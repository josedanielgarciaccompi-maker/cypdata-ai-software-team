# ARCHITECTURE v1 — Rafting Peru & Kayak Adventures
Solution Architect · CypData · Junio 2026
Entradas: business.md · research-report-002.md · opportunity-report-002.md · epics.md · features.md · user-stories.md · acceptance-criteria.md
Arquitectura funcional y lógica de alto nivel. Sin selección de tecnologías, diseño físico de BD, UI/UX, código, priorización ni estimaciones.

==================================================================
1. RESUMEN EJECUTIVO
==================================================================

La solución para RPKA se concibe como una plataforma digital de turismo
de aventura cuyo propósito central es convertir el tráfico en consultas y
reservas, cerrando la brecha de "materialización digital" identificada en
la investigación. La arquitectura se organiza en torno a cuatro grandes
capacidades de negocio: descubrir (ser encontrado y comunicar la oferta),
captar (convertir interés en consulta), convertir (formalizar reserva y
pago) y operar (gestionar consultas, cotizaciones, reservas y pedidos).

El diseño prioriza una capa pública orientada al contenido y al
descubrimiento (multi-destino, bilingüe, citable por buscadores y motores
de IA), una capa conversacional/transaccional para captar y convertir, y
una capa de gestión interna (back-office) para que el negocio atienda y dé
seguimiento. La línea de comercialización de equipos se modela como un
dominio independiente del negocio turístico, dada su naturaleza distinta
(producto físico bajo importación).

La arquitectura es modular y desacoplada por dominio, de modo que las
capacidades de bajo costo y alto valor (captación conversacional,
transparencia de oferta, confianza, visibilidad) puedan existir y operar
con independencia de las capacidades de mayor complejidad (motor de
reservas y pagos), que se incorporan cuando el negocio lo requiera, sin
rediseñar el conjunto.

==================================================================
2. DOMINIOS DE NEGOCIO
==================================================================

D1 · Descubrimiento y Contenido
Responsable de la presencia pública, el contenido por destino, la oferta,
la narrativa de marca y la visibilidad ante buscadores y motores de IA.
Epics asociados: EP-002, EP-004, EP-005, EP-006

D2 · Captación y Relación con el Cliente (CRM/Lead)
Responsable de capturar, centralizar, atender y dar seguimiento a las
consultas y a las solicitudes de cotización por cualquier canal.
Epics asociados: EP-001, EP-007

D3 · Confianza y Credibilidad
Responsable de la información de seguridad, certificaciones, perfiles de
guías, reseñas y preguntas frecuentes que sustentan la decisión de compra.
Epic asociado: EP-003

D4 · Reservas y Pagos (Booking)
Responsable de la disponibilidad, la formalización de reservas, el cobro
en línea, la confirmación y la gestión operativa de las reservas.
Epic asociado: EP-008

D5 · Comercialización de Equipos
Responsable del catálogo de equipos, las solicitudes de cotización/pedido
bajo importación y el seguimiento de pedidos. Dominio independiente del
negocio turístico.
Epic asociado: EP-009

Nota: los dominios se definen por afinidad de capacidad de negocio, no por
canal ni por tecnología. D2 agrupa la relación con el cliente tanto del
turista individual (consultas) como del canal B2B (cotizaciones) por
compartir el mismo núcleo de gestión de la relación.

==================================================================
3. MÓDULOS FUNCIONALES
==================================================================

M1 · Portal de Contenido y Catálogo (D1)
Presentación de destinos por río, catálogo de paquetes y experiencias,
detalle de "qué incluye/opcional", comparación, precios de referencia,
navegación segmentada por perfil de cliente y soporte bilingüe.
Cubre: FT-006, FT-007, FT-008, FT-009, FT-018, FT-019, FT-022, FT-023, FT-024

M2 · Motor de Visibilidad y Descubrimiento (D1)
Estructuración de contenido y metadatos para indexación por buscadores y
citabilidad por motores de IA, y gestión de fichas de presencia local por
sede.
Cubre: FT-020, FT-021

M3 · Contenido de Marca y Multimedia (D1)
Gestión y publicación de galería audiovisual por destino, integración de
redes sociales y contenido editorial/blog.
Cubre: FT-015, FT-016, FT-017

