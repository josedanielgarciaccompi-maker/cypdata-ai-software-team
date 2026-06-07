# PRODUCT BACKLOG — Rafting Peru & Kayak Adventures
Product Manager · CypData · Junio 2026
Entradas: architecture-v1.md · user-stories.md · acceptance-criteria.md
Backlog priorizado con dependencias y releases. Sin diseño técnico, desarrollo, diseño UI ni arquitectura.

## Criterio de priorización
Hereda la lógica del roadmap de negocio (presupuesto limitado; objetivo: maximizar
consultas y reservas) y respeta las dependencias funcionales de la arquitectura.
La "Prioridad" es el orden del backlog (1 = más alta).

## Releases
- R1 · Capturar consultas (0–30 días): captación, transparencia de oferta, confianza y ficha local.
- R2 · Atraer y convertir tráfico (30–60 días): visibilidad/GEO, contenido, segmentación y canal B2B.
- R3 · Automatizar y escalar (60–90 días): reservas y pago en línea, y comercialización de equipos.

## Product Backlog priorizado

| Prioridad | Story | Epic | Dependencias | Release |
| --------- | ----- | ---- | ------------ | ------- |
| 1 | US-001 · Iniciar conversación por WhatsApp | EP-001 | — | R1 |
| 2 | US-003 · Enviar consulta mediante formulario | EP-001 | — | R1 |
| 3 | US-002 · Mensaje de WhatsApp precargado por destino | EP-001 | US-001 | R1 |
| 4 | US-004 · Ver consultas centralizadas | EP-001 | US-001, US-003 | R1 |
| 5 | US-005 · Registrar y actualizar estado de la consulta | EP-001 | US-004 | R1 |
| 6 | US-006 · Atención en el idioma del visitante | EP-001 | US-004 | R1 |
| 7 | US-007 · Ver paquetes y experiencias por destino y dificultad | EP-002 | — | R1 |
| 8 | US-008 · Ver qué incluye y qué es opcional por paquete | EP-002 | US-007 | R1 |
| 9 | US-010 · Ver precios de referencia y condiciones | EP-002 | US-007 | R1 |
| 10 | US-011 · Ver licencias y certificaciones del operador | EP-003 | — | R1 |
| 11 | US-012 · Conocer al equipo de guías y sus credenciales | EP-003 | — | R1 |
| 12 | US-013 · Conocer protocolos y equipo de seguridad | EP-003 | — | R1 |
| 13 | US-015 · Encontrar respuestas en preguntas frecuentes | EP-003 | — | R1 |
| 14 | US-022 · Encontrar la ficha local por sede (GMB) | EP-005 | — | R1 |
| 15 | US-019 · Encontrar página dedicada por río | EP-005 | US-007 | R2 |
| 16 | US-020 · Navegar el sitio en inglés | EP-005 | US-019 | R2 |
| 17 | US-021 · Aparecer en buscadores y asistentes de IA | EP-005 | US-019 | R2 |
| 18 | US-009 · Comparar paquetes | EP-002 | US-007, US-008 | R2 |
| 19 | US-016 · Ver fotos y videos reales por río | EP-004 | US-019 | R2 |
| 20 | US-017 · Ver contenido de redes sociales en el sitio | EP-004 | — | R2 |
| 21 | US-018 · Leer guías y artículos (blog) | EP-004 | — | R2 |
| 22 | US-014 · Leer reseñas y testimonios | EP-003 | — | R2 |
| 23 | US-023 · Explorar la oferta filtrada por perfil | EP-006 | US-007 | R2 |
| 24 | US-024 · Conocer edad mínima y nivel de esfuerzo | EP-006 | US-007 | R2 |
| 25 | US-025 · Recibir recomendaciones por perfil | EP-006 | US-023, US-024 | R2 |
| 26 | US-026 · Solicitar cotización para grupos | EP-007 | — | R2 |
| 27 | US-027 · Registrar y responder cotizaciones | EP-007 | US-026 | R2 |
| 28 | US-028 · Conocer la propuesta de valor corporativa | EP-007 | — | R2 |
| 29 | US-029 · Descargar material informativo B2B | EP-007 | — | R2 |
| 30 | US-030 · Consultar disponibilidad y calendario | EP-008 | US-007 | R3 |
| 31 | US-031 · Reservar en línea | EP-008 | US-030 | R3 |
| 32 | US-032 · Pagar la reserva en línea | EP-008 | US-031 | R3 |
| 33 | US-033 · Recibir confirmación automática de reserva | EP-008 | US-032 | R3 |
| 34 | US-034 · Administrar reservas | EP-008 | US-031 | R3 |
| 35 | US-035 · Ver el catálogo de equipos | EP-009 | — | R3 |
| 36 | US-036 · Solicitar cotización o pedido de equipos | EP-009 | US-035 | R3 |
| 37 | US-037 · Ver oferta de equipos por tipo de cliente | EP-009 | US-035 | R3 |
| 38 | US-038 · Registrar y dar seguimiento a pedidos de equipos | EP-009 | US-036 | R3 |

## Notas de priorización y dependencias

- Las US de captación (US-001 a US-006) encabezan el backlog: son la palanca de conversión de mayor valor y menor costo, y no dependen de otras historias.
- US-007 (catálogo de paquetes) es base de varias historias (transparencia, segmentación, disponibilidad), por eso se prioriza dentro de R1 aunque su explotación completa ocurra en R2/R3.
- US-014 (reseñas) se ubica en R2 porque depende de acumular testimonios; las señales de confianza esenciales (licencias, guías, protocolos, FAQ) se entregan en R1.
- El bloque de reservas (US-030 → US-031 → US-032 → US-033) es estrictamente secuencial: disponibilidad habilita reserva, reserva habilita pago, pago habilita confirmación. Es el núcleo de R3 por su mayor costo y riesgo técnico.
- El dominio de equipos (US-035 a US-038) cierra el backlog, alineado con su prioridad P3 en el Opportunity Report (no es objetivo primario y requiere estudio de mercado previo).

## Nota de trazabilidad
Cada Story conserva su Epic de origen y la cadena Oportunidad → Epic → Feature → User Story.
Observación heredada del backlog: US-038 referencia FT-038, mientras que features.md
numera esa funcionalidad como un segundo FT-037. Se recomienda renumerar esa feature a
FT-038 para mantener IDs únicos.