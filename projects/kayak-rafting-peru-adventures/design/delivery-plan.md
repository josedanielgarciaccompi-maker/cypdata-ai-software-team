# DELIVERY PLAN — Rafting Peru & Kayak Adventures
Delivery Manager · CypData · Junio 2026
Entradas: product-backlog.md · architecture-v1.md · technical-architecture-v1.md · backend-specification.md · api-specification.md · database-design.md · frontend-contract.md · qa-specification.md
Plan de ejecución (sin modificar US ni prioridades). Puente entre product-backlog y sprint-backlog.

---

# DELIVERY PLAN

## Resumen Ejecutivo

### Objetivo del Delivery
Convertir el product-backlog priorizado (38 US, 9 Epics, releases R1/R2/R3) en un plan de
ejecución realista y trazable que respete las dependencias funcionales y técnicas, gestione
los riesgos y permita entregas incrementales de valor. El plan conserva el orden del backlog
(la captación encabeza por su mayor valor/menor costo; el motor de reservas-pagos cierra por
su mayor complejidad) y alimenta directamente al Sprint Planner.

### Estrategia de Ejecución
Entrega incremental por release alineada al roadmap de negocio (0-30 / 30-60 / 60-90 días),
sobre un backend modular (modular monolith) que se construye dominio por dominio. Se prioriza
levantar primero la "plomería" transversal (identidad, contenido publicable, envoltura de
API, esquema de BD base) para no bloquear las historias de captación, y se deja el flujo
transaccional (reserva-pago-confirmación) como el último bloque por su acoplamiento secuencial
y su riesgo de concurrencia/idempotencia. Cada entregable se cierra con su capa de QA
correspondiente (los TC/TCN/TCA/TCI del qa-specification), no al final.

### Supuestos
- El equipo opera con presupuesto limitado; se favorece construir una capacidad transversal una vez y reutilizarla (formularios, envoltura de error, listados con filtros).
- La capa pública (contenido, confianza, captación) aporta valor antes de que exista el motor de reservas.
- Integraciones externas (WhatsApp, pasarela, fichas locales) se consumen vía adaptadores; su disponibilidad es un supuesto de entorno.
- El stack es el de la arquitectura técnica (Next.js/React/TS, Node/NestJS, PostgreSQL); no se replantea aquí.
- La numeración FT-038 (pedidos de equipos) se da por corregida en las matrices; el features.md raíz aún conserva el doble FT-037 (ver riesgo R-F4).

### Restricciones
- No se modifican User Stories ni las prioridades del product-backlog.
- No se introducen funcionalidades nuevas; todo deriva de los artefactos existentes.
- El bloque de reservas es estrictamente secuencial (US-030→031→032→033); no se paraleliza dentro de sí mismo.
- Los datos de pago no residen en la plataforma (tokenización); condiciona el diseño del entregable transaccional.

---

# ANÁLISIS DE DEPENDENCIAS

## Dependencias Funcionales

| Historia | Depende de | Justificación |
| -------- | ---------- | ------------- |
| US-002 | US-001 | El mensaje contextual de WhatsApp requiere primero el acceso/deep link base. |
| US-004 | US-001, US-003 | La bandeja centraliza leads que provienen de WhatsApp y del formulario. |
| US-005 | US-004 | El cambio de estado opera sobre las consultas listadas en la bandeja. |
| US-006 | US-004 | La atención en idioma se refleja sobre el lead ya capturado/listado. |
| US-008 | US-007 | El detalle incluido/opcional cuelga del catálogo de experiencias. |
| US-010 | US-007 | El precio/condiciones se muestra sobre la experiencia del catálogo. |
| US-009 | US-007, US-008 | Comparar requiere catálogo y detalle de componentes. |
| US-016 | US-019 | La galería por río cuelga de la página de destino. |
| US-020 | US-019 | El soporte EN se aplica sobre las páginas de destino/contenido ya existentes. |
| US-021 | US-019 | Los metadatos estructurados describen páginas de destino publicadas. |
| US-023 | US-007 | El filtro por perfil opera sobre el catálogo de experiencias. |
| US-024 | US-007 | Los requisitos (edad/esfuerzo) se muestran en el detalle de la experiencia. |
| US-025 | US-023, US-024 | La recomendación combina perfil y requisitos. |
| US-027 | US-026 | Responder cotizaciones requiere solicitudes B2B registradas. |
| US-031 | US-030 | Reservar requiere consultar disponibilidad/cupo primero. |
| US-032 | US-031 | Pagar requiere una reserva en estado INICIADA. |
| US-033 | US-032 | La confirmación automática se emite tras el pago exitoso. |
| US-034 | US-031 | La gestión de reservas opera sobre reservas existentes. |
| US-036 | US-035 | Solicitar pedido requiere el catálogo de equipos. |
| US-037 | US-035 | La segmentación por tipo de cliente opera sobre el catálogo. |
| US-038 | US-036 | El seguimiento de importación opera sobre pedidos registrados. |

