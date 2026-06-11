# FRONTEND CONTRACT — Rafting Peru & Kayak Adventures
CypData · Junio 2026
Entradas: ui-specification.md · api-specification.md · backend-specification.md
Contrato funcional Frontend (sin código). Une pantallas/flujos (UI Spec) con endpoints (API Spec).

---

# RESUMEN GENERAL

Este contrato traduce la especificación de experiencia (sitemap, pantallas P01–P26,
flujos F1–F10, formularios FORM-1..5, estados) en un contrato implementable por el
equipo Frontend (Next.js/React/TS, según la arquitectura técnica), enlazando cada
pantalla con los endpoints REST que consume (`/api/v1`), sus formularios, validaciones
de interfaz, estados de UI y reglas de navegación.

Principios heredados de la UI Spec: contacto a un toque (WhatsApp/consulta persistente),
transparencia de la oferta, bilingüe (ES/EN) y prioridad móvil. Toda llamada a API usa
la envoltura estándar `{ success, data }` / `{ success, errorCode, message, details }`;
el idioma se propaga como parámetro `lang` en endpoints públicos de contenido.

Alcance por release (de la UI Spec): R1 captación (WhatsApp, formularios, confianza,
contenido); R3 transaccional (disponibilidad, reserva, pago, confirmación) y tienda de
equipos. El back-office (P22–P26) es la capa autenticada.

Convención de manejo de respuestas en Frontend:
- 2xx → render de `data`. Listados usan `data.items` + `meta.{page,pageSize,total}`.
- 4xx → mapear `errorCode` a mensaje localizado y, si hay `details[]`, marcar campos.
- 401/403 → redirigir a login (back-office) o mostrar acceso restringido.
- 429 → mensaje "intenta más tarde" en formularios públicos.
- 5xx → aviso no intrusivo con opción de reintentar.

---

# CATÁLOGO DE PANTALLAS

## Pantalla: P01 Inicio
### Objetivo: Presentar propuesta de valor, accesos a destinos/experiencias, confianza resumida y contacto persistente.
### Componentes: Encabezado+selector idioma, hero, tarjetas de destino, bloque confianza resumido, acceso WhatsApp flotante, pie.
### Formularios: —
### Campos: —
### Validaciones: —
### Estados UI: carga (tarjetas), vacío (sin destinos publicados → "contenido en preparación"), error de carga (reintentar).
### APIs Consumidas: GET /destinations; GET /trust-center (resumen); GET /contacto/whatsapp (deep link)
### Navegación: → P02 (destinos), → P04 (experiencias), → P08 (confianza), WhatsApp (externo).

## Pantalla: P02 Índice de Destinos
### Objetivo: Listar los 4 ríos con entrada a cada uno.
### Componentes: Tarjeta de destino, migas de pan.
### Formularios: —
### Validaciones: —
### Estados UI: carga, vacío, error.
### APIs Consumidas: GET /destinations?lang
### Navegación: → P03 (detalle de destino).

## Pantalla: P03 Página de Destino/Río
### Objetivo: Contenido propio del destino + experiencias + galería + vía de contacto (US-019, US-016).
### Componentes: Encabezado de destino, galería multimedia (carga progresiva), tarjetas de experiencia, acceso WhatsApp contextual.
### Formularios: —
### Validaciones: —
### Estados UI: carga (galería progresiva, no bloqueante), vacío (sin experiencias), error.
### APIs Consumidas: GET /destinations/{id}?lang; GET /media?destinoId; GET /contacto/whatsapp?destinoId
### Navegación: → P05 (detalle experiencia), WhatsApp contextual.

