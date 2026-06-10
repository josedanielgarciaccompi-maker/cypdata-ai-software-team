# API SPECIFICATION — Rafting Peru & Kayak Adventures
Arquitecto de APIs Senior · CypData · Junio 2026
Entradas: backend-specification.md · domain-model.md · domain-catalog.md · architecture-v1.md · technical-architecture-v1.md · ui-specification.md · user-stories.md · acceptance-criteria.md
Contrato API REST (sin código fuente). Base para OpenAPI/Swagger, contrato Frontend↔Backend y pruebas QA.

---

# 1. RESUMEN GENERAL

## Objetivo de las APIs
Exponer, mediante contratos REST/JSON estables, todas las capacidades backend de
la plataforma: contenido y catálogo, captación (consultas/cotizaciones), confianza,
disponibilidad, reservas y pagos, y catálogo/pedidos de equipos, además de las
operaciones de back-office. Cada endpoint deriva de un Command o Query de la
backend-specification, sin introducir funcionalidades nuevas.

## Alcance
Cubre los endpoints públicos de lectura (contenido, catálogo, confianza), los
endpoints públicos de escritura sin sesión (consulta, cotización, pedido de equipo),
el flujo de reserva y pago del cliente, el webhook de pago y los endpoints de
back-office autenticados. Quedan fuera: implementación, esquema físico de BD, SQL,
diseño visual y código.

## Arquitectura API
- Estilo: REST/JSON sobre HTTPS, versionado bajo el prefijo `/api/v1`.
- Organización por recurso de dominio (consultas, cotizaciones, experiencias,
  disponibilidad, reservas, pagos, equipos, pedidos-equipo, sedes, contenido).
- Punto de entrada vía API Gateway/BFF; servicios de dominio detrás (content,
  lead, quote, booking, payment, equipment, visibility), según technical-architecture.
- Operaciones de lectura → GET; creación → POST; cambios de estado → PATCH.
- Internacionalización por parámetro `lang` (ES/EN) en endpoints públicos de contenido.

## Convenciones Utilizadas
- URLs sustantivas en plural, sin verbos: `/activities`, `/reservations`, `/payments`.
- Identificadores de recurso en la ruta: `/reservations/{reservationId}`.
- Sub-recursos para acciones de dominio acotadas: `/reservations/{id}/payment`.
- Paginación uniforme: `page`, `pageSize`; orden: `sortBy`, `sortDirection`.
- Cuerpos y respuestas en JSON con envoltura estándar (§3).
- Recursos en inglés en la URL; los enums conservan los valores del domain-catalog.
- Mapa de recursos ↔ dominio:
  `/activities` = Experiencia · `/destinations` = Destino · `/inquiries` = Consulta (Lead) ·
  `/quote-requests` = SolicitudCotizacion · `/availability` = Disponibilidad ·
  `/reservations` = Reserva · `payment` = Pago · `/equipment` = Equipo ·
  `/equipment-orders` = PedidoEquipo · `/locations` = Sede · `/guides` = Guia ·
  `/reviews` = Resena · `/faqs` = Faq · `/articles` = Articulo.

---

# 2. SEGURIDAD

## Método de Autenticación
- **JWT Bearer Token** para back-office y para el cliente que gestiona su propia reserva.
  El token transporta el rol (claim `role`) y, para clientes, el identificador de su reserva/sesión.
- **Endpoints públicos**: sin autenticación (solo lectura de contenido publicado y alta de
  consulta/cotización/pedido), protegidos por límite de tasa y anti-bot.
- **Webhook de pago**: autenticación por **firma del proveedor** (cabecera de firma) verificada
  e idempotente; no usa JWT.
- Transporte: HTTPS/TLS obligatorio. Tokens con expiración; renovación vía endpoint de identidad
  (gestionado por identity-access, fuera de este catálogo de negocio).

## Roles del Sistema
| Rol | Descripción |
| --- | ----------- |
| PUBLIC | Visitante no autenticado; lectura de contenido publicado y alta de consulta/cotización/pedido de equipo. |
| CLIENT | Cliente que consulta disponibilidad, crea su reserva, paga y lee su confirmación. |
| RESPONSABLE_COMERCIAL | Back-office: gestiona consultas (leads) y cotizaciones. |
| GESTOR_OPERACIONES | Back-office: gestiona reservas, pedidos de equipos, sincroniza fichas y publica contenido. |
| SYSTEM | Identidad de máquina para el webhook de pago (firma válida). |

## Matriz de Acceso
| Endpoint | Rol | Permitido |
| -------- | --- | --------- |
| GET /destinations, /destinations/{id} | PUBLIC | Sí |
| GET /activities, /activities/{id} | PUBLIC | Sí |
| GET /activities/compare | PUBLIC | Sí |
| GET /activities/recommendations | PUBLIC | Sí |
| GET /trust-center, /guides, /reviews, /faqs | PUBLIC | Sí |
| GET /articles, /articles/{id} | PUBLIC | Sí |
| GET /media, /social-feed | PUBLIC | Sí |
| GET /seo/metadata | PUBLIC (buscadores/IA) | Sí |
| GET /locations/{id} | PUBLIC | Sí |
| GET /corporate, /corporate/material | PUBLIC | Sí |
| GET /equipment, /equipment/{id} | PUBLIC | Sí |
| GET /availability | PUBLIC / CLIENT | Sí |
| POST /inquiries | PUBLIC | Sí |
| POST /quote-requests | PUBLIC | Sí |
| POST /equipment-orders | PUBLIC | Sí |
| POST /reservations | CLIENT | Sí |
| POST /reservations/{id}/payment | CLIENT | Sí |
| GET /reservations/{id} (propia) | CLIENT | Sí |
| POST /payments/webhook | SYSTEM (firma) | Sí |
| GET /inquiries · PATCH /inquiries/{id} | RESPONSABLE_COMERCIAL | Sí |
| GET /quote-requests · POST /quote-requests/{id}/quotation | RESPONSABLE_COMERCIAL | Sí |
| GET /reservations · PATCH /reservations/{id} | GESTOR_OPERACIONES | Sí |
| GET /equipment-orders · PATCH /equipment-orders/{id} | GESTOR_OPERACIONES | Sí |
| PATCH /locations/{id} (sync ficha) | GESTOR_OPERACIONES | Sí |
| POST /content/{type}/{id}/publish · /unpublish | GESTOR_OPERACIONES | Sí |
| Cualquier endpoint back-office | PUBLIC / sin rol | No |

---

# 3. ESTÁNDARES DE RESPUESTA

## Respuesta Exitosa
```json
{
  "success": true,
  "data": {}
}
```
Para colecciones, `data` es un arreglo y se acompaña de `meta` de paginación:
```json
{
  "success": true,
  "data": [],
  "meta": { "page": 1, "pageSize": 20, "total": 134 }
}
```

## Respuesta de Error
```json
{
  "success": false,
  "errorCode": "ERR-010",
  "message": "Descripción legible del error",
  "details": [ { "field": "medioContacto", "error": "formato inválido" } ]
}
```

Convención de códigos HTTP (detalle en §7): 200 OK lectura, 201 Created alta,
200/204 cambios de estado, 400 validación sintáctica, 401 sin token, 403 rol
insuficiente, 404 recurso inexistente, 409 conflicto de estado/cupo, 422 regla de
dominio no satisfecha, 429 límite de tasa, 500 error interno.

---

# 4. CATÁLOGO DE ENDPOINTS

> Cada Command y Query de la backend-specification genera al menos un endpoint.
> Orden: lectura pública, escritura pública, reservas/pagos, back-office.

