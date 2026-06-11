# QA SPECIFICATION — Rafting Peru & Kayak Adventures
QA Lead · CypData · Junio 2026
Entradas: user-stories.md · acceptance-criteria.md · backend-specification.md · api-specification.md · frontend-contract.md
Especificación de pruebas funcionales (sin código de automatización). Trazabilidad US → AC → caso de prueba.

---

# ESTRATEGIA DE QA

## Objetivo
Verificar que la plataforma cumple las 38 User Stories y sus Acceptance Criteria
(Gherkin), validando comportamiento funcional, contratos de API, interacción de UI,
reglas de negocio (BR-001..BR-020) y manejo de errores (ERR-010..ERR-032), preservando
la cadena de trazabilidad Oportunidad → Epic → Feature → US → AC → caso de prueba.

## Niveles y tipos de prueba
- **Funcional (caja negra)**: cada AC se traduce en al menos un caso positivo; cada flujo alternativo/regla en al menos un caso negativo.
- **API (contrato)**: por endpoint, validar método, auth/rol, request/response, códigos HTTP y errores de la API Spec.
- **UI (interfaz)**: por pantalla (P01–P26), validar estados (carga/vacío/error), validación de formularios, navegación y consumo de APIs según el frontend-contract.
- **Integración (E2E)**: flujos completos que cruzan front + API + dominio (captación, reserva-pago-confirmación, cotización B2B, pedido de equipos, operación back-office).

## Convenciones
- **ID de caso**: `TC-<US>-<n>` (positivo) y `TCN-<US>-<n>` (negativo). API: `TCA-<endpoint>`. UI: `TCU-<Pxx>`. Integración: `TCI-<flujo>`.
- **Formato**: Precondición → Pasos → Resultado esperado, alineado al Gherkin del AC.
- **Datos de prueba**: catálogos sembrados (4 destinos, experiencias publicadas/no publicadas, disponibilidades con/sin cupo, equipos, actores RESPONSABLE_COMERCIAL y GESTOR_OPERACIONES).
- **Entornos**: pruebas en staging con base de datos sembrada; pagos contra pasarela en modo sandbox; webhook simulado con firma válida/ inválida.

## Prioridad (de releases)
- **P0/R1 (captación)**: US-001..US-006, confianza US-011..US-015, contenido y descubrimiento.
- **P1/R2 (atracción)**: contenido de marca, segmentación por perfil, B2B.
- **P2/R3 (transaccional)**: reserva-pago-confirmación US-030..US-034, equipos US-035..US-038.

## Criterios de entrada/salida
- **Entrada**: documentos de especificación congelados; entorno sembrado disponible.
- **Salida**: 100% de AC con al menos un caso ejecutado; 0 defectos críticos abiertos en el flujo de reserva-pago; cobertura de los catálogos de error de la API.

---

# CASOS DE PRUEBA FUNCIONALES (POSITIVOS)

> Un caso por AC positivo. Se listan los representativos por US; los AC se citan entre paréntesis.

## EP-001 Captación
- **TC-001-1** (US-001/AC-001,002): Dado cualquier página pública, al observar la pantalla se ve el acceso WhatsApp persistente; al tocarlo abre conversación con el número del negocio. (frontend: acceso persistente; GET /contacto/whatsapp)
- **TC-002-1** (US-002/AC-001): Estando en P03 (destino) o P05 (experiencia), al iniciar WhatsApp el mensaje viene precargado con la referencia a ese destino/experiencia.
- **TC-002-2** (US-002/AC-002): Desde página general, el mensaje precargado es genérico.
- **TC-003-1** (US-003/AC-001,003): En FORM-1 (P17), al completar requeridos y enviar, POST /inquiries → 201 con `recepcionConfirmada=true` y estado NUEVA. (BR-001)
- **TC-004-1** (US-004/AC-001,002): Como RESPONSABLE_COMERCIAL, GET /inquiries → 200 lista unificada con datos asociados (canal, contacto, destino, mensaje, fecha).
- **TC-004-2** (US-004/AC-003): Al ingresar una nueva consulta, aparece en el listado sin recargar (refresco/polling de P22).
- **TC-005-1** (US-005/AC-001): Como RESPONSABLE_COMERCIAL, PATCH /inquiries/{id} con `nuevoEstado` válido → 200 estado guardado. (BR-002)
- **TC-005-2** (US-005/AC-002): Al consultar el historial, se ven los cambios de estado registrados.
- **TC-005-3** (US-005/AC-003): Filtro por estado en GET /inquiries devuelve solo consultas de ese estado.
- **TC-006-1** (US-006/AC-001,002): Con sitio en EN, el flujo de consulta se presenta en inglés; la consulta registrada conserva el idioma (BR-019).

