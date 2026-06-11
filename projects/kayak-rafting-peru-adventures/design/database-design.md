# DATABASE DESIGN — Rafting Peru & Kayak Adventures
Arquitecto de Datos Senior · CypData · Junio 2026
Entradas: domain-model.md · domain-catalog.md · backend-specification.md · api-specification.md · technical-architecture-v1.md
Diseño físico relacional (PostgreSQL). Consistente con el domain-catalog (entidades, enums, relaciones).

---

# RESUMEN GENERAL

Diseño físico de la base de datos relacional principal de RPKA sobre **PostgreSQL**,
motor seleccionado en la arquitectura técnica por la naturaleza transaccional de
reservas/pagos y la integridad referencial entre entidades. El esquema se organiza
por límites de dominio (C1–C5 + transversal), de modo que cada dominio es dueño de
sus tablas; el acoplamiento entre dominios se resuelve por **clave foránea lógica
por identidad** (columna `*_id`), preservando la posibilidad de extraer servicios
en el futuro (modular monolith → servicios).

Decisiones transversales:
- **Claves primarias**: UUID (`uuid`, default `gen_random_uuid()` vía pgcrypto/pgcrypto o `gen_random_uuid()` nativo en PG13+). Identificadores opacos, alineados con los ids string de la API.
- **Enums**: tipos `ENUM` nativos de PostgreSQL (un tipo por cada enum del domain-catalog), para integridad a nivel de motor.
- **Dinero**: `NUMERIC(12,2)` para el monto + columna de moneda (`enum moneda`), materializando el VO `Money`.
- **Auditoría**: columnas `created_at`, `updated_at` en todas las tablas + tablas de historial de estado para entidades con ciclo de vida (consulta, cotización, reserva, pedido).
- **Soft delete**: por estado de publicación en contenido (`estado_publicacion`) y por columna `deleted_at` (nullable) en entidades administrables; las entidades transaccionales (reserva, pago, confirmación) **no** se borran, solo cambian de estado.
- **Esquemas**: un esquema lógico por dominio (`contenido`, `captacion`, `confianza`, `reservas`, `equipos`, `core`) para reflejar las fronteras; alternativamente prefijo de tabla si se prefiere un único esquema.

---

# MODELO RELACIONAL

Esquemas y tablas (raíz de agregado en **negrita**):

- **core**: `actor_operativo`
- **contenido (C1)**: `sede`, **`destino`**, **`experiencia`**, `componente`, `activo_multimedia`, **`articulo`**, `experiencia_perfil`, `articulo_relacionado`, `destino_multimedia`/`experiencia_multimedia` (resueltas como FK nullable en `activo_multimedia`)
- **captacion (C2)**: **`consulta`**, `consulta_cambio_estado`, **`solicitud_cotizacion`**, `cotizacion`, `solicitud_cambio_estado`
- **confianza (C3)**: **`guia`**, **`credencial`**, `guia_credencial`, **`resena`**, **`faq`**
- **reservas (C4)**: **`disponibilidad`**, **`reserva`**, `pago`, `confirmacion`, `reserva_cambio_estado`
- **equipos (C5)**: **`equipo`**, `equipo_perfil`, **`pedido_equipo`**, `linea_pedido_equipo`, `pedido_cambio_estado`

Relación entre dominios por identidad (FK lógica): p. ej. `reservas.reserva.experiencia_id`
referencia `contenido.experiencia.id`. Se permite FK física para integridad mientras
sea un monolito modular; si se extraen servicios, estas FK se degradan a referencia lógica.