## Endpoint
### Nombre: Listar experiencias
### Caso de Uso Asociado: UC-07, UC-23 (Query Q-01)
### Método HTTP: GET
### URL: /api/v1/activities
### Descripción: Lista experiencias publicadas con filtros por destino, dificultad y perfil.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Request Headers
| Header | Requerido |
| ------ | --------- |
| Accept: application/json | Sí |
| Accept-Language | No |
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| — | — | — |
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| destino | string | No |
| dificultad | enum(NivelDificultad) | No |
| perfilCliente | enum(PerfilCliente) | No |
| lang | enum(Idioma) | No |
| page | int | No |
| pageSize | int | No |
| sortBy | string | No |
| sortDirection | enum(asc,desc) | No |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| — | — | — |
### Ejemplo Request
`GET /api/v1/activities?destino=DST-URU&dificultad=III&lang=ES&page=1&pageSize=20`
### Response 200
Lista paginada de experiencias publicadas (resumen).
### Ejemplo Response
```json
{ "success": true, "data": [
  { "id": "EXP-URU-01", "nombre": "Rafting Urubamba día completo", "destinoId": "DST-URU",
    "dificultad": "III", "duracion": "1 día", "precioReferencia": { "monto": 45, "moneda": "USD" },
    "disponibilidadIndicativa": "DISPONIBLE" } ],
  "meta": { "page": 1, "pageSize": 20, "total": 12 } }
```
### Validaciones: enums válidos; paginación dentro de límites.
### Reglas de Negocio: BR-016 (solo PUBLICADO).
### Errores Posibles: 400 (ERR-010).
### User Stories Relacionadas: US-007, US-023

## Endpoint
### Nombre: Obtener detalle de experiencia
### Caso de Uso Asociado: UC-08, UC-10, UC-24 (Query Q-02)
### Método HTTP: GET
### URL: /api/v1/activities/{activityId}
### Descripción: Devuelve detalle con componentes incluido/opcional, precio, condiciones y requisitos.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Request Headers
| Header | Requerido |
| ------ | --------- |
| Accept: application/json | Sí |
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| activityId | string | Sí |
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| lang | enum(Idioma) | No |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| — | — | — |
### Ejemplo Request
`GET /api/v1/activities/EXP-URU-01?lang=ES`
### Response 200
Detalle de la experiencia (ExperienciaDetalleResponse).
### Ejemplo Response
```json
{ "success": true, "data": {
  "id": "EXP-URU-01", "nombre": "Rafting Urubamba día completo", "destinoId": "DST-URU",
  "duracion": "1 día", "dificultad": "III", "precioReferencia": { "monto": 45, "moneda": "USD" },
  "baseCalculoPrecio": "POR_PERSONA", "condiciones": "Incluye traslado local",
  "componentes": [ { "nombre": "Almuerzo", "tipo": "ALIMENTACION", "condicion": "INCLUIDO" },
                   { "nombre": "Hospedaje", "tipo": "HOSPEDAJE", "condicion": "OPCIONAL" } ],
  "requisitos": { "edadMinima": 12, "nivelEsfuerzo": "MEDIO" } } }
```
### Validaciones: activityId existente.
### Reglas de Negocio: BR-016, BR-004 (componentes inequívocos), BR-005-precio (base de cálculo).
### Errores Posibles: 404 (ERR-011 / ERR-020).
### User Stories Relacionadas: US-008, US-010, US-024

## Endpoint
### Nombre: Comparar experiencias
### Caso de Uso Asociado: UC-09 (Query Q-03)
### Método HTTP: GET
### URL: /api/v1/activities/compare
### Descripción: Devuelve dos o más experiencias para comparación lado a lado.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| ids | string (CSV de ids) | Sí |
| lang | enum(Idioma) | No |
### Ejemplo Request
`GET /api/v1/activities/compare?ids=EXP-URU-01,EXP-LUN-02&lang=ES`
### Response 200
Matriz comparativa (componentes, duración, precio).
### Ejemplo Response
```json
{ "success": true, "data": { "items": [
  { "id": "EXP-URU-01", "duracion": "1 día", "precioReferencia": { "monto": 45, "moneda": "USD" } },
  { "id": "EXP-LUN-02", "duracion": "medio día", "precioReferencia": { "monto": 30, "moneda": "USD" } } ] } }
```
### Validaciones: 2..N ids; todos publicados.
### Reglas de Negocio: BR-016.
### Errores Posibles: 400 (ERR-010), 404 (ERR-011).
### User Stories Relacionadas: US-009

## Endpoint
### Nombre: Recomendar experiencias por perfil
### Caso de Uso Asociado: UC-25 (Query Q-14)
### Método HTTP: GET
### URL: /api/v1/activities/recommendations
### Descripción: Devuelve experiencias acordes a un perfil/preferencias.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| perfilCliente | enum(PerfilCliente) | Sí |
| lang | enum(Idioma) | No |
### Ejemplo Request
`GET /api/v1/activities/recommendations?perfilCliente=FAMILIA&lang=ES`
### Response 200
Lista priorizada de experiencias.
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "EXP-LUN-02", "nombre": "Rafting familiar Lunahuaná" } ] }
```
### Validaciones: perfilCliente válido.
### Reglas de Negocio: BR-016.
### Errores Posibles: 400 (ERR-010).
### User Stories Relacionadas: US-025

## Endpoint
### Nombre: Obtener página de destino
### Caso de Uso Asociado: UC-19 (Query Q-11)
### Método HTTP: GET
### URL: /api/v1/destinations/{destinationId}
### Descripción: Contenido del río + experiencias del destino + vía de contacto.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| destinationId | string (id o slug) | Sí |
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| lang | enum(Idioma) | No |
### Ejemplo Request
`GET /api/v1/destinations/maranon?lang=ES`
### Response 200
Destino + experiencias asociadas.
### Ejemplo Response
```json
{ "success": true, "data": { "id": "DST-MAR", "nombre": "Marañón", "experiencias": [] } }
```
### Validaciones: destino existente y publicado.
### Reglas de Negocio: BR-016.
### Errores Posibles: 404 (ERR-011).
### User Stories Relacionadas: US-019

## Endpoint
### Nombre: Listar destinos
### Caso de Uso Asociado: UC-19 (soporte índice)
### Método HTTP: GET
### URL: /api/v1/destinations
### Descripción: Índice de los destinos publicados.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| lang | enum(Idioma) | No |
### Response 200
Lista de destinos publicados.
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "DST-URU", "nombre": "Urubamba" } ] }
```
### Validaciones: —
### Reglas de Negocio: BR-016.
### Errores Posibles: —
### User Stories Relacionadas: US-019

## Endpoint
### Nombre: Obtener centro de confianza
### Caso de Uso Asociado: UC-11, UC-13 (Query Q-04)
### Método HTTP: GET
### URL: /api/v1/trust-center
### Descripción: Certificaciones/licencias (con entidad emisora) y protocolos de seguridad.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| lang | enum(Idioma) | No |
### Response 200
Credenciales + protocolos.
### Ejemplo Response
```json
{ "success": true, "data": { "credenciales": [ { "tipo": "Licencia", "entidadEmisora": "MINCETUR" } ],
  "protocolos": [ "Briefing de seguridad", "Equipo de rescate certificado" ] } }
```
### Validaciones: —
### Reglas de Negocio: BR-016 (RN-16 emisor).
### Errores Posibles: —
### User Stories Relacionadas: US-011, US-013

## Endpoint
### Nombre: Listar guías
### Caso de Uso Asociado: UC-12 (Query Q-05)
### Método HTTP: GET
### URL: /api/v1/guides
### Descripción: Perfiles de guías con credenciales.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| lang | enum(Idioma) | No |
### Response 200
Lista de guías con credenciales.
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "GUI-01", "nombre": "Carlos R.", "credenciales": [ "Rescate en aguas rápidas" ] } ] }
```
### Validaciones: —
### Reglas de Negocio: RN-16.
### Errores Posibles: —
### User Stories Relacionadas: US-012

## Endpoint
### Nombre: Listar reseñas
### Caso de Uso Asociado: UC-14 (Query Q-06)
### Método HTTP: GET
### URL: /api/v1/reviews
### Descripción: Reseñas, opcionalmente por destino/experiencia.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| objetivoRef | string | No |
| lang | enum(Idioma) | No |
| page | int | No |
| pageSize | int | No |
### Response 200
Lista de reseñas (puede ser vacía sin error).
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "REV-01", "autor": "Ana", "valoracion": 5, "texto": "Excelente" } ],
  "meta": { "page": 1, "pageSize": 20, "total": 1 } }
```
### Validaciones: —
### Reglas de Negocio: RN-17 (ausencia no es error).
### Errores Posibles: —
### User Stories Relacionadas: US-014

