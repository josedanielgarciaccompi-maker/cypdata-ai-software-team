# BACKEND SPECIFICATION — Rafting Peru & Kayak Adventures
Arquitecto Backend Senior · CypData · Junio 2026
Entradas: opportunity-report-002.md · epics.md · features.md · user-stories.md · acceptance-criteria.md · architecture-v1.md · technical-architecture-v1.md · domain-model.md · domain-catalog.md
Contrato técnico backend (sin código fuente). Trazabilidad completa US/FT/EP.

---

# 1. RESUMEN EJECUTIVO

## Objetivo del Sistema
Plataforma digital que convierte tráfico en consultas y reservas para un operador
de turismo de aventura multi-destino, con una línea secundaria de comercialización
de equipos. El backend expone las capacidades de captar (consultas/cotizaciones),
publicar oferta y confianza, reservar y cobrar, y operar (back-office), preservando
el desacoplamiento por dominio definido en la arquitectura.

## Alcance Backend
Cubre el comportamiento de servidor de los módulos M1–M10: contenido y catálogo,
visibilidad, contenido de marca, confianza, captación conversacional/formularios,
gestión de leads, cotizaciones/B2B, disponibilidad y reservas, pagos y confirmación,
y catálogo/pedidos de equipos. Quedan fuera: código fuente, diseño físico de BD,
SQL, UI, despliegue (cubiertos por otros agentes). Las integraciones externas
(WhatsApp, pasarela, fichas locales) se describen por contrato, no por implementación.

## Contextos de Negocio Cubiertos
- C1 Contenido y Oferta · C2 Captación y Relación · C3 Confianza · C4 Reservas y Pagos
- C5 Equipos · CX Transversal (Actor Operativo / autorización)

---

# 2. TRAZABILIDAD FUNCIONAL

| Epic | Feature | User Story | Casos de Uso |
| ---- | ------- | ---------- | ------------ |
| EP-001 | FT-001 | US-001, US-002 | UC-01, UC-02 |
| EP-001 | FT-002 | US-003 | UC-03 |
| EP-001 | FT-003 | US-004 | UC-04 |
| EP-001 | FT-004 | US-005 | UC-05 |
| EP-001 | FT-005 | US-006 | UC-06 |
| EP-002 | FT-006 | US-007 | UC-07 |
| EP-002 | FT-007 | US-008 | UC-08 |
| EP-002 | FT-008 | US-009 | UC-09 |
| EP-002 | FT-009 | US-010 | UC-10 |
| EP-003 | FT-010 | US-011 | UC-11 |
| EP-003 | FT-011 | US-012 | UC-12 |
| EP-003 | FT-012 | US-013 | UC-13 |
| EP-003 | FT-013 | US-014 | UC-14 |
| EP-003 | FT-014 | US-015 | UC-15 |
| EP-004 | FT-015 | US-016 | UC-16 |
| EP-004 | FT-016 | US-017 | UC-17 |
| EP-004 | FT-017 | US-018 | UC-18 |
| EP-005 | FT-018 | US-019 | UC-19 |
| EP-005 | FT-019 | US-020 | UC-20 |
| EP-005 | FT-020 | US-021 | UC-21 |
| EP-005 | FT-021 | US-022 | UC-22 |
| EP-006 | FT-022 | US-023 | UC-23 |
| EP-006 | FT-023 | US-024 | UC-24 |
| EP-006 | FT-024 | US-025 | UC-25 |
| EP-007 | FT-025 | US-026 | UC-26 |
| EP-007 | FT-026 | US-027 | UC-27 |
| EP-007 | FT-027 | US-028 | UC-28 |
| EP-007 | FT-028 | US-029 | UC-29 |
| EP-008 | FT-029 | US-030 | UC-30 |
| EP-008 | FT-030 | US-031 | UC-31 |
| EP-008 | FT-031 | US-032 | UC-32 |
| EP-008 | FT-032 | US-033 | UC-33 |
| EP-008 | FT-033 | US-034 | UC-34 |
| EP-009 | FT-034 | US-035 | UC-35 |
| EP-009 | FT-035 | US-036 | UC-36 |
| EP-009 | FT-036 | US-037 | UC-37 |
| EP-009 | FT-038 | US-038 | UC-38 |

Cobertura: 38/38 User Stories · 9/9 Epics · 37/37 Features.

---

# 3. CATÁLOGO DE CASOS DE USO

> Nota: UC-01, UC-02, UC-07–UC-25, UC-28, UC-35, UC-37 son de consulta (lectura) o
> de presentación servida por el backend; UC-03, UC-04, UC-05, UC-26, UC-27, UC-30–UC-34,
> UC-36, UC-38 modifican estado. Cada uno indica Commands/Queries en §4 y §5.

## Caso de Uso
### Identificador: UC-03
### Nombre: Registrar consulta por formulario
### Descripción: El visitante envía una consulta con sus datos; el sistema la registra y confirma recepción.
### Actor: Visitante
### Contexto de Negocio: C2 Captación
### Precondiciones: Formulario disponible; idioma activo conocido.
### Flujo Principal:
1. El visitante envía nombre, medio de contacto, destino de interés (opcional), mensaje, idioma y canal.
2. El sistema valida los campos requeridos (BR-001, VAL).
3. El sistema crea la Consulta en estado NUEVA y registra canal e idioma.
4. El sistema emite ConsultaRecibida y devuelve confirmación de recepción.
### Flujos Alternativos:
- A1: campos requeridos vacíos/ inválidos → ERR-010; no se crea la consulta.
### Postcondiciones: Consulta persistida en estado NUEVA; evento emitido.
### Reglas de Negocio: BR-001, BR-010
### Eventos Generados: ConsultaRecibida
### Entidades Afectadas: Consulta
### User Stories Relacionadas: US-003

## Caso de Uso
### Identificador: UC-04
### Nombre: Listar consultas centralizadas
### Descripción: El responsable comercial consulta el listado unificado de leads de todos los canales.
### Actor: Responsable Comercial
### Contexto de Negocio: C2 Captación
### Precondiciones: Actor autenticado con rol autorizado.
### Flujo Principal:
1. El actor solicita el listado (con filtros opcionales por estado, canal, fecha).
2. El sistema valida autorización (BR-015).
3. El sistema devuelve las consultas con sus datos asociados.
### Flujos Alternativos:
- A1: no autenticado/ sin rol → ERR-030.
### Postcondiciones: Sin cambios de estado.
### Reglas de Negocio: BR-015
### Eventos Generados: —
### Entidades Afectadas: Consulta (lectura)
### User Stories Relacionadas: US-004

## Caso de Uso
### Identificador: UC-05
### Nombre: Actualizar estado de consulta
### Descripción: El responsable comercial cambia el estado de una consulta y el sistema historiza la transición.
### Actor: Responsable Comercial
### Contexto de Negocio: C2 Captación
### Precondiciones: Consulta existente; actor autorizado.
### Flujo Principal:
1. El actor selecciona una consulta y un nuevo estado válido.
2. El sistema valida la transición y la autorización (BR-002, BR-015).
3. El sistema persiste el nuevo estado y registra el cambio en el historial.
4. El sistema emite ConsultaCambioEstado.
### Flujos Alternativos:
- A1: consulta inexistente → ERR-011.
- A2: transición no válida → ERR-012.
### Postcondiciones: Estado actualizado e historizado.
### Reglas de Negocio: BR-002, BR-015
### Eventos Generados: ConsultaCambioEstado
### Entidades Afectadas: Consulta, CambioEstadoConsulta
### User Stories Relacionadas: US-005

