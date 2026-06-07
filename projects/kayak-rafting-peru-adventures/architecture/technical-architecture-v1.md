# TECHNICAL ARCHITECTURE v1 — Rafting Peru & Kayak Adventures
Technical Architect · CypData · Junio 2026
Entrada: architecture-v1.md
Arquitectura técnica implementable. Sin priorización, roadmap, sprint planning, diseño visual ni código.

Nota de alcance: este documento selecciona stack y define componentes, APIs,
persistencia, integraciones, seguridad y despliegue. La selección tecnológica
es una recomendación justificada, alineada a un ecosistema JS/TS moderno; las
alternativas se indican donde es relevante. No incluye planificación temporal
ni código.

==================================================================
1. STACK TECNOLÓGICO
==================================================================

Principio: stack unificado JavaScript/TypeScript de extremo a extremo, para
reducir costo de operación (un solo lenguaje), favorecer el renderizado
orientado a descubrimiento (SSR/SSG) y permitir crecimiento modular por
dominio, en línea con el principio rector de architecture-v1.

Frontend / Capa pública (A)
- Framework: Next.js (App Router) con React y TypeScript.
- Renderizado: SSG/ISR para páginas de destino y catálogo (descubrimiento),
  SSR para vistas con datos dinámicos. Justificación: la "descubribilidad"
  (indexación + citabilidad por motores de IA) y la velocidad de carga son
  requisitos no funcionales centrales (RNF de D1).
- Internacionalización: i18n nativo de Next.js (rutas /es y /en) para la
  paridad ES/EN exigida.
- Estilos: framework de utilidades CSS (p. ej. Tailwind) — sin diseño visual
  en este documento, solo la elección técnica.

Backend / Servicios de dominio (B, C, D)
- Runtime: Node.js con TypeScript.
- Estilo de servicio: API modular por dominio (módulo de servicios por cada
  dominio D1–D5), desplegable como un backend modular (modular monolith)
  inicialmente, con límites de dominio explícitos para extraer servicios si
  el volumen lo exige. Justificación: presupuesto limitado + necesidad de
  desacoplar por dominio sin sobrecostos de microservicios prematuros.
- Framework de API: framework Node maduro (p. ej. NestJS por su modularidad
  por dominio, o Express/Fastify si se prefiere ligereza).

Capa de administración / Back-office (C)
- Aplicación administrativa basada en el mismo stack (Next.js) con acceso
  autenticado, separada de la capa pública por rutas y autorización.

Persistencia
- Base de datos relacional como almacén principal (p. ej. PostgreSQL), por
  la naturaleza transaccional de reservas/pagos y la integridad referencial
  entre entidades.
- Almacenamiento de objetos para activos multimedia (galerías/video).
- Capa de búsqueda/indexación opcional para catálogo y contenido si el
  volumen lo justifica.

Mensajería / Procesos asíncronos
- Mecanismo de cola/eventos para tareas asíncronas (envío de
  confirmaciones, notificaciones, sincronización con fichas locales),
  desacoplando las integraciones externas del flujo síncrono.

Alternativas válidas (no excluyentes): el dominio público podría resolverse
con un CMS headless para contenido editorial/multimedia (D1/M3); el backend
podría apoyarse en un BaaS para acelerar el arranque del MVP. Se documentan
como opciones; la decisión final depende de restricciones operativas del
negocio.

==================================================================
2. COMPONENTES
==================================================================

Capa pública (A)
- web-public: aplicación Next.js que sirve portal de contenido y catálogo
  (M1), centro de confianza (M4) y contenido de marca (M3). Responsable del
  SSR/SSG y del SEO/GEO técnico.
- content-service: provee contenido de destinos, paquetes, multimedia,
  reseñas y artículos a la capa pública (soporta M1, M3, M4).
- visibility-service: genera y expone metadatos estructurados (datos
  estructurados/schema), sitemaps y feeds para indexación y citabilidad
  (M2), y orquesta la sincronización de fichas locales.

Capa de captación y conversión (B)
- lead-service: recibe y registra consultas de formularios y del canal
  conversacional, normaliza y centraliza leads (M5, M6).