## Pantalla: P04 Índice de Experiencias/Paquetes
### Objetivo: Listado con filtros (destino, dificultad, perfil) y estado de disponibilidad (US-007).
### Componentes: Filtros, tarjeta de experiencia, paginación.
### Formularios: filtros (no es POST; query params).
### Campos: destino, dificultad, perfilCliente (todos opcionales).
### Validaciones: enums válidos antes de llamar.
### Estados UI: carga, vacío (filtro sin coincidencias → mensaje + ajustar filtros), error.
### APIs Consumidas: GET /activities?destino&dificultad&perfilCliente&lang&page&pageSize&sortBy&sortDirection
### Navegación: → P05 (detalle), → P06 (comparador, al seleccionar 2+).

## Pantalla: P05 Detalle de Experiencia/Paquete
### Objetivo: Incluido/opcional, precio de referencia, condiciones, acción de consulta/reserva (US-008, US-010).
### Componentes: Bloque incluido/opcional, requisitos (edad, esfuerzo), CTA consulta (WhatsApp/form), CTA reserva (R3), acceso a comparación.
### Formularios: — (acceso a FORM-1 o flujo de reserva).
### Validaciones: —
### Estados UI: carga, error (404 → "experiencia no disponible"); no contratable si no publicada.
### APIs Consumidas: GET /activities/{id}?lang; (R3) GET /availability?activityId
### Navegación: → P06 (comparar), → P17/F1 (consulta), → P18 (disponibilidad, R3).

## Pantalla: P06 Comparador de Paquetes
### Objetivo: Comparación lado a lado (US-009).
### Componentes: Tabla/panel comparador (apilado en móvil).
### Validaciones: 2..N ids seleccionados.
### Estados UI: carga, error (400 si <2 ids; 404 si alguno no publicado).
### APIs Consumidas: GET /activities/compare?ids=...&lang
### Navegación: ← P04/P05; → P05 (volver a detalle).

## Pantalla: P07 Aventura para todos (por perfil)
### Objetivo: Navegación por perfil con requisitos visibles + recomendaciones (US-023, US-024, US-025).
### Componentes: Selector/segmentador de perfil, tarjetas con requisitos.
### Validaciones: perfilCliente válido.
### Estados UI: carga, vacío (sin experiencias para el perfil → mensaje), error.
### APIs Consumidas: GET /activities?perfilCliente; GET /activities/recommendations?perfilCliente
### Navegación: → P05 (detalle).

## Pantalla: P08 Seguridad y Confianza
### Objetivo: Certificaciones, guías, protocolos (US-011, US-012, US-013).
### Componentes: Bloque certificaciones (con entidad emisora), tarjetas de guía, bloque protocolos.
### Validaciones: —
### Estados UI: carga, error.
### APIs Consumidas: GET /trust-center?lang; GET /guides?lang
### Navegación: → P10 (reseñas), → P09 (FAQ).

## Pantalla: P09 FAQ
### Objetivo: Búsqueda de términos (US-015).
### Componentes: Acordeón FAQ, buscador.
### Formularios: búsqueda (query param, no POST).
### Campos: termino, categoria (opcionales).
### Validaciones: —
### Estados UI: carga, vacío (sin coincidencias), error.
### APIs Consumidas: GET /faqs?termino&categoria&lang
### Navegación: ← P08.

## Pantalla: P10 Reseñas y Testimonios
### Objetivo: Valoraciones, eventualmente por destino/experiencia (US-014).
### Componentes: Componente de reseñas, paginación.
### Validaciones: —
### Estados UI: carga, vacío (sin reseñas → mensaje neutro, no error), error.
### APIs Consumidas: GET /reviews?objetivoRef&lang&page&pageSize
### Navegación: ← P08/P05.

## Pantalla: P11 Galería y Blog
### Objetivo: Multimedia por destino y artículos (US-016, US-017, US-018).
### Componentes: Galería progresiva, bloque social embebido (degradación elegante), listado de artículos.
### Validaciones: —
### Estados UI: carga progresiva (multimedia); bloque social ausente sin romper página (available=false); error parcial tolerado.
### APIs Consumidas: GET /media; GET /social-feed; GET /articles?lang&page&pageSize
### Navegación: → detalle de artículo (GET /articles/{id}).