## Dependencias Técnicas

| Componente | Dependencia | Impacto |
| ---------- | ----------- | ------- |
| Esquema BD base (tipos ENUM, core.actor_operativo, contenido.*) | Migraciones en orden de FK (database-design §Migraciones) | Bloquea cualquier persistencia; debe existir antes de la primera US con escritura. |
| identity-access (JWT por rol) | — (componente transversal) | Bloquea todo el back-office (US-004,005,027,034,038) y el flujo CLIENT de reserva. |
| Envoltura de respuesta + catálogo de errores (API §3/§7) | — (contrato transversal) | Todas las pantallas dependen del mapeo de error uniforme (frontend-contract). |
| content-service + comando publicar/despublicar (C-12/C-13) | Esquema contenido + estado_publicacion | Habilita BR-016 (solo PUBLICADO visible); precondición de todo el contenido público. |
| lead-service (POST /inquiries, GET/PATCH /inquiries) | Esquema captacion + identity-access (para BO) | Soporta el bloque de captación completo (EP-001). |
| booking-service (disponibilidad, reserva) | Esquema reservas + bloqueo de fila (SELECT FOR UPDATE) | Núcleo transaccional; depende de la consistencia de cupo del database-design. |
| payment-service + webhook idempotente | integration-adapters (pasarela) + índice único referencia_pasarela | Cierra el flujo de reserva; depende de la idempotencia a nivel de BD (BR-020). |
| notification-service | cola/eventos asíncronos | Emite confirmaciones (US-033); desacoplado, no bloquea el camino síncrono. |
| messaging-gateway (WhatsApp) | integration-adapters | Soporta US-001/002 (deep link); degradable. |
| visibility-service (seo/metadata, fichas locales) | content-service publicado | Soporta US-021/022; consume contenido ya publicado. |

---

# ANÁLISIS DE RIESGOS

## Riesgos Funcionales

| Riesgo | Impacto | Probabilidad | Mitigación |
| ------ | ------- | ------------ | ---------- |
| R-F1 · Captación sin contenido suficiente (formularios listos pero catálogo vacío) reduce conversión | Medio | Media | Sembrar catálogo mínimo (4 destinos + experiencias base) en R1; publicar contenido antes de exponer captación. |
| R-F2 · Reseñas escasas al inicio debilitan la señal de confianza | Bajo | Alta | US-014 ya está en R2 (depende de acumular testimonios); en R1 se cubren licencias/guías/protocolos/FAQ. |
| R-F3 · Equipos sin estudio de mercado previo (línea P3) | Medio | Media | Mantener EP-009 al cierre (R3); validar demanda antes de invertir en el catálogo. |
| R-F4 · Doble FT-037 en features.md raíz rompe trazabilidad de auditoría | Bajo | Alta | Renumerar la última feature de EP-009 a FT-038 en features.md (todas las matrices ya lo aplican). |

## Riesgos Técnicos

| Riesgo | Impacto | Probabilidad | Mitigación |
| ------ | ------- | ------------ | ---------- |
| R-T1 · Sobreventa de cupo bajo concurrencia | Alto | Media | SELECT FOR UPDATE sobre disponibilidad + índice único (experiencia_id,fecha); caso TCI-reserva-concurrencia obligatorio antes de release. |
| R-T2 · Acoplamiento del modular monolith degrada la futura extracción de servicios | Medio | Media | Esquemas por dominio + FK lógicas entre dominios desde el día uno (database-design). |
| R-T3 · Generación de metadatos para IA se vuelve obsoleta | Bajo | Media | Mantener visibility-service desacoplado y actualizable; no acoplar a un formato fijo. |
| R-T4 · Paridad ES/EN divergente | Medio | Media | Una fuente de verdad por entidad de contenido; lang transversal verificado en TCI-bilingue. |