## Endpoint
### Nombre: Buscar FAQ
### Caso de Uso Asociado: UC-15 (Query Q-07)
### Método HTTP: GET
### URL: /api/v1/faqs
### Descripción: Busca FAQ por término/categoría.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| termino | string | No |
| categoria | string | No |
| lang | enum(Idioma) | No |
### Response 200
Lista de FAQ relevantes.
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "FAQ-03", "pregunta": "¿Es seguro para niños?", "respuesta": "Sí, desde 8 años con guía." } ] }
```
### Validaciones: —
### Reglas de Negocio: —
### Errores Posibles: —
### User Stories Relacionadas: US-015

## Endpoint
### Nombre: Obtener galería multimedia
### Caso de Uso Asociado: UC-16 (Query Q-08)
### Método HTTP: GET
### URL: /api/v1/media
### Descripción: Activos multimedia por destino/experiencia.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| destinoId | string | No |
| experienciaId | string | No |
| lang | enum(Idioma) | No |
### Response 200
Lista de activos multimedia (con referencia de almacenamiento).
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "MED-10", "tipo": "VIDEO", "referenciaAlmacenamiento": "s3://.../maranon.mp4" } ] }
```
### Validaciones: al menos un filtro (destinoId o experienciaId) recomendado.
### Reglas de Negocio: BR-016.
### Errores Posibles: —
### User Stories Relacionadas: US-016

## Endpoint
### Nombre: Obtener contenido social
### Caso de Uso Asociado: UC-17 (Query Q-09)
### Método HTTP: GET
### URL: /api/v1/social-feed
### Descripción: Publicaciones sociales embebibles; tolerante a indisponibilidad.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Response 200
Bloque de contenido social (puede venir vacío → degradación elegante).
### Ejemplo Response
```json
{ "success": true, "data": { "posts": [], "available": false } }
```
### Validaciones: —
### Reglas de Negocio: —
### Errores Posibles: — (nunca rompe la página; 200 con available=false)
### User Stories Relacionadas: US-017

## Endpoint
### Nombre: Listar/leer artículos
### Caso de Uso Asociado: UC-18 (Query Q-10)
### Método HTTP: GET
### URL: /api/v1/articles  ·  /api/v1/articles/{articleId}
### Descripción: Lista o detalle de artículos publicados.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| relacionadoRef | string | No |
| lang | enum(Idioma) | No |
| page | int | No |
| pageSize | int | No |
### Response 200
Lista/detalle de artículos (PUBLICADO).
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "ART-05", "titulo": "Guía del Marañón", "idioma": "ES" } ] }
```
### Validaciones: artículo existente (detalle).
### Reglas de Negocio: BR-016.
### Errores Posibles: 404 (ERR-011).
### User Stories Relacionadas: US-018

## Endpoint
### Nombre: Obtener metadatos estructurados
### Caso de Uso Asociado: UC-21 (Query Q-12)
### Método HTTP: GET
### URL: /api/v1/seo/metadata
### Descripción: Metadatos/schema y sitemaps de un recurso publicado.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC (buscadores/IA)
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| recursoRef | string | Sí |
| lang | enum(Idioma) | No |
### Response 200
Metadatos estructurados.
### Ejemplo Response
```json
{ "success": true, "data": { "@type": "TouristTrip", "name": "Rafting Urubamba" } }
```
### Validaciones: recurso existente y publicado.
### Reglas de Negocio: BR-016.
### Errores Posibles: 404 (ERR-011).
### User Stories Relacionadas: US-021

## Endpoint
### Nombre: Obtener ficha local de sede
### Caso de Uso Asociado: UC-22 lectura (Query Q-13)
### Método HTTP: GET
### URL: /api/v1/locations/{locationId}
### Descripción: Información de contacto, ubicación y reseñas de la sede.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| locationId | string | Sí |
### Response 200
Ficha local de la sede.
### Ejemplo Response
```json
{ "success": true, "data": { "id": "SED-CUS", "nombre": "Cusco", "contacto": "+51...", "reseñas": [] } }
```
### Validaciones: sede existente.
### Reglas de Negocio: BR-017.
### Errores Posibles: 404 (ERR-011).
### User Stories Relacionadas: US-022

## Endpoint
### Nombre: Obtener propuesta corporativa
### Caso de Uso Asociado: UC-28 (Query Q-15)
### Método HTTP: GET
### URL: /api/v1/corporate
### Descripción: Propuesta de valor B2B y vía de contacto.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| lang | enum(Idioma) | No |
### Response 200
Contenido corporativo.
### Ejemplo Response
```json
{ "success": true, "data": { "propuesta": "Team building en los 4 ríos", "contacto": "/quote-requests" } }
```
### Validaciones: —
### Reglas de Negocio: —
### Errores Posibles: —
### User Stories Relacionadas: US-028

## Endpoint
### Nombre: Obtener material B2B
### Caso de Uso Asociado: UC-29 (Query Q-16)
### Método HTTP: GET
### URL: /api/v1/corporate/material
### Descripción: Material descargable vigente para agencias.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| lang | enum(Idioma) | No |
### Response 200
Referencia al documento vigente.
### Ejemplo Response
```json
{ "success": true, "data": { "version": "2026-06", "url": "https://.../oferta-b2b.pdf" } }
```
### Validaciones: —
### Reglas de Negocio: versión vigente (US-029 AC-002).
### Errores Posibles: 404 (ERR-011).
### User Stories Relacionadas: US-029

## Endpoint
### Nombre: Listar catálogo de equipos
### Caso de Uso Asociado: UC-35, UC-37 (Query Q-17)
### Método HTTP: GET
### URL: /api/v1/equipment  ·  /api/v1/equipment/{equipmentId}
### Descripción: Catálogo de equipos, con filtro por tipo de cliente; detalle con condición de venta.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| tipoCliente | enum(PerfilCliente) | No |
| lang | enum(Idioma) | No |
| page | int | No |
| pageSize | int | No |
### Response 200
Lista de equipos; si no hay oferta específica, oferta general (BR-018).
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "EQ-KAYAK-01", "nombre": "Kayak travesía", "condicionVenta": "IMPORTACION" } ] }
```
### Validaciones: tipoCliente válido si se envía.
### Reglas de Negocio: BR-016, BR-018.
### Errores Posibles: 400 (ERR-010), 404 (ERR-025 detalle).
### User Stories Relacionadas: US-035, US-037

## Endpoint
### Nombre: Consultar disponibilidad
### Caso de Uso Asociado: UC-30 (consulta de disponibilidad)
### Método HTTP: GET
### URL: /api/v1/availability
### Descripción: Fechas y cupos disponibles de una experiencia.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC, CLIENT
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| activityId | string | Sí |
| from | date | No |
| to | date | No |
### Ejemplo Request
`GET /api/v1/availability?activityId=EXP-URU-01&from=2026-07-01&to=2026-07-31`
### Response 200
Fechas con cupo y estado (DISPONIBLE/PARCIAL/AGOTADO).
### Ejemplo Response
```json
{ "success": true, "data": { "experienciaId": "EXP-URU-01",
  "fechas": [ { "fecha": "2026-07-20", "cupoDisponible": 6, "estado": "DISPONIBLE" } ] } }
```
### Validaciones: experiencia publicada.
### Reglas de Negocio: BR-006.
### Errores Posibles: 404 (ERR-020).
### User Stories Relacionadas: US-030