## Pantalla: P12 Empresas y Grupos (B2B)
### Objetivo: Propuesta de valor + acceso a cotización + material descargable (US-028, US-029).
### Componentes: Bloque propuesta corporativa, botón cotizar, enlace material.
### Validaciones: —
### Estados UI: carga, error (404 material → "no disponible").
### APIs Consumidas: GET /corporate?lang; GET /corporate/material?lang
### Navegación: → P13 (cotización de grupos).

## Pantalla: P13 Solicitud de Cotización de Grupos (FORM-2)
### Objetivo: Registrar solicitud B2B (US-026).
### Componentes: Formulario, mensaje de confirmación.
### Formularios: FORM-2.
### Campos: organizacion (req.), numeroPersonas (req., ≥1), destinoId (req.), fechaSolicitada (req., ≥ hoy), contacto {nombre req., email?, telefono?}.
### Validaciones: requeridos marcados; numeroPersonas entero ≥1; fecha ≥ hoy; bloquear envío y señalar campos si falta algo (mapear ERR-013/details).
### Estados UI: envío en progreso (bloquear reenvío), éxito (confirmación de recepción), error de validación (por campo), 429 (reintentar luego).
### APIs Consumidas: POST /quote-requests; (selector destino) GET /destinations
### Navegación: → confirmación; ← P12.

## Pantalla: P14 Catálogo de Equipos
### Objetivo: Listado con filtro por tipo de cliente (US-035, US-037).
### Componentes: Filtro tipoCliente, tarjeta de equipo, paginación.
### Validaciones: tipoCliente válido si se envía.
### Estados UI: carga, vacío (sin oferta específica → muestra oferta general, BR-018), error.
### APIs Consumidas: GET /equipment?tipoCliente&lang&page&pageSize
### Navegación: → P15 (detalle de equipo).

## Pantalla: P15 Detalle de Equipo
### Objetivo: Ficha + condición bajo pedido + acceso a solicitud (US-036).
### Componentes: Ficha de equipo, aviso "bajo pedido/importación", CTA solicitar.
### Validaciones: —
### Estados UI: carga, error (404 → ERR-025 "equipo no disponible").
### APIs Consumidas: GET /equipment/{equipmentId}?lang
### Navegación: → P16 (solicitud/pedido, equipo precargado).

## Pantalla: P16 Solicitud de Cotización/Pedido de Equipo (FORM-3)
### Objetivo: Registrar pedido de equipos (US-036).
### Componentes: Formulario con equipo precargado, aviso de importación, confirmación.
### Formularios: FORM-3.
### Campos: lineas[{equipoId precargado, cantidad ≥1}], tipoCliente (req.), contacto {nombre req., email?, telefono?}.
### Validaciones: ≥1 línea; cantidad ≥1; aviso explícito "sujeto a disponibilidad de importación"; mapear ERR-010/ERR-025.
### Estados UI: envío en progreso, éxito (sujetoImportacion=true visible), error por campo, 429.
### APIs Consumidas: POST /equipment-orders
### Navegación: → confirmación; ← P15.

## Pantalla: P17 Contacto / Consulta (FORM-1)
### Objetivo: Formulario general de consulta (US-003).
### Componentes: Formulario, confirmación de recepción.
### Formularios: FORM-1.
### Campos: nombreContacto (req.), medioContacto (req., email o teléfono), destinoInteresId (opcional), mensaje (req.), canal (FORMULARIO por defecto), idioma (del selector activo).
### Validaciones: requeridos no vacíos; formato de medioContacto; preservar datos ante error; foco al primer campo inválido (mapear ERR-010/details).
### Estados UI: envío en progreso (bloquear reenvío), éxito (recepcionConfirmada=true), error por campo, 429.
### APIs Consumidas: POST /inquiries; (selector destino) GET /destinations
### Navegación: → confirmación.