## Riesgos de Integración

| Riesgo | Impacto | Probabilidad | Mitigación |
| ------ | ------- | ------------ | ---------- |
| R-I1 · Webhook de pago perdido/duplicado deja reservas inconsistentes | Alto | Media | Idempotencia por referencia_pasarela (índice único, BR-020) + reintentos + conciliación; TCI-webhook-idempotente. |
| R-I2 · Indisponibilidad de WhatsApp/pasarela/fichas locales | Medio | Media | Adaptadores aislados (integration-adapters); degradación elegante (social-feed available=false). |
| R-I3 · Firma de webhook no verificable | Alto | Baja | Validación de firma (ERR-031); rechazar evento no verificable; TCN-032-3. |
| R-I4 · Cambios de API de proveedores externos | Medio | Media | Contratos internos estables por adaptador; aislar el detalle del proveedor. |

---

# ESTIMACIÓN DE COMPLEJIDAD

| Historia | Complejidad |
| -------- | ----------- |
| US-001 | Baja |
| US-002 | Baja |
| US-003 | Media |
| US-004 | Media |
| US-005 | Media |
| US-006 | Baja |
| US-007 | Media |
| US-008 | Baja |
| US-009 | Media |
| US-010 | Baja |
| US-011 | Baja |
| US-012 | Baja |
| US-013 | Baja |
| US-014 | Baja |
| US-015 | Media |
| US-016 | Media |
| US-017 | Media |
| US-018 | Baja |
| US-019 | Media |
| US-020 | Media |
| US-021 | Alta |
| US-022 | Media |
| US-023 | Media |
| US-024 | Baja |
| US-025 | Media |
| US-026 | Media |
| US-027 | Media |
| US-028 | Baja |
| US-029 | Baja |
| US-030 | Media |
| US-031 | Alta |
| US-032 | Muy Alta |
| US-033 | Media |
| US-034 | Media |
| US-035 | Baja |
| US-036 | Media |
| US-037 | Media |
| US-038 | Media |

> Notas: US-032 (pago) es Muy Alta por la integración con pasarela, el webhook idempotente y
> la integridad reserva-pago bajo concurrencia. US-031 y US-021 son Alta por el control de
> cupo transaccional y la generación de metadatos estructurados respectivamente.

---

# AGRUPACIÓN DE ENTREGABLES

## Entregable 0 — Cimientos (habilitador, no es US)
### Objetivo: Levantar la base que desbloquea todo: esquema BD por dominio (ENUM, core, contenido), identity-access (JWT por rol), envoltura de API + catálogo de errores, y el comando publicar/despublicar (BR-016).
### Historias Incluidas: — (habilitador técnico transversal)
### Dependencias: Ninguna; es precondición de los demás entregables.

## Entregable 1 — Captación
### Objetivo: Convertir tráfico en consultas: WhatsApp contextual, formulario de consulta y bandeja de leads con estados e idioma.
### Historias Incluidas: US-001, US-003, US-002, US-004, US-005, US-006.
### Dependencias: Entregable 0 (identity-access para la bandeja; lead-service; esquema captacion).

## Entregable 2 — Oferta y Confianza
### Objetivo: Comunicar la oferta con transparencia y exhibir señales de confianza antes de pedir acción.
### Historias Incluidas: US-007, US-008, US-010, US-011, US-012, US-013, US-015.
### Dependencias: Entregable 0 (contenido publicable). US-008/US-010 dependen de US-007.

## Entregable 3 — Descubrimiento y ficha local
### Objetivo: Ser encontrado: páginas por río, bilingüe, metadatos para buscadores/IA y ficha local por sede.
### Historias Incluidas: US-022, US-019, US-020, US-021.
### Dependencias: Entregable 2 (US-019 depende de US-007); visibility-service.

## Entregable 4 — Contenido de marca y segmentación
### Objetivo: Atraer con contenido (galería, social, blog, reseñas) y segmentar la oferta por perfil.
### Historias Incluidas: US-009, US-016, US-017, US-018, US-014, US-023, US-024, US-025.
### Dependencias: Entregables 2 y 3 (US-016 depende de US-019; US-023/024/025 de US-007).