Tipos ENUM (PostgreSQL) — un tipo por enum del domain-catalog:
```sql
CREATE TYPE idioma AS ENUM ('ES','EN');
CREATE TYPE canal AS ENUM ('WHATSAPP','FORMULARIO','OTRO');
CREATE TYPE estado_publicacion AS ENUM ('BORRADOR','PUBLICADO','NO_DISPONIBLE');
CREATE TYPE nivel_dificultad AS ENUM ('II','III','IV','MIXTO');
CREATE TYPE nivel_esfuerzo AS ENUM ('BAJO','MEDIO','ALTO');
CREATE TYPE tipo_componente AS ENUM ('ALIMENTACION','HOSPEDAJE','EQUIPO','TRANSPORTE','GUIA');
CREATE TYPE condicion_componente AS ENUM ('INCLUIDO','OPCIONAL');
CREATE TYPE base_calculo_precio AS ENUM ('FIJO','POR_PERSONA','POR_TEMPORADA');
CREATE TYPE perfil_cliente AS ENUM ('FAMILIA','NINO','ADULTO_MAYOR','PRINCIPIANTE','CORPORATIVO','PRACTICANTE','TURISTA_NACIONAL','TURISTA_INTERNACIONAL');
CREATE TYPE tipo_multimedia AS ENUM ('FOTO','VIDEO');
CREATE TYPE tipo_contenido_ref AS ENUM ('DESTINO','EXPERIENCIA');
CREATE TYPE estado_consulta AS ENUM ('NUEVA','EN_PROCESO','RESPONDIDA','CERRADA');
CREATE TYPE estado_cotizacion AS ENUM ('RECIBIDA','EN_PROCESO','COTIZACION_EMITIDA','CERRADA');
CREATE TYPE estado_reserva AS ENUM ('INICIADA','PAGADA','CONFIRMADA','NO_CONFIRMADA','CANCELADA','COMPLETADA');
CREATE TYPE estado_pago AS ENUM ('INICIADO','EXITOSO','FALLIDO','RECHAZADO');
CREATE TYPE estado_disponibilidad AS ENUM ('DISPONIBLE','PARCIAL','AGOTADO');
CREATE TYPE estado_pedido_equipo AS ENUM ('RECIBIDO','EN_COTIZACION','CONFIRMADO','EN_IMPORTACION','ENTREGADO','CANCELADO');
CREATE TYPE condicion_venta_equipo AS ENUM ('BAJO_PEDIDO','IMPORTACION');
CREATE TYPE titular_credencial AS ENUM ('NEGOCIO','GUIA');
CREATE TYPE rol_operativo AS ENUM ('RESPONSABLE_COMERCIAL','GESTOR_OPERACIONES');
CREATE TYPE moneda AS ENUM ('PEN','USD');
```

---

# CATÁLOGO DE TABLAS

## Tabla: core.actor_operativo
### Descripción: Usuario interno (back-office). Las credenciales de acceso viven en el contexto de identidad, fuera de esta tabla.
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| nombre | text | No |
| rol | rol_operativo | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: —
### Índices: idx_actor_rol (rol)
### Constraints: nombre <> ''
### Reglas de Integridad: RN-15 (autorización por rol) se aplica en backend; aquí solo se persiste el rol.

## Tabla: contenido.sede
### Descripción: Ubicación operativa asociada a un destino y a una ficha local externa.
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| nombre | text | No |
| destino_id | uuid | No |
| referencia_ficha_local | text | Sí |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: destino_id → contenido.destino(id)
### Índices: idx_sede_destino (destino_id)
### Constraints: UNIQUE(destino_id) — relación Destino 1–1 Sede
### Reglas de Integridad: cada sede pertenece a un destino (1–1).

## Tabla: contenido.destino
### Descripción: Río/destino con su contenido propio (AGG-Destino).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| nombre | text | No |
| descripcion | text | Sí |
| idiomas | idioma[] | No |
| estado_publicacion | estado_publicacion | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: — (Sede referencia a Destino, no al revés)
### Índices: idx_destino_estado (estado_publicacion); idx_destino_nombre (lower(nombre))
### Constraints: nombre <> ''; default estado_publicacion = 'BORRADOR'
### Reglas de Integridad: RN-03/BR-016 (visible solo si PUBLICADO) — filtrado en consultas públicas.

## Tabla: contenido.experiencia
### Descripción: Paquete/experiencia turística (AGG-Experiencia).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| nombre | text | No |
| destino_id | uuid | No |
| duracion | text | Sí |
| dificultad | nivel_dificultad | No |
| precio_referencia_monto | numeric(12,2) | No |
| precio_referencia_moneda | moneda | No |
| base_calculo_precio | base_calculo_precio | No |
| condiciones | text | Sí |
| edad_minima | int | Sí |
| nivel_esfuerzo | nivel_esfuerzo | Sí |
| estado_publicacion | estado_publicacion | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: destino_id → contenido.destino(id)
### Índices: idx_exp_destino (destino_id); idx_exp_estado (estado_publicacion); idx_exp_dificultad (dificultad)
### Constraints: precio_referencia_monto >= 0; edad_minima IS NULL OR edad_minima >= 0; default estado_publicacion = 'BORRADOR'
### Reglas de Integridad: RN-05/BR-016 (precio de referencia + base de cálculo; visible solo si PUBLICADO).