## Pantalla: P18 Disponibilidad/Calendario (R3)
### Objetivo: Consultar fechas y cupos de una experiencia (US-030).
### Componentes: Calendario/selector de disponibilidad, indicador de carga.
### Validaciones: experiencia publicada.
### Estados UI: carga (resolviendo calendario); fechas sin cupo marcadas no disponibles; si ninguna → mensaje a consultar por otros medios; error (404 ERR-020).
### APIs Consumidas: GET /availability?activityId&from&to
### Navegación: → P19 (reserva, al elegir fecha con cupo).

## Pantalla: P19 Reserva (FORM-4 parte 1, R3)
### Objetivo: Selección de fecha/participantes + datos (US-031).
### Componentes: Resumen de reserva, selector participantes, datos del cliente.
### Formularios: FORM-4 (parte datos).
### Campos: experienciaId (precargado), disponibilidadId (de P18), fecha, numeroParticipantes (≥1, ≤ cupo), cliente {nombre req., email?, telefono?}.
### Validaciones: no permitir más participantes que cupo (mapear 409 ERR-021/ERR-022, informar cupo disponible).
### Estados UI: validación de cupo en vivo; error de cupo; requiere sesión CLIENT (Bearer).
### APIs Consumidas: POST /reservations (requiere Authorization: Bearer)
### Navegación: → P20 (pago) tras crear reserva INICIADA.

## Pantalla: P20 Pago (FORM-4 parte 2, R3)
### Objetivo: Pagar la reserva (US-032).
### Componentes: Componente de pago (delegado a pasarela), indicador de operación en curso.
### Formularios: FORM-4 (parte pago).
### Campos: monto {monto, moneda} (coincide con la reserva).
### Validaciones: reserva en estado INICIADA (409 ERR-026); monto consistente (422 ERR-024); bloquear duplicación del intento mientras procesa.
### Estados UI: procesando pago (no duplicar), redirección a pasarela (redirectUrl), pago fallido → no confirma, ofrecer reintento (ERR-023).
### APIs Consumidas: POST /reservations/{id}/payment (Bearer). El resultado real llega por webhook al backend (POST /payments/webhook); el front consulta estado en P21.
### Navegación: → P21 (confirmación) al volver de la pasarela.

## Pantalla: P21 Confirmación de Reserva (R3)
### Objetivo: Mostrar confirmación automática con el detalle (US-033).
### Componentes: Resumen de confirmación (experiencia, fecha, participantes, datos).
### Validaciones: —
### Estados UI: carga (consultando estado), CONFIRMADA → muestra confirmacionId; NO_CONFIRMADA → mensaje de pago no completado con reintento.
### APIs Consumidas: GET /reservations/{id} (Bearer; propia)
### Navegación: ← P20; → inicio.

## Pantalla: P22 Bandeja de Consultas/Leads (back-office)
### Objetivo: Ver y gestionar leads (US-004, US-005).
### Componentes: Listado con filtros por estado/canal/fecha, detalle de consulta, control de cambio de estado, historial.
### Formularios: filtros (query); cambio de estado (PATCH).
### Campos (cambio estado): nuevoEstado (enum EstadoConsulta).
### Validaciones: rol RESPONSABLE_COMERCIAL; transición válida (409/422 ERR-012).
### Estados UI: carga, vacío ("sin consultas"), 401/403 (acceso restringido), actualización optimista con reversión si falla.
### APIs Consumidas: GET /inquiries (Bearer); PATCH /inquiries/{id} (Bearer)
### Navegación: detalle ↔ listado.

## Pantalla: P23 Gestión de Cotizaciones (back-office)
### Objetivo: Ver solicitudes y emitir cotización (US-027).
### Componentes: Listado de solicitudes, detalle, editor de contenido de cotización.
### Formularios: emitir cotización (POST).
### Campos: contenido (req., no vacío).
### Validaciones: rol RESPONSABLE_COMERCIAL; solicitud existente (404 ERR-014).
### Estados UI: carga, vacío, 401/403, éxito (estado → COTIZACION_EMITIDA).
### APIs Consumidas: GET /quote-requests (Bearer); POST /quote-requests/{id}/quotation (Bearer)
### Navegación: detalle ↔ listado.