## Entregable 5 — Canal B2B
### Objetivo: Atender grupos y corporativos: cotización, propuesta de valor y material descargable.
### Historias Incluidas: US-026, US-027, US-028, US-029.
### Dependencias: Entregable 0 (quote-service; BO para US-027 depende de US-026).

## Entregable 6 — Reservas y pago (transaccional)
### Objetivo: Automatizar la conversión: disponibilidad → reserva → pago → confirmación, y gestión de reservas.
### Historias Incluidas: US-030, US-031, US-032, US-033, US-034.
### Dependencias: Entregable 0 + Entregable 2 (catálogo); cadena secuencial interna; integration-adapters (pasarela).

## Entregable 7 — Comercialización de equipos
### Objetivo: Catálogo y pedidos de equipos bajo importación con seguimiento.
### Historias Incluidas: US-035, US-036, US-037, US-038.
### Dependencias: Entregable 0 (equipment-service); US-036/037 dependen de US-035; US-038 de US-036.

---

# PROPUESTA DE RELEASES

## Release 1 (0-30 días) — Capturar consultas
### Alcance: Entregable 0 (cimientos) + Entregable 1 (captación) + Entregable 2 (oferta y confianza) + la ficha local (US-022) del Entregable 3.
### Valor Entregado: El negocio empieza a recibir y gestionar consultas con oferta transparente y señales de confianza — la palanca de conversión de mayor valor y menor costo. Coincide con R1 del product-backlog.
### Riesgos: R-F1 (catálogo vacío) y R-T2 (límites de dominio); mitigados con seed de contenido y esquemas por dominio desde el inicio.

## Release 2 (30-60 días) — Atraer y convertir tráfico
### Alcance: Resto del Entregable 3 (US-019, US-020, US-021) + Entregable 4 (contenido y segmentación) + Entregable 5 (B2B).
### Valor Entregado: Tráfico cualificado vía descubrimiento (SEO/GEO, bilingüe), contenido que motiva, segmentación por perfil y canal institucional. Coincide con R2.
### Riesgos: R-T1 paridad ES/EN, R-T3 obsolescencia de metadatos; mitigados con fuente de verdad por entidad y visibility-service desacoplado.

## Release 3 (60-90 días) — Automatizar y escalar
### Alcance: Entregable 6 (reservas y pago) + Entregable 7 (equipos).
### Valor Entregado: Reserva y pago en línea (automatiza la conversión) y comercialización de equipos. Es el bloque de mayor costo y riesgo, por eso va al final. Coincide con R3.
### Riesgos: R-T1 (sobreventa), R-I1 (webhook), R-I3 (firma); mitigados con bloqueo de fila, idempotencia por BD y validación de firma, todos con casos E2E obligatorios.

---

# ROADMAP DE EJECUCIÓN

| Fase | Objetivo | Entregables |
| ---- | -------- | ----------- |
| Fase 0 (días 0-7) | Cimientos técnicos que desbloquean todo | Entregable 0 (BD, identidad, envoltura API, publicar contenido) |
| Fase 1 (días 7-30) | Capturar consultas con oferta y confianza | Entregables 1 y 2 + US-022 |
| Fase 2 (días 30-45) | Ser encontrado y servir contenido | Entregable 3 (US-019/020/021) |
| Fase 3 (días 45-60) | Atraer, segmentar y abrir B2B | Entregables 4 y 5 |
| Fase 4 (días 60-80) | Reserva-pago-confirmación | Entregable 6 (secuencial) |
| Fase 5 (días 80-90) | Comercialización de equipos | Entregable 7 |

---

# PREPARACIÓN PARA SPRINT PLANNING

## Historias Candidatas Sprint 1
Foco: cimientos + captación. US-001, US-003, US-002, US-004, US-005, US-006. (Habilitador previo: Entregable 0 — esquema BD, identity-access, envoltura API, publicar contenido.)

## Historias Candidatas Sprint 2
Foco: oferta, confianza y ficha local. US-007, US-008, US-010, US-011, US-012, US-013, US-015, US-022.

## Historias Candidatas Sprint 3
Foco: descubrimiento, contenido, segmentación y B2B (entrada a R2). US-019, US-020, US-021, US-009, US-016, US-017, US-018, US-014, US-023, US-024, US-025, US-026, US-027, US-028, US-029.

> El bloque transaccional (US-030→034) y equipos (US-035→038) se planifican en sprints
> posteriores (R3), respetando la secuencia estricta de reservas y dejando US-032 (Muy Alta)
> con holgura por su riesgo de integración.