M4 · Centro de Confianza (D3)
Presentación de seguridad y certificaciones, perfiles de guías, protocolos,
reseñas/testimonios y preguntas frecuentes.
Cubre: FT-010, FT-011, FT-012, FT-013, FT-014

M5 · Captación Conversacional y Formularios (D2)
Contacto por WhatsApp con mensaje contextual por destino, formularios de
consulta y atención multilingüe del flujo de consulta.
Cubre: FT-001, FT-002, FT-005

M6 · Gestión de Consultas y Leads (D2)
Captura y centralización de consultas multicanal, seguimiento y estados.
Cubre: FT-003, FT-004

M7 · Gestión de Grupos y Canal B2B (D2)
Solicitud y gestión de cotizaciones de grupos/corporativos, sección de
propuesta de valor institucional y material descargable para intermediarios.
Cubre: FT-025, FT-026, FT-027, FT-028

M8 · Disponibilidad y Reservas (D4)
Consulta de disponibilidad/calendario, formalización de reservas y gestión
operativa de reservas.
Cubre: FT-029, FT-030, FT-033

M9 · Pagos y Confirmación (D4)
Procesamiento del pago de la reserva y emisión de confirmación automática.
Cubre: FT-031, FT-032

M10 · Catálogo y Pedidos de Equipos (D5)
Catálogo de equipos, segmentación por tipo de cliente, solicitud de
cotización/pedido bajo importación y seguimiento de pedidos.
Cubre: FT-034, FT-035, FT-036, FT-037 (gestión de pedidos)

Módulo transversal · Back-office / Administración
Capa de gestión interna que da soporte a M6, M7, M8 y M10 para los actores
operativos (responsable comercial, gestor de operaciones): bandejas,
listados, estados y seguimiento. Es una capacidad transversal de los
dominios D2, D4 y D5, no un dominio en sí mismo.

==================================================================
4. ENTIDADES PRINCIPALES
==================================================================

(Modelo conceptual de negocio; no constituye diseño físico de base de datos)

- Destino / Río: unidad geográfica de la oferta (Urubamba, Lunahuaná, Cotahuasi, Marañón) con su contenido propio.
- Sede: ubicación operativa asociada a fichas locales (Cusco, Lima, Arequipa, Marañón).
- Experiencia / Paquete: oferta turística con destino, duración, dificultad, componentes incluidos y opcionales, y precio de referencia.
- Componente de Paquete: elemento incluido u opcional (alimentación, hospedaje, equipo, transporte, guía).
- Perfil de Cliente: segmento al que se adapta la oferta (familia, niño, adulto mayor, principiante, corporativo, practicante).
- Consulta / Lead: solicitud de información de un visitante, con canal de origen, datos de contacto, destino de interés, mensaje, idioma y estado.
- Solicitud de Cotización: petición de grupo/corporativo con organización, número de personas, destino, fecha y estado de atención.
- Cotización: respuesta elaborada a una solicitud, asociada a su solicitud.
- Reserva: compromiso sobre una experiencia, con fecha, número de participantes, estado y vínculo a su pago.
- Disponibilidad / Cupo: capacidad por experiencia y fecha.
- Pago: transacción asociada a una reserva, con su resultado/estado.
- Confirmación: comprobante emitido tras una reserva pagada.
- Guía: integrante del equipo con experiencia y certificaciones.
- Certificación / Licencia: credencial institucional o de seguridad del negocio o del guía.
- Reseña / Testimonio: valoración de cliente, eventualmente asociada a un destino o experiencia.
- Contenido (Multimedia / Editorial): activos audiovisuales y artículos asociados a destinos o experiencias.
- Equipo (Producto): artículo del catálogo de equipos, con su información y condición de venta bajo pedido.
- Pedido de Equipo: solicitud de cotización/compra de equipo con su estado de importación.
- Usuario Interno / Actor Operativo: responsable comercial y gestor de operaciones, con acceso al back-office.

==================================================================
5. INTEGRACIONES EXTERNAS
==================================================================

(Identificadas por capacidad requerida; sin selección de proveedor específico)