## Pantalla: P24 Gestión de Reservas (back-office)
### Objetivo: Ver y actualizar estado de reservas (US-034).
### Componentes: Listado con filtros, detalle, control de cambio de estado, historial.
### Formularios: cambio de estado (PATCH).
### Campos: nuevoEstado (enum EstadoReserva).
### Validaciones: rol GESTOR_OPERACIONES; transición válida (409/422 ERR-012); cupo se ajusta si CANCELADA.
### Estados UI: carga, vacío, 401/403.
### APIs Consumidas: GET /reservations (Bearer); GET /reservations/{id} (Bearer); PATCH /reservations/{id} (Bearer)
### Navegación: detalle ↔ listado.

## Pantalla: P25 Gestión de Pedidos de Equipos (back-office)
### Objetivo: Ver y actualizar estado de importación (US-038).
### Componentes: Listado con filtro por estado de importación, detalle, control de cambio de estado, historial.
### Formularios: cambio de estado (PATCH).
### Campos: nuevoEstado (enum EstadoPedidoEquipo).
### Validaciones: rol GESTOR_OPERACIONES; transición válida.
### Estados UI: carga, vacío, 401/403.
### APIs Consumidas: GET /equipment-orders (Bearer); PATCH /equipment-orders/{id} (Bearer)
### Navegación: detalle ↔ listado.

## Pantalla: P26 Acceso / Autenticación interna (FORM-5)
### Objetivo: Autenticación de actores operativos (back-office).
### Componentes: Formulario de acceso.
### Formularios: FORM-5.
### Campos: identificación de usuario, credencial.
### Validaciones: acceso solo a roles autorizados; mostrar error sin filtrar detalle.
### Estados UI: envío en progreso, error de credenciales, redirección por rol (RESPONSABLE_COMERCIAL→P22/P23; GESTOR_OPERACIONES→P24/P25).
### APIs Consumidas: endpoint de identidad (identity-access; fuera del catálogo de negocio) → emite JWT Bearer.
### Navegación: → bandeja según rol.

> Pantalla transversal de publicación de contenido (no numerada en UI Spec) para
> GESTOR_OPERACIONES: consume POST /content/{type}/{id}/publish · /unpublish y
> PATCH /locations/{id} (sincronizar ficha local). Se integra como acción en las
> vistas de administración de contenido.

---

# CATÁLOGO DE COMPONENTES

Componentes reutilizables (de la UI Spec) y su contrato de datos/API:

- **Encabezado + selector de idioma**: estado global `lang` (ES/EN); propaga `lang` a las llamadas de contenido. Sin API propia.
- **Acceso WhatsApp persistente**: consume GET /contacto/whatsapp (deep link, contextual por destino). Visible en todas las pantallas públicas.
- **Tarjeta de destino**: datos de GET /destinations. Navega a P03.
- **Tarjeta de experiencia/paquete**: datos de GET /activities (resumen) — incluye disponibilidadIndicativa. Navega a P05.
- **Bloque incluido/opcional**: datos de GET /activities/{id} (componentes); distinción visual incluido vs opcional (BR-004).
- **Filtros (destino/dificultad/perfil)**: traducen a query params de /activities; panel desplegable en móvil.
- **Comparador**: GET /activities/compare; vista apilada en móvil.
- **Indicador de requisitos (edad/esfuerzo)**: de GET /activities/{id}.requisitos.
- **Galería multimedia**: GET /media; carga progresiva, no bloqueante (US-016).
- **Bloque social embebido**: GET /social-feed; degradación elegante (available=false no rompe).
- **Bloque certificaciones/guías/protocolos**: GET /trust-center, GET /guides.
- **Componente de reseñas**: GET /reviews; estado vacío neutro.
- **Acordeón FAQ con búsqueda**: GET /faqs.
- **Formularios (consulta, cotización, pedido, reserva)**: ver §Catálogo de Pantallas; validación por campo + envoltura de error.
- **Calendario/selector de disponibilidad**: GET /availability; marca fechas sin cupo.
- **Componente de pago**: POST /reservations/{id}/payment → redirectUrl a pasarela.
- **Mensaje/pantalla de confirmación**: GET /reservations/{id}.
- **Bandeja/listado back-office con filtros**: GET de cada recurso con `page/pageSize/sortBy/sortDirection`.
- **Control de cambio de estado**: PATCH del recurso; muestra historial.
- **Feedback transversal**: notificaciones (éxito/error/info), indicadores de carga, estados vacíos — alimentados por el resultado de las APIs.

