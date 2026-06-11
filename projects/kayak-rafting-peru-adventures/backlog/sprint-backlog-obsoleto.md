# SPRINT BACKLOG — Rafting Peru & Kayak Adventures
Agile Delivery Manager · CypData · Junio 2026
Entrada: product-backlog.md
Sprint Backlog ejecutable. Respeta el orden de prioridad del Product Backlog (no se modifica).
Agrupa historias por sprint balanceando dependencias y entregas incrementales.

## Criterio de planificación
- Un sprint por release del Product Backlog (R1 → Sprint 1, R2 → Sprint 2, R3 → Sprint 3),
  preservando cada entrega incremental como unidad de valor.
- Dentro de cada sprint, las historias se listan en su orden de prioridad del backlog.
- No se alteran prioridades ni dependencias; solo se agrupa y secuencia para ejecución.
- Cada sprint cierra con un incremento entregable y con valor de negocio por sí mismo.

==================================================================
### Sprint 1 — Capturar consultas (Release R1)
==================================================================
Meta del sprint: habilitar la captación de consultas y la información mínima que
sustenta la decisión (oferta clara, confianza y ficha local), sin desarrollo pesado.
Incremento entregable: el sitio puede recibir y centralizar consultas, mostrar la
oferta con transparencia y comunicar confianza.

Historias incluidas (en orden de prioridad):
- US-001 · Iniciar conversación por WhatsApp (EP-001) — sin dependencias
- US-003 · Enviar consulta mediante formulario (EP-001) — sin dependencias
- US-002 · Mensaje de WhatsApp precargado por destino (EP-001) — dep. US-001
- US-004 · Ver consultas centralizadas (EP-001) — dep. US-001, US-003
- US-005 · Registrar y actualizar estado de la consulta (EP-001) — dep. US-004
- US-006 · Atención en el idioma del visitante (EP-001) — dep. US-004
- US-007 · Ver paquetes y experiencias por destino y dificultad (EP-002) — sin dependencias
- US-008 · Ver qué incluye y qué es opcional por paquete (EP-002) — dep. US-007
- US-010 · Ver precios de referencia y condiciones (EP-002) — dep. US-007
- US-011 · Ver licencias y certificaciones del operador (EP-003) — sin dependencias
- US-012 · Conocer al equipo de guías y sus credenciales (EP-003) — sin dependencias
- US-013 · Conocer protocolos y equipo de seguridad (EP-003) — sin dependencias
- US-015 · Encontrar respuestas en preguntas frecuentes (EP-003) — sin dependencias
- US-022 · Encontrar la ficha local por sede / GMB (EP-005) — sin dependencias

Secuencia interna sugerida (por dependencias):
1) US-007 (catálogo, base para US-008 y US-010) y, en paralelo, US-001 y US-003.
2) US-002 (tras US-001); US-004 (tras US-001 y US-003); luego US-005 y US-006.
3) US-008 y US-010 (tras US-007).
4) US-011, US-012, US-013, US-015 y US-022 (independientes, en paralelo).

==================================================================
### Sprint 2 — Atraer y convertir tráfico (Release R2)
==================================================================
Meta del sprint: generar tráfico calificado y mejorar la conversión mediante
visibilidad/GEO, contenido, segmentación por perfil y atención al canal B2B.
Incremento entregable: páginas por destino bilingües y descubribles, contenido de
marca, navegación segmentada y flujo de cotización para grupos.
Precondición: Sprint 1 entregado (US-007 disponible como base).