## Tabla: contenido.componente
### Descripción: Componente de una experiencia (incluido/opcional). Entidad interna del agregado Experiencia.
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| experiencia_id | uuid | No |
| nombre | text | No |
| tipo | tipo_componente | No |
| condicion | condicion_componente | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
### Primary Key: (id)
### Foreign Keys: experiencia_id → contenido.experiencia(id) ON DELETE CASCADE
### Índices: idx_componente_exp (experiencia_id)
### Constraints: condicion IN ('INCLUIDO','OPCIONAL') — garantizado por el enum (RN-04/BR-004)
### Reglas de Integridad: RN-04 (cada componente es inequívocamente incluido u opcional).

## Tabla: contenido.experiencia_perfil
### Descripción: Relación N–N Experiencia ↔ PerfilCliente (perfiles aptos).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| experiencia_id | uuid | No |
| perfil | perfil_cliente | No |
### Primary Key: (experiencia_id, perfil)
### Foreign Keys: experiencia_id → contenido.experiencia(id) ON DELETE CASCADE
### Índices: idx_exp_perfil_perfil (perfil)
### Constraints: PK compuesta evita duplicados
### Reglas de Integridad: soporta filtro/recomendación por perfil (US-023, US-025).

## Tabla: contenido.activo_multimedia
### Descripción: Foto/video asociado a un destino o experiencia (la referencia binaria vive en object storage).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| tipo | tipo_multimedia | No |
| referencia_almacenamiento | text | No |
| destino_id | uuid | Sí |
| experiencia_id | uuid | Sí |
| idioma | idioma | Sí |
| caption | text | Sí |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: destino_id → contenido.destino(id); experiencia_id → contenido.experiencia(id)
### Índices: idx_media_destino (destino_id); idx_media_exp (experiencia_id)
### Constraints: CHECK (destino_id IS NOT NULL OR experiencia_id IS NOT NULL) — debe asociarse a algo
### Reglas de Integridad: soporte de galería por destino/experiencia (US-016).

## Tabla: contenido.articulo
### Descripción: Contenido editorial/blog (AGG-Articulo).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| titulo | text | No |
| cuerpo | text | No |
| idioma | idioma | No |
| estado_publicacion | estado_publicacion | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: —
### Índices: idx_articulo_estado (estado_publicacion); idx_articulo_idioma (idioma)
### Constraints: titulo <> ''; default estado_publicacion = 'BORRADOR'
### Reglas de Integridad: BR-016 (visible solo si PUBLICADO).

## Tabla: contenido.articulo_relacionado
### Descripción: Relación N–N entre Artículo y Destino/Experiencia (referencia polimórfica controlada).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| articulo_id | uuid | No |
| tipo_ref | tipo_contenido_ref | No |
| ref_id | uuid | No |
### Primary Key: (articulo_id, tipo_ref, ref_id)
### Foreign Keys: articulo_id → contenido.articulo(id) ON DELETE CASCADE
### Índices: idx_art_rel_ref (tipo_ref, ref_id)
### Constraints: PK compuesta; ref_id validado por aplicación (referencia polimórfica)
### Reglas de Integridad: enlaza artículos con destinos/experiencias mencionados (US-018).

## Tabla: captacion.consulta
### Descripción: Lead/consulta de un visitante (AGG-Consulta).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| nombre_contacto | text | No |
| medio_contacto | text | No |
| destino_interes_id | uuid | Sí |
| mensaje | text | No |
| canal | canal | No |
| idioma | idioma | No |
| fecha | timestamptz | No |
| estado | estado_consulta | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
### Primary Key: (id)
### Foreign Keys: destino_interes_id → contenido.destino(id) (lógica entre dominios)
### Índices: idx_consulta_estado (estado); idx_consulta_canal (canal); idx_consulta_fecha (fecha DESC); idx_consulta_destino (destino_interes_id)
### Constraints: nombre_contacto <> ''; mensaje <> ''; default estado = 'NUEVA'
### Reglas de Integridad: RN-01/BR-001 (registra canal e idioma; requeridos obligatorios). Sin soft delete: el lead se cierra por estado.

## Tabla: captacion.consulta_cambio_estado
### Descripción: Historial de cambios de estado de una consulta (auditoría/trazabilidad).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| consulta_id | uuid | No |
| estado_anterior | estado_consulta | Sí |
| estado_nuevo | estado_consulta | No |
| fecha | timestamptz | No |
| actor_id | uuid | Sí |
### Primary Key: (id)
### Foreign Keys: consulta_id → captacion.consulta(id) ON DELETE CASCADE; actor_id → core.actor_operativo(id)
### Índices: idx_cce_consulta (consulta_id, fecha)
### Constraints: estado_nuevo <> estado_anterior (cuando anterior no es null)
### Reglas de Integridad: RN-02/BR-002 (toda transición historizada).