---

# GESTIÓN DE ESTADO

- **Estado global**: `lang` (ES/EN, persistente en navegación — US-020/BR-019); sesión/JWT y rol (para back-office y cliente); carrito/flujo de reserva en curso (experienciaId, disponibilidadId, fecha, participantes) entre P18→P19→P20→P21.
- **Estado de servidor (cache de datos)**: listados y detalles de contenido (cacheables; apoyados por SSG/ISR en la capa pública); disponibilidad y reservas no se cachean (datos vivos).
- **Estado de UI local**: filtros activos, paginación, estados de formulario (valores, errores por campo, envío en progreso), selección del comparador.
- **Reglas de consistencia**:
  - El idioma seleccionado se conserva entre páginas y se envía a las APIs de contenido y en la consulta (BR-019).
  - El flujo de reserva mantiene `disponibilidadId` desde P18; si el cupo cambió, P19 revalida contra 409 (ERR-021/ERR-022).
  - El pago bloquea reenvíos mientras procesa; el resultado se refleja al consultar la reserva (el webhook lo resuelve en backend).
  - Tras 401, limpiar sesión y redirigir a P26.
- **Idempotencia en UI**: deshabilitar botones de envío durante peticiones POST/PATCH; en pago, impedir doble intento.

---

# MATRIZ DE NAVEGACIÓN

| Origen | Destino | Disparador |
| ------ | ------- | ---------- |
| P01 Inicio | P02 Destinos | "Ver destinos" |
| P01 | P04 Experiencias | "Ver experiencias" |
| P01 | P08 Confianza | "Seguridad" |
| P01 | WhatsApp | Acceso persistente |
| P02 | P03 Destino | Seleccionar río |
| P03 | P05 Detalle experiencia | Seleccionar experiencia |
| P04 | P05 | Seleccionar experiencia |
| P04 | P06 Comparador | Seleccionar 2+ |
| P05 | P06 | "Comparar" |
| P05 | P17 Consulta / WhatsApp | CTA consulta |
| P05 | P18 Disponibilidad (R3) | CTA reservar |
| P07 | P05 | Seleccionar recomendación |
| P08 | P09 FAQ / P10 Reseñas | Navegación confianza |
| P11 | Detalle artículo | Abrir artículo |
| P12 | P13 Cotización grupos | "Cotizar" |
| P14 | P15 Detalle equipo | Seleccionar equipo |
| P15 | P16 Solicitud equipo | "Solicitar" |
| P18 | P19 Reserva | Elegir fecha con cupo |
| P19 | P20 Pago | Confirmar datos (reserva INICIADA) |
| P20 | Pasarela externa | redirectUrl |
| Pasarela | P21 Confirmación | Retorno |
| P26 Login | P22/P23 | Rol RESPONSABLE_COMERCIAL |
| P26 Login | P24/P25 | Rol GESTOR_OPERACIONES |
| P22/P23/P24/P25 | Detalle ítem | Seleccionar fila |

Transversal: selector de idioma y acceso WhatsApp disponibles en todas las pantallas
públicas; migas de pan en P03/P05/P15.

---

