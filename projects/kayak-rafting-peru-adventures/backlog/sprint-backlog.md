# SPRINT BACKLOG — Rafting Peru & Kayak Adventures
Agile Delivery Lead / Scrum Master · CypData · Junio 2026
Entradas: product-backlog.md · delivery-plan.md · architecture-v1.md · technical-architecture-v1.md · backend-specification.md · api-specification.md · database-design.md · frontend-contract.md · qa-specification.md
Plan de ejecución iterativo. No modifica User Stories, prioridades ni dependencias; las ordena en sprints.

---

## Resumen Ejecutivo

### Objetivo General
Transformar el Product Backlog priorizado (38 User Stories, 9 Epics, releases R1/R2/R3) y el
Delivery Plan (7 entregables E0–E7, 6 fases en 90 días) en una secuencia de sprints ejecutable
por los equipos de desarrollo, que respete el orden del backlog (la captación encabeza por su
mayor valor/menor costo; el motor de reservas-pago cierra por su mayor complejidad y riesgo),
cumpla las dependencias funcionales y técnicas, y entregue valor de forma incremental.

### Estrategia de Iteraciones
- **Sprints de duración uniforme (2 semanas)**, alineados a las fases del Delivery Plan y a las
  ventanas de release de negocio (R1: 0–30 días; R2: 30–60; R3: 60–90).
- **Cimientos primero**: el Entregable 0 (esquema BD por dominio, identity-access, envoltura de
  API + catálogo de errores, comando publicar/despublicar) se construye al inicio del Sprint 1
  como habilitador técnico, ya que desbloquea toda persistencia y todo el back-office.
- **Valor temprano**: R1 (captación, oferta, confianza, ficha local) se entrega en los dos
  primeros sprints, la palanca de conversión de mayor valor y menor costo.
- **Riesgo aislado, no concentrado en exceso**: el bloque transaccional estrictamente secuencial
  (US-030→031→032→033→034) se reserva a un sprint propio (Sprint 5) para contener su riesgo de
  concurrencia/idempotencia; los demás sprints mantienen una mezcla equilibrada de complejidad
  Baja/Media para nivelar la carga.
- **Cada sprint cierra con su capa de QA correspondiente** (TC/TCN/TCA/TCU/TCI del qa-specification),
  no al final del proyecto.

### Número Estimado de Sprints
**6 sprints de 2 semanas (≈ 12 semanas), encajados en la ventana de 90 días del roadmap de negocio.**
El Entregable 0 (cimientos) se integra como trabajo habilitador en la primera semana del Sprint 1.

---

# SPRINT 1

## Objetivo
Levantar los cimientos técnicos transversales y poner en marcha la captación de consultas
multicanal (WhatsApp contextual, formulario y bandeja de leads con estados e idioma). Es la
palanca de conversión de mayor valor y menor costo: el negocio empieza a recibir y gestionar
consultas. Cierra EP-001 completo.

## Historias Incluidas

| Historia | Epic | Prioridad |
| -------- | ---- | --------- |
| US-001 · Iniciar conversación por WhatsApp | EP-001 | 1 |
| US-003 · Enviar consulta mediante formulario | EP-001 | 2 |
| US-002 · Mensaje de WhatsApp precargado por destino | EP-001 | 3 |
| US-004 · Ver consultas centralizadas | EP-001 | 4 |
| US-005 · Registrar y actualizar estado de la consulta | EP-001 | 5 |
| US-006 · Atención en el idioma del visitante | EP-001 | 6 |

## Dependencias
- **Habilitador previo (Entregable 0):** esquema BD por dominio (tipos ENUM, `core.actor_operativo`,
  esquema `captacion`), `identity-access` (JWT por rol), envoltura de respuesta + catálogo de
  errores (API §3/§7) y comando publicar/despublicar (BR-016). Es precondición de toda escritura
  y de la bandeja de back-office.
- US-002 depende de US-001 (deep link base); US-004 de US-001 y US-003 (centraliza ambos canales);
  US-005 y US-006 de US-004. Todas resueltas dentro del mismo sprint por orden de ejecución.

## Riesgos
- **R-F4 · Doble FT-037 en features.md** (Probabilidad Alta): renumerar la última feature de EP-009
  a FT-038 en `features.md` raíz al inicio del proyecto; todas las matrices del pipeline ya usan
  FT-038. Resolverlo aquí evita arrastrar deuda de trazabilidad.