## EP-002 Oferta
- **TC-007-1** (US-007/AC-001,002): GET /activities (con/sin filtros destino/dificultad) → 200 solo publicadas (BR-016).
- **TC-007-2** (US-007/AC-003): Una experiencia no publicada no aparece como contratable.
- **TC-008-1** (US-008/AC-001,002): GET /activities/{id} → 200 con componentes marcados INCLUIDO/OPCIONAL de forma inequívoca (BR-004).
- **TC-009-1** (US-009/AC-001,002): GET /activities/compare?ids=a,b → 200 matriz lado a lado con diferencias de componentes/duración/precio.
- **TC-010-1** (US-010/AC-001,002): GET /activities/{id} → precio de referencia + baseCalculoPrecio cuando varía por persona/temporada.

## EP-003 Confianza
- **TC-011-1** (US-011/AC-001,002): GET /trust-center → 200 credenciales con entidad emisora (RN-16).
- **TC-012-1** (US-012/AC-001,002): GET /guides → 200 perfiles con nombre, experiencia y credenciales.
- **TC-013-1** (US-013/AC-001,002): GET /trust-center → protocolos de seguridad, incluyendo medidas para niños/adultos mayores.
- **TC-014-1** (US-014/AC-001,002): GET /reviews y GET /reviews?objetivoRef → 200 reseñas (incluye lista vacía sin error, RN-17).
- **TC-015-1** (US-015/AC-001,002): GET /faqs y GET /faqs?termino → 200 FAQ relevantes.

## EP-004 Contenido
- **TC-016-1** (US-016/AC-001,002): GET /media?destinoId → 200 fotos/videos del destino; carga progresiva en UI (P03/P11).
- **TC-017-1** (US-017/AC-001): GET /social-feed → 200 con posts cuando hay contenido.
- **TC-018-1** (US-018/AC-001,002): GET /articles y GET /articles/{id} → 200; el artículo enlaza destinos/experiencias relacionados.

## EP-005 Descubrimiento
- **TC-019-1** (US-019/AC-001,002): GET /destinations/{id} → 200 contenido propio + experiencias + vía de contacto.
- **TC-020-1** (US-020/AC-001,002): Con `lang=EN`, el contenido se sirve en inglés; al navegar a otra página se mantiene el idioma (BR-019).
- **TC-021-1** (US-021/AC-001,002): GET /seo/metadata?recursoRef de un recurso publicado → 200 metadatos estructurados.
- **TC-022-1** (US-022/AC-001): GET /locations/{id} → 200 contacto, ubicación y reseñas de la sede.
- **TC-022-2** (US-022/AC-002): Tras PATCH /locations/{id} (GESTOR_OPERACIONES), la ficha refleja la información vigente (BR-017).

## EP-006 Segmentación
- **TC-023-1** (US-023/AC-001): GET /activities?perfilCliente=FAMILIA → 200 solo experiencias aptas para el perfil.
- **TC-024-1** (US-024/AC-001,002): GET /activities/{id} → requisitos (edadMinima, nivelEsfuerzo) visibles.
- **TC-025-1** (US-025/AC-001,002): GET /activities/recommendations?perfilCliente → 200 selección acorde, con acceso a cada detalle.