## Endpoint
### Nombre: Crear consulta
### Caso de Uso Asociado: UC-03, UC-06 (Command C-01)
### Método HTTP: POST
### URL: /api/v1/inquiries
### Descripción: Registra una consulta de un visitante por formulario o canal conversacional.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Request Headers
| Header | Requerido |
| ------ | --------- |
| Content-Type: application/json | Sí |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| nombreContacto | string | Sí |
| medioContacto | string | Sí |
| destinoInteresId | string | No |
| mensaje | string | Sí |
| canal | enum(Canal) | Sí |
| idioma | enum(Idioma) | Sí |
### Ejemplo Request
```json
{ "nombreContacto": "Ana Pérez", "medioContacto": "ana@example.com",
  "destinoInteresId": "DST-URU", "mensaje": "Quiero info de rafting", "canal": "FORMULARIO", "idioma": "ES" }
```
### Response 201
Consulta creada (ConsultaResponse).
### Ejemplo Response
```json
{ "success": true, "data": { "id": "LEAD-5001", "estado": "NUEVA", "canal": "FORMULARIO",
  "idioma": "ES", "fecha": "2026-06-10T14:02:00Z", "recepcionConfirmada": true } }
```
### Validaciones: requeridos presentes; medioContacto email/teléfono; destino existente si se envía.
### Reglas de Negocio: BR-001, BR-010, BR-019.
### Errores Posibles: 400 (ERR-010), 404 (ERR-011), 429 (ERR-032).
### User Stories Relacionadas: US-003, US-006

## Endpoint
### Nombre: Crear solicitud de cotización
### Caso de Uso Asociado: UC-26 (Command C-03)
### Método HTTP: POST
### URL: /api/v1/quote-requests
### Descripción: Registra una solicitud de cotización de grupo/B2B.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| organizacion | string | Sí |
| numeroPersonas | int | Sí |
| destinoId | string | Sí |
| fechaSolicitada | date | Sí |
| contacto | object {nombre, email?, telefono?} | Sí |
### Ejemplo Request
```json
{ "organizacion": "Colegio X", "numeroPersonas": 25, "destinoId": "DST-LUN",
  "fechaSolicitada": "2026-08-15", "contacto": { "nombre": "Luis", "telefono": "+51..." } }
```
### Response 201
Solicitud creada en estado RECIBIDA.
### Ejemplo Response
```json
{ "success": true, "data": { "id": "QR-2001", "estado": "RECIBIDA", "recepcionConfirmada": true } }
```
### Validaciones: requeridos; numeroPersonas ≥ 1; destino existente; fecha ≥ hoy.
### Reglas de Negocio: BR-010, BR-011.
### Errores Posibles: 400/422 (ERR-013), 404 (ERR-011), 429 (ERR-032).
### User Stories Relacionadas: US-026

## Endpoint
### Nombre: Crear pedido/solicitud de equipo
### Caso de Uso Asociado: UC-36 (Command C-09)
### Método HTTP: POST
### URL: /api/v1/equipment-orders
### Descripción: Registra un pedido/solicitud de cotización de equipos bajo importación.
### Autenticación Requerida: No
### Roles Permitidos: PUBLIC
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| lineas | array[{equipoId: string, cantidad: int}] | Sí |
| tipoCliente | enum(PerfilCliente) | Sí |
| contacto | object {nombre, email?, telefono?} | Sí |
### Ejemplo Request
```json
{ "lineas": [ { "equipoId": "EQ-KAYAK-01", "cantidad": 2 } ],
  "tipoCliente": "CLUB", "contacto": { "nombre": "Club Andino", "email": "club@x.com" } }
```
### Response 201
Pedido creado en estado RECIBIDO; aviso de importación.
### Ejemplo Response
```json
{ "success": true, "data": { "id": "PED-3001", "estadoImportacion": "RECIBIDO", "sujetoImportacion": true } }
```
### Validaciones: ≥ 1 línea; cantidades ≥ 1; equipos publicados.
### Reglas de Negocio: BR-013, BR-014.
### Errores Posibles: 400 (ERR-010), 404 (ERR-025), 429 (ERR-032).
### User Stories Relacionadas: US-036

## Endpoint
### Nombre: Crear reserva
### Caso de Uso Asociado: UC-31 (Command C-05)
### Método HTTP: POST
### URL: /api/v1/reservations
### Descripción: Crea una reserva validando el cupo de la disponibilidad.
### Autenticación Requerida: Sí
### Roles Permitidos: CLIENT
### Request Headers
| Header | Requerido |
| ------ | --------- |
| Authorization: Bearer <token> | Sí |
| Content-Type: application/json | Sí |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| experienciaId | string | Sí |
| disponibilidadId | string | Sí |
| fecha | date | Sí |
| numeroParticipantes | int | Sí |
| cliente | object {nombre, email?, telefono?} | Sí |
### Ejemplo Request
```json
{ "experienciaId": "EXP-URU-01", "disponibilidadId": "DISP-0912",
  "fecha": "2026-07-20", "numeroParticipantes": 3, "cliente": { "nombre": "John", "email": "john@x.com" } }
```
### Response 201
Reserva en estado INICIADA; cupo reservado provisionalmente.
### Ejemplo Response
```json
{ "success": true, "data": { "id": "RES-1001", "estado": "INICIADA", "experienciaId": "EXP-URU-01",
  "fecha": "2026-07-20", "numeroParticipantes": 3 } }
```
### Validaciones: experiencia publicada; disponibilidad existente; participantes ≥ 1 y ≤ cupo.
### Reglas de Negocio: BR-003, BR-004.
### Errores Posibles: 409 (ERR-021 cupo, ERR-022 fecha), 404 (ERR-020), 400 (ERR-010).
### User Stories Relacionadas: US-031

## Endpoint
### Nombre: Iniciar pago de reserva
### Caso de Uso Asociado: UC-32 (Command C-06)
### Método HTTP: POST
### URL: /api/v1/reservations/{reservationId}/payment
### Descripción: Inicia el pago de una reserva pendiente; devuelve referencia/redirección de pasarela.
### Autenticación Requerida: Sí
### Roles Permitidos: CLIENT
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| reservationId | string | Sí |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| monto | object {monto: number, moneda: enum(Moneda)} | Sí |
### Ejemplo Request
```json
{ "monto": { "monto": 135.00, "moneda": "USD" } }
```
### Response 201
Pago en estado INICIADO; referencia/redirección de pasarela.
### Ejemplo Response
```json
{ "success": true, "data": { "pagoId": "PAY-9001", "estado": "INICIADO",
  "referenciaPasarela": "tok_abc", "redirectUrl": "https://pasarela/..." } }
```
### Validaciones: reserva en estado INICIADA; monto coincide con la reserva; moneda válida.
### Reglas de Negocio: BR-005.
### Errores Posibles: 409 (ERR-026), 422 (ERR-024), 404 (ERR-011).
### User Stories Relacionadas: US-032

## Endpoint
### Nombre: Webhook de resultado de pago
### Caso de Uso Asociado: UC-32 (Command C-07)
### Método HTTP: POST
### URL: /api/v1/payments/webhook
### Descripción: Recibe el resultado del pago desde la pasarela; idempotente. Confirma o no la reserva.
### Autenticación Requerida: Sí (firma del proveedor)
### Roles Permitidos: SYSTEM
### Request Headers
| Header | Requerido |
| ------ | --------- |
| X-Signature (firma del proveedor) | Sí |
| Content-Type: application/json | Sí |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| reservaId | string | Sí |
| referenciaPasarela | string | Sí |
| resultado | enum(EstadoPago) | Sí |
### Ejemplo Request
```json
{ "reservaId": "RES-1001", "referenciaPasarela": "tok_abc", "resultado": "EXITOSO" }
```
### Response 200
Reconocimiento del evento (idempotente).
### Ejemplo Response
```json
{ "success": true, "data": { "reservaId": "RES-1001", "estado": "CONFIRMADA" } }
```
### Validaciones: firma/origen válidos; idempotencia por referencia; reserva existente.
### Reglas de Negocio: BR-005, BR-007, BR-020.
### Errores Posibles: 401 (ERR-031 firma), 409 (duplicado tratado idempotente), 404 (ERR-011).
### User Stories Relacionadas: US-032