- **R-T2 · Acoplamiento del modular monolith** (Impacto Medio): establecer esquemas por dominio y
  FK lógicas entre dominios desde el día uno (database-design).
- **R-I2 · Indisponibilidad de WhatsApp**: adaptador aislado (`messaging-gateway`) con degradación
  elegante; el deep link no debe bloquear la página.

## Entregables
- Entregable 0 (cimientos) operativo: migraciones base, identity-access, envoltura de API.
- Entregable 1 (captación): `lead-service` (POST/GET/PATCH `/inquiries`), `messaging-gateway`,
  bandeja de leads (P22) con filtros, estados e historial; idioma persistido en la consulta.
- QA: TC-001-1, TC-002-1/2, TC-003-1, TC-004-1/2, TC-005-1/2/3, TC-006-1; negativos
  TCN-001-1, TCN-003-1/2/3, TCN-004-1, TCN-005-1; integración TCI-captacion-form,
  TCI-captacion-whatsapp.

## Criterios de Finalización
- E0 desplegado en staging con migraciones versionadas y reversibles.
- Las 6 US con sus AC positivos y negativos ejecutados (verde).
- Un lead creado por formulario aparece en la bandeja y su transición de estado queda historizada
  (BR-002); idioma conservado (BR-019).
- Endpoints de back-office protegidos por rol (401/403, BR-015).

---

# SPRINT 2

## Objetivo
Comunicar la oferta con transparencia y exhibir las señales de confianza antes de pedir acción,
e incorporar la ficha local por sede (señal de descubrimiento de bajo costo). Con esto se completa
el alcance de **Release 1**.

## Historias Incluidas

| Historia | Epic | Prioridad |
| -------- | ---- | --------- |
| US-007 · Ver paquetes y experiencias por destino y dificultad | EP-002 | 7 |
| US-008 · Ver qué incluye y qué es opcional por paquete | EP-002 | 8 |
| US-010 · Ver precios de referencia y condiciones | EP-002 | 9 |
| US-011 · Ver licencias y certificaciones del operador | EP-003 | 10 |
| US-012 · Conocer al equipo de guías y sus credenciales | EP-003 | 11 |
| US-013 · Conocer protocolos y equipo de seguridad | EP-003 | 12 |
| US-015 · Encontrar respuestas en preguntas frecuentes | EP-003 | 13 |
| US-022 · Encontrar la ficha local por sede (GMB) | EP-005 | 14 |

## Dependencias
- US-008 y US-010 dependen de US-007 (catálogo base); resueltas dentro del sprint por orden.
- US-007 requiere `content-service` + comando publicar/despublicar del Entregable 0 (Sprint 1).
- US-022 requiere `visibility-service`; lectura pública (Q-13) + sincronización por
  GESTOR_OPERACIONES (PATCH `/locations/{id}`, BR-017).

## Riesgos
- **R-F1 · Captación sin contenido suficiente** (Probabilidad Media): sembrar catálogo mínimo
  (4 destinos + experiencias base) y publicar contenido antes de exponerlo; mitiga el riesgo
  abierto en Sprint 1 de formularios listos sobre catálogo vacío.

## Entregables
- `content-service` sirviendo catálogo (`/activities`, `/activities/{id}`), confianza
  (`/trust-center`, `/guides`, `/faqs`) y ficha local (`/locations/{id}`).
- Pantallas P04, P05, P08, P09; sincronización de ficha local (acción back-office).
- QA: TC-007-1/2, TC-008-1, TC-010-1, TC-011-1, TC-012-1, TC-013-1, TC-015-1, TC-022-1/2;
  negativos TCN-007-1, TCN-008-1.

## Criterios de Finalización
- Solo contenido PUBLICADO visible/contratable (BR-016); experiencia no publicada no aparece.
- Componentes marcados de forma inequívoca INCLUIDO/OPCIONAL (BR-004); precio + base de cálculo visibles.
- Credenciales con entidad emisora (RN-16); ficha local refleja la información vigente tras sync (BR-017).
- **Hito: Release 1 completa.**

---

# SPRINT 3

## Objetivo
Ser encontrado y servir contenido de marca: páginas por río, navegación bilingüe, metadatos
estructurados para buscadores/IA, galería, contenido social, blog, reseñas y el comparador.
Entrada a **Release 2**.

## Historias Incluidas