- messaging-gateway: adaptador hacia la mensajería conversacional
  (WhatsApp) para contacto contextual por destino (M5).
- quote-service: gestiona solicitudes de cotización de grupos/B2B y su ciclo
  de respuesta (M7).
- booking-service: disponibilidad, creación y gestión de reservas, control
  de cupos (M8).
- payment-service: orquesta el cobro con la pasarela y coordina el cambio de
  estado de la reserva (M9).
- notification-service: emite confirmaciones y comunicaciones por correo o
  mensajería de forma asíncrona (M9, M6).

Capa de gestión interna / Back-office (C)
- admin-app: aplicación autenticada para responsable comercial y gestor de
  operaciones; consume lead-service, quote-service, booking-service y
  equipment-service para bandejas, estados y seguimiento.

Capa de dominio de equipos (D)
- equipment-service: catálogo de equipos, segmentación por tipo de cliente,
  solicitudes de cotización/pedido y seguimiento de estado de importación
  (M10).

Capa de integraciones (E)
- integration-adapters: conjunto de adaptadores aislados por proveedor
  (mensajería, pagos, fichas locales, redes sociales, notificaciones,
  OTAs), que encapsulan el detalle externo y exponen contratos internos
  estables.

Componentes transversales
- identity-access: autenticación y autorización para back-office y, si
  aplica, para clientes que gestionen sus reservas.
- api-gateway / BFF: punto de entrada que orquesta las llamadas de las apps
  (web-public y admin-app) hacia los servicios de dominio.
- observability: registro de eventos de negocio (consultas por canal,
  destino, idioma) y técnico (logs, métricas, trazas).

==================================================================
3. APIs
==================================================================

Estilo: APIs REST/JSON sobre HTTPS, versionadas (p. ej. /api/v1), con
contratos por dominio. Se documentan con especificación abierta (OpenAPI).
Donde el consumo del frontend lo justifique, puede exponerse un BFF que
agregue respuestas. (Definición de contratos a nivel lógico; no incluye
implementación.)

Contenido y catálogo (content-service)
- Consulta de destinos y su contenido.
- Consulta de paquetes/experiencias con filtros por destino, dificultad y
  perfil de cliente.
- Consulta de detalle de paquete (incluidos/opcionales, precio de
  referencia, condiciones).
- Consulta de reseñas, FAQ, perfiles de guías y certificaciones.

Visibilidad (visibility-service)
- Generación de metadatos estructurados y sitemaps (consumo interno de
  web-public).
- Sincronización de información de fichas locales (hacia integration-adapters).

Leads y consultas (lead-service)
- Alta de consulta (formulario / conversacional) con datos de contacto,
  destino, idioma y canal de origen.
- Consulta y actualización de estado de leads (uso del back-office).

Cotizaciones (quote-service)
- Alta de solicitud de cotización (grupos/B2B).
- Consulta, elaboración y respuesta de cotizaciones; cambio de estado.

Reservas (booking-service)
- Consulta de disponibilidad por experiencia y fecha.
- Creación de reserva (valida cupo) y actualización de su estado.
- Gestión operativa de reservas (back-office).

Pagos (payment-service)
- Inicio de pago asociado a una reserva.
- Recepción de resultado de pago (callback/webhook desde la pasarela) y
  actualización del estado de la reserva.

Notificaciones (notification-service)
- Emisión de confirmación de reserva y comunicaciones de seguimiento
  (invocación interna/asíncrona).

Equipos (equipment-service)
- Consulta de catálogo y segmentación por tipo de cliente.
- Alta de solicitud de cotización/pedido de equipo.
- Consulta y actualización de estado de pedido/importación (back-office).

Convención transversal: autenticación requerida para endpoints de
back-office y de gestión de estado; endpoints públicos de solo lectura para
contenido y catálogo.

==================================================================
4. PERSISTENCIA
==================================================================

(Modelo lógico de almacenamiento; no constituye diseño físico de base de datos)

Almacén relacional principal (transaccional), organizado por límites de
dominio:
- Dominio Contenido (D1): Destino, Paquete/Experiencia, Componente de
  Paquete, Perfil de Cliente, Reseña, Contenido editorial (metadatos), Guía,
  Certificación.