- Mensajería conversacional (WhatsApp): canal de contacto y captación (M5).
- Pasarela de pago: procesamiento del cobro de reservas (M9). El negocio
  opera en Perú, por lo que se prevé soporte de medios locales e
  internacionales.
- Fichas de presencia local (tipo Google My Business): visibilidad local
  por sede (M2).
- Buscadores y motores de IA / búsqueda generativa: consumo del contenido
  estructurado para descubrimiento (M2).
- Redes sociales (Instagram, Facebook, TikTok): origen de contenido a
  integrar en el sitio (M3).
- OTAs / plataformas de terceros (Viator, GetYourGuide, TripAdvisor):
  canales de descubrimiento y, potencialmente, de reseñas; relación a
  considerar a nivel de presencia y consistencia de información.
- Notificaciones (correo electrónico y/o mensajería): envío de
  confirmaciones de reserva y comunicaciones de seguimiento (M9, M6).
- Servicios de mapas/geolocalización: ubicación de sedes y destinos
  (soporte a M1/M2), de ser requerido.

==================================================================
6. REQUISITOS NO FUNCIONALES
==================================================================

- Rendimiento y velocidad de carga: la carga rápida es crítica para el
  descubrimiento y la conversión; el contenido multimedia debe presentarse
  de forma progresiva sin bloquear la página (alineado con AC de US-016).
- Multilingüe (ES/EN): el contenido público y los flujos de consulta deben
  soportar ambos idiomas y mantener el idioma seleccionado en la navegación.
- Disponibilidad: la capa pública y de captación debe mantener alta
  disponibilidad, por ser la puerta de entrada de consultas y reservas.
- Accesibilidad y diseño responsivo: experiencia adecuada en dispositivos
  móviles, dado el comportamiento del usuario y el peso del canal móvil.
- Visibilidad / "descubribilidad": el contenido debe ser indexable y
  citable por buscadores y motores de IA (capacidad central del dominio D1).
- Seguridad y protección de datos: tratamiento adecuado de datos personales
  de consultas, reservas y pagos, y de los datos de pago según estándares
  aplicables.
- Trazabilidad y auditabilidad: registro de estados y cambios en consultas,
  cotizaciones, reservas y pedidos (soporta los AC de seguimiento de estado).
- Consistencia de la información: la información publicada (precios,
  disponibilidad, fichas locales) debe poder mantenerse vigente y coherente
  entre canales.
- Escalabilidad modular: capacidad de incorporar el dominio de reservas/pagos
  y el de equipos sin rediseñar los dominios ya operativos.
- Observabilidad de negocio: capacidad de medir consultas por canal, destino
  e idioma (necesidad declarada en el roadmap de negocio, a nivel de métrica).
- Mantenibilidad: separación por dominios para evolución independiente.
- Confiabilidad transaccional: integridad entre reserva, pago y
  confirmación (una reserva no se confirma si el pago no se completa, según
  AC de US-032).

==================================================================
7. ARQUITECTURA LÓGICA DE ALTO NIVEL
==================================================================

La solución se estructura en capas lógicas desacopladas:

(A) Capa de Experiencia Pública
Expone el portal de contenido y catálogo (M1), el motor de visibilidad
(M2), el contenido de marca (M3) y el centro de confianza (M4). Es la cara
de descubrimiento y comunicación de la oferta. Orientada a ser indexable y
citable, bilingüe y responsiva.

(B) Capa de Captación y Conversión
Comprende la captación conversacional y formularios (M5), la entrada de
solicitudes de cotización (parte de M7) y el flujo de disponibilidad y
reserva (M8) con pagos y confirmación (M9). Es donde el interés se convierte
en consulta o reserva.

(C) Capa de Gestión Interna (Back-office)
Da soporte a los actores operativos: gestión de consultas y leads (M6),
gestión de cotizaciones y B2B (M7), gestión de reservas (M8) y gestión de
pedidos de equipos (M10). Centraliza el seguimiento y los estados.

(D) Capa de Dominio de Equipos
Catálogo y pedidos de equipos (M10), desacoplada del negocio turístico por
su naturaleza de producto físico bajo importación. Comparte la capa de
gestión interna para el seguimiento de pedidos.