## Tabla: captacion.solicitud_cotizacion
### Descripción: Solicitud de cotización de grupo/B2B (AGG-SolicitudCotizacion).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| organizacion | text | No |
| numero_personas | int | No |
| destino_id | uuid | No |
| fecha_solicitada | date | No |
| contacto_nombre | text | No |
| contacto_email | text | Sí |
| contacto_telefono | text | Sí |
| estado | estado_cotizacion | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
### Primary Key: (id)
### Foreign Keys: destino_id → contenido.destino(id)
### Índices: idx_solc_estado (estado); idx_solc_destino (destino_id); idx_solc_fecha (fecha_solicitada)
### Constraints: numero_personas >= 1; organizacion <> ''; default estado = 'RECIBIDA'
### Reglas de Integridad: RN-10/BR-011 (campos obligatorios para registrar).

## Tabla: captacion.cotizacion
### Descripción: Respuesta a una solicitud (entidad dependiente, 0..1 por solicitud).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| solicitud_id | uuid | No |
| contenido | text | No |
| fecha_emision | timestamptz | No |
| autor_id | uuid | No |
### Primary Key: (id)
### Foreign Keys: solicitud_id → captacion.solicitud_cotizacion(id) ON DELETE CASCADE; autor_id → core.actor_operativo(id)
### Índices: idx_cotizacion_solicitud (solicitud_id)
### Constraints: UNIQUE(solicitud_id) — una cotización por solicitud (0..1); contenido <> ''
### Reglas de Integridad: RN-11/BR-012 (cotización asociada a su solicitud).

## Tabla: captacion.solicitud_cambio_estado
### Descripción: Historial de estado de la solicitud de cotización.
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| solicitud_id | uuid | No |
| estado_anterior | estado_cotizacion | Sí |
| estado_nuevo | estado_cotizacion | No |
| fecha | timestamptz | No |
| actor_id | uuid | Sí |
### Primary Key: (id)
### Foreign Keys: solicitud_id → captacion.solicitud_cotizacion(id) ON DELETE CASCADE; actor_id → core.actor_operativo(id)
### Índices: idx_sce_solicitud (solicitud_id, fecha)
### Constraints: —
### Reglas de Integridad: RN-02/BR-002.

## Tabla: confianza.guia
### Descripción: Integrante del equipo de guías (AGG-Guia).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| nombre | text | No |
| experiencia | text | Sí |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: —
### Índices: idx_guia_nombre (lower(nombre))
### Constraints: nombre <> ''
### Reglas de Integridad: perfiles de guías (US-012).

## Tabla: confianza.credencial
### Descripción: Certificación/licencia del negocio o de un guía (AGG-Credencial).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| tipo | text | No |
| entidad_emisora | text | No |
| titular_tipo | titular_credencial | No |
| titular_guia_id | uuid | Sí |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: titular_guia_id → confianza.guia(id)
### Índices: idx_credencial_titular (titular_tipo); idx_credencial_guia (titular_guia_id)
### Constraints: CHECK ((titular_tipo='GUIA' AND titular_guia_id IS NOT NULL) OR (titular_tipo='NEGOCIO' AND titular_guia_id IS NULL))
### Reglas de Integridad: RN-16 (identifica emisora y titular).

## Tabla: confianza.guia_credencial
### Descripción: Relación N–N Guía ↔ Credencial (una credencial puede pertenecer a varios guías si es compartida; un guía tiene varias).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| guia_id | uuid | No |
| credencial_id | uuid | No |
### Primary Key: (guia_id, credencial_id)
### Foreign Keys: guia_id → confianza.guia(id) ON DELETE CASCADE; credencial_id → confianza.credencial(id) ON DELETE CASCADE
### Índices: idx_gc_credencial (credencial_id)
### Constraints: PK compuesta
### Reglas de Integridad: asociación guía–credencial (US-012).