## Caso de Uso
### Identificador: UC-26
### Nombre: Registrar solicitud de cotización
### Descripción: Un cliente corporativo/institucional solicita una cotización para un grupo.
### Actor: Cliente Corporativo
### Contexto de Negocio: C2 Captación (B2B)
### Precondiciones: Formulario de cotización disponible.
### Flujo Principal:
1. El cliente envía organización, número de personas, destino y fecha + contacto.
2. El sistema valida los campos requeridos (BR-010, BR-011).
3. El sistema crea la Solicitud de Cotización en estado RECIBIDA.
4. El sistema emite SolicitudCotizacionRecibida y confirma recepción.
### Flujos Alternativos:
- A1: datos incompletos → ERR-013; no se registra.
### Postcondiciones: Solicitud persistida en estado RECIBIDA.
### Reglas de Negocio: BR-010, BR-011
### Eventos Generados: SolicitudCotizacionRecibida
### Entidades Afectadas: SolicitudCotizacion
### User Stories Relacionadas: US-026

## Caso de Uso
### Identificador: UC-27
### Nombre: Registrar y responder cotización
### Descripción: El responsable comercial elabora y emite la cotización asociada a una solicitud.
### Actor: Responsable Comercial
### Contexto de Negocio: C2 Captación (B2B)
### Precondiciones: Solicitud existente; actor autorizado.
### Flujo Principal:
1. El actor abre una solicitud y registra el contenido de la cotización.
2. El sistema valida autorización (BR-015).
3. El sistema asocia la Cotización a la Solicitud y cambia su estado a COTIZACION_EMITIDA.
4. El sistema emite CotizacionEmitida y SolicitudCotizacionCambioEstado.
### Flujos Alternativos:
- A1: solicitud inexistente → ERR-014.
### Postcondiciones: Cotización asociada; estado e historial actualizados.
### Reglas de Negocio: BR-012, BR-002, BR-015
### Eventos Generados: CotizacionEmitida, SolicitudCotizacionCambioEstado
### Entidades Afectadas: SolicitudCotizacion, Cotizacion
### User Stories Relacionadas: US-027

## Caso de Uso
### Identificador: UC-30
### Nombre: Consultar disponibilidad
### Descripción: El cliente consulta fechas y cupos disponibles de una experiencia.
### Actor: Cliente
### Contexto de Negocio: C4 Reservas
### Precondiciones: Experiencia publicada y reservable.
### Flujo Principal:
1. El cliente solicita disponibilidad por experiencia (y rango de fechas).
2. El sistema devuelve las fechas con cupo y marca como no disponibles las sin cupo.
### Flujos Alternativos:
- A1: experiencia inexistente/ no publicada → ERR-020.
### Postcondiciones: Sin cambios de estado.
### Reglas de Negocio: BR-006
### Eventos Generados: DisponibilidadConsultada
### Entidades Afectadas: Disponibilidad (lectura)
### User Stories Relacionadas: US-030

## Caso de Uso
### Identificador: UC-31
### Nombre: Crear reserva
### Descripción: El cliente formaliza una reserva sobre una experiencia en una fecha con cupo.
### Actor: Cliente
### Contexto de Negocio: C4 Reservas
### Precondiciones: Disponibilidad con cupo suficiente.
### Flujo Principal:
1. El cliente selecciona experiencia, fecha y número de participantes.
2. El sistema valida cupo disponible (BR-003, BR-004).
3. El sistema crea la Reserva en estado INICIADA y reserva (bloquea) el cupo.
4. El sistema emite ReservaIniciada.
### Flujos Alternativos:
- A1: participantes > cupo → ERR-021; no se crea la reserva.
- A2: fecha sin cupo → ERR-022.
### Postcondiciones: Reserva en estado INICIADA; cupo reservado provisionalmente.
### Reglas de Negocio: BR-003, BR-004
### Eventos Generados: ReservaIniciada
### Entidades Afectadas: Reserva, Disponibilidad
### User Stories Relacionadas: US-031

## Caso de Uso
### Identificador: UC-32
### Nombre: Pagar reserva
### Descripción: El cliente paga una reserva pendiente; el resultado del pago determina la confirmación.
### Actor: Cliente
### Contexto de Negocio: C4 Reservas / Pagos
### Precondiciones: Reserva en estado INICIADA.
### Flujo Principal:
1. El cliente inicia el pago de la reserva (PagoIniciado).
2. La pasarela procesa y notifica el resultado vía webhook (idempotente).
3. Si el pago es EXITOSO, la Reserva pasa a PAGADA y luego CONFIRMADA (BR-005).
4. El sistema emite PagoConfirmado y ReservaConfirmada.
### Flujos Alternativos:
- A1: pago FALLIDO/RECHAZADO → la Reserva no se confirma (BR-007); ERR-023; libera cupo provisional.
- A2: webhook duplicado → se procesa de forma idempotente sin doble efecto.
### Postcondiciones: Reserva CONFIRMADA con Pago EXITOSO, o NO_CONFIRMADA.
### Reglas de Negocio: BR-005, BR-007
### Eventos Generados: PagoIniciado, PagoConfirmado | PagoRechazado, ReservaConfirmada
### Entidades Afectadas: Reserva, Pago, Disponibilidad
### User Stories Relacionadas: US-032

## Caso de Uso
### Identificador: UC-33
### Nombre: Emitir confirmación de reserva
### Descripción: Al confirmarse la reserva, el sistema genera y entrega la confirmación con su detalle.
### Actor: Sistema (desencadenado por ReservaConfirmada)
### Contexto de Negocio: C4 Reservas
### Precondiciones: Reserva CONFIRMADA.
### Flujo Principal:
1. El sistema genera la Confirmación con experiencia, fecha, participantes y datos.
2. El sistema emite ConfirmacionEmitida y dispara la notificación (asíncrona).
### Flujos Alternativos:
- A1: fallo en notificación → reintento; la Confirmación persiste igualmente.
### Postcondiciones: Confirmación persistida y entregada.
### Reglas de Negocio: BR-008
### Eventos Generados: ConfirmacionEmitida
### Entidades Afectadas: Confirmacion, Reserva
### User Stories Relacionadas: US-033

## Caso de Uso
### Identificador: UC-34
### Nombre: Gestionar reservas (back-office)
### Descripción: El gestor de operaciones consulta y actualiza el estado de reservas.
### Actor: Gestor de Operaciones
### Contexto de Negocio: C4 Reservas
### Precondiciones: Actor autorizado.
### Flujo Principal:
1. El actor lista reservas (con filtros por estado/fecha).
2. El actor actualiza el estado de una reserva (p. ej., CANCELADA, COMPLETADA).
3. El sistema valida la transición (BR-002, BR-015), historiza y, si corresponde, ajusta cupo (BR-009).
### Flujos Alternativos:
- A1: transición no válida → ERR-012.
### Postcondiciones: Estado actualizado e historizado; cupo consistente.
### Reglas de Negocio: BR-002, BR-009, BR-015
### Eventos Generados: ReservaCancelada (si aplica), CupoActualizado
### Entidades Afectadas: Reserva, Disponibilidad
### User Stories Relacionadas: US-034