## Endpoint
### Nombre: Obtener reserva (propia)
### Caso de Uso Asociado: UC-33 lectura de confirmación
### Método HTTP: GET
### URL: /api/v1/reservations/{reservationId}
### Descripción: Devuelve el detalle y la confirmación de la reserva del propio cliente.
### Autenticación Requerida: Sí
### Roles Permitidos: CLIENT (propia), GESTOR_OPERACIONES
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| reservationId | string | Sí |
### Response 200
Detalle de reserva + confirmación (si CONFIRMADA).
### Ejemplo Response
```json
{ "success": true, "data": { "id": "RES-1001", "estado": "CONFIRMADA",
  "experienciaId": "EXP-URU-01", "fecha": "2026-07-20", "numeroParticipantes": 3,
  "pago": { "estado": "EXITOSO" }, "confirmacionId": "CONF-7001" } }
```
### Validaciones: reserva existente; pertenencia al cliente o rol operativo.
### Reglas de Negocio: BR-008, BR-009.
### Errores Posibles: 403 (ERR-030), 404 (ERR-011).
### User Stories Relacionadas: US-033

## Endpoint
### Nombre: Listar consultas (back-office)
### Caso de Uso Asociado: UC-04 (Query Q-18)
### Método HTTP: GET
### URL: /api/v1/inquiries
### Descripción: Lista leads centralizados con filtros.
### Autenticación Requerida: Sí
### Roles Permitidos: RESPONSABLE_COMERCIAL
### Request Headers
| Header | Requerido |
| ------ | --------- |
| Authorization: Bearer <token> | Sí |
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| estado | enum(EstadoConsulta) | No |
| canal | enum(Canal) | No |
| from | date | No |
| to | date | No |
| page | int | No |
| pageSize | int | No |
| sortBy | string | No |
| sortDirection | enum(asc,desc) | No |
### Response 200
Lista paginada de consultas con datos asociados.
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "LEAD-5001", "estado": "NUEVA", "canal": "FORMULARIO",
  "destinoInteresId": "DST-URU", "fecha": "2026-06-10T14:02:00Z" } ],
  "meta": { "page": 1, "pageSize": 20, "total": 42 } }
```
### Validaciones: rol autorizado.
### Reglas de Negocio: BR-015.
### Errores Posibles: 401 (sin token), 403 (ERR-030).
### User Stories Relacionadas: US-004

## Endpoint
### Nombre: Actualizar estado de consulta
### Caso de Uso Asociado: UC-05 (Command C-02)
### Método HTTP: PATCH
### URL: /api/v1/inquiries/{inquiryId}
### Descripción: Cambia el estado de una consulta e historiza la transición.
### Autenticación Requerida: Sí
### Roles Permitidos: RESPONSABLE_COMERCIAL
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| inquiryId | string | Sí |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| nuevoEstado | enum(EstadoConsulta) | Sí |
### Ejemplo Request
```json
{ "nuevoEstado": "RESPONDIDA" }
```
### Response 200
Estado actualizado e historizado.
### Ejemplo Response
```json
{ "success": true, "data": { "id": "LEAD-5001", "estado": "RESPONDIDA" } }
```
### Validaciones: consulta existente; transición válida; rol autorizado.
### Reglas de Negocio: BR-002, BR-015.
### Errores Posibles: 404 (ERR-011), 409/422 (ERR-012), 403 (ERR-030).
### User Stories Relacionadas: US-005

## Endpoint
### Nombre: Listar solicitudes de cotización (back-office)
### Caso de Uso Asociado: UC-27 lectura (Query Q-19)
### Método HTTP: GET
### URL: /api/v1/quote-requests
### Descripción: Lista solicitudes de cotización con su estado.
### Autenticación Requerida: Sí
### Roles Permitidos: RESPONSABLE_COMERCIAL
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| estado | enum(EstadoCotizacion) | No |
| destino | string | No |
| from | date | No |
| to | date | No |
| page | int | No |
| pageSize | int | No |
### Response 200
Lista paginada de solicitudes.
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "QR-2001", "estado": "RECIBIDA", "organizacion": "Colegio X" } ],
  "meta": { "page": 1, "pageSize": 20, "total": 8 } }
```
### Validaciones: rol autorizado.
### Reglas de Negocio: BR-015.
### Errores Posibles: 401, 403 (ERR-030).
### User Stories Relacionadas: US-027

## Endpoint
### Nombre: Emitir cotización
### Caso de Uso Asociado: UC-27 (Command C-04)
### Método HTTP: POST
### URL: /api/v1/quote-requests/{quoteRequestId}/quotation
### Descripción: Asocia y emite una cotización para una solicitud; cambia su estado.
### Autenticación Requerida: Sí
### Roles Permitidos: RESPONSABLE_COMERCIAL
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| quoteRequestId | string | Sí |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| contenido | string | Sí |
### Ejemplo Request
```json
{ "contenido": "Propuesta para 25 personas en Lunahuaná, incluye traslados." }
```
### Response 201
Cotización asociada; estado COTIZACION_EMITIDA.
### Ejemplo Response
```json
{ "success": true, "data": { "solicitudId": "QR-2001", "cotizacionId": "QT-4001", "estado": "COTIZACION_EMITIDA" } }
```
### Validaciones: solicitud existente; contenido no vacío; rol autorizado.
### Reglas de Negocio: BR-012, BR-002, BR-015.
### Errores Posibles: 404 (ERR-014), 403 (ERR-030), 400 (ERR-010).
### User Stories Relacionadas: US-027

## Endpoint
### Nombre: Listar reservas (back-office)
### Caso de Uso Asociado: UC-34 lectura (Query Q-20)
### Método HTTP: GET
### URL: /api/v1/reservations
### Descripción: Lista reservas con su estado.
### Autenticación Requerida: Sí
### Roles Permitidos: GESTOR_OPERACIONES
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| estado | enum(EstadoReserva) | No |
| experienciaId | string | No |
| from | date | No |
| to | date | No |
| page | int | No |
| pageSize | int | No |
### Response 200
Lista paginada de reservas.
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "RES-1001", "estado": "CONFIRMADA", "fecha": "2026-07-20" } ],
  "meta": { "page": 1, "pageSize": 20, "total": 15 } }
```
### Validaciones: rol autorizado.
### Reglas de Negocio: BR-015.
### Errores Posibles: 401, 403 (ERR-030).
### User Stories Relacionadas: US-034

## Endpoint
### Nombre: Actualizar estado de reserva (back-office)
### Caso de Uso Asociado: UC-34 (Command C-08)
### Método HTTP: PATCH
### URL: /api/v1/reservations/{reservationId}
### Descripción: Cambia el estado operativo de una reserva; ajusta cupo si CANCELADA.
### Autenticación Requerida: Sí
### Roles Permitidos: GESTOR_OPERACIONES
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| reservationId | string | Sí |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| nuevoEstado | enum(EstadoReserva) | Sí |
### Ejemplo Request
```json
{ "nuevoEstado": "CANCELADA" }
```
### Response 200
Estado actualizado e historizado; cupo ajustado si corresponde.
### Ejemplo Response
```json
{ "success": true, "data": { "id": "RES-1001", "estado": "CANCELADA" } }
```
### Validaciones: reserva existente; transición válida; rol autorizado.
### Reglas de Negocio: BR-002, BR-009, BR-015.
### Errores Posibles: 404 (ERR-011), 409/422 (ERR-012), 403 (ERR-030).
### User Stories Relacionadas: US-034

## Endpoint
### Nombre: Listar pedidos de equipos (back-office)
### Caso de Uso Asociado: UC-38 lectura (Query Q-21)
### Método HTTP: GET
### URL: /api/v1/equipment-orders
### Descripción: Lista pedidos de equipos con su estado de importación.
### Autenticación Requerida: Sí
### Roles Permitidos: GESTOR_OPERACIONES
### Query Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| estadoImportacion | enum(EstadoPedidoEquipo) | No |
| from | date | No |
| to | date | No |
| page | int | No |
| pageSize | int | No |
### Response 200
Lista paginada de pedidos.
### Ejemplo Response
```json
{ "success": true, "data": [ { "id": "PED-3001", "estadoImportacion": "RECIBIDO" } ],
  "meta": { "page": 1, "pageSize": 20, "total": 5 } }