## EP-007 B2B
- **TC-026-1** (US-026/AC-001): POST /quote-requests con datos completos → 201 estado RECIBIDA y confirmación (BR-011).
- **TC-027-1** (US-027/AC-001): Como RESPONSABLE_COMERCIAL, GET /quote-requests → 200 solicitudes con datos.
- **TC-027-2** (US-027/AC-002): POST /quote-requests/{id}/quotation → 201 cotización asociada, estado COTIZACION_EMITIDA (BR-012).
- **TC-028-1** (US-028/AC-001,002): GET /corporate → 200 propuesta B2B + vía de contacto.
- **TC-029-1** (US-029/AC-001,002): GET /corporate/material → 200 versión vigente del material.

## EP-008 Reserva y pago
- **TC-030-1** (US-030/AC-001,002): GET /availability?activityId&from&to → 200 fechas con cupo; las sin cupo marcadas no disponibles (BR-006).
- **TC-031-1** (US-031/AC-001): POST /reservations (CLIENT) con fecha con cupo y participantes válidos → 201 estado INICIADA y cupo reservado (BR-003/004).
- **TC-032-1** (US-032/AC-001): POST /reservations/{id}/payment (INICIADA) → 201 pago INICIADO; webhook EXITOSO → reserva CONFIRMADA (BR-005/007).
- **TC-033-1** (US-033/AC-001,002): Tras confirmación, GET /reservations/{id} → 200 con confirmacionId y detalle (experiencia, fecha, participantes) (BR-008).
- **TC-034-1** (US-034/AC-001,002): Como GESTOR_OPERACIONES, GET /reservations → 200; PATCH /reservations/{id} con transición válida → 200 estado guardado.

## EP-009 Equipos
- **TC-035-1** (US-035/AC-001,002): GET /equipment y GET /equipment/{id} → 200 catálogo + condición de venta bajo pedido.
- **TC-036-1** (US-036/AC-001,002): POST /equipment-orders → 201 estado RECIBIDO con `sujetoImportacion=true` (BR-014).
- **TC-037-1** (US-037/AC-001): GET /equipment?tipoCliente → 200 oferta del tipo de cliente.
- **TC-038-1** (US-038/AC-001,002): Como GESTOR_OPERACIONES, GET /equipment-orders → 200; PATCH /equipment-orders/{id} → 200 estado de importación guardado e historizado.

---

# CASOS NEGATIVOS

> Derivados de flujos alternativos (A1/A2) y reglas de validación. Verifican rechazo correcto y código de error.