## Caso de Uso
### Identificador: UC-36
### Nombre: Registrar solicitud/pedido de equipo
### Descripción: Un practicante/club solicita la cotización o pedido de equipos bajo importación.
### Actor: Practicante / Club
### Contexto de Negocio: C5 Equipos
### Precondiciones: Equipo publicado.
### Flujo Principal:
1. El cliente envía equipo(s), cantidad, tipo de cliente y contacto.
2. El sistema valida los datos requeridos (BR-013).
3. El sistema crea el Pedido de Equipo en estado RECIBIDO e informa la sujeción a importación (BR-014).
4. El sistema emite PedidoEquipoRecibido y confirma recepción.
### Flujos Alternativos:
- A1: datos incompletos → ERR-013.
### Postcondiciones: Pedido persistido en estado RECIBIDO.
### Reglas de Negocio: BR-013, BR-014
### Eventos Generados: PedidoEquipoRecibido
### Entidades Afectadas: PedidoEquipo, LineaPedidoEquipo
### User Stories Relacionadas: US-036

## Caso de Uso
### Identificador: UC-38
### Nombre: Gestionar pedidos de equipos (back-office)
### Descripción: El gestor de operaciones actualiza el estado de importación de un pedido.
### Actor: Gestor de Operaciones
### Contexto de Negocio: C5 Equipos
### Precondiciones: Pedido existente; actor autorizado.
### Flujo Principal:
1. El actor lista pedidos con su estado de importación.
2. El actor actualiza el estado (RECIBIDO→EN_COTIZACION→…→ENTREGADO/CANCELADO).
3. El sistema valida la transición (BR-002, BR-015) e historiza.
4. El sistema emite PedidoEquipoCambioEstadoImportacion.
### Flujos Alternativos:
- A1: transición no válida → ERR-012.
### Postcondiciones: Estado de importación actualizado e historizado.
### Reglas de Negocio: BR-002, BR-015
### Eventos Generados: PedidoEquipoCambioEstadoImportacion
### Entidades Afectadas: PedidoEquipo, CambioEstadoPedido
### User Stories Relacionadas: US-038

## Casos de Uso de consulta/presentación (resumen)
> Mismo patrón: Actor (Visitante salvo indicación), contexto indicado, sin cambios de
> estado, resultado de lectura, autorización pública salvo back-office.

- UC-01 Abrir contacto WhatsApp (C2, US-001) · genera deep link contextual; sin persistencia salvo registro opcional de evento.
- UC-02 Precargar mensaje WhatsApp por destino (C2, US-002).
- UC-06 Atender consulta en idioma del visitante (C2, US-006) · el idioma se persiste en la Consulta.
- UC-07 Listar experiencias por destino/dificultad (C1, US-007) → Query Q-01.
- UC-08 Ver detalle incluido/opcional (C1, US-008) → Q-02.
- UC-09 Comparar paquetes (C1, US-009) → Q-03.
- UC-10 Ver precios y condiciones (C1, US-010) → Q-02.
- UC-11 Ver licencias/certificaciones (C3, US-011) → Q-04.
- UC-12 Ver guías y credenciales (C3, US-012) → Q-05.
- UC-13 Ver protocolos de seguridad (C3, US-013) → Q-04.
- UC-14 Ver reseñas (C3, US-014) → Q-06.
- UC-15 Buscar en FAQ (C3, US-015) → Q-07.
- UC-16 Ver galería por destino (C1/C3 contenido, US-016) → Q-08.
- UC-17 Ver contenido social embebido (C1, US-017) → Q-09 (tolerante a indisponibilidad).
- UC-18 Listar/leer artículos (C1, US-018) → Q-10.
- UC-19 Ver página de destino (C1, US-019) → Q-11.
- UC-20 Servir contenido en idioma seleccionado (C1, US-020) → parámetro idioma transversal.
- UC-21 Exponer metadatos estructurados/sitemaps (C1 visibilidad, US-021) → Q-12.
- UC-22 Consultar/actualizar ficha local por sede (C1 visibilidad, US-022) → Q-13 (lectura) / Command C-10 (sync).
- UC-23 Filtrar oferta por perfil (C1, US-023) → Q-01 con filtro perfil.
- UC-24 Ver requisitos (edad/esfuerzo) (C1, US-024) → Q-02.
- UC-25 Recomendar experiencias por perfil (C1, US-025) → Q-14.
- UC-28 Ver propuesta de valor corporativa (C2, US-028) → Q-15.
- UC-29 Descargar material B2B (C2, US-029) → Q-16.
- UC-35 Ver catálogo de equipos (C5, US-035) → Q-17.
- UC-37 Ver equipos por tipo de cliente (C5, US-037) → Q-17 con filtro.

---

# 4. CATÁLOGO DE COMMANDS

## Command
### Nombre: C-01 CrearConsulta
### Descripción: Registra una consulta de un visitante por formulario o canal conversacional.
### Aggregate Responsable: Consulta (AGG-Consulta)
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| nombreContacto | Text | Sí |
| medioContacto | Text | Sí |
| destinoInteres | Ref(Destino) | No |
| mensaje | LongText | Sí |
| canal | Enum(Canal) | Sí |
| idioma | Enum(Idioma) | Sí |
### Validaciones: requeridos presentes; medioContacto con formato válido (email o teléfono); destinoInteres existente si se envía.
### Reglas de Negocio: BR-001, BR-010
### Resultado Esperado: Consulta creada en estado NUEVA; id devuelto.
### Eventos Generados: ConsultaRecibida

## Command
### Nombre: C-02 ActualizarEstadoConsulta
### Descripción: Cambia el estado de una consulta e historiza la transición.
### Aggregate Responsable: Consulta
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| consultaId | Id | Sí |
| nuevoEstado | Enum(EstadoConsulta) | Sí |
### Validaciones: consulta existente; transición permitida; actor autorizado.
### Reglas de Negocio: BR-002, BR-015
### Resultado Esperado: Estado actualizado; registro en historial.
### Eventos Generados: ConsultaCambioEstado

## Command
### Nombre: C-03 CrearSolicitudCotizacion
### Descripción: Registra una solicitud de cotización de grupo/B2B.
### Aggregate Responsable: SolicitudCotizacion (AGG-SolicitudCotizacion)
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| organizacion | Text | Sí |
| numeroPersonas | Int | Sí |
| destino | Ref(Destino) | Sí |
| fechaSolicitada | Date | Sí |
| datosContacto | DatosContacto | Sí |
### Validaciones: requeridos presentes; numeroPersonas > 0; destino existente; fecha válida.
### Reglas de Negocio: BR-010, BR-011
### Resultado Esperado: Solicitud creada en estado RECIBIDA.
### Eventos Generados: SolicitudCotizacionRecibida

## Command
### Nombre: C-04 EmitirCotizacion
### Descripción: Asocia y emite una cotización para una solicitud.
### Aggregate Responsable: SolicitudCotizacion
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| solicitudId | Id | Sí |
| contenido | LongText | Sí |
### Validaciones: solicitud existente; actor autorizado; contenido no vacío.
### Reglas de Negocio: BR-012, BR-002, BR-015
### Resultado Esperado: Cotización asociada; estado COTIZACION_EMITIDA.
### Eventos Generados: CotizacionEmitida, SolicitudCotizacionCambioEstado

## Command
### Nombre: C-05 CrearReserva
### Descripción: Crea una reserva validando cupo de la disponibilidad.
### Aggregate Responsable: Reserva (AGG-Reserva)
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| experienciaId | Id | Sí |
| disponibilidadId | Id | Sí |
| fecha | Date | Sí |
| numeroParticipantes | Int | Sí |
| datosCliente | DatosContacto | Sí |
### Validaciones: experiencia publicada; disponibilidad existente; numeroParticipantes ≤ cupoDisponible; numeroParticipantes > 0.
### Reglas de Negocio: BR-003, BR-004
### Resultado Esperado: Reserva en estado INICIADA; cupo reservado provisionalmente.
### Eventos Generados: ReservaIniciada