- Dominio Captación (D2): Consulta/Lead, Solicitud de Cotización, Cotización.
- Dominio Reservas (D4): Reserva, Disponibilidad/Cupo, Pago, Confirmación.
- Dominio Equipos (D5): Equipo (producto), Pedido de Equipo.
- Transversal: Usuario Interno / Actor Operativo, registros de auditoría de
  estados.

Principios de persistencia:
- Integridad transaccional fuerte en el dominio de reservas/pagos (relación
  Reserva–Pago–Confirmación; el cupo se ajusta de forma consistente).
- Cada dominio es dueño de sus datos; el acoplamiento entre dominios se
  resuelve por referencia lógica, no por consultas cruzadas directas, para
  permitir la futura extracción de servicios.
- Auditoría: las entidades con ciclo de vida por estados (lead, cotización,
  reserva, pedido) registran su historial de cambios (RNF de trazabilidad).
- Activos multimedia en almacenamiento de objetos; en la base solo se guarda
  su referencia/metadato.
- Datos personales y de pago segregados y tratados con los controles
  descritos en Seguridad.

==================================================================
5. INTEGRACIONES
==================================================================

Cada integración se implementa como adaptador aislado (patrón puerto/
adaptador) bajo integration-adapters, exponiendo un contrato interno estable
y absorbiendo el detalle del proveedor.

- Mensajería conversacional (WhatsApp): vía API de negocio del proveedor de
  mensajería; mensajes contextuales y, si aplica, recepción de mensajes
  entrantes hacia lead-service. Comunicación saliente preferentemente
  asíncrona.
- Pasarela de pago: integración por API + webhooks para confirmación de
  cobro; soporte de medios locales (Perú) e internacionales. El resultado de
  pago actualiza el estado de la reserva de forma idempotente.
- Fichas de presencia local (tipo Google Business Profile): sincronización
  de datos de sede; preferentemente vía API del proveedor.
- Buscadores y motores de IA: no es una integración por API sino por
  exposición de contenido (datos estructurados/schema, sitemaps, feeds)
  generados por visibility-service.
- Redes sociales (Instagram, Facebook, TikTok): consumo de contenido para
  su exhibición en la capa pública (M3), tolerante a indisponibilidad
  (degradación elegante).
- Notificaciones (correo/mensajería): envío transaccional de confirmaciones
  y seguimientos, de forma asíncrona vía cola.
- OTAs (Viator, GetYourGuide, TripAdvisor): a nivel técnico, consistencia de
  información; integración profunda (disponibilidad/reservas) queda como
  capacidad evolutiva, no obligatoria en v1.

Patrones de integración:
- Comunicación saliente desacoplada por cola/eventos cuando no requiere
  respuesta inmediata.
- Webhooks entrantes (pagos, mensajería) validados y procesados de forma
  idempotente.
- Reintentos con backoff y registro de fallos en la capa de adaptadores.

==================================================================
6. SEGURIDAD
==================================================================

- Transporte: HTTPS/TLS obligatorio en todos los canales.
- Autenticación y autorización: identidad para back-office con control de
  acceso por rol (responsable comercial, gestor de operaciones); separación
  estricta entre superficie pública (solo lectura de contenido) y superficie
  administrada.
- Datos personales: tratamiento de datos de consultas, reservas y clientes
  conforme a la normativa peruana de protección de datos personales (Ley
  N.º 29733) y buenas prácticas; minimización y control de acceso.
- Datos de pago: no almacenar datos sensibles de tarjeta en la plataforma;
  delegar el manejo a la pasarela (tokenización), alineado con estándares de
  la industria de medios de pago. payment-service trabaja con referencias/
  tokens, no con datos crudos.
- Validación de webhooks: verificación de firma/origen de los webhooks de
  pago y mensajería; procesamiento idempotente para evitar duplicados.
- Protección de superficie pública: defensas ante abuso de formularios
  (límite de tasa, protección anti-bot) en lead-service y quote-service.
- Gestión de secretos: credenciales de integraciones en un gestor de
  secretos, fuera del código y de la configuración versionada.