# MATRIZ DE TRAZABILIDAD

| Pantalla | User Stories | APIs Consumidas |
| -------- | ------------ | --------------- |
| P01 Inicio | (entrada general) | GET /destinations, /trust-center, /contacto/whatsapp |
| P02 Destinos | US-019 | GET /destinations |
| P03 Destino | US-019, US-016 | GET /destinations/{id}, /media, /contacto/whatsapp |
| P04 Experiencias | US-007, US-023 | GET /activities |
| P05 Detalle experiencia | US-008, US-010, US-024 | GET /activities/{id}, GET /availability (R3) |
| P06 Comparador | US-009 | GET /activities/compare |
| P07 Aventura para todos | US-023, US-024, US-025 | GET /activities (perfil), /activities/recommendations |
| P08 Confianza | US-011, US-012, US-013 | GET /trust-center, /guides |
| P09 FAQ | US-015 | GET /faqs |
| P10 Reseñas | US-014 | GET /reviews |
| P11 Galería/Blog | US-016, US-017, US-018 | GET /media, /social-feed, /articles |
| P12 B2B | US-028, US-029 | GET /corporate, /corporate/material |
| P13 Cotización grupos | US-026 | POST /quote-requests, GET /destinations |
| P14 Catálogo equipos | US-035, US-037 | GET /equipment |
| P15 Detalle equipo | US-036 | GET /equipment/{id} |
| P16 Solicitud equipo | US-036 | POST /equipment-orders |
| P17 Consulta | US-003, US-006 | POST /inquiries, GET /destinations |
| P18 Disponibilidad | US-030 | GET /availability |
| P19 Reserva | US-031 | POST /reservations |
| P20 Pago | US-032 | POST /reservations/{id}/payment |
| P21 Confirmación | US-033 | GET /reservations/{id} |
| P22 Consultas (BO) | US-004, US-005 | GET /inquiries, PATCH /inquiries/{id} |
| P23 Cotizaciones (BO) | US-027 | GET /quote-requests, POST /quote-requests/{id}/quotation |
| P24 Reservas (BO) | US-034 | GET /reservations, PATCH /reservations/{id} |
| P25 Pedidos equipo (BO) | US-038 | GET /equipment-orders, PATCH /equipment-orders/{id} |
| P26 Login | (acceso BO) | identity-access (JWT) |
| Acción publicación (BO) | US-007, US-021, US-022 | POST /content/{type}/{id}/publish·unpublish, PATCH /locations/{id} |

Cobertura: 26 pantallas (P01–P26) + acciones de administración; todas con al menos una
API asociada, y todos los endpoints de la API Spec consumidos por al menos una pantalla
(las APIs públicas de contenido se reparten entre P01–P14; las de back-office entre
P22–P25 + acción de publicación). US-001/US-002 (WhatsApp) se resuelven con el deep link
(GET /contacto/whatsapp) presente en pantallas públicas; US-020 (idioma) es transversal.

---

# CHECKLIST DE VALIDACIÓN
- [x] Todas las pantallas documentadas (P01–P26 + acción de publicación BO).
- [x] Todas las APIs asociadas (cada pantalla lista sus endpoints; ver matriz).
- [x] Validaciones UI definidas (por formulario: FORM-1..5, con mapeo de errores).
- [x] Navegación definida (matriz de navegación + navegación por pantalla).
- [x] Sin pantallas sin API ni APIs sin pantalla (verificado en matriz de trazabilidad).

---

# NOTA DE TRAZABILIDAD
Contrato derivado de ui-specification.md (pantallas, flujos, formularios, estados) y
api-specification.md (endpoints, métodos, auth, errores), consistente con la
backend-specification (BR/ERR). No introduce pantallas ni endpoints nuevos; enlaza los
existentes. El idioma (lang) y el acceso WhatsApp son transversales. Se conserva la
corrección heredada FT-037 → FT-038 (US-038 / P25 gestión de pedidos de equipos).