## Command
### Nombre: C-06 IniciarPago
### Descripción: Inicia el pago de una reserva pendiente.
### Aggregate Responsable: Reserva (entidad Pago)
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| reservaId | Id | Sí |
| monto | Money | Sí |
### Validaciones: reserva en estado INICIADA; monto coincide con la reserva.
### Reglas de Negocio: BR-005
### Resultado Esperado: Pago en estado INICIADO; referencia de pasarela generada.
### Eventos Generados: PagoIniciado

## Command
### Nombre: C-07 RegistrarResultadoPago
### Descripción: Procesa el resultado del pago recibido por webhook (idempotente).
### Aggregate Responsable: Reserva (entidad Pago)
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| reservaId | Id | Sí |
| referenciaPasarela | Text | Sí |
| resultado | Enum(EstadoPago) | Sí |
### Validaciones: firma/origen del webhook válidos; idempotencia por referencia; reserva existente.
### Reglas de Negocio: BR-005, BR-007
### Resultado Esperado: Si EXITOSO → Reserva PAGADA→CONFIRMADA; si FALLIDO/RECHAZADO → NO_CONFIRMADA y liberación de cupo.
### Eventos Generados: PagoConfirmado | PagoRechazado, ReservaConfirmada (si aplica), CupoActualizado

## Command
### Nombre: C-08 ActualizarEstadoReserva
### Descripción: Cambia el estado operativo de una reserva (back-office).
### Aggregate Responsable: Reserva
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| reservaId | Id | Sí |
| nuevoEstado | Enum(EstadoReserva) | Sí |
### Validaciones: reserva existente; transición permitida; actor autorizado.
### Reglas de Negocio: BR-002, BR-009, BR-015
### Resultado Esperado: Estado actualizado e historizado; cupo ajustado si CANCELADA.
### Eventos Generados: ReservaCancelada (si aplica), CupoActualizado (si aplica)

## Command
### Nombre: C-09 CrearPedidoEquipo
### Descripción: Registra un pedido/solicitud de cotización de equipos.
### Aggregate Responsable: PedidoEquipo (AGG-PedidoEquipo)
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| lineas | List<{equipoId: Id, cantidad: Int}> | Sí |
| tipoCliente | Enum(PerfilCliente) | Sí |
| datosContacto | DatosContacto | Sí |
### Validaciones: al menos una línea; equipos publicados; cantidades > 0.
### Reglas de Negocio: BR-013, BR-014
### Resultado Esperado: Pedido en estado RECIBIDO; aviso de sujeción a importación.
### Eventos Generados: PedidoEquipoRecibido

## Command
### Nombre: C-10 ActualizarEstadoPedidoEquipo
### Descripción: Actualiza el estado de importación de un pedido (back-office).
### Aggregate Responsable: PedidoEquipo
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| pedidoId | Id | Sí |
| nuevoEstado | Enum(EstadoPedidoEquipo) | Sí |
### Validaciones: pedido existente; transición permitida; actor autorizado.
### Reglas de Negocio: BR-002, BR-015
### Resultado Esperado: Estado de importación actualizado e historizado.
### Eventos Generados: PedidoEquipoCambioEstadoImportacion

## Command
### Nombre: C-11 SincronizarFichaLocal
### Descripción: Actualiza/propaga la información de la ficha local de una sede.
### Aggregate Responsable: Sede (C1) / visibility
### Inputs
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| sedeId | Id | Sí |
| datosFicha | Text | Sí |
### Validaciones: sede existente; actor autorizado.
### Reglas de Negocio: BR-017
### Resultado Esperado: Ficha local consistente con la información vigente.
### Eventos Generados: — (operación de sincronización vía adaptador)

> Nota: la publicación/despublicación de contenido (Destino/Experiencia/Artículo/Equipo)
> se gestiona por comandos de administración de contenido (C-12 PublicarContenido /
> C-13 DespublicarContenido), sujetos a BR-016 y autorización (BR-015), emitiendo
> ContenidoPublicado / ContenidoDespublicado.

---

# 5. CATÁLOGO DE QUERIES

## Query
### Nombre: Q-01 ListarExperiencias
### Descripción: Lista experiencias publicadas, con filtros.
### Parámetros: idioma
### Filtros: destino, dificultad, perfilCliente, estado=PUBLICADO
### Ordenamiento: por destino, por dificultad
### Paginación: page, pageSize
### Resultado: lista de Experiencia (resumen) con disponibilidad indicativa.
### Restricciones: solo contenido publicado (BR-016); pública.

## Query
### Nombre: Q-02 ObtenerDetalleExperiencia
### Descripción: Devuelve el detalle de una experiencia (componentes incluido/opcional, precio, condiciones, requisitos).
### Parámetros: experienciaId, idioma
### Filtros: —
### Ordenamiento: componentes por tipo
### Paginación: —
### Resultado: Experiencia detallada + Componentes + RequisitosParticipacion.
### Restricciones: solo si PUBLICADO; pública.

## Query
### Nombre: Q-03 CompararPaquetes
### Descripción: Devuelve dos o más experiencias para comparación lado a lado.
### Parámetros: experienciaIds[], idioma
### Filtros: —
### Resultado: matriz comparativa (componentes, duración, precio).
### Restricciones: 2..N ids publicados; pública.

## Query
### Nombre: Q-04 ObtenerCentroConfianza
### Descripción: Devuelve certificaciones/licencias y protocolos de seguridad.
### Parámetros: idioma
### Resultado: Credenciales (con entidad emisora) + protocolos.
### Restricciones: pública.

## Query
### Nombre: Q-05 ListarGuias
### Descripción: Devuelve perfiles de guías con credenciales.
### Parámetros: idioma
### Resultado: lista de Guia + credenciales.
### Restricciones: pública.

## Query
### Nombre: Q-06 ListarResenas
### Descripción: Devuelve reseñas, opcionalmente por destino/experiencia.
### Parámetros: objetivoRef (opcional), idioma
### Paginación: page, pageSize
### Resultado: lista de Resena.
### Restricciones: pública; ausencia de reseñas no es error (BR-derivada de US-014).

## Query
### Nombre: Q-07 BuscarFaq
### Descripción: Busca FAQ por término/categoría.
### Parámetros: termino (opcional), categoria (opcional), idioma
### Resultado: lista de Faq relevantes.
### Restricciones: pública.

## Query
### Nombre: Q-08 ObtenerGaleria
### Descripción: Devuelve activos multimedia por destino/experiencia.
### Parámetros: destinoId | experienciaId, idioma
### Resultado: lista de ActivoMultimedia (con referencia de almacenamiento).
### Restricciones: pública; carga progresiva a nivel de presentación.

## Query
### Nombre: Q-09 ObtenerContenidoSocial
### Descripción: Devuelve publicaciones sociales embebibles.
### Parámetros: —
### Resultado: bloque de contenido social.
### Restricciones: tolerante a indisponibilidad (degradación elegante); pública.

## Query
### Nombre: Q-10 ListarArticulos
### Descripción: Lista/lee artículos del blog.
### Parámetros: idioma, relacionadoRef (opcional)
### Paginación: page, pageSize
### Resultado: lista/detalle de Articulo (PUBLICADO).
### Restricciones: solo PUBLICADO; pública.

## Query
### Nombre: Q-11 ObtenerPaginaDestino
### Descripción: Devuelve el contenido de la página de un río + experiencias del destino.
### Parámetros: destinoId | slug, idioma
### Resultado: Destino + experiencias asociadas + vía de contacto.
### Restricciones: solo PUBLICADO; pública.