## Tabla: confianza.resena
### Descripción: Valoración de cliente, opcionalmente asociada a destino/experiencia (AGG-Resena).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| autor | text | No |
| valoracion | int | No |
| texto | text | Sí |
| objetivo_tipo | tipo_contenido_ref | Sí |
| objetivo_id | uuid | Sí |
| fecha | timestamptz | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: — (objetivo es referencia polimórfica validada por aplicación)
### Índices: idx_resena_objetivo (objetivo_tipo, objetivo_id); idx_resena_fecha (fecha DESC)
### Constraints: valoracion BETWEEN 1 AND 5; CHECK ((objetivo_tipo IS NULL AND objetivo_id IS NULL) OR (objetivo_tipo IS NOT NULL AND objetivo_id IS NOT NULL))
### Reglas de Integridad: RN-17 (asociación opcional; su ausencia no rompe la página).

## Tabla: confianza.faq
### Descripción: Pregunta frecuente (AGG-Faq).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| pregunta | text | No |
| respuesta | text | No |
| categoria | text | Sí |
| idioma | idioma | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: —
### Índices: idx_faq_categoria (categoria); idx_faq_idioma (idioma); idx_faq_busqueda (GIN sobre to_tsvector(pregunta || ' ' || respuesta))
### Constraints: pregunta <> ''; respuesta <> ''
### Reglas de Integridad: búsqueda por término (US-015) — índice de texto completo.

## Tabla: reservas.disponibilidad
### Descripción: Cupo por experiencia y fecha (AGG-Disponibilidad).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| experiencia_id | uuid | No |
| fecha | date | No |
| cupo_total | int | No |
| cupo_disponible | int | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
### Primary Key: (id)
### Foreign Keys: experiencia_id → contenido.experiencia(id) (lógica entre dominios)
### Índices: UNIQUE idx_disp_exp_fecha (experiencia_id, fecha); idx_disp_fecha (fecha)
### Constraints: cupo_total >= 0; cupo_disponible >= 0; CHECK (cupo_disponible <= cupo_total)
### Reglas de Integridad: RN-06/RN-07/BR-009 (consistencia de cupo). El estado DISPONIBLE/PARCIAL/AGOTADO se deriva en consulta; opcionalmente columna generada.

## Tabla: reservas.reserva
### Descripción: Reserva sobre una experiencia en una fecha (raíz del agregado transaccional).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| experiencia_id | uuid | No |
| disponibilidad_id | uuid | No |
| fecha | date | No |
| numero_participantes | int | No |
| cliente_nombre | text | No |
| cliente_email | text | Sí |
| cliente_telefono | text | Sí |
| estado | estado_reserva | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
### Primary Key: (id)
### Foreign Keys: experiencia_id → contenido.experiencia(id) (lógica); disponibilidad_id → reservas.disponibilidad(id)
### Índices: idx_reserva_estado (estado); idx_reserva_exp (experiencia_id); idx_reserva_disp (disponibilidad_id); idx_reserva_fecha (fecha)
### Constraints: numero_participantes >= 1; default estado = 'INICIADA'
### Reglas de Integridad: RN-06/BR-003 (participantes ≤ cupo, validado transaccionalmente con bloqueo de fila de disponibilidad); RN-08/BR-007 (CONFIRMADA solo con pago exitoso). Sin borrado: solo cambios de estado.

## Tabla: reservas.pago
### Descripción: Pago asociado a una reserva (1–1). No almacena datos de tarjeta; solo token/referencia de pasarela.
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| reserva_id | uuid | No |
| monto | numeric(12,2) | No |
| moneda | moneda | No |
| estado | estado_pago | No |
| referencia_pasarela | text | Sí |
| fecha | timestamptz | No |
### Primary Key: (id)
### Foreign Keys: reserva_id → reservas.reserva(id) ON DELETE RESTRICT
### Índices: UNIQUE idx_pago_reserva (reserva_id); UNIQUE idx_pago_referencia (referencia_pasarela) WHERE referencia_pasarela IS NOT NULL
### Constraints: monto >= 0; default estado = 'INICIADO'
### Reglas de Integridad: RN-08 (resultado determina confirmación); BR-020 (idempotencia por referencia_pasarela → índice único). Sin datos sensibles de tarjeta (cumplimiento).

## Tabla: reservas.confirmacion
### Descripción: Comprobante emitido tras reserva pagada (0..1 por reserva).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| reserva_id | uuid | No |
| detalle | text | No |
| fecha_emision | timestamptz | No |
### Primary Key: (id)
### Foreign Keys: reserva_id → reservas.reserva(id) ON DELETE RESTRICT
### Índices: UNIQUE idx_confirmacion_reserva (reserva_id)
### Constraints: UNIQUE(reserva_id) — una confirmación por reserva
### Reglas de Integridad: RN-09/BR-008 (toda reserva confirmada genera confirmación).