- Auditoría: registro de accesos y cambios de estado en entidades sensibles
  (reservas, pagos, pedidos) para trazabilidad.
- Aislamiento de dominios: el dominio de reservas/pagos se trata como zona de
  mayor sensibilidad, con controles reforzados.

==================================================================
7. DESPLIEGUE
==================================================================

(Estrategia técnica; sin planificación temporal)

- Empaquetado: componentes contenedorizados para portabilidad y paridad
  entre entornos.
- Topología inicial: backend modular (modular monolith) desplegado como
  servicio contenedorizado + aplicación(es) Next.js, con base de datos
  gestionada y almacenamiento de objetos. Justificación: minimizar costo y
  complejidad operativa en el arranque, conservando límites de dominio para
  evolucionar a servicios independientes si el volumen lo exige.
- Entornos: separación de desarrollo, prueba (staging) y producción.
- Integración y entrega continuas (CI/CD): canal automatizado de build,
  pruebas y despliegue, con migraciones de base de datos versionadas.
- Escalado: la capa pública (A) escala horizontalmente de forma
  independiente (es la de mayor tráfico y la puerta de descubrimiento);
  los servicios de dominio escalan según demanda; los procesos asíncronos
  (notificaciones, sincronizaciones) corren como trabajadores separados.
- Distribución de contenido: red de distribución (CDN) para activos
  estáticos y multimedia, alineada con el RNF de velocidad de carga.
- Respaldo y recuperación: respaldo regular de la base relacional y
  estrategia de recuperación para el dominio transaccional.
- Observabilidad operativa: centralización de logs, métricas y trazas;
  métricas de negocio (consultas por canal/destino/idioma) expuestas para el
  back-office.

==================================================================
8. RIESGOS TÉCNICOS
==================================================================

- Acoplamiento del modular monolith: si los límites de dominio no se
  respetan en la implementación, se dificulta la futura extracción de
  servicios. Mitigación: contratos internos por dominio y propiedad de datos
  por dominio desde el inicio.
- Dependencia de webhooks externos (pagos/mensajería): pérdida o duplicación
  de eventos puede dejar reservas en estado inconsistente. Mitigación:
  idempotencia, reintentos y conciliación periódica.
- Integridad reserva–pago bajo concurrencia: riesgo de sobreventa de cupos.
  Mitigación: control de concurrencia en booking-service y confirmación
  condicionada al resultado del pago.
- Dependencia de proveedores externos: cambios de API o de políticas
  (mensajería, fichas locales, OTAs) afectan adaptadores. Mitigación:
  aislamiento en integration-adapters y contratos internos estables.
- Estrategia de visibilidad ante IA en evolución: los mecanismos de
  citabilidad por motores generativos cambian rápido; riesgo de
  obsolescencia técnica de la generación de metadatos. Mitigación: mantener
  la generación de datos estructurados desacoplada y actualizable.
- Consistencia multicanal y multilingüe: divergencia de contenido/precios
  entre sitio, OTAs y fichas locales. Mitigación: una fuente de verdad por
  entidad y sincronización controlada.
- Costo operativo vs. presupuesto: servicios gestionados aceleran el arranque
  pero pueden elevar el costo recurrente. Mitigación: arquitectura que
  permite empezar simple y escalar componentes de forma selectiva.
- Seguridad de superficie pública: formularios y endpoints públicos
  expuestos a abuso. Mitigación: límites de tasa, protección anti-bot y
  validación estricta.
- Tratamiento de datos personales y de pago: el incumplimiento normativo es
  un riesgo material. Mitigación: tokenización de pagos, minimización de
  datos y controles de acceso descritos en Seguridad.

==================================================================
NOTA DE TRAZABILIDAD
==================================================================
Cada componente y servicio técnico mapea a un módulo funcional (M1–M10) y,
a través de él, a las Features y Epics del backlog, conservando la cadena
Oportunidad → Epic → Feature → Módulo funcional → Componente técnico.

Se mantiene la observación heredada del backlog sobre la doble numeración
"FT-037" (gestión de pedidos de equipos), cuya capacidad se implementa en
equipment-service (M10); se recomienda renumerar esa feature a FT-038 para
conservar IDs únicos en toda la trazabilidad.