```
### Validaciones: rol autorizado.
### Reglas de Negocio: BR-015.
### Errores Posibles: 401, 403 (ERR-030).
### User Stories Relacionadas: US-038

## Endpoint
### Nombre: Actualizar estado de pedido de equipo (back-office)
### Caso de Uso Asociado: UC-38 (Command C-10)
### Método HTTP: PATCH
### URL: /api/v1/equipment-orders/{orderId}
### Descripción: Actualiza el estado de importación de un pedido e historiza.
### Autenticación Requerida: Sí
### Roles Permitidos: GESTOR_OPERACIONES
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| orderId | string | Sí |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| nuevoEstado | enum(EstadoPedidoEquipo) | Sí |
### Ejemplo Request
```json
{ "nuevoEstado": "EN_IMPORTACION" }
```
### Response 200
Estado de importación actualizado e historizado.
### Ejemplo Response
```json
{ "success": true, "data": { "id": "PED-3001", "estadoImportacion": "EN_IMPORTACION" } }
```
### Validaciones: pedido existente; transición válida; rol autorizado.
### Reglas de Negocio: BR-002, BR-015.
### Errores Posibles: 404 (ERR-011), 409/422 (ERR-012), 403 (ERR-030).
### User Stories Relacionadas: US-038

## Endpoint
### Nombre: Sincronizar ficha local (back-office)
### Caso de Uso Asociado: UC-22 escritura (Command C-11)
### Método HTTP: PATCH
### URL: /api/v1/locations/{locationId}
### Descripción: Actualiza/propaga la información de la ficha local de una sede.
### Autenticación Requerida: Sí
### Roles Permitidos: GESTOR_OPERACIONES
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| locationId | string | Sí |
### Request Body
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| datosFicha | object | Sí |
### Ejemplo Request
```json
{ "datosFicha": { "contacto": "+51 999 888 777", "horario": "L-D 8:00-18:00" } }
```
### Response 200
Ficha local consistente con la información vigente.
### Ejemplo Response
```json
{ "success": true, "data": { "id": "SED-CUS", "actualizada": true } }
```
### Validaciones: sede existente; rol autorizado.
### Reglas de Negocio: BR-017, BR-015.
### Errores Posibles: 404 (ERR-011), 403 (ERR-030).
### User Stories Relacionadas: US-022

## Endpoint
### Nombre: Publicar / despublicar contenido (back-office)
### Caso de Uso Asociado: soporte BR-016 (Commands C-12 / C-13)
### Método HTTP: POST
### URL: /api/v1/content/{type}/{id}/publish  ·  /api/v1/content/{type}/{id}/unpublish
### Descripción: Cambia el estado de publicación de un recurso de contenido.
### Autenticación Requerida: Sí
### Roles Permitidos: GESTOR_OPERACIONES
### Path Parameters
| Campo | Tipo | Requerido |
| ----- | ---- | --------- |
| type | enum(destino,experiencia,articulo,equipo) | Sí |
| id | string | Sí |
### Response 200
Estado de publicación actualizado.
### Ejemplo Response
```json
{ "success": true, "data": { "type": "experiencia", "id": "EXP-URU-01", "estadoPublicacion": "PUBLICADO" } }
```
### Validaciones: recurso existente; rol autorizado.
### Reglas de Negocio: BR-016, BR-015.
### Errores Posibles: 404 (ERR-011), 403 (ERR-030).
### User Stories Relacionadas: US-007, US-021 (habilitador de contenido publicado)

---

# 5. FILTROS Y BÚSQUEDAS

Formato estándar transversal: `page` (≥1), `pageSize` (1..100), `sortBy` (campo),
`sortDirection` (`asc`|`desc`). El idioma de contenido se pasa por `lang` (ES/EN).

## /activities
- Filtros: `destino`, `dificultad`, `perfilCliente`, `lang`.
- Ordenamiento: `sortBy` ∈ {destino, dificultad, precio, nombre}.
- Paginación: `page`, `pageSize`.

## /reviews
- Filtros: `objetivoRef`, `lang`.
- Ordenamiento: `sortBy` ∈ {fecha, valoracion}; default fecha desc.
- Paginación: `page`, `pageSize`.

## /articles
- Filtros: `relacionadoRef`, `lang`.
- Ordenamiento: `sortBy` ∈ {fecha, titulo}.
- Paginación: `page`, `pageSize`.

## /equipment
- Filtros: `tipoCliente`, `lang`.
- Ordenamiento: `sortBy` ∈ {nombre, tipo}.
- Paginación: `page`, `pageSize`.

## /availability
- Filtros: `activityId` (req.), `from`, `to`.
- Ordenamiento: por fecha asc (implícito).
- Paginación: no aplica (rango acotado por fechas).

## /inquiries (back-office)
- Filtros: `estado`, `canal`, `from`, `to`.
- Ordenamiento: `sortBy` ∈ {fecha, estado}; default fecha desc.
- Paginación: `page`, `pageSize`.

## /quote-requests (back-office)
- Filtros: `estado`, `destino`, `from`, `to`.
- Ordenamiento: `sortBy` ∈ {fecha, estado}.
- Paginación: `page`, `pageSize`.

## /reservations (back-office)
- Filtros: `estado`, `experienciaId`, `from`, `to`.
- Ordenamiento: `sortBy` ∈ {fecha, estado}.
- Paginación: `page`, `pageSize`.

## /equipment-orders (back-office)
- Filtros: `estadoImportacion`, `from`, `to`.
- Ordenamiento: `sortBy` ∈ {fecha, estadoImportacion}.
- Paginación: `page`, `pageSize`.

---

# 6. DTO CATALOG

## Request DTOs

### Nombre: CrearConsultaRequest
### Campos: nombreContacto: string; medioContacto: string; destinoInteresId?: string; mensaje: string; canal: enum(Canal); idioma: enum(Idioma)
### Restricciones: nombreContacto y mensaje no vacíos; medioContacto email/teléfono; enums válidos.
### Ejemplo:
```json
{ "nombreContacto": "Ana Pérez", "medioContacto": "ana@example.com", "destinoInteresId": "DST-URU",
  "mensaje": "Quiero info de rafting", "canal": "FORMULARIO", "idioma": "ES" }
```

### Nombre: CrearSolicitudCotizacionRequest
### Campos: organizacion: string; numeroPersonas: int; destinoId: string; fechaSolicitada: date; contacto: {nombre, email?, telefono?}
### Restricciones: numeroPersonas ≥ 1; destinoId existente; fechaSolicitada ≥ hoy.
### Ejemplo:
```json
{ "organizacion": "Colegio X", "numeroPersonas": 25, "destinoId": "DST-LUN",
  "fechaSolicitada": "2026-08-15", "contacto": { "nombre": "Luis", "telefono": "+51..." } }
```

### Nombre: CrearReservaRequest
### Campos: experienciaId: string; disponibilidadId: string; fecha: date; numeroParticipantes: int; cliente: {nombre, email?, telefono?}
### Restricciones: numeroParticipantes ≥ 1; fecha con cupo; experiencia publicada.
### Ejemplo:
```json
{ "experienciaId": "EXP-URU-01", "disponibilidadId": "DISP-0912", "fecha": "2026-07-20",
  "numeroParticipantes": 3, "cliente": { "nombre": "John", "email": "john@x.com" } }
```

### Nombre: IniciarPagoRequest
### Campos: monto: {monto: number, moneda: enum(Moneda)}
### Restricciones: reserva (de la ruta) en estado INICIADA; moneda ∈ {PEN, USD}; monto coincide con la reserva.
### Ejemplo:
```json
{ "monto": { "monto": 135.00, "moneda": "USD" } }
```

### Nombre: PagoWebhookRequest
### Campos: reservaId: string; referenciaPasarela: string; resultado: enum(EstadoPago)
### Restricciones: firma válida (cabecera); idempotente por referenciaPasarela.
### Ejemplo:
```json
{ "reservaId": "RES-1001", "referenciaPasarela": "tok_abc", "resultado": "EXITOSO" }
```

### Nombre: CrearPedidoEquipoRequest
### Campos: lineas: [{equipoId: string, cantidad: int}]; tipoCliente: enum(PerfilCliente); contacto: {nombre, email?, telefono?}
### Restricciones: ≥ 1 línea; cantidad ≥ 1; equipos publicados.
### Ejemplo:
```json
{ "lineas": [ { "equipoId": "EQ-KAYAK-01", "cantidad": 2 } ], "tipoCliente": "CLUB",
  "contacto": { "nombre": "Club Andino", "email": "club@x.com" } }