| Historia | Epic | Prioridad |
| -------- | ---- | --------- |
| US-019 · Encontrar página dedicada por río | EP-005 | 15 |
| US-020 · Navegar el sitio en inglés | EP-005 | 16 |
| US-021 · Aparecer en buscadores y asistentes de IA | EP-005 | 17 |
| US-009 · Comparar paquetes | EP-002 | 18 |
| US-016 · Ver fotos y videos reales por río | EP-004 | 19 |
| US-017 · Ver contenido de redes sociales en el sitio | EP-004 | 20 |
| US-018 · Leer guías y artículos (blog) | EP-004 | 21 |
| US-014 · Leer reseñas y testimonios | EP-003 | 22 |

## Dependencias
- US-019 depende de US-007 (Sprint 2). US-020, US-021 y US-016 dependen de US-019; resueltas
  dentro del sprint por orden de ejecución.
- US-009 depende de US-007 y US-008 (Sprint 2).
- US-017, US-018 y US-014 sin dependencias funcionales.

## Riesgos
- **R-T4 · Paridad ES/EN divergente** (Impacto Medio): una sola fuente de verdad por entidad de
  contenido; `lang` transversal verificado en TCI-bilingue.
- **R-T3 · Obsolescencia de metadatos para IA** (Probabilidad Media): mantener `visibility-service`
  desacoplado y actualizable; US-021 es de complejidad **Alta** por la generación de datos
  estructurados.
- **R-F2 · Reseñas escasas al inicio** (Probabilidad Alta): US-014 tolera lista vacía sin error (RN-17).
- **R-I2 · Indisponibilidad de redes sociales**: `social-feed` con `available=false` sin romper la página.

## Entregables
- `visibility-service` (`/seo/metadata`, sitemaps); páginas de destino bilingües (P03);
  galería progresiva, bloque social tolerante a fallos y blog (P11); comparador (P06); reseñas (P10).
- QA: TC-019-1, TC-020-1, TC-021-1, TC-009-1, TC-016-1, TC-017-1, TC-018-1, TC-014-1;
  negativos/UI TCN-009-1, TCU-P11, TCU-P10; integración TCI-bilingue, TCI-publicacion-contenido.

## Criterios de Finalización
- Página de destino con contenido propio + experiencias + vía de contacto; idioma conservado al navegar (BR-019).
- Metadatos estructurados emitidos solo para recursos publicados (BR-016).
- Comparador lado a lado con 2..N ids; galería con carga progresiva no bloqueante.
- **Hito: descubrimiento y contenido publicados.**

---

# SPRINT 4

## Objetivo
Segmentar la oferta por perfil de cliente y abrir el canal institucional B2B (cotización de grupos,
propuesta de valor corporativa y material descargable). Con esto se completa **Release 2**.

## Historias Incluidas

| Historia | Epic | Prioridad |
| -------- | ---- | --------- |
| US-023 · Explorar la oferta filtrada por perfil | EP-006 | 23 |
| US-024 · Conocer edad mínima y nivel de esfuerzo | EP-006 | 24 |
| US-025 · Recibir recomendaciones por perfil | EP-006 | 25 |
| US-026 · Solicitar cotización para grupos | EP-007 | 26 |
| US-027 · Registrar y responder cotizaciones | EP-007 | 27 |
| US-028 · Conocer la propuesta de valor corporativa | EP-007 | 28 |
| US-029 · Descargar material informativo B2B | EP-007 | 29 |

## Dependencias
- US-023 y US-024 dependen de US-007 (Sprint 2); US-025 de US-023 y US-024 (mismo sprint, por orden).
- US-027 depende de US-026 (solicitudes B2B registradas); resuelto dentro del sprint.
- US-028, US-029 sin dependencias funcionales.

## Riesgos
- Carga equilibrada (4 Media + 3 Baja) sin riesgos técnicos altos; sprint de nivelación previo al
  bloque transaccional. Atención a la consistencia del **material B2B vigente** (US-029 AC-002).

## Entregables
- Filtro y recomendación por perfil (`/activities?perfilCliente`, `/activities/recommendations`, P07).
- `quote-service` (POST `/quote-requests`, gestión en P23 por RESPONSABLE_COMERCIAL); contenido
  corporativo y material (`/corporate`, `/corporate/material`, P12, P13).
- QA: TC-023-1, TC-024-1, TC-025-1, TC-026-1, TC-027-1/2, TC-028-1, TC-029-1; negativos
  TCN-026-1, TCN-027-1, TCU-P07; integración TCI-cotizacion-b2b.