## Query
### Nombre: Q-12 ObtenerMetadatosEstructurados
### Descripción: Genera metadatos/schema y sitemaps para una página publicada.
### Parámetros: recursoRef, idioma
### Resultado: metadatos estructurados.
### Restricciones: solo recursos PUBLICADOS; consumo de buscadores/IA.

## Query
### Nombre: Q-13 ObtenerFichaLocal
### Descripción: Devuelve la información de la ficha local de una sede.
### Parámetros: sedeId
### Resultado: datos de contacto, ubicación y reseñas de la sede.
### Restricciones: pública.

## Query
### Nombre: Q-14 RecomendarExperiencias
### Descripción: Devuelve experiencias acordes a un perfil/preferencias.
### Parámetros: perfilCliente, idioma
### Resultado: lista priorizada de Experiencia.
### Restricciones: solo PUBLICADO; pública.

## Query
### Nombre: Q-15 ObtenerPropuestaCorporativa
### Descripción: Devuelve la propuesta de valor B2B y vía de contacto.
### Parámetros: idioma
### Resultado: contenido corporativo.
### Restricciones: pública.

## Query
### Nombre: Q-16 ObtenerMaterialB2B
### Descripción: Devuelve el material descargable vigente para agencias.
### Parámetros: idioma
### Resultado: referencia al documento vigente.
### Restricciones: pública; versión vigente (US-029 AC-002).

## Query
### Nombre: Q-17 ListarEquipos
### Descripción: Lista el catálogo de equipos, con filtro por tipo de cliente.
### Parámetros: idioma
### Filtros: tipoCliente (opcional), estado=PUBLICADO
### Paginación: page, pageSize
### Resultado: lista de Equipo; si no hay oferta específica, oferta general (BR-018).
### Restricciones: solo PUBLICADO; pública.

## Query
### Nombre: Q-18 ListarConsultas (back-office)
### Descripción: Lista leads centralizados con filtros.
### Parámetros: —
### Filtros: estado, canal, rango de fechas
### Ordenamiento: por fecha desc
### Paginación: page, pageSize
### Resultado: lista de Consulta con datos asociados.
### Restricciones: requiere autorización (BR-015).

## Query
### Nombre: Q-19 ListarSolicitudesCotizacion (back-office)
### Descripción: Lista solicitudes de cotización con su estado.
### Filtros: estado, destino, rango de fechas
### Paginación: page, pageSize
### Resultado: lista de SolicitudCotizacion.
### Restricciones: requiere autorización (BR-015).

## Query
### Nombre: Q-20 ListarReservas (back-office)
### Descripción: Lista reservas con su estado.
### Filtros: estado, experiencia, rango de fechas
### Paginación: page, pageSize
### Resultado: lista de Reserva.
### Restricciones: requiere autorización (BR-015).

## Query
### Nombre: Q-21 ListarPedidosEquipo (back-office)
### Descripción: Lista pedidos de equipos con su estado de importación.
### Filtros: estadoImportacion, rango de fechas
### Paginación: page, pageSize
### Resultado: lista de PedidoEquipo.
### Restricciones: requiere autorización (BR-015).

---

# 6. CATÁLOGO DE DTOs

## Request DTOs

### Nombre: CrearConsultaRequest
### Campos: nombreContacto: string; medioContacto: string; destinoInteresId?: string; mensaje: string; canal: enum; idioma: enum
### Restricciones: nombreContacto, mensaje no vacíos; medioContacto formato email/teléfono; canal e idioma de los enums.
### Ejemplo:
```
{ "nombreContacto": "Ana Pérez", "medioContacto": "ana@example.com",
  "destinoInteresId": "DST-URU", "mensaje": "Quiero info de rafting", "canal": "FORMULARIO", "idioma": "ES" }
```

### Nombre: CrearSolicitudCotizacionRequest
### Campos: organizacion: string; numeroPersonas: int; destinoId: string; fechaSolicitada: date; contacto: {nombre, email?, telefono?}
### Restricciones: numeroPersonas ≥ 1; destinoId existente; fechaSolicitada ≥ hoy.
### Ejemplo:
```
{ "organizacion": "Colegio X", "numeroPersonas": 25, "destinoId": "DST-LUN",
  "fechaSolicitada": "2026-08-15", "contacto": { "nombre": "Luis", "telefono": "+51..." } }
```

### Nombre: CrearReservaRequest
### Campos: experienciaId: string; disponibilidadId: string; fecha: date; numeroParticipantes: int; cliente: {nombre, email?, telefono?}
### Restricciones: numeroParticipantes ≥ 1; fecha con cupo; experiencia publicada.
### Ejemplo:
```
{ "experienciaId": "EXP-URU-01", "disponibilidadId": "DISP-0912",
  "fecha": "2026-07-20", "numeroParticipantes": 3, "cliente": { "nombre": "John", "email": "john@x.com" } }
```

### Nombre: IniciarPagoRequest
### Campos: reservaId: string; monto: {monto: number, moneda: enum}
### Restricciones: reservaId en estado INICIADA; moneda en {PEN, USD}.
### Ejemplo:
```
{ "reservaId": "RES-1001", "monto": { "monto": 135.00, "moneda": "USD" } }
```

### Nombre: CrearPedidoEquipoRequest
### Campos: lineas: [{equipoId: string, cantidad: int}]; tipoCliente: enum; contacto: {...}
### Restricciones: ≥ 1 línea; cantidad ≥ 1; equipos publicados.
### Ejemplo:
```
{ "lineas": [ { "equipoId": "EQ-KAYAK-01", "cantidad": 2 } ],
  "tipoCliente": "CLUB", "contacto": { "nombre": "Club Andino", "email": "club@x.com" } }
```

### Nombre: ActualizarEstadoRequest (genérico consulta/cotización/reserva/pedido)
### Campos: id: string; nuevoEstado: enum
### Restricciones: transición permitida según la máquina de estados de la entidad.
### Ejemplo:
```
{ "id": "RES-1001", "nuevoEstado": "CANCELADA" }
```

## Response DTOs

### Nombre: ConsultaResponse
### Campos: id; estado; canal; idioma; destinoInteresId?; fecha; recepcionConfirmada: bool
### Ejemplo:
```
{ "id": "LEAD-5001", "estado": "NUEVA", "canal": "FORMULARIO", "idioma": "ES",
  "destinoInteresId": "DST-URU", "fecha": "2026-06-10T14:02:00Z", "recepcionConfirmada": true }
```

### Nombre: ExperienciaDetalleResponse
### Campos: id; nombre; destinoId; duracion; dificultad; precioReferencia {monto,moneda}; baseCalculoPrecio; condiciones; componentes [{nombre,tipo,condicion}]; requisitos {edadMinima,nivelEsfuerzo}
### Ejemplo:
```
{ "id": "EXP-URU-01", "nombre": "Rafting Urubamba día completo", "destinoId": "DST-URU",
  "duracion": "1 día", "dificultad": "III", "precioReferencia": { "monto": 45, "moneda": "USD" },
  "baseCalculoPrecio": "POR_PERSONA", "condiciones": "...", 
  "componentes": [ { "nombre": "Almuerzo", "tipo": "ALIMENTACION", "condicion": "INCLUIDO" } ],
  "requisitos": { "edadMinima": 12, "nivelEsfuerzo": "MEDIO" } }
```

### Nombre: DisponibilidadResponse
### Campos: experienciaId; fechas [{fecha, cupoDisponible, estado}]
### Ejemplo:
```
{ "experienciaId": "EXP-URU-01", "fechas": [ { "fecha": "2026-07-20", "cupoDisponible": 6, "estado": "DISPONIBLE" } ] }
```