Historias incluidas (en orden de prioridad):
- US-019 · Encontrar página dedicada por río (EP-005) — dep. US-007
- US-020 · Navegar el sitio en inglés (EP-005) — dep. US-019
- US-021 · Aparecer en buscadores y asistentes de IA (EP-005) — dep. US-019
- US-009 · Comparar paquetes (EP-002) — dep. US-007, US-008
- US-016 · Ver fotos y videos reales por río (EP-004) — dep. US-019
- US-017 · Ver contenido de redes sociales en el sitio (EP-004) — sin dependencias
- US-018 · Leer guías y artículos / blog (EP-004) — sin dependencias
- US-014 · Leer reseñas y testimonios (EP-003) — sin dependencias
- US-023 · Explorar la oferta filtrada por perfil (EP-006) — dep. US-007
- US-024 · Conocer edad mínima y nivel de esfuerzo (EP-006) — dep. US-007
- US-025 · Recibir recomendaciones por perfil (EP-006) — dep. US-023, US-024
- US-026 · Solicitar cotización para grupos (EP-007) — sin dependencias
- US-027 · Registrar y responder cotizaciones (EP-007) — dep. US-026
- US-028 · Conocer la propuesta de valor corporativa (EP-007) — sin dependencias
- US-029 · Descargar material informativo B2B (EP-007) — sin dependencias

Secuencia interna sugerida (por dependencias):
1) US-019 (página por río) primero: habilita US-020, US-021 y US-016.
2) US-023 y US-024 (sobre US-007) y luego US-025.
3) US-026 → US-027 (cotización); US-009 (tras US-008).
4) US-014, US-017, US-018, US-028, US-029 (independientes, en paralelo).

==================================================================
### Sprint 3 — Automatizar y escalar (Release R3)
==================================================================
Meta del sprint: cerrar el ciclo de venta directa con reservas y pago en línea, y
habilitar la comercialización de equipos.
Incremento entregable: motor de reservas con pago y confirmación operativo, y
catálogo/pedidos de equipos.
Precondición: Sprint 1 entregado (US-007 disponible como base para disponibilidad).

Historias incluidas (en orden de prioridad):
- US-030 · Consultar disponibilidad y calendario (EP-008) — dep. US-007
- US-031 · Reservar en línea (EP-008) — dep. US-030
- US-032 · Pagar la reserva en línea (EP-008) — dep. US-031
- US-033 · Recibir confirmación automática de reserva (EP-008) — dep. US-032
- US-034 · Administrar reservas (EP-008) — dep. US-031
- US-035 · Ver el catálogo de equipos (EP-009) — sin dependencias
- US-036 · Solicitar cotización o pedido de equipos (EP-009) — dep. US-035
- US-037 · Ver oferta de equipos por tipo de cliente (EP-009) — dep. US-035
- US-038 · Registrar y dar seguimiento a pedidos de equipos (EP-009) — dep. US-036

Secuencia interna sugerida (por dependencias):
1) Cadena de reservas estricta: US-030 → US-031 → US-032 → US-033; US-034 tras US-031.
2) Equipos en paralelo a la cadena de reservas: US-035 → (US-036, US-037); US-038 tras US-036.

==================================================================
## Notas de planificación
==================================================================
- Mapeo 1 sprint = 1 release para preservar cada incremento como unidad de valor de
  negocio. Si la operación usa sprints de 2 semanas, cada sprint puede partirse en dos
  sub-iteraciones respetando la secuencia interna indicada (p. ej. Sprint 1a: captación
  EP-001; Sprint 1b: oferta + confianza + ficha local).
- Las historias de gestión interna (US-004, US-005, US-027, US-034, US-038) acompañan a
  las de cara al cliente dentro del mismo sprint para que cada incremento sea operable
  por el negocio (no solo visible para el visitante).
- Sprint 1 y Sprint 3 comparten la dependencia de US-007; al entregarse US-007 en
  Sprint 1, queda disponible como base para la disponibilidad de reservas del Sprint 3.
- No se modificaron prioridades del Product Backlog; el orden dentro de cada sprint es
  el del backlog, ajustado solo en su secuencia de ejecución por dependencias.

## Nota de trazabilidad
Cada historia conserva su Epic y la cadena Oportunidad → Epic → Feature → User Story →
Release → Sprint. Se mantiene la observación heredada sobre la doble numeración FT-037
(US-038); se recomienda renumerar esa feature a FT-038 para conservar IDs únicos.