## Criterios de Finalización
- Solo experiencias aptas por perfil; requisitos (edad/esfuerzo) visibles.
- Solicitud B2B en estado RECIBIDA con confirmación (BR-011); cotización asociada y estado
  COTIZACION_EMITIDA historizado (BR-012).
- **Hito: Release 2 completa.**

---

# SPRINT 5

## Objetivo
Automatizar la conversión con el motor transaccional completo: disponibilidad → reserva → pago →
confirmación, más la gestión operativa de reservas. Es el bloque de mayor costo y riesgo (cadena
estrictamente secuencial), aislado en su propio sprint. Núcleo de **Release 3**.

## Historias Incluidas

| Historia | Epic | Prioridad |
| -------- | ---- | --------- |
| US-030 · Consultar disponibilidad y calendario | EP-008 | 30 |
| US-031 · Reservar en línea | EP-008 | 31 |
| US-032 · Pagar la reserva en línea | EP-008 | 32 |
| US-033 · Recibir confirmación automática de reserva | EP-008 | 33 |
| US-034 · Administrar reservas | EP-008 | 34 |

## Dependencias
- US-030 depende de US-007 (catálogo, Sprint 2). Cadena interna estrictamente secuencial:
  US-031 (de US-030) → US-032 (de US-031) → US-033 (de US-032). US-034 depende de US-031.
  **No se paraleliza dentro del bloque.**
- `booking-service` (esquema `reservas` + bloqueo de fila), `payment-service` + webhook idempotente,
  `notification-service` e `integration-adapters` (pasarela), todos sobre el Entregable 0.

## Riesgos
> Concentración deliberada del riesgo transaccional en este sprint; se le da holgura por la
> complejidad **Muy Alta** de US-032 y **Alta** de US-031.
- **R-T1 · Sobreventa de cupo bajo concurrencia** (Impacto Alto): `SELECT FOR UPDATE` sobre
  disponibilidad + índice único `(experiencia_id, fecha)`; TCI-reserva-concurrencia obligatorio.
- **R-I1 · Webhook de pago perdido/duplicado** (Impacto Alto): idempotencia por
  `referencia_pasarela` (índice único, BR-020) + reintentos + conciliación; TCI-webhook-idempotente.
- **R-I3 · Firma de webhook no verificable** (Impacto Alto): validación de firma (ERR-031);
  rechazar evento no verificable (TCN-032-3).
- **R-I4 · Cambios de API de la pasarela**: contrato interno estable en `integration-adapters`.

## Entregables
- Flujo de reserva-pago-confirmación (P18→P19→P20→P21) y gestión de reservas (P24).
- Datos de pago tokenizados (sin datos de tarjeta en la plataforma).
- QA: TC-030-1, TC-031-1, TC-032-1, TC-033-1, TC-034-1; negativos TCN-030-1, TCN-031-1/2/3,
  TCN-032-1/2/3, TCN-034-1; integración TCI-reserva-feliz, TCI-reserva-pago-fallido,
  TCI-reserva-concurrencia, TCI-webhook-idempotente.

## Criterios de Finalización
- Reserva CONFIRMADA solo con pago EXITOSO (BR-007); pago fallido deja NO_CONFIRMADA y libera cupo.
- **0 defectos críticos abiertos en el flujo de reserva-pago** (criterio de salida del qa-specification).
- Concurrencia y webhook duplicado verificados sin sobreventa ni doble efecto.
- **Hito: motor de reservas-pago operativo.**

---

# SPRINT 6

## Objetivo
Comercializar la línea de equipos: catálogo segmentado por tipo de cliente, solicitud/pedido bajo
importación y seguimiento de estado en back-office. Cierra el backlog y completa **Release 3**.

## Historias Incluidas

| Historia | Epic | Prioridad |
| -------- | ---- | --------- |
| US-035 · Ver el catálogo de equipos | EP-009 | 35 |
| US-036 · Solicitar cotización o pedido de equipos | EP-009 | 36 |
| US-037 · Ver oferta de equipos por tipo de cliente | EP-009 | 37 |
| US-038 · Registrar y dar seguimiento a pedidos de equipos | EP-009 | 38 |

## Dependencias
- US-036 y US-037 dependen de US-035 (catálogo); US-038 de US-036 (seguimiento sobre pedidos
  registrados). Todas resueltas dentro del sprint por orden de ejecución.