## Tabla: reservas.reserva_cambio_estado
### Descripción: Historial de estado de la reserva.
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| reserva_id | uuid | No |
| estado_anterior | estado_reserva | Sí |
| estado_nuevo | estado_reserva | No |
| fecha | timestamptz | No |
| actor_id | uuid | Sí |
### Primary Key: (id)
### Foreign Keys: reserva_id → reservas.reserva(id) ON DELETE CASCADE; actor_id → core.actor_operativo(id)
### Índices: idx_rce_reserva (reserva_id, fecha)
### Constraints: —
### Reglas de Integridad: RN-02/BR-002.

## Tabla: equipos.equipo
### Descripción: Producto del catálogo de equipos (AGG-Equipo), bajo pedido/importación.
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| nombre | text | No |
| descripcion | text | Sí |
| tipo | text | Sí |
| condicion_venta | condicion_venta_equipo | No |
| estado_publicacion | estado_publicacion | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
| deleted_at | timestamptz | Sí |
### Primary Key: (id)
### Foreign Keys: —
### Índices: idx_equipo_estado (estado_publicacion); idx_equipo_condicion (condicion_venta)
### Constraints: nombre <> ''; default estado_publicacion = 'BORRADOR'
### Reglas de Integridad: RN-12/BR-016 (bajo pedido; visible solo si PUBLICADO).

## Tabla: equipos.equipo_perfil
### Descripción: Relación N–N Equipo ↔ PerfilCliente (segmentos aplicables).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| equipo_id | uuid | No |
| perfil | perfil_cliente | No |
### Primary Key: (equipo_id, perfil)
### Foreign Keys: equipo_id → equipos.equipo(id) ON DELETE CASCADE
### Índices: idx_eq_perfil_perfil (perfil)
### Constraints: PK compuesta
### Reglas de Integridad: RN-13/BR-018 (segmentación; oferta general si no hay específica — resuelto en consulta).

## Tabla: equipos.pedido_equipo
### Descripción: Pedido/solicitud de cotización de equipos (AGG-PedidoEquipo).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| tipo_cliente | perfil_cliente | No |
| contacto_nombre | text | No |
| contacto_email | text | Sí |
| contacto_telefono | text | Sí |
| estado_importacion | estado_pedido_equipo | No |
| fecha | timestamptz | No |
| created_at | timestamptz | No |
| updated_at | timestamptz | No |
### Primary Key: (id)
### Foreign Keys: —
### Índices: idx_pedido_estado (estado_importacion); idx_pedido_fecha (fecha)
### Constraints: contacto_nombre <> ''; default estado_importacion = 'RECIBIDO'
### Reglas de Integridad: RN-12/BR-014 (sujeto a importación).

## Tabla: equipos.linea_pedido_equipo
### Descripción: Línea de un pedido de equipo (entidad interna; N–N resuelta entre pedido y equipo con cantidad).
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| pedido_id | uuid | No |
| equipo_id | uuid | No |
| cantidad | int | No |
### Primary Key: (id)
### Foreign Keys: pedido_id → equipos.pedido_equipo(id) ON DELETE CASCADE; equipo_id → equipos.equipo(id)
### Índices: idx_linea_pedido (pedido_id); idx_linea_equipo (equipo_id); UNIQUE(pedido_id, equipo_id)
### Constraints: cantidad >= 1
### Reglas de Integridad: RN-13/BR-013 (al menos una línea con cantidad > 0 — la regla "al menos una" se valida en backend al crear el pedido).

## Tabla: equipos.pedido_cambio_estado
### Descripción: Historial de estado de importación del pedido.
### Columnas
| Campo | Tipo | Nullable |
| ----- | ---- | -------- |
| id | uuid | No |
| pedido_id | uuid | No |
| estado_anterior | estado_pedido_equipo | Sí |
| estado_nuevo | estado_pedido_equipo | No |
| fecha | timestamptz | No |
| actor_id | uuid | Sí |
### Primary Key: (id)
### Foreign Keys: pedido_id → equipos.pedido_equipo(id) ON DELETE CASCADE; actor_id → core.actor_operativo(id)
### Índices: idx_pce_pedido (pedido_id, fecha)
### Constraints: —
### Reglas de Integridad: RN-02/BR-002 (US-038).

---

# AUDITORÍA

Estrategia de auditoría en dos niveles:

1. **Columnas de tiempo en cada tabla**: `created_at timestamptz NOT NULL DEFAULT now()` y
   `updated_at timestamptz NOT NULL DEFAULT now()`, esta última mantenida por trigger
   `BEFORE UPDATE` (`set_updated_at()`).

2. **Tablas de historial de estado** para entidades con ciclo de vida (RN-02/BR-002):
   `consulta_cambio_estado`, `solicitud_cambio_estado`, `reserva_cambio_estado`,
   `pedido_cambio_estado`. Cada transición inserta una fila con estado anterior, nuevo,
   fecha y actor responsable. Esto satisface la trazabilidad exigida por los RNF de
   auditoría de la arquitectura técnica (registro de cambios en entidades sensibles:
   reservas, pagos, pedidos).

Trigger de updated_at (patrón):
```sql
CREATE OR REPLACE FUNCTION set_updated_at() RETURNS trigger AS $$
BEGIN NEW.updated_at = now(); RETURN NEW; END;
$$ LANGUAGE plpgsql;
-- aplicar: CREATE TRIGGER trg_set_updated_at BEFORE UPDATE ON <tabla>
--          FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

Registro de acceso/operaciones (logins, lecturas sensibles) se delega a la capa de
observabilidad/identidad (technical-architecture), fuera del esquema de negocio.

---

# SOFT DELETE

Política diferenciada por naturaleza del dato:

- **Contenido administrable** (destino, experiencia, articulo, equipo, sede, guia, credencial, resena, faq, activo_multimedia): columna `deleted_at timestamptz NULL`. Una fila con `deleted_at` no nula se considera eliminada lógicamente y se excluye en todas las consultas (vistas/filtros `WHERE deleted_at IS NULL`). El contenido público adicionalmente usa `estado_publicacion` (BORRADOR/PUBLICADO/NO_DISPONIBLE) para visibilidad, que es independiente del borrado lógico.
- **Entidades transaccionales** (reserva, pago, confirmacion, disponibilidad): **no** admiten borrado (ni físico ni lógico). Su ciclo de vida se gestiona por estado (p. ej. reserva CANCELADA/NO_CONFIRMADA), preservando la integridad financiera y la trazabilidad.
- **Leads y solicitudes** (consulta, solicitud_cotizacion): se cierran por estado (CERRADA), no se borran, para conservar el historial comercial.
- **Tablas puente e historiales**: sin soft delete; se eliminan en cascada con su agregado raíz cuando corresponde (`ON DELETE CASCADE`), o se conservan como registro inmutable (historiales).

Recomendación operativa: aplicar `deleted_at` mediante un patrón de repositorio en
backend y, opcionalmente, **Row Level Security** o vistas (`v_destino_activo`, etc.)
que encapsulen `WHERE deleted_at IS NULL` para evitar fugas.

---

# ESTRATEGIA DE MIGRACIONES

- **Herramienta**: migraciones versionadas y reversibles (p. ej. Prisma Migrate, Knex,
  node-pg-migrate o Flyway), integradas en el pipeline CI/CD descrito en la arquitectura
  técnica. Cada cambio de esquema es un archivo versionado con `up`/`down`.
- **Orden de creación** (respeta dependencias de FK): (1) tipos ENUM; (2) `core.actor_operativo`;
  (3) `contenido.destino` → `contenido.sede` → `contenido.experiencia` → `componente`,
  `experiencia_perfil`, `activo_multimedia`, `articulo`, `articulo_relacionado`;
  (4) `confianza.*`; (5) `captacion.*`; (6) `reservas.disponibilidad` → `reserva` → `pago`,
  `confirmacion`, `reserva_cambio_estado`; (7) `equipos.equipo` → `equipo_perfil`,
  `pedido_equipo` → `linea_pedido_equipo`, `pedido_cambio_estado`.
- **Datos semilla (seed)**: catálogos iniciales de sedes/destinos (Urubamba, Lunahuaná,
  Cotahuasi, Marañón) y actores operativos base, en una migración de seed separada.
- **Cambios de enum**: añadir valores con `ALTER TYPE ... ADD VALUE` (no destructivo);
  retirar valores requiere migración de datos controlada.
- **Concurrencia**: para la reducción de cupo (RN-07) usar transacciones con
  `SELECT ... FOR UPDATE` sobre la fila de `disponibilidad`, garantizando que no haya
  sobreventa bajo reservas simultáneas (riesgo identificado en domain-model).
- **Índices**: crear índices con `CREATE INDEX CONCURRENTLY` en producción para evitar bloqueos.
- **Entornos**: las mismas migraciones se aplican en dev → staging → producción; respaldos
  previos a migraciones destructivas.

---

# MATRIZ DE TRAZABILIDAD

| Entidad (domain-catalog) | Tabla(s) física(s) | Agregado | Contexto |
| ------------------------ | ------------------ | -------- | -------- |
| ActorOperativo | core.actor_operativo | AGG-ActorOperativo | CX |
| Sede | contenido.sede | (parte de C1) | C1 |
| Destino | contenido.destino | AGG-Destino | C1 |
| Experiencia | contenido.experiencia | AGG-Experiencia | C1 |
| Componente | contenido.componente | AGG-Experiencia (interna) | C1 |
| PerfilApto (Experiencia↔Perfil) | contenido.experiencia_perfil | AGG-Experiencia | C1 |
| ActivoMultimedia | contenido.activo_multimedia | (C1) | C1 |
| Articulo | contenido.articulo (+ articulo_relacionado) | AGG-Articulo | C1 |
| Consulta | captacion.consulta | AGG-Consulta | C2 |
| CambioEstadoConsulta | captacion.consulta_cambio_estado | AGG-Consulta (interna) | C2 |
| SolicitudCotizacion | captacion.solicitud_cotizacion | AGG-SolicitudCotizacion | C2 |
| Cotizacion | captacion.cotizacion | AGG-SolicitudCotizacion (interna) | C2 |
| CambioEstadoCotizacion | captacion.solicitud_cambio_estado | AGG-SolicitudCotizacion (interna) | C2 |
| Guia | confianza.guia | AGG-Guia | C3 |
| Credencial | confianza.credencial (+ guia_credencial) | AGG-Credencial | C3 |
| Resena | confianza.resena | AGG-Resena | C3 |
| Faq | confianza.faq | AGG-Faq | C3 |
| Disponibilidad | reservas.disponibilidad | AGG-Disponibilidad | C4 |
| Reserva | reservas.reserva | AGG-Reserva | C4 |
| Pago | reservas.pago | AGG-Reserva (interna) | C4 |
| Confirmacion | reservas.confirmacion | AGG-Reserva (interna) | C4 |
| CambioEstadoReserva | reservas.reserva_cambio_estado | AGG-Reserva (interna) | C4 |
| Equipo | equipos.equipo (+ equipo_perfil) | AGG-Equipo | C5 |
| PedidoEquipo | equipos.pedido_equipo | AGG-PedidoEquipo | C5 |
| LineaPedidoEquipo | equipos.linea_pedido_equipo | AGG-PedidoEquipo (interna) | C5 |
| CambioEstadoPedido | equipos.pedido_cambio_estado | AGG-PedidoEquipo (interna) | C5 |

Cobertura: todas las entidades persistentes del domain-catalog tienen tabla; los VO
(Money, DatosContacto) se materializan como columnas dentro de su entidad contenedora;
PerfilCliente y demás enums se materializan como tipos ENUM y tablas puente N–N.

---

# CHECKLIST DE VALIDACIÓN
- [x] Todas las entidades persistentes tienen tabla (ver matriz de trazabilidad).
- [x] Todas las relaciones están definidas (FK físicas intra-dominio; FK lógicas entre dominios).
- [x] Todas las PK están definidas (UUID en cada tabla; PK compuesta en puentes).
- [x] Todas las FK están definidas (con ON DELETE CASCADE/RESTRICT según naturaleza).
- [x] Existen índices críticos (estados, fechas, FKs, unicidad de cupo y de referencia de pago, texto completo en FAQ).
- [x] Existe estrategia de auditoría (created/updated_at + historiales de estado).

---

# NOTA DE TRAZABILIDAD
Diseño derivado exclusivamente del domain-catalog (entidades, enums, relaciones),
la backend-specification (reglas BR, validaciones) y la arquitectura técnica
(PostgreSQL, integridad transaccional, tokenización de pagos). No introduce entidades
nuevas. Decisiones físicas clave: UUID como PK, ENUM nativos, índice único
(experiencia_id, fecha) en disponibilidad para una sola fila de cupo por fecha,
índice único sobre referencia_pasarela para idempotencia (BR-020), y bloqueo de fila
para consistencia de cupo (RN-06/RN-07). Se conserva la corrección heredada
FT-037 → FT-038 (gestión de pedidos de equipos = tabla equipos.pedido_equipo).