### Nombre: ReservaResponse
### Campos: id; estado; experienciaId; fecha; numeroParticipantes; pago {estado}; confirmacionId?
### Ejemplo:
```
{ "id": "RES-1001", "estado": "CONFIRMADA", "experienciaId": "EXP-URU-01",
  "fecha": "2026-07-20", "numeroParticipantes": 3, "pago": { "estado": "EXITOSO" }, "confirmacionId": "CONF-7001" }
```

### Nombre: PedidoEquipoResponse
### Campos: id; estadoImportacion; lineas [{equipoId,cantidad}]; sujetoImportacion: bool
### Ejemplo:
```
{ "id": "PED-3001", "estadoImportacion": "RECIBIDO",
  "lineas": [ { "equipoId": "EQ-KAYAK-01", "cantidad": 2 } ], "sujetoImportacion": true }
```

### Nombre: ErrorResponse (transversal)
### Campos: codigo; mensaje; detalles? [{campo, error}]
### Ejemplo:
```
{ "codigo": "ERR-010", "mensaje": "Datos de consulta inválidos",
  "detalles": [ { "campo": "medioContacto", "error": "formato inválido" } ] }
```

---

# 7. REGLAS DE NEGOCIO

| ID | Regla |
| -- | ----- |
| BR-001 | Una Consulta requiere nombre, medio de contacto y mensaje; sin ellos no se registra. |
| BR-002 | Todo cambio de estado (consulta, cotización, reserva, pedido) debe ser una transición válida y quedar historizado. |
| BR-003 | Una Reserva no puede registrar más participantes que el cupo disponible. |
| BR-004 | El número de participantes de una Reserva debe ser mayor que cero. |
| BR-005 | El pago de una Reserva solo procede si la reserva está en estado INICIADA. |
| BR-006 | Una fecha sin cupo disponible se presenta como no disponible. |
| BR-007 | Una Reserva solo alcanza CONFIRMADA si su Pago es EXITOSO; pago fallido/rechazado deja la reserva NO_CONFIRMADA. |
| BR-008 | Toda Reserva confirmada genera una Confirmación con su detalle. |
| BR-009 | Al confirmarse o cancelarse una Reserva, el cupo de su Disponibilidad se ajusta de forma consistente. |
| BR-010 | Los campos requeridos de un formulario deben validarse antes de registrar; si faltan, no se registra. |
| BR-011 | Una Solicitud de Cotización requiere organización, número de personas, destino y fecha. |
| BR-012 | Una Cotización queda siempre asociada a su Solicitud y a un estado de atención. |
| BR-013 | Un Pedido de Equipo requiere al menos una línea con cantidad mayor que cero. |
| BR-014 | Todo Pedido/Solicitud de Equipo informa la sujeción a disponibilidad de importación. |
| BR-015 | Las operaciones de back-office requieren Actor Operativo autenticado y autorizado por rol. |
| BR-016 | Solo el contenido en estado PUBLICADO es visible/contratable públicamente. |
| BR-017 | La ficha local de una sede refleja la información vigente tras su actualización. |
| BR-018 | Si no existe oferta de equipos específica para un tipo de cliente, se presenta la oferta general. |
| BR-019 | El idioma seleccionado se conserva durante la navegación y se registra en la Consulta. |
| BR-020 | Los webhooks de pago/mensajería se procesan de forma idempotente (no duplican efectos). |

---

# 8. VALIDACIONES

## Validaciones Sintácticas
| Campo | Regla | Error |
| ----- | ----- | ----- |
| medioContacto | Formato email o teléfono válido | ERR-010 |
| numeroPersonas / numeroParticipantes / cantidad | Entero ≥ 1 | ERR-010 |
| fecha / fechaSolicitada | Formato fecha válido | ERR-010 |
| canal / idioma / estado | Valor dentro del enum permitido | ERR-012 |
| monto.moneda | En {PEN, USD} | ERR-010 |

## Validaciones Semánticas
| Campo | Regla | Error |
| ----- | ----- | ----- |
| destinoInteres / destinoId | Debe existir y estar publicado | ERR-011 |
| experienciaId | Debe existir y estar PUBLICADO | ERR-020 |
| fechaSolicitada / fecha | No anterior a hoy | ERR-010 |
| monto | Debe coincidir con el importe de la reserva | ERR-024 |
| lineas[].equipoId | Equipo existente y publicado | ERR-025 |

## Validaciones de Dominio
| Campo | Regla | Error |
| ----- | ----- | ----- |
| numeroParticipantes | ≤ cupoDisponible (BR-003) | ERR-021 |
| disponibilidad.fecha | Debe tener cupo (BR-006) | ERR-022 |
| reserva.estado | Pago solo si INICIADA (BR-005) | ERR-026 |
| reserva→confirmación | Confirmar solo con Pago EXITOSO (BR-007) | ERR-023 |
| transición de estado | Debe ser válida en la máquina de estados (BR-002) | ERR-012 |
| solicitudCotizacion | Campos obligatorios completos (BR-011) | ERR-013 |

## Validaciones de Seguridad
| Campo | Regla | Error |
| ----- | ----- | ----- |
| sesión/rol | Operación back-office requiere rol autorizado (BR-015) | ERR-030 |
| webhook | Firma/origen válidos e idempotencia (BR-020) | ERR-031 |
| formularios públicos | Límite de tasa / anti-bot | ERR-032 |
| acceso a recurso | Recurso no publicado no accesible públicamente (BR-016) | ERR-011 |

---

# 9. DOMAIN EVENTS

## Evento
### Nombre: ConsultaRecibida
### Descripción: Se registró una nueva consulta.
### Aggregate Origen: Consulta
### Disparador: C-01 CrearConsulta
### Payload: { consultaId, canal, idioma, destinoInteresId? }
### Consumidores Potenciales: notification-service (aviso interno), observability (métrica por canal/destino/idioma)

## Evento
### Nombre: ConsultaCambioEstado
### Descripción: Cambió el estado de una consulta.
### Aggregate Origen: Consulta
### Disparador: C-02 ActualizarEstadoConsulta
### Payload: { consultaId, estadoNuevo }
### Consumidores Potenciales: observability

## Evento
### Nombre: SolicitudCotizacionRecibida
### Descripción: Se registró una solicitud de cotización B2B.
### Aggregate Origen: SolicitudCotizacion
### Disparador: C-03 CrearSolicitudCotizacion
### Payload: { solicitudId, destinoId }
### Consumidores Potenciales: notification-service, observability

## Evento
### Nombre: CotizacionEmitida
### Descripción: Se emitió la cotización de una solicitud.
### Aggregate Origen: SolicitudCotizacion
### Disparador: C-04 EmitirCotizacion
### Payload: { solicitudId, cotizacionId }
### Consumidores Potenciales: notification-service

## Evento
### Nombre: SolicitudCotizacionCambioEstado
### Descripción: Cambió el estado de una solicitud de cotización.
### Aggregate Origen: SolicitudCotizacion
### Disparador: C-04 / actualización de estado
### Payload: { solicitudId, estadoNuevo }
### Consumidores Potenciales: observability

## Evento
### Nombre: DisponibilidadConsultada
### Descripción: Se consultó la disponibilidad de una experiencia.
### Aggregate Origen: Disponibilidad
### Disparador: Q-... consulta de disponibilidad (UC-30)
### Payload: { experienciaId, fecha }
### Consumidores Potenciales: observability (demanda por experiencia/fecha)

## Evento
### Nombre: ReservaIniciada
### Descripción: Se creó una reserva pendiente de pago.
### Aggregate Origen: Reserva
### Disparador: C-05 CrearReserva
### Payload: { reservaId, experienciaId, fecha, numeroParticipantes }
### Consumidores Potenciales: payment-service (inicio de pago), observability

## Evento
### Nombre: PagoIniciado
### Descripción: Se inició el pago de una reserva.
### Aggregate Origen: Reserva (Pago)
### Disparador: C-06 IniciarPago
### Payload: { reservaId, pagoId, monto }
### Consumidores Potenciales: integration-adapters (pasarela)

## Evento
### Nombre: PagoConfirmado
### Descripción: El pago resultó exitoso.
### Aggregate Origen: Reserva (Pago)
### Disparador: C-07 RegistrarResultadoPago (EXITOSO)
### Payload: { reservaId, pagoId }
### Consumidores Potenciales: booking-service (confirmar), notification-service

## Evento
### Nombre: PagoRechazado
### Descripción: El pago falló o fue rechazado.
### Aggregate Origen: Reserva (Pago)
### Disparador: C-07 RegistrarResultadoPago (FALLIDO/RECHAZADO)
### Payload: { reservaId, pagoId, motivo? }
### Consumidores Potenciales: booking-service (liberar cupo), notification-service

## Evento
### Nombre: ReservaConfirmada
### Descripción: La reserva quedó confirmada tras pago exitoso.
### Aggregate Origen: Reserva
### Disparador: C-07 (tras PagoConfirmado)
### Payload: { reservaId }
### Consumidores Potenciales: notification-service (confirmación), observability

## Evento
### Nombre: ConfirmacionEmitida
### Descripción: Se generó la confirmación de la reserva.
### Aggregate Origen: Reserva (Confirmacion)
### Disparador: UC-33
### Payload: { reservaId, confirmacionId }
### Consumidores Potenciales: notification-service

## Evento
### Nombre: ReservaCancelada
### Descripción: Una reserva fue cancelada.
### Aggregate Origen: Reserva
### Disparador: C-08 ActualizarEstadoReserva (CANCELADA)
### Payload: { reservaId }
### Consumidores Potenciales: booking-service (liberar cupo), notification-service

## Evento
### Nombre: CupoActualizado
### Descripción: Cambió el cupo disponible de una disponibilidad.
### Aggregate Origen: Disponibilidad
### Disparador: confirmación/cancelación/liberación de reserva
### Payload: { disponibilidadId, cupoDisponible }
### Consumidores Potenciales: content-service (presentación), observability

## Evento
### Nombre: PedidoEquipoRecibido
### Descripción: Se registró un pedido de equipos.
### Aggregate Origen: PedidoEquipo
### Disparador: C-09 CrearPedidoEquipo
### Payload: { pedidoId }
### Consumidores Potenciales: notification-service, observability

## Evento
### Nombre: PedidoEquipoCambioEstadoImportacion
### Descripción: Cambió el estado de importación de un pedido.
### Aggregate Origen: PedidoEquipo
### Disparador: C-10 ActualizarEstadoPedidoEquipo
### Payload: { pedidoId, estadoNuevo }
### Consumidores Potenciales: notification-service (aviso al cliente)

## Evento
### Nombre: ContenidoPublicado / ContenidoDespublicado
### Descripción: Se publicó/despublicó contenido (destino, experiencia, artículo, equipo).
### Aggregate Origen: Destino | Experiencia | Articulo | Equipo
### Disparador: C-12 / C-13
### Payload: { tipo, id }
### Consumidores Potenciales: visibility-service (sitemaps/metadatos), content-service (caché)

---

# 10. MATRIZ DE AUTORIZACIÓN

| Rol | Acción | Permitido |
| --- | ------ | --------- |
| Visitante / Público | Lectura de contenido publicado (Q-01..Q-17) | Sí |
| Visitante / Público | Crear Consulta (C-01) | Sí |
| Visitante / Público | Crear Solicitud de Cotización (C-03) | Sí |
| Visitante / Público | Crear Pedido de Equipo (C-09) | Sí |
| Cliente | Consultar disponibilidad (UC-30) | Sí |
| Cliente | Crear Reserva (C-05) | Sí |
| Cliente | Iniciar Pago (C-06) | Sí |
| Cliente | Lectura de su Confirmación | Sí (propia) |
| Cliente | Operaciones de back-office | No |
| Responsable Comercial | Listar/leer consultas y cotizaciones (Q-18, Q-19) | Sí |
| Responsable Comercial | Actualizar estado de consulta (C-02) | Sí |
| Responsable Comercial | Emitir cotización (C-04) | Sí |
| Responsable Comercial | Gestionar reservas/pedidos | No (salvo lectura) |
| Gestor de Operaciones | Listar/gestionar reservas (Q-20, C-08) | Sí |
| Gestor de Operaciones | Listar/gestionar pedidos de equipo (Q-21, C-10) | Sí |
| Gestor de Operaciones | Publicar/despublicar contenido (C-12/C-13) | Sí (según política) |
| Gestor de Operaciones | Sincronizar ficha local (C-11) | Sí |
| Sistema (webhook) | Registrar resultado de pago (C-07) | Sí (con firma válida) |
| Cualquier rol no autenticado | Operaciones especiales/eliminación back-office | No |

> Operaciones especiales (publicación, sincronización, cambios de estado) y
> eliminación lógica de contenido quedan restringidas a roles operativos.
> No se define eliminación física de datos transaccionales (solo cambios de estado).

---

# 11. CATÁLOGO DE ERRORES

## Error
### Código: ERR-010
### Mensaje: Datos de formulario inválidos
### Causa: Campos requeridos vacíos o con formato incorrecto.
### Acción Correctiva: Señalar los campos a corregir; no registrar.

## Error
### Código: ERR-011
### Mensaje: Recurso no encontrado o no disponible
### Causa: Destino/experiencia/recurso inexistente o no publicado.
### Acción Correctiva: Verificar el identificador o el estado de publicación.

## Error
### Código: ERR-012
### Mensaje: Transición de estado no válida
### Causa: Cambio de estado no permitido por la máquina de estados.
### Acción Correctiva: Usar una transición válida.

## Error
### Código: ERR-013
### Mensaje: Solicitud de cotización incompleta
### Causa: Falta organización, número de personas, destino o fecha.
### Acción Correctiva: Completar los campos requeridos.

## Error
### Código: ERR-014
### Mensaje: Solicitud de cotización no encontrada
### Causa: La solicitud referida no existe.
### Acción Correctiva: Verificar el identificador de la solicitud.

## Error
### Código: ERR-020
### Mensaje: Experiencia no disponible
### Causa: Experiencia inexistente o no publicada.
### Acción Correctiva: Seleccionar una experiencia publicada.

## Error
### Código: ERR-021
### Mensaje: Capacidad insuficiente
### Causa: Participantes solicitados superan el cupo disponible.
### Acción Correctiva: Reducir participantes o elegir otra fecha; se informa el cupo.

## Error
### Código: ERR-022
### Mensaje: Fecha sin disponibilidad
### Causa: La fecha seleccionada no tiene cupo.
### Acción Correctiva: Elegir una fecha con cupo.

## Error
### Código: ERR-023
### Mensaje: Pago no completado
### Causa: El pago falló o fue rechazado.
### Acción Correctiva: Reintentar el pago; la reserva no se confirma.

## Error
### Código: ERR-024
### Mensaje: Monto de pago inconsistente
### Causa: El monto no coincide con el importe de la reserva.
### Acción Correctiva: Recalcular el importe de la reserva.

## Error
### Código: ERR-025
### Mensaje: Equipo no disponible
### Causa: Equipo inexistente o no publicado en una línea de pedido.
### Acción Correctiva: Seleccionar un equipo publicado.

## Error
### Código: ERR-026
### Mensaje: Estado de reserva no apto para pago
### Causa: La reserva no está en estado INICIADA.
### Acción Correctiva: Verificar el estado de la reserva.

## Error
### Código: ERR-030
### Mensaje: Usuario sin permisos
### Causa: Operación back-office sin autenticación o rol autorizado.
### Acción Correctiva: Autenticarse con el rol correspondiente.

## Error
### Código: ERR-031
### Mensaje: Webhook no válido
### Causa: Firma/origen inválidos o evento duplicado.
### Acción Correctiva: Rechazar el evento; procesar solo eventos válidos e idempotentes.

## Error
### Código: ERR-032
### Mensaje: Límite de solicitudes excedido
### Causa: Abuso de formulario / superación de límite de tasa.
### Acción Correctiva: Reintentar más tarde; aplicar protección anti-bot.

---

# 12. REQUISITOS NO FUNCIONALES BACKEND

## Rendimiento
Endpoints públicos de lectura optimizados para baja latencia (apoyados por
SSG/ISR y CDN en la capa de presentación); la consulta de disponibilidad y el
detalle de experiencia deben responder con rapidez por su rol en la conversión.

## Escalabilidad
Servicios de dominio escalables de forma independiente; la capa pública escala
horizontalmente; procesos asíncronos (notificaciones, sincronizaciones) como
trabajadores separados. Límites de dominio explícitos para extraer servicios.

## Seguridad
HTTPS/TLS; autorización por rol en back-office; tokenización de pagos (sin datos
de tarjeta en la plataforma); validación de webhooks; protección anti-abuso en
formularios; gestión de secretos; cumplimiento de la Ley N.º 29733 (datos
personales).

## Auditoría
Historial de cambios de estado en consultas, cotizaciones, reservas y pedidos;
registro de accesos y operaciones sobre entidades sensibles (reservas, pagos,
pedidos).

## Observabilidad
Logs, métricas y trazas; métricas de negocio (consultas por canal/destino/idioma,
disponibilidad consultada, reservas confirmadas) expuestas para el back-office.

## Trazabilidad
Cada operación mantiene el vínculo Oportunidad → Epic → Feature → User Story →
Caso de Uso → Command/Query → Entidad (ver §13).

---

# 13. MATRIZ DE TRAZABILIDAD

| Epic | Feature | User Story | Use Case | Command | Query | Entidad |
| ---- | ------- | ---------- | -------- | ------- | ----- | ------- |
| EP-001 | FT-001 | US-001 | UC-01 | — | — | Consulta |
| EP-001 | FT-001 | US-002 | UC-02 | — | — | Consulta |
| EP-001 | FT-002 | US-003 | UC-03 | C-01 | — | Consulta |
| EP-001 | FT-003 | US-004 | UC-04 | — | Q-18 | Consulta |
| EP-001 | FT-004 | US-005 | UC-05 | C-02 | — | Consulta, CambioEstadoConsulta |
| EP-001 | FT-005 | US-006 | UC-06 | C-01 | — | Consulta |
| EP-002 | FT-006 | US-007 | UC-07 | — | Q-01 | Experiencia |
| EP-002 | FT-007 | US-008 | UC-08 | — | Q-02 | Experiencia, Componente |
| EP-002 | FT-008 | US-009 | UC-09 | — | Q-03 | Experiencia |
| EP-002 | FT-009 | US-010 | UC-10 | — | Q-02 | Experiencia |
| EP-003 | FT-010 | US-011 | UC-11 | — | Q-04 | Credencial |
| EP-003 | FT-011 | US-012 | UC-12 | — | Q-05 | Guia, Credencial |
| EP-003 | FT-012 | US-013 | UC-13 | — | Q-04 | (Protocolos) |
| EP-003 | FT-013 | US-014 | UC-14 | — | Q-06 | Resena |
| EP-003 | FT-014 | US-015 | UC-15 | — | Q-07 | Faq |
| EP-004 | FT-015 | US-016 | UC-16 | — | Q-08 | ActivoMultimedia |
| EP-004 | FT-016 | US-017 | UC-17 | — | Q-09 | (Contenido social) |
| EP-004 | FT-017 | US-018 | UC-18 | — | Q-10 | Articulo |
| EP-005 | FT-018 | US-019 | UC-19 | — | Q-11 | Destino, Experiencia |
| EP-005 | FT-019 | US-020 | UC-20 | — | Q-01..Q-17 (idioma) | (Contenido) |
| EP-005 | FT-020 | US-021 | UC-21 | — | Q-12 | (Metadatos) |
| EP-005 | FT-021 | US-022 | UC-22 | C-11 | Q-13 | Sede |
| EP-006 | FT-022 | US-023 | UC-23 | — | Q-01 (perfil) | Experiencia |
| EP-006 | FT-023 | US-024 | UC-24 | — | Q-02 | Experiencia (requisitos) |
| EP-006 | FT-024 | US-025 | UC-25 | — | Q-14 | Experiencia |
| EP-007 | FT-025 | US-026 | UC-26 | C-03 | — | SolicitudCotizacion |
| EP-007 | FT-026 | US-027 | UC-27 | C-04 | Q-19 | SolicitudCotizacion, Cotizacion |
| EP-007 | FT-027 | US-028 | UC-28 | — | Q-15 | (Contenido corporativo) |
| EP-007 | FT-028 | US-029 | UC-29 | — | Q-16 | (Material B2B) |
| EP-008 | FT-029 | US-030 | UC-30 | — | (disponibilidad) | Disponibilidad |
| EP-008 | FT-030 | US-031 | UC-31 | C-05 | — | Reserva, Disponibilidad |
| EP-008 | FT-031 | US-032 | UC-32 | C-06, C-07 | — | Reserva, Pago |
| EP-008 | FT-032 | US-033 | UC-33 | — | — | Confirmacion, Reserva |
| EP-008 | FT-033 | US-034 | UC-34 | C-08 | Q-20 | Reserva, Disponibilidad |
| EP-009 | FT-034 | US-035 | UC-35 | — | Q-17 | Equipo |
| EP-009 | FT-035 | US-036 | UC-36 | C-09 | — | PedidoEquipo, LineaPedidoEquipo |
| EP-009 | FT-036 | US-037 | UC-37 | — | Q-17 (filtro) | Equipo |
| EP-009 | FT-038 | US-038 | UC-38 | C-10 | Q-21 | PedidoEquipo, CambioEstadoPedido |

Cobertura: 38/38 US · 9/9 EP · 37/37 FT · todos los Aggregates con operaciones asociadas.

---

# CHECKLIST DE VALIDACIÓN
- [x] Todas las User Stories tienen cobertura (38/38).
- [x] Todas las Features tienen cobertura (37/37).
- [x] Todas las Epics tienen cobertura (9/9).
- [x] Todos los Aggregates tienen operaciones asociadas.
- [x] Todos los Commands poseen validaciones.
- [x] Todas las Queries poseen resultado definido.
- [x] Todas las reglas de negocio están documentadas (BR-001..BR-020).
- [x] Todos los eventos están documentados con origen.
- [x] Existe matriz de autorización.
- [x] Existe catálogo de errores.
- [x] Existe trazabilidad completa (§2 y §13).

---

# NOTA DE TRAZABILIDAD
Documento derivado exclusivamente de los artefactos de entrada; no introduce
funcionalidades nuevas. Se mantiene la observación heredada: US-038 referencia
FT-038, mientras features.md numera esa funcionalidad como un segundo FT-037; se
recomienda renumerarla a FT-038 para conservar IDs únicos (aplicado así en §2 y §13).