- **TCN-001-1** (US-001/AC-003): WhatsApp fuera de horario → mensaje con horario y tiempo estimado de respuesta (no error técnico).
- **TCN-003-1** (US-003/AC-002): POST /inquiries con requeridos vacíos/ inválidos → 400 ERR-010; no se crea la consulta; UI marca campos (P17).
- **TCN-003-2** (US-003): POST /inquiries con `medioContacto` mal formado → 400 ERR-010 detail en `medioContacto`.
- **TCN-003-3** (US-003): Exceso de envíos del formulario → 429 ERR-032 (anti-bot).
- **TCN-004-1** (US-004): GET /inquiries sin token → 401; con rol distinto → 403 ERR-030.
- **TCN-005-1** (US-005): PATCH /inquiries/{id} con transición inválida → 409/422 ERR-012; inexistente → 404 ERR-011.
- **TCN-007-1** (US-007): GET /activities con `dificultad` fuera del enum → 400 ERR-010.
- **TCN-008-1** (US-008): GET /activities/{id} inexistente/no publicada → 404 ERR-011/ERR-020.
- **TCN-009-1** (US-009): GET /activities/compare con 1 solo id → 400 ERR-010; con id no publicado → 404 ERR-011.
- **TCN-026-1** (US-026/AC-002): POST /quote-requests incompleto → 400/422 ERR-013; no registra; UI señala faltantes (P13).
- **TCN-027-1** (US-027): POST /quote-requests/{id}/quotation sobre solicitud inexistente → 404 ERR-014; sin rol → 403 ERR-030.
- **TCN-030-1** (US-030): GET /availability de experiencia inexistente/no publicada → 404 ERR-020.
- **TCN-031-1** (US-031/AC-002): POST /reservations con participantes > cupo → 409 ERR-021, informa cupo; no se crea reserva.
- **TCN-031-2** (US-031): POST /reservations en fecha sin cupo → 409 ERR-022; participantes ≤ 0 → 400 ERR-010.
- **TCN-031-3** (US-031): POST /reservations sin token CLIENT → 401.
- **TCN-032-1** (US-032/AC-002): Webhook resultado FALLIDO/RECHAZADO → reserva NO_CONFIRMADA, libera cupo, informa resultado (ERR-023). (BR-007)
- **TCN-032-2** (US-032): POST /reservations/{id}/payment sobre reserva no INICIADA → 409 ERR-026; monto inconsistente → 422 ERR-024.
- **TCN-032-3** (US-032): Webhook con firma inválida → 401 ERR-031; webhook duplicado → idempotente sin doble efecto (BR-020).
- **TCN-034-1** (US-034): PATCH /reservations/{id} con transición inválida → 409/422 ERR-012; sin rol → 403 ERR-030.
- **TCN-036-1** (US-036): POST /equipment-orders sin líneas o cantidad < 1 → 400 ERR-010; equipo no publicado → 404 ERR-025.
- **TCN-038-1** (US-038): PATCH /equipment-orders/{id} con transición inválida → 409/422 ERR-012; sin rol → 403 ERR-030.

---

# CASOS API

> Por endpoint: método, auth, request válida/ inválida, código HTTP y forma de respuesta (envoltura `{success,data}` / error).

- **TCA-activities-GET**: 200 lista paginada (`data[]` + `meta`); filtros enum válidos; 400 con enum inválido.
- **TCA-activities-id-GET**: 200 detalle con componentes/requisitos; 404 ERR-011/020.
- **TCA-activities-compare-GET**: 200 con 2..N ids; 400 con <2; 404 si alguno no publicado.
- **TCA-availability-GET**: 200 fechas+estado; 404 ERR-020.
- **TCA-inquiries-POST**: 201 ConsultaResponse; 400 ERR-010; 429 ERR-032.
- **TCA-inquiries-GET (BO)**: 200 paginado; 401 sin token; 403 ERR-030.
- **TCA-inquiries-PATCH (BO)**: 200; 404 ERR-011; 409/422 ERR-012; 403.
- **TCA-quote-requests-POST**: 201 RECIBIDA; 400/422 ERR-013; 404 ERR-011.
- **TCA-quote-requests-GET / quotation-POST (BO)**: 200 / 201 COTIZACION_EMITIDA; 404 ERR-014; 403.
- **TCA-reservations-POST**: 201 INICIADA; 409 ERR-021/022; 404 ERR-020; 401.
- **TCA-reservations-payment-POST**: 201 pago INICIADO + redirectUrl; 409 ERR-026; 422 ERR-024.
- **TCA-payments-webhook-POST**: 200 idempotente; 401 ERR-031 (firma); 404 ERR-011; duplicado sin doble efecto.
- **TCA-reservations-id-GET**: 200 detalle+confirmación; 403 ERR-030 (no propia); 404.
- **TCA-reservations-GET/PATCH (BO)**: 200; 401/403; 409/422 ERR-012.
- **TCA-equipment-GET**: 200 lista (oferta general si no hay específica, BR-018); 400 tipoCliente inválido.
- **TCA-equipment-orders-POST**: 201 RECIBIDO; 400 ERR-010; 404 ERR-025; 429.
- **TCA-equipment-orders-GET/PATCH (BO)**: 200; 401/403; 409/422 ERR-012.
- **TCA-locations-id-GET/PATCH**: 200; 404 ERR-011; PATCH requiere GESTOR_OPERACIONES (403 si no).
- **TCA-content-publish/unpublish-POST (BO)**: 200 estadoPublicacion; 404 ERR-011; 403 ERR-030.
- **TCA-trust-center / guides / reviews / faqs / media / social-feed / articles / seo-metadata / corporate / corporate-material-GET**: 200 públicos (social-feed nunca rompe: 200 con `available=false`).

Transversal API: HTTPS obligatorio; cabeceras `Accept`/`Content-Type`; paginación `page≥1`, `pageSize 1..100`; estructura de error con `errorCode`+`message`(+`details`).

---

# CASOS UI

> Por pantalla del frontend-contract: estados de UI, validación de formularios, navegación.

- **TCU-P01**: estados carga/vacío ("contenido en preparación")/error; navegación a P02/P04/P08; acceso WhatsApp visible.
- **TCU-P03**: galería con carga progresiva no bloqueante; vacío sin experiencias; WhatsApp contextual.
- **TCU-P04**: filtros → query params; estado vacío con sugerencia de ajustar filtros; paginación.
- **TCU-P05**: bloque incluido/opcional diferenciado; 404 → "experiencia no disponible"; CTA a P17/P18.
- **TCU-P06**: comparador apilado en móvil; error con <2 ids.
- **TCU-P07**: selector de perfil; estado vacío sin experiencias para el perfil.
- **TCU-P09**: búsqueda FAQ; estado vacío sin coincidencias.
- **TCU-P10**: reseñas; estado vacío neutro sin romper página (US-014).
- **TCU-P11**: bloque social ausente (available=false) sin romper diseño (US-017).
- **TCU-P13 (FORM-2)**: requeridos marcados; bloqueo de envío con faltantes; foco al primer error; preservación de datos; éxito con confirmación; 429.
- **TCU-P16 (FORM-3)**: equipo precargado; aviso "sujeto a importación"; confirmación.
- **TCU-P17 (FORM-1)**: validación por campo (nombre/medioContacto/mensaje); foco al primer inválido; datos preservados; éxito recepcionConfirmada.
- **TCU-P18**: calendario marca fechas sin cupo; si ninguna, mensaje a otros medios; indicador de carga.
- **TCU-P19 (FORM-4 datos)**: no permite participantes > cupo (informa cupo); requiere sesión CLIENT.
- **TCU-P20 (FORM-4 pago)**: indicador en curso; bloqueo de doble intento; pago fallido → no confirma + reintento.
- **TCU-P21**: CONFIRMADA muestra confirmacionId; NO_CONFIRMADA muestra resultado + reintento.
- **TCU-P22**: listado con filtros; vacío "sin consultas"; 401/403 acceso restringido; actualización optimista con reversión.
- **TCU-P23/P24/P25**: listados con filtros; control de cambio de estado con historial; errores de transición y de rol.
- **TCU-P26 (FORM-5)**: error de credenciales sin filtrar detalle; redirección por rol.
- **TCU-transversal**: selector de idioma persistente entre páginas (US-020); WhatsApp persistente; responsive prioridad móvil.

---

# CASOS DE INTEGRACIÓN (E2E)

- **TCI-captacion-form**: Visitante completa FORM-1 (P17) → POST /inquiries → aparece en bandeja P22 (RESPONSABLE_COMERCIAL) → cambio de estado historizado. (US-003 → US-004 → US-005)
- **TCI-captacion-whatsapp**: Desde P05 → WhatsApp contextual con mensaje precargado del destino. (US-002)
- **TCI-reserva-feliz**: P18 disponibilidad → P19 crea reserva INICIADA → P20 inicia pago → webhook EXITOSO → reserva CONFIRMADA → P21 muestra confirmación; cupo reducido y visible en nueva consulta de disponibilidad. (US-030→031→032→033)
- **TCI-reserva-pago-fallido**: igual hasta P20 → webhook FALLIDO → reserva NO_CONFIRMADA → cupo liberado → P21 ofrece reintento. (US-032 AC-002)
- **TCI-reserva-concurrencia**: dos reservas simultáneas sobre la última plaza → solo una CONFIRMADA, la otra recibe 409 ERR-021 (consistencia de cupo, BR-003/009).
- **TCI-webhook-idempotente**: reenvío del mismo webhook (misma referenciaPasarela) → sin doble confirmación ni doble ajuste de cupo (BR-020).
- **TCI-cotizacion-b2b**: P13 POST /quote-requests → P23 RESPONSABLE_COMERCIAL emite cotización → estado COTIZACION_EMITIDA e historizado. (US-026 → US-027)
- **TCI-pedido-equipo**: P16 POST /equipment-orders (RECIBIDO, sujetoImportacion) → P25 GESTOR_OPERACIONES avanza estado de importación hasta ENTREGADO/CANCELADO, historizado. (US-036 → US-038)
- **TCI-publicacion-contenido**: GESTOR_OPERACIONES publica una experiencia (publish) → pasa a aparecer en GET /activities público; unpublish → deja de ser contratable (BR-016).
- **TCI-bilingue**: navegación EN a través de P01→P04→P05 conserva idioma y sirve contenido EN; la consulta registrada queda en EN. (US-020/US-006)
- **TCI-autorizacion**: CLIENT intenta endpoints de back-office → 403; PUBLIC intenta GET /inquiries → 401. (BR-015)

---

# COBERTURA DE REQUISITOS

- **User Stories**: 38/38 con al menos un caso positivo (TC-001-1 .. TC-038-1).
- **Acceptance Criteria**: cada AC de cada US tiene un caso positivo y, donde define flujo alternativo, un caso negativo asociado.
- **Reglas de negocio**: BR-001..BR-020 cubiertas (captación BR-001/010/019; reserva-pago BR-003..009; back-office BR-002/015; equipos BR-013/014/018; contenido BR-016; ficha BR-017; webhook BR-020).
- **Errores**: ERR-010, 011, 012, 013, 014, 020, 021, 022, 023, 024, 025, 026, 030, 031, 032 cubiertos por casos negativos/API.
- **Endpoints**: todos los de la API Spec con al menos un caso (positivo y, donde aplica, negativo).
- **Pantallas**: P01–P26 + acción de publicación cubiertas por casos UI.

---

# MATRIZ DE TRAZABILIDAD

| US | AC (positivos) | Caso(s) positivo(s) | Caso(s) negativo(s) | Endpoint / Pantalla |
| -- | -------------- | ------------------- | ------------------- | ------------------- |
| US-001 | AC-001,002,003 | TC-001-1 | TCN-001-1 | GET /contacto/whatsapp · P01 |
| US-002 | AC-001,002 | TC-002-1, TC-002-2 | — | (deep link) · P03/P05 |
| US-003 | AC-001,002,003 | TC-003-1 | TCN-003-1/2/3 | POST /inquiries · P17 |
| US-004 | AC-001,002,003 | TC-004-1, TC-004-2 | TCN-004-1 | GET /inquiries · P22 |
| US-005 | AC-001,002,003 | TC-005-1/2/3 | TCN-005-1 | PATCH /inquiries/{id} · P22 |
| US-006 | AC-001,002 | TC-006-1 | — | POST /inquiries (idioma) · P17 |
| US-007 | AC-001,002,003 | TC-007-1, TC-007-2 | TCN-007-1 | GET /activities · P04 |
| US-008 | AC-001,002 | TC-008-1 | TCN-008-1 | GET /activities/{id} · P05 |
| US-009 | AC-001,002 | TC-009-1 | TCN-009-1 | GET /activities/compare · P06 |
| US-010 | AC-001,002 | TC-010-1 | — | GET /activities/{id} · P05 |
| US-011 | AC-001,002 | TC-011-1 | — | GET /trust-center · P08 |
| US-012 | AC-001,002 | TC-012-1 | — | GET /guides · P08 |
| US-013 | AC-001,002 | TC-013-1 | — | GET /trust-center · P08 |
| US-014 | AC-001,002 | TC-014-1 | — | GET /reviews · P10 |
| US-015 | AC-001,002 | TC-015-1 | — | GET /faqs · P09 |
| US-016 | AC-001,002 | TC-016-1 | — | GET /media · P03/P11 |
| US-017 | AC-001,002 | TC-017-1 | TCU-P11 (vacío) | GET /social-feed · P11 |
| US-018 | AC-001,002 | TC-018-1 | — | GET /articles · P11 |
| US-019 | AC-001,002 | TC-019-1 | — | GET /destinations/{id} · P03 |
| US-020 | AC-001,002 | TC-020-1 | — | lang transversal · todas |
| US-021 | AC-001,002 | TC-021-1 | — | GET /seo/metadata |
| US-022 | AC-001,002 | TC-022-1, TC-022-2 | — | GET/PATCH /locations/{id} |
| US-023 | AC-001,002 | TC-023-1 | TCU-P07 (vacío) | GET /activities (perfil) · P07 |
| US-024 | AC-001,002 | TC-024-1 | — | GET /activities/{id} · P05 |
| US-025 | AC-001,002 | TC-025-1 | — | GET /activities/recommendations · P07 |
| US-026 | AC-001,002 | TC-026-1 | TCN-026-1 | POST /quote-requests · P13 |
| US-027 | AC-001,002 | TC-027-1, TC-027-2 | TCN-027-1 | GET /quote-requests · POST .../quotation · P23 |
| US-028 | AC-001,002 | TC-028-1 | — | GET /corporate · P12 |
| US-029 | AC-001,002 | TC-029-1 | — | GET /corporate/material · P12 |
| US-030 | AC-001,002 | TC-030-1 | TCN-030-1 | GET /availability · P18 |
| US-031 | AC-001,002 | TC-031-1 | TCN-031-1/2/3 | POST /reservations · P19 |
| US-032 | AC-001,002 | TC-032-1 | TCN-032-1/2/3 | POST .../payment · POST /payments/webhook · P20 |
| US-033 | AC-001,002 | TC-033-1 | — | GET /reservations/{id} · P21 |
| US-034 | AC-001,002 | TC-034-1 | TCN-034-1 | GET/PATCH /reservations · P24 |
| US-035 | AC-001,002 | TC-035-1 | — | GET /equipment · P14/P15 |
| US-036 | AC-001,002 | TC-036-1 | TCN-036-1 | POST /equipment-orders · P16 |
| US-037 | AC-001,002 | TC-037-1 | — | GET /equipment (tipoCliente) · P14 |
| US-038 | AC-001,002 | TC-038-1 | TCN-038-1 | GET/PATCH /equipment-orders · P25 |

Cobertura: 38/38 US · todos los AC con caso positivo · flujos alternativos con caso negativo · endpoints y pantallas referenciados.

---

# CHECKLIST DE VALIDACIÓN
- [x] Todas las User Stories cubiertas (38/38, ver matriz).
- [x] Todos los Acceptance Criteria cubiertos (caso positivo por AC; negativo donde hay flujo alternativo).
- [x] Casos positivos definidos (TC-*).
- [x] Casos negativos definidos (TCN-* + casos de error de API).
- [x] APIs cubiertas (TCA-* sobre todos los endpoints de la API Spec).

---

# NOTA DE TRAZABILIDAD
Especificación derivada de user-stories.md, acceptance-criteria.md (Gherkin),
backend-specification.md (BR/ERR/validaciones), api-specification.md (endpoints/HTTP) y
frontend-contract.md (pantallas/estados). No introduce requisitos nuevos; verifica los
existentes. Se conserva la corrección heredada FT-037 → FT-038 (US-038 / pedidos de
equipos), validada en TC-038-1, TCN-038-1 y TCI-pedido-equipo.