- `equipment-service` sobre el Entregable 0.

## Riesgos
- **R-F3 · Equipos sin estudio de mercado previo** (línea P3, Impacto Medio): validar demanda antes
  de invertir en el catálogo; dominio desacoplado del negocio turístico, por eso va al cierre.

## Entregables
- `equipment-service` (`/equipment`, POST `/equipment-orders`, gestión en P25 por GESTOR_OPERACIONES).
- Catálogo (P14/P15) con condición de venta bajo pedido/importación; formulario de pedido (P16).
- QA: TC-035-1, TC-036-1, TC-037-1, TC-038-1; negativos TCN-036-1, TCN-038-1; integración
  TCI-pedido-equipo.

## Criterios de Finalización
- Si no hay oferta específica por tipo de cliente, se muestra la oferta general (BR-018).
- Pedido en estado RECIBIDO con `sujetoImportacion=true` (BR-014); avance de estado historizado (BR-002).
- **Hito: Release 3 completa — fin del proyecto.**

---

# HITOS DEL PROYECTO

| Hito | Sprint |
| ---- | ------ |
| H1 · Cimientos técnicos operativos (Entregable 0: BD, identidad, envoltura API, publicación) | Sprint 1 |
| H2 · Captación de consultas activa (EP-001 completo) | Sprint 1 |
| H3 · Release 1 completa (oferta, confianza y ficha local) | Sprint 2 |
| H4 · Descubrimiento y contenido de marca publicados (bilingüe + metadatos IA) | Sprint 3 |
| H5 · Release 2 completa (segmentación por perfil + canal B2B) | Sprint 4 |
| H6 · Motor de reservas-pago-confirmación operativo (0 críticos abiertos) | Sprint 5 |
| H7 · Release 3 completa (comercialización de equipos) — cierre del proyecto | Sprint 6 |

---

# ROADMAP DE SPRINTS

| Sprint | Objetivo | Entregables |
| ------ | -------- | ----------- |
| Sprint 1 (sem. 1-2) | Cimientos + captación de consultas | E0 (cimientos) + E1 (captación EP-001) |
| Sprint 2 (sem. 3-4) | Oferta, confianza y ficha local — completa R1 | E2 (oferta + confianza) + US-022 de E3 |
| Sprint 3 (sem. 5-6) | Descubrimiento y contenido de marca — entrada a R2 | E3 (US-019/020/021) + parte de E4 (US-009/014/016/017/018) |
| Sprint 4 (sem. 7-8) | Segmentación por perfil y canal B2B — completa R2 | Resto de E4 (US-023/024/025) + E5 (B2B) |
| Sprint 5 (sem. 9-10) | Reserva-pago-confirmación (transaccional) | E6 (reservas y pago) |
| Sprint 6 (sem. 11-12) | Comercialización de equipos — completa R3 | E7 (equipos) |

---

# DISTRIBUCIÓN DE RIESGOS

| Riesgo | Sprint |
| ------ | ------ |
| R-F4 · Doble FT-037 en features.md (renumerar a FT-038) | Sprint 1 |
| R-T2 · Acoplamiento del modular monolith (límites de dominio desde el inicio) | Sprint 1 |
| R-I2 · Indisponibilidad de WhatsApp / redes sociales (adaptadores aislados, degradación) | Sprint 1 / Sprint 3 |
| R-F1 · Captación sin contenido suficiente (seed de catálogo, publicar antes de exponer) | Sprint 2 |
| R-T4 · Paridad ES/EN divergente (una fuente de verdad por entidad) | Sprint 3 |
| R-T3 · Obsolescencia de metadatos para IA (visibility-service desacoplado) | Sprint 3 |
| R-F2 · Reseñas escasas al inicio (ausencia no es error, RN-17) | Sprint 3 |
| R-T1 · Sobreventa de cupo bajo concurrencia (SELECT FOR UPDATE + índice único) | Sprint 5 |
| R-I1 · Webhook de pago perdido/duplicado (idempotencia BR-020 + conciliación) | Sprint 5 |
| R-I3 · Firma de webhook no verificable (validación de firma, ERR-031) | Sprint 5 |
| R-I4 · Cambios de API de proveedores externos (contratos internos estables) | Sprint 5 |
| R-F3 · Equipos sin estudio de mercado previo (validar demanda antes de invertir) | Sprint 6 |

---