(E) Capa de Integraciones
Media la comunicación con servicios externos (mensajería, pagos, fichas
locales, buscadores/IA, redes sociales, notificaciones, OTAs).

Relación entre capas:
- La capa pública (A) alimenta a la de captación (B): el contenido y la
  confianza habilitan la consulta y la reserva.
- La captación (B) registra en la gestión interna (C): toda consulta,
  cotización y reserva queda disponible para el back-office.
- La capa de integraciones (E) es transversal y sirve a (A), (B) y (D).
- El dominio de equipos (D) es independiente pero reutiliza (C) y (E).

Principio rector: desacoplamiento por dominio para que las capacidades de
alto valor/bajo costo operen sin depender de las de mayor complejidad, y
para que la solución crezca por incorporación de dominios y no por
rediseño.

==================================================================
8. RIESGOS ARQUITECTÓNICOS
==================================================================

- Dependencia de servicios externos: mensajería, pagos y fichas locales son
  provistos por terceros; su disponibilidad y condiciones afectan
  capacidades críticas. Mitigación a nivel de diseño: aislar las
  integraciones en una capa propia (E) para reducir el acoplamiento.
- Acoplamiento del dominio de reservas/pagos: por su complejidad y manejo de
  datos sensibles, podría arrastrar al resto si no se mantiene desacoplado.
- Consistencia entre canales: la información publicada en el sitio, las OTAs
  y las fichas locales puede divergir; riesgo de incoherencia de precios o
  disponibilidad.
- Integridad transaccional reserva-pago-confirmación: requiere un manejo
  cuidadoso de estados para evitar reservas confirmadas sin pago o cupos mal
  liberados.
- Datos personales y de pago: el tratamiento de información sensible
  introduce requisitos de seguridad y cumplimiento que condicionan el diseño.
- Visibilidad ante motores de IA: al ser un terreno en evolución, la
  capacidad de ser citado depende de factores externos cambiantes; riesgo de
  obsolescencia de la estrategia de estructuración de contenido.
- Multilingüe y consistencia de contenido: mantener paridad ES/EN y
  coherencia del contenido por destino añade complejidad de gestión.
- Doble naturaleza del negocio: el dominio de equipos (producto físico bajo
  importación) tiene dinámicas distintas al turístico; forzar su encaje en
  el mismo modelo introduciría complejidad innecesaria.

==================================================================
9. SUPUESTOS
==================================================================

- El objetivo primario de la solución es maximizar consultas y reservas; las
  capacidades se conciben al servicio de ese objetivo.
- El negocio opera con presupuesto inicial limitado, por lo que la
  arquitectura debe permitir activar capacidades de forma incremental por
  dominio, sin rediseño.
- Las capacidades de captación conversacional, transparencia, confianza y
  visibilidad pueden existir y aportar valor antes de que exista el motor de
  reservas y pagos.
- Los procesos de back-office serán operados por roles del negocio
  (responsable comercial, gestor de operaciones).
- La línea de equipos se comercializa bajo pedido y sujeta a importación;
  su modelo de cumplimiento (fulfillment) no es inmediato.
- La información de oferta, precios y disponibilidad es provista y mantenida
  por el negocio.
- La definición de tecnologías concretas, el diseño físico de datos, el
  diseño de interfaces y la planificación de ejecución corresponden a etapas
  y roles posteriores.
- Se asume continuidad de los canales de descubrimiento actuales (buscadores,
  IA, redes, OTAs, fichas locales) como contexto de la arquitectura.

==================================================================
NOTA DE TRAZABILIDAD
==================================================================
La arquitectura deriva directamente del backlog: cada Módulo Funcional
referencia las Features (FT) que cubre, y cada Dominio agrupa los Epics (EP)
correspondientes, conservando la cadena Oportunidad → Epic → Feature →
Módulo → Dominio.

Observación heredada del backlog: en user-stories.md la US-038 referencia
"FT-038", mientras que features.md numera esa funcionalidad como un segundo
"FT-037". En esta arquitectura, la gestión de pedidos de equipos se ubica en
el módulo M10. Se mantiene la recomendación de renumerar esa feature a
FT-038 para conservar IDs únicos en toda la trazabilidad.