---

# MATRIZ DE TRAZABILIDAD

| Epic | Feature | Historia | Entregable | Release |
| ---- | ------- | -------- | ---------- | ------- |
| EP-001 | FT-001 | US-001 | E1 | R1 |
| EP-001 | FT-001 | US-002 | E1 | R1 |
| EP-001 | FT-002 | US-003 | E1 | R1 |
| EP-001 | FT-003 | US-004 | E1 | R1 |
| EP-001 | FT-004 | US-005 | E1 | R1 |
| EP-001 | FT-005 | US-006 | E1 | R1 |
| EP-002 | FT-006 | US-007 | E2 | R1 |
| EP-002 | FT-007 | US-008 | E2 | R1 |
| EP-002 | FT-008 | US-009 | E4 | R2 |
| EP-002 | FT-009 | US-010 | E2 | R1 |
| EP-003 | FT-010 | US-011 | E2 | R1 |
| EP-003 | FT-011 | US-012 | E2 | R1 |
| EP-003 | FT-012 | US-013 | E2 | R1 |
| EP-003 | FT-013 | US-014 | E4 | R2 |
| EP-003 | FT-014 | US-015 | E2 | R1 |
| EP-004 | FT-015 | US-016 | E4 | R2 |
| EP-004 | FT-016 | US-017 | E4 | R2 |
| EP-004 | FT-017 | US-018 | E4 | R2 |
| EP-005 | FT-018 | US-019 | E3 | R2 |
| EP-005 | FT-019 | US-020 | E3 | R2 |
| EP-005 | FT-020 | US-021 | E3 | R2 |
| EP-005 | FT-021 | US-022 | E3 | R1 |
| EP-006 | FT-022 | US-023 | E4 | R2 |
| EP-006 | FT-023 | US-024 | E4 | R2 |
| EP-006 | FT-024 | US-025 | E4 | R2 |
| EP-007 | FT-025 | US-026 | E5 | R2 |
| EP-007 | FT-026 | US-027 | E5 | R2 |
| EP-007 | FT-027 | US-028 | E5 | R2 |
| EP-007 | FT-028 | US-029 | E5 | R2 |
| EP-008 | FT-029 | US-030 | E6 | R3 |
| EP-008 | FT-030 | US-031 | E6 | R3 |
| EP-008 | FT-031 | US-032 | E6 | R3 |
| EP-008 | FT-032 | US-033 | E6 | R3 |
| EP-008 | FT-033 | US-034 | E6 | R3 |
| EP-009 | FT-034 | US-035 | E7 | R3 |
| EP-009 | FT-035 | US-036 | E7 | R3 |
| EP-009 | FT-036 | US-037 | E7 | R3 |
| EP-009 | FT-038 | US-038 | E7 | R3 |

> Nota: US-022 (ficha local) pertenece al Entregable 3 por afinidad técnica (visibility-service)
> pero se entrega en R1 porque el product-backlog la prioriza en R1 (señal de descubrimiento
> de bajo costo). El resto del Entregable 3 (US-019/020/021) va en R2 conforme al backlog.

---

# CHECKLIST DE VALIDACIÓN
- [x] Todas las historias fueron consideradas (38/38 en complejidad, dependencias y trazabilidad).
- [x] Todas las dependencias fueron identificadas (funcionales y técnicas).
- [x] Todos los riesgos tienen mitigación (funcionales, técnicos, integración).
- [x] Existe roadmap (6 fases, días 0-90).
- [x] Existen releases (R1/R2/R3 alineadas al product-backlog).
- [x] Existe trazabilidad completa (Epic → Feature → Historia → Entregable → Release).

---

# NOTA DE TRAZABILIDAD
Plan derivado del product-backlog (orden y releases), la arquitectura (dominios/módulos),
las specs técnicas (componentes, esquema, contrato API/Frontend) y el qa-specification (puntos
de control por entregable). No modifica User Stories ni prioridades; reordena en entregables y
fases respetando el orden del backlog. La única reubicación respecto al orden lineal del backlog
es agrupar US-022 en el Entregable 3 por afinidad técnica, conservando su release R1. Se mantiene
la recomendación de renumerar el doble FT-037 a FT-038 en features.md (riesgo R-F4); todas las
matrices de este plan ya usan FT-038 para US-038.