# MATRIZ DE TRAZABILIDAD

| Epic | Feature | Historia | Sprint |
| ---- | ------- | -------- | ------ |
| EP-001 | FT-001 | US-001 | Sprint 1 |
| EP-001 | FT-001 | US-002 | Sprint 1 |
| EP-001 | FT-002 | US-003 | Sprint 1 |
| EP-001 | FT-003 | US-004 | Sprint 1 |
| EP-001 | FT-004 | US-005 | Sprint 1 |
| EP-001 | FT-005 | US-006 | Sprint 1 |
| EP-002 | FT-006 | US-007 | Sprint 2 |
| EP-002 | FT-007 | US-008 | Sprint 2 |
| EP-002 | FT-009 | US-010 | Sprint 2 |
| EP-003 | FT-010 | US-011 | Sprint 2 |
| EP-003 | FT-011 | US-012 | Sprint 2 |
| EP-003 | FT-012 | US-013 | Sprint 2 |
| EP-003 | FT-014 | US-015 | Sprint 2 |
| EP-005 | FT-021 | US-022 | Sprint 2 |
| EP-005 | FT-018 | US-019 | Sprint 3 |
| EP-005 | FT-019 | US-020 | Sprint 3 |
| EP-005 | FT-020 | US-021 | Sprint 3 |
| EP-002 | FT-008 | US-009 | Sprint 3 |
| EP-004 | FT-015 | US-016 | Sprint 3 |
| EP-004 | FT-016 | US-017 | Sprint 3 |
| EP-004 | FT-017 | US-018 | Sprint 3 |
| EP-003 | FT-013 | US-014 | Sprint 3 |
| EP-006 | FT-022 | US-023 | Sprint 4 |
| EP-006 | FT-023 | US-024 | Sprint 4 |
| EP-006 | FT-024 | US-025 | Sprint 4 |
| EP-007 | FT-025 | US-026 | Sprint 4 |
| EP-007 | FT-026 | US-027 | Sprint 4 |
| EP-007 | FT-027 | US-028 | Sprint 4 |
| EP-007 | FT-028 | US-029 | Sprint 4 |
| EP-008 | FT-029 | US-030 | Sprint 5 |
| EP-008 | FT-030 | US-031 | Sprint 5 |
| EP-008 | FT-031 | US-032 | Sprint 5 |
| EP-008 | FT-032 | US-033 | Sprint 5 |
| EP-008 | FT-033 | US-034 | Sprint 5 |
| EP-009 | FT-034 | US-035 | Sprint 6 |
| EP-009 | FT-035 | US-036 | Sprint 6 |
| EP-009 | FT-036 | US-037 | Sprint 6 |
| EP-009 | FT-038 | US-038 | Sprint 6 |

Cobertura: 38/38 User Stories · 9/9 Epics · 37/37 Features asignadas a un sprint.

---

# CHECKLIST DE VALIDACIÓN

- [x] Todas las historias fueron asignadas (38/38 en la matriz de trazabilidad).
- [x] Todas las dependencias fueron respetadas (ninguna US precede a aquellas de las que depende, ni entre sprints ni dentro del sprint por orden de ejecución).
- [x] Existe cobertura completa del Product Backlog (las 38 US, en orden de prioridad 1–38).
- [x] Existe roadmap de sprints (6 sprints de 2 semanas, alineados a fases y releases).
- [x] Existen hitos definidos (H1–H7).
- [x] Existe trazabilidad completa (Epic → Feature → Historia → Sprint).
- [x] No existen sprints vacíos (cada sprint contiene User Stories; el Entregable 0 es habilitador integrado en Sprint 1).

---

# NOTA DE TRAZABILIDAD
Plan derivado del product-backlog (orden y releases R1/R2/R3) y del delivery-plan (entregables
E0–E7, dependencias, riesgos y estimación de complejidad), consistente con la arquitectura
(dominios/módulos), las specs técnicas (componentes, esquema, contrato API/Frontend) y el
qa-specification (puntos de control por sprint). No modifica User Stories, prioridades ni
dependencias; las agrupa en sprints de duración uniforme respetando el orden del backlog. El
bloque transaccional secuencial (US-030→034) se aísla en el Sprint 5 para contener su riesgo.
Se conserva la corrección heredada FT-037 → FT-038 (US-038 / gestión de pedidos de equipos),
cuya resolución en `features.md` raíz se programa en el Sprint 1 (riesgo R-F4).