```

### Nombre: EmitirCotizacionRequest
### Campos: contenido: string
### Restricciones: contenido no vacío; solicitud (de la ruta) existente.
### Ejemplo:
```json
{ "contenido": "Propuesta para 25 personas en Lunahuaná." }
```

### Nombre: CambiarEstadoRequest (consulta/reserva/pedido)
### Campos: nuevoEstado: enum (según recurso)
### Restricciones: transición válida en la máquina de estados de la entidad.
### Ejemplo:
```json
{ "nuevoEstado": "CANCELADA" }
```

### Nombre: SincronizarFichaLocalRequest
### Campos: datosFicha: object
### Restricciones: sede existente; rol autorizado.
### Ejemplo:
```json
{ "datosFicha": { "contacto": "+51 999 888 777", "horario": "L-D 8:00-18:00" } }
```

## Response DTOs

### Nombre: ConsultaResponse
### Campos: id; estado; canal; idioma; destinoInteresId?; fecha; recepcionConfirmada
### Ejemplo:
```json
{ "id": "LEAD-5001", "estado": "NUEVA", "canal": "FORMULARIO", "idioma": "ES",
  "destinoInteresId": "DST-URU", "fecha": "2026-06-10T14:02:00Z", "recepcionConfirmada": true }
```

### Nombre: ExperienciaResumenResponse
### Campos: id; nombre; destinoId; dificultad; duracion; precioReferencia {monto,moneda}; disponibilidadIndicativa
### Ejemplo:
```json
{ "id": "EXP-URU-01", "nombre": "Rafting Urubamba día completo", "destinoId": "DST-URU",
  "dificultad": "III", "duracion": "1 día", "precioReferencia": { "monto": 45, "moneda": "USD" },
  "disponibilidadIndicativa": "DISPONIBLE" }
```

### Nombre: ExperienciaDetalleResponse
### Campos: id; nombre; destinoId; duracion; dificultad; precioReferencia; baseCalculoPrecio; condiciones; componentes[]; requisitos
### Ejemplo:
```json
{ "id": "EXP-URU-01", "nombre": "Rafting Urubamba día completo", "destinoId": "DST-URU",
  "duracion": "1 día", "dificultad": "III", "precioReferencia": { "monto": 45, "moneda": "USD" },
  "baseCalculoPrecio": "POR_PERSONA", "condiciones": "...",
  "componentes": [ { "nombre": "Almuerzo", "tipo": "ALIMENTACION", "condicion": "INCLUIDO" } ],
  "requisitos": { "edadMinima": 12, "nivelEsfuerzo": "MEDIO" } }
```

### Nombre: DisponibilidadResponse
### Campos: experienciaId; fechas[{fecha, cupoDisponible, estado}]
### Ejemplo:
```json
{ "experienciaId": "EXP-URU-01", "fechas": [ { "fecha": "2026-07-20", "cupoDisponible": 6, "estado": "DISPONIBLE" } ] }
```

### Nombre: ReservaResponse
### Campos: id; estado; experienciaId; fecha; numeroParticipantes; pago{estado}; confirmacionId?
### Ejemplo:
```json
{ "id": "RES-1001", "estado": "CONFIRMADA", "experienciaId": "EXP-URU-01", "fecha": "2026-07-20",
  "numeroParticipantes": 3, "pago": { "estado": "EXITOSO" }, "confirmacionId": "CONF-7001" }
```

### Nombre: PagoResponse
### Campos: pagoId; estado; referenciaPasarela; redirectUrl?
### Ejemplo:
```json
{ "pagoId": "PAY-9001", "estado": "INICIADO", "referenciaPasarela": "tok_abc", "redirectUrl": "https://pasarela/..." }
```

### Nombre: CotizacionResponse
### Campos: solicitudId; cotizacionId; estado
### Ejemplo:
```json
{ "solicitudId": "QR-2001", "cotizacionId": "QT-4001", "estado": "COTIZACION_EMITIDA" }
```

### Nombre: PedidoEquipoResponse
### Campos: id; estadoImportacion; lineas[{equipoId,cantidad}]; sujetoImportacion
### Ejemplo:
```json
{ "id": "PED-3001", "estadoImportacion": "RECIBIDO",
  "lineas": [ { "equipoId": "EQ-KAYAK-01", "cantidad": 2 } ], "sujetoImportacion": true }
```

### Nombre: ErrorResponse
### Campos: success(false); errorCode; message; details?[{field, error}]
### Ejemplo:
```json
{ "success": false, "errorCode": "ERR-021", "message": "Capacidad insuficiente",
  "details": [ { "field": "numeroParticipantes", "error": "supera el cupo disponible (6)" } ] }
```

---

# 7. CATÁLOGO DE ERRORES HTTP

## 400 Bad Request
- Código interno: ERR-010
- Descripción: Petición mal formada o con validación sintáctica fallida.
- Causa: Campos requeridos vacíos, formato inválido, enum fuera de rango, paginación inválida.
- Acción Correctiva: Corregir el cuerpo/parámetros según `details` y reintentar.

## 401 Unauthorized
- Código interno: ERR-030 (sin token) · ERR-031 (firma de webhook inválida)
- Descripción: Falta autenticación o credenciales/firma inválidas.
- Causa: Token ausente/expirado en back-office; firma de webhook no verificable.
- Acción Correctiva: Autenticarse y renovar token; en webhook, verificar la firma del proveedor.

## 403 Forbidden
- Código interno: ERR-030
- Descripción: Autenticado pero sin rol suficiente para la operación.
- Causa: Rol distinto al requerido (p. ej. PUBLIC/CLIENT en endpoint de back-office).
- Acción Correctiva: Usar una cuenta con el rol adecuado.

## 404 Not Found
- Código interno: ERR-011 (recurso) · ERR-014 (solicitud cotización) · ERR-020 (experiencia) · ERR-025 (equipo)
- Descripción: El recurso referenciado no existe o no está publicado.
- Causa: Identificador inexistente o contenido no publicado.
- Acción Correctiva: Verificar el identificador o el estado de publicación.

## 409 Conflict
- Código interno: ERR-021 (cupo) · ERR-022 (fecha sin cupo) · ERR-026 (estado no apto para pago) · ERR-012 (transición)
- Descripción: La operación choca con el estado actual del recurso.
- Causa: Cupo insuficiente, fecha agotada, reserva no en INICIADA, transición de estado inválida.
- Acción Correctiva: Ajustar la solicitud al estado vigente del recurso.

## 422 Unprocessable Entity
- Código interno: ERR-013 (cotización incompleta) · ERR-024 (monto inconsistente) · ERR-023 (pago no completado) · ERR-012 (regla de transición)
- Descripción: Sintaxis correcta pero la regla de dominio no se satisface.
- Causa: Datos semánticamente inválidos o invariante de dominio no cumplida.
- Acción Correctiva: Revisar la regla de negocio asociada (BR-XXX) y corregir.

## 500 Internal Server Error
- Código interno: ERR-500
- Descripción: Error inesperado del servidor.
- Causa: Fallo no controlado.
- Acción Correctiva: Reintentar; si persiste, escalar a soporte. No exponer detalles internos.

> Nota: 429 Too Many Requests (ERR-032) aplica a endpoints públicos de escritura
> (inquiries, quote-requests, equipment-orders) por límite de tasa / anti-bot.

---

# 8. MATRIZ DE TRAZABILIDAD

| Epic | Feature | User Story | Use Case | Endpoint |
| ---- | ------- | ---------- | -------- | -------- |
| EP-001 | FT-001 | US-001 | UC-01 | (cliente WhatsApp; deep link, sin endpoint REST de negocio) |
| EP-001 | FT-001 | US-002 | UC-02 | (deep link contextual; sin endpoint REST) |
| EP-001 | FT-002 | US-003 | UC-03 | POST /inquiries |
| EP-001 | FT-003 | US-004 | UC-04 | GET /inquiries |
| EP-001 | FT-004 | US-005 | UC-05 | PATCH /inquiries/{id} |
| EP-001 | FT-005 | US-006 | UC-06 | POST /inquiries (idioma) |
| EP-002 | FT-006 | US-007 | UC-07 | GET /activities |
| EP-002 | FT-007 | US-008 | UC-08 | GET /activities/{id} |
| EP-002 | FT-008 | US-009 | UC-09 | GET /activities/compare |
| EP-002 | FT-009 | US-010 | UC-10 | GET /activities/{id} |
| EP-003 | FT-010 | US-011 | UC-11 | GET /trust-center |
| EP-003 | FT-011 | US-012 | UC-12 | GET /guides |
| EP-003 | FT-012 | US-013 | UC-13 | GET /trust-center |
| EP-003 | FT-013 | US-014 | UC-14 | GET /reviews |
| EP-003 | FT-014 | US-015 | UC-15 | GET /faqs |
| EP-004 | FT-015 | US-016 | UC-16 | GET /media |
| EP-004 | FT-016 | US-017 | UC-17 | GET /social-feed |
| EP-004 | FT-017 | US-018 | UC-18 | GET /articles |
| EP-005 | FT-018 | US-019 | UC-19 | GET /destinations/{id} |
| EP-005 | FT-019 | US-020 | UC-20 | (param lang transversal en endpoints públicos) |
| EP-005 | FT-020 | US-021 | UC-21 | GET /seo/metadata |
| EP-005 | FT-021 | US-022 | UC-22 | GET /locations/{id} · PATCH /locations/{id} |
| EP-006 | FT-022 | US-023 | UC-23 | GET /activities (perfilCliente) |
| EP-006 | FT-023 | US-024 | UC-24 | GET /activities/{id} |
| EP-006 | FT-024 | US-025 | UC-25 | GET /activities/recommendations |
| EP-007 | FT-025 | US-026 | UC-26 | POST /quote-requests |
| EP-007 | FT-026 | US-027 | UC-27 | GET /quote-requests · POST /quote-requests/{id}/quotation |
| EP-007 | FT-027 | US-028 | UC-28 | GET /corporate |
| EP-007 | FT-028 | US-029 | UC-29 | GET /corporate/material |
| EP-008 | FT-029 | US-030 | UC-30 | GET /availability |
| EP-008 | FT-030 | US-031 | UC-31 | POST /reservations |
| EP-008 | FT-031 | US-032 | UC-32 | POST /reservations/{id}/payment · POST /payments/webhook |
| EP-008 | FT-032 | US-033 | UC-33 | GET /reservations/{id} |
| EP-008 | FT-033 | US-034 | UC-34 | GET /reservations · PATCH /reservations/{id} |
| EP-009 | FT-034 | US-035 | UC-35 | GET /equipment |
| EP-009 | FT-035 | US-036 | UC-36 | POST /equipment-orders |
| EP-009 | FT-036 | US-037 | UC-37 | GET /equipment (tipoCliente) |
| EP-009 | FT-038 | US-038 | UC-38 | GET /equipment-orders · PATCH /equipment-orders/{id} |

Cobertura: 38/38 US · 9/9 EP · 37/37 FT. Todos los Commands (C-01..C-13) y Queries
(Q-01..Q-21) de backend-specification tienen al menos un endpoint.
US-001/US-002/US-020 se satisfacen por mecanismos del cliente (deep link de WhatsApp,
parámetro de idioma) sin endpoint REST de negocio dedicado, según ui-specification.

---

# 9. OPENAPI READINESS

| Endpoint | Método | Recurso | Auth |
| -------- | ------ | ------- | ---- |
| /api/v1/destinations | GET | destinations | No |
| /api/v1/destinations/{destinationId} | GET | destinations | No |
| /api/v1/activities | GET | activities | No |
| /api/v1/activities/{activityId} | GET | activities | No |
| /api/v1/activities/compare | GET | activities | No |
| /api/v1/activities/recommendations | GET | activities | No |
| /api/v1/trust-center | GET | trust-center | No |
| /api/v1/guides | GET | guides | No |
| /api/v1/reviews | GET | reviews | No |
| /api/v1/faqs | GET | faqs | No |
| /api/v1/media | GET | media | No |
| /api/v1/social-feed | GET | social-feed | No |
| /api/v1/articles | GET | articles | No |
| /api/v1/articles/{articleId} | GET | articles | No |
| /api/v1/seo/metadata | GET | seo | No |
| /api/v1/locations/{locationId} | GET | locations | No |
| /api/v1/corporate | GET | corporate | No |
| /api/v1/corporate/material | GET | corporate | No |
| /api/v1/equipment | GET | equipment | No |
| /api/v1/equipment/{equipmentId} | GET | equipment | No |
| /api/v1/availability | GET | availability | No |
| /api/v1/inquiries | POST | inquiries | No |
| /api/v1/inquiries | GET | inquiries | Bearer (RESPONSABLE_COMERCIAL) |
| /api/v1/inquiries/{inquiryId} | PATCH | inquiries | Bearer (RESPONSABLE_COMERCIAL) |
| /api/v1/quote-requests | POST | quote-requests | No |
| /api/v1/quote-requests | GET | quote-requests | Bearer (RESPONSABLE_COMERCIAL) |
| /api/v1/quote-requests/{quoteRequestId}/quotation | POST | quote-requests | Bearer (RESPONSABLE_COMERCIAL) |
| /api/v1/equipment-orders | POST | equipment-orders | No |
| /api/v1/equipment-orders | GET | equipment-orders | Bearer (GESTOR_OPERACIONES) |
| /api/v1/equipment-orders/{orderId} | PATCH | equipment-orders | Bearer (GESTOR_OPERACIONES) |
| /api/v1/reservations | POST | reservations | Bearer (CLIENT) |
| /api/v1/reservations | GET | reservations | Bearer (GESTOR_OPERACIONES) |
| /api/v1/reservations/{reservationId} | GET | reservations | Bearer (CLIENT propia / GESTOR_OPERACIONES) |
| /api/v1/reservations/{reservationId} | PATCH | reservations | Bearer (GESTOR_OPERACIONES) |
| /api/v1/reservations/{reservationId}/payment | POST | payments | Bearer (CLIENT) |
| /api/v1/payments/webhook | POST | payments | Firma (SYSTEM) |
| /api/v1/locations/{locationId} | PATCH | locations | Bearer (GESTOR_OPERACIONES) |
| /api/v1/content/{type}/{id}/publish | POST | content | Bearer (GESTOR_OPERACIONES) |
| /api/v1/content/{type}/{id}/unpublish | POST | content | Bearer (GESTOR_OPERACIONES) |

---

# CHECKLIST DE VALIDACIÓN
- [x] Todos los Commands tienen endpoint (C-01..C-13 → POST/PATCH).
- [x] Todas las Queries tienen endpoint (Q-01..Q-21 → GET).
- [x] Todas las APIs tienen autenticación definida (No / Bearer / Firma).
- [x] Todas las APIs tienen autorización definida (matriz de acceso §2).
- [x] Todos los endpoints tienen Request (parámetros/body).
- [x] Todos los endpoints tienen Response (200/201 + ejemplo).
- [x] Todos los endpoints tienen errores definidos.
- [x] Existe catálogo de DTOs (§6).
- [x] Existe catálogo de errores HTTP (§7).
- [x] Existe matriz de trazabilidad (§8).
- [x] Existe resumen OpenAPI (§9).

---

# NOTA DE TRAZABILIDAD
Documento derivado exclusivamente de backend-specification.md, domain-catalog.md,
domain-model.md y las User Stories; no introduce endpoints sin respaldo de
Command/Query/Use Case. Consistente con la arquitectura técnica (REST/JSON,
/api/v1, JWT, webhook firmado e idempotente). Se conserva la corrección heredada
FT-037 → FT-038 (US-038), aplicada en §8.