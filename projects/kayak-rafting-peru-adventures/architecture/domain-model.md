# DOMAIN MODEL — Rafting Peru & Kayak Adventures
Domain-Driven Design · CypData · Junio 2026
Entradas: architecture-v1.md · technical-architecture-v1.md · user-stories.md · acceptance-criteria.md · ui-specification.md
Modelo de dominio conceptual. Sin diseño físico de BD, SQL, código, APIs, arquitectura técnica, priorización ni UI/UX.

==================================================================
RESUMEN DEL DOMINIO
==================================================================

El dominio de RPKA modela un operador de turismo de aventura que convierte
visitantes en consultas y reservas, y que opera además una línea secundaria de
comercialización de equipos. El modelo se organiza en cinco contextos
delimitados (bounded contexts), heredados de los dominios de la arquitectura
funcional, cada uno con sus agregados y lenguaje propio:

- C1 · Contenido y Oferta (Descubrimiento): qué se ofrece y dónde.
- C2 · Captación y Relación (CRM): consultas y cotizaciones de clientes.
- C3 · Confianza: credenciales, guías, reseñas y FAQ.
- C4 · Reservas y Pagos (Booking): disponibilidad, reserva, pago y confirmación.
- C5 · Equipos: catálogo y pedidos de equipos bajo importación.

Cada contexto es dueño de sus agregados y no accede a los datos internos de
otro: las relaciones entre contextos se expresan por referencia (identidad),
no por composición directa, para preservar el desacoplamiento por dominio que
define la arquitectura. La terminología de este documento es la fuente de
verdad para los agentes posteriores (Backend, Database, API, QA).

==================================================================
GLOSARIO
==================================================================

(Lenguaje ubicuo; los términos se usan de forma consistente en todo el dominio)

- Destino (Río): lugar donde se ofrecen experiencias (Urubamba, Lunahuaná, Cotahuasi, Marañón).
- Sede: ubicación operativa del negocio asociada a un destino y a una ficha local.
- Experiencia (Paquete): servicio turístico ofertado en un destino, con duración y dificultad.
- Componente: elemento de una experiencia, marcado como incluido u opcional.
- Perfil de Cliente: categoría de visitante a la que se adapta la oferta (familia, niño, adulto mayor, principiante, corporativo, practicante).
- Visitante: persona que navega el sitio; aún no identificada como cliente.
- Consulta (Lead): manifestación de interés de un visitante por un canal (WhatsApp o formulario).
- Solicitud de Cotización: petición de propuesta para un grupo o cliente institucional.
- Cotización: respuesta elaborada por el negocio a una Solicitud de Cotización.
- Reserva: compromiso formal de un cliente sobre una Experiencia en una fecha.
- Disponibilidad (Cupo): plazas ofertadas para una Experiencia en una fecha.
- Pago: transacción económica asociada a una Reserva.
- Confirmación: comprobante emitido cuando una Reserva queda pagada.
- Guía: persona del equipo que conduce las actividades, con credenciales.
- Credencial (Certificación/Licencia): acreditación del negocio o de un Guía.
- Reseña: valoración de un cliente sobre el negocio, un destino o una experiencia.
- Artículo: contenido editorial (guía/blog) asociado a destinos o experiencias.
- Activo Multimedia: foto o video asociado a un destino o experiencia.
- Equipo: producto del catálogo de equipos, comercializado bajo pedido.
- Pedido de Equipo: solicitud de cotización/compra de uno o más Equipos.
- Actor Operativo: usuario interno (responsable comercial o gestor de operaciones).
- Canal: medio por el que llega una consulta (WhatsApp, formulario, etc.).
- Idioma: ES o EN, aplicable a contenido y a la atención de consultas.

==================================================================
ENTIDADES
==================================================================

(Se indican agregados [AR = raíz de agregado], entidades internas y objetos de valor [VO].
Atributos principales a nivel conceptual; no es diseño físico.)

C1 · Contenido y Oferta
- Destino [AR]
  Atributos: identificador, nombre, región/sede asociada, descripción, idioma(s) de contenido, estado de publicación.
  Contiene: referencias a Activos Multimedia y Artículos del destino.
- Experiencia (Paquete) [AR]
  Atributos: identificador, nombre, destino (ref.), duración, nivel de dificultad, precio de referencia, condiciones, perfiles aptos, estado (publicada / no disponible).
  Contiene: Componentes (entidad interna).
- Componente [entidad interna de Experiencia]
  Atributos: nombre, tipo (alimentación, hospedaje, equipo, transporte, guía), condición (incluido / opcional).
- Perfil de Cliente [VO / catálogo]
  Atributos: clave (familia, niño, adulto mayor, principiante, corporativo, practicante), descripción, requisitos asociados (edad mínima, nivel de esfuerzo) cuando aplica a una Experiencia.
- Activo Multimedia [entidad]
  Atributos: identificador, tipo (foto/video), referencia de almacenamiento, destino/experiencia asociada, idioma/caption.
- Artículo [AR]
  Atributos: identificador, título, cuerpo, idioma, destinos/experiencias relacionadas, estado de publicación.

C2 · Captación y Relación
- Consulta (Lead) [AR]
  Atributos: identificador, datos de contacto, destino de interés (ref. opcional), mensaje, canal de origen, idioma, fecha, estado.
  Contiene: historial de cambios de estado (entidad interna).
- Solicitud de Cotización [AR]
  Atributos: identificador, organización, número de personas, destino (ref.), fecha solicitada, datos de contacto, estado de atención.
  Contiene: Cotización asociada (cuando se emite) e historial de estado.
- Cotización [entidad dependiente de Solicitud de Cotización]
  Atributos: identificador, contenido de la propuesta, fecha de emisión, autor (Actor Operativo).

C3 · Confianza
- Guía [AR]
  Atributos: identificador, nombre, experiencia, credenciales (ref. a Credencial).
- Credencial (Certificación/Licencia) [entidad]
  Atributos: identificador, tipo, entidad emisora, titular (negocio o Guía).
- Reseña [AR]
  Atributos: identificador, autor/cliente, valoración, texto, destino/experiencia asociada (opcional), fecha.
- FAQ [AR]
  Atributos: identificador, pregunta, respuesta, categoría/tema, idioma.

C4 · Reservas y Pagos
- Reserva [AR] (raíz del agregado transaccional)
  Atributos: identificador, experiencia (ref.), fecha, número de participantes, datos del cliente, estado, referencia de Pago, referencia de Confirmación.
  Contiene: historial de estado.
- Disponibilidad (Cupo) [AR]
  Atributos: experiencia (ref.), fecha, cupo total, cupo disponible.
- Pago [entidad dependiente de Reserva]
  Atributos: identificador, monto, estado/resultado, referencia/token de pasarela (sin datos sensibles de tarjeta), fecha.
- Confirmación [entidad dependiente de Reserva]
  Atributos: identificador, detalle de la reserva, fecha de emisión.

C5 · Equipos
- Equipo (Producto) [AR]
  Atributos: identificador, nombre, descripción, tipo, segmentos de cliente aplicables, condición de venta (bajo pedido / importación), estado de publicación.
- Pedido de Equipo [AR]
  Atributos: identificador, equipo(s) y cantidad(es) (ref.), tipo de cliente, datos de contacto, estado de importación, fecha.
  Contiene: historial de estado.

Transversal
- Actor Operativo [AR]
  Atributos: identificador, nombre, rol (responsable comercial / gestor de operaciones), credenciales de acceso (gestionadas por el contexto de identidad, fuera del modelo de negocio).

==================================================================
RELACIONES
==================================================================

(Relaciones conceptuales; "ref." indica relación por identidad entre contextos)

- Destino 1 — N Experiencia (una experiencia pertenece a un destino).
- Destino 1 — 1 Sede (cada destino tiene una sede operativa asociada).
- Experiencia 1 — N Componente (composición interna del agregado Experiencia).
- Experiencia N — N Perfil de Cliente (una experiencia es apta para varios perfiles).
- Destino 1 — N Activo Multimedia; Experiencia 1 — N Activo Multimedia.
- Destino N — N Artículo (un artículo puede relacionar varios destinos/experiencias).
- Consulta N — 1 Destino (ref. opcional: la consulta puede referir a un destino).
- Consulta N — 1 Canal; Consulta N — 1 Idioma.
- Solicitud de Cotización 1 — 0..1 Cotización (una solicitud puede tener una cotización emitida).
- Solicitud de Cotización N — 1 Destino (ref.).
- Guía N — N Credencial; Negocio 1 — N Credencial (titularidad de credenciales).
- Reseña N — 1 Destino/Experiencia (ref. opcional).
- Experiencia 1 — N Disponibilidad (una experiencia tiene cupos por fecha).
- Reserva N — 1 Experiencia (ref.); Reserva N — 1 Disponibilidad (por experiencia+fecha).
- Reserva 1 — 1 Pago; Reserva 1 — 0..1 Confirmación (la confirmación existe solo si la reserva queda pagada).
- Equipo N — N Perfil de Cliente (segmentación de la oferta de equipos).
- Pedido de Equipo N — N Equipo (un pedido puede incluir varios equipos).
- Actor Operativo 1 — N Consulta / Solicitud de Cotización / Reserva / Pedido (atención/gestión; relación operativa, no de propiedad del dato del cliente).

Fronteras de contexto: las referencias entre C1–C5 se hacen por identidad
(p. ej., Reserva referencia Experiencia por su identificador), nunca por
composición directa, para permitir la evolución independiente de cada contexto.

==================================================================
ESTADOS
==================================================================

(Ciclos de vida derivados de los Acceptance Criteria)

Consulta (Lead):
  Nueva → En proceso → Respondida → Cerrada
  (cada transición queda historizada; AC de US-005.)

Solicitud de Cotización:
  Recibida → En proceso → Cotización emitida → Cerrada
  (la emisión de la Cotización asocia la respuesta a la solicitud; AC de US-027.)

Reserva:
  Iniciada (pendiente de pago) → Pagada → Confirmada
                                   ↘ Pago fallido/rechazado → No confirmada
  Reserva (operación posterior): Confirmada → Cancelada / Completada
  (una reserva no pasa a Confirmada sin Pago exitoso; AC de US-032/US-033.)

Pago:
  Iniciado → Exitoso | Fallido/Rechazado
  (idempotente respecto del resultado recibido de la pasarela.)

Disponibilidad (Cupo):
  Disponible → Parcial → Agotado
  (derivado del cupo disponible vs. total; una fecha sin cupo se muestra no disponible; AC de US-030.)

Pedido de Equipo:
  Recibido → En cotización → Confirmado → En importación → Entregado
                                                        ↘ Cancelado
  (estado de importación visible para seguimiento; AC de US-038.)

Contenido (Destino / Experiencia / Artículo / Equipo):
  Borrador → Publicado → No disponible/Despublicado
  (solo el contenido publicado y disponible es contratable/visible; AC de US-007.)

==================================================================
REGLAS DE NEGOCIO
==================================================================

(Derivadas de las User Stories y sus Acceptance Criteria)

RN-01 · Toda Consulta registra su canal de origen, idioma y, si aplica, el
destino de interés (US-003, US-004, US-006).
RN-02 · El cambio de estado de una Consulta, Solicitud de Cotización, Reserva o
Pedido queda historizado (US-005, US-027, US-034, US-038; RNF de trazabilidad).
RN-03 · Una Experiencia solo es contratable/visible si está publicada y
disponible (US-007).
RN-04 · Cada Componente de una Experiencia es, o bien incluido, o bien opcional,
de forma inequívoca (US-008).
RN-05 · Una Experiencia expone un precio de referencia y, si su precio varía,
la base de cálculo (por personas o temporada) (US-010).
RN-06 · Una Reserva no puede registrar más participantes que el cupo disponible
de la Disponibilidad correspondiente (US-031).
RN-07 · Al confirmarse una Reserva, el cupo disponible de su Disponibilidad se
reduce de forma consistente (US-031).
RN-08 · Una Reserva solo alcanza el estado Confirmada si su Pago es exitoso; si
el Pago falla o es rechazado, la Reserva no se confirma (US-032).
RN-09 · Toda Reserva Confirmada genera una Confirmación con su detalle
(experiencia, fecha, número de participantes, datos de la reserva) (US-033).
RN-10 · Una Solicitud de Cotización requiere organización, número de personas,
destino y fecha; sin esos datos no se registra (US-026).
RN-11 · Una Cotización queda siempre asociada a la Solicitud que la origina y a
un estado de atención (US-027).
RN-12 · Un Equipo se comercializa bajo pedido; toda Solicitud/Pedido de Equipo
informa que está sujeta a disponibilidad de importación (US-036).
RN-13 · La oferta de Equipos puede segmentarse por tipo de cliente; si no hay
oferta específica para un tipo, se presenta la oferta general (US-037).
RN-14 · El contenido público mantiene paridad y coherencia entre idiomas ES/EN;
el idioma seleccionado se conserva durante la navegación (US-020).
RN-15 · Las acciones de gestión (bandejas, estados) requieren un Actor Operativo
autenticado y autorizado por rol (AC de US-004, US-034).
RN-16 · Una Credencial identifica su entidad emisora y su titular (negocio o
Guía) (US-011, US-012).
RN-17 · Una Reseña puede asociarse a un Destino o Experiencia; su ausencia no
impide mostrar la página correspondiente (US-014).

==================================================================
EVENTOS DE DOMINIO
==================================================================

(Hechos relevantes del negocio; insumo para Backend/QA, sin implementación)

- ConsultaRecibida (alta de Lead por cualquier canal).
- ConsultaCambióDeEstado.
- SolicitudDeCotizaciónRecibida.
- CotizaciónEmitida.
- SolicitudDeCotizaciónCambióDeEstado.
- DisponibilidadConsultada.
- ReservaIniciada.
- PagoIniciado.
- PagoConfirmado.
- PagoRechazado.
- ReservaConfirmada (desencadena ConfirmaciónEmitida).
- ConfirmaciónEmitida.
- ReservaCancelada.
- CupoActualizado (al confirmarse o cancelarse una reserva).
- PedidoDeEquipoRecibido.
- PedidoDeEquipoCambióDeEstadoDeImportación.
- ContenidoPublicado / ContenidoDespublicado (destino, experiencia, artículo, equipo).

==================================================================
SUPUESTOS
==================================================================

- El modelo es conceptual y agnóstico de tecnología; la persistencia, las APIs
  y el diseño físico corresponden a otros agentes.
- La identidad y el control de acceso de los Actores Operativos se gestionan en
  un contexto de identidad transversal, fuera del modelo de negocio.
- Los datos sensibles de pago no residen en el dominio: el Pago referencia un
  token/identificador de la pasarela (coherente con la arquitectura técnica).
- Una Reserva corresponde a una Experiencia en una fecha; la composición de
  varias experiencias en un mismo flujo de compra no se modela en esta versión.
- La gestión de cupos asume una Disponibilidad por Experiencia y fecha; reglas
  de temporada o cupos compartidos entre experiencias quedan como evolución.
- El Pedido de Equipo modela el seguimiento de importación, no la logística
  detallada de aduanas/transporte.
- La oferta y los precios son provistos y mantenidos por el negocio.

==================================================================
RIESGOS
==================================================================

- Consistencia de cupos bajo concurrencia: dos reservas simultáneas sobre la
  misma Disponibilidad podrían sobrevender si la reducción de cupo no es
  consistente (mitigación: tratar Reserva–Disponibilidad como invariante del
  agregado transaccional).
- Integridad Reserva–Pago–Confirmación: el estado debe reflejar fielmente el
  resultado del Pago para no confirmar reservas no pagadas.
- Coherencia multilingüe: mantener paridad ES/EN del contenido puede generar
  divergencias si no hay una fuente de verdad por entidad de contenido.
- Ambigüedad de "cotización": existe tanto en grupos/B2B (C2) como,
  potencialmente, en equipos (C5/Pedido); el modelo las mantiene separadas para
  evitar mezclar lenguajes de contexto.
- Doble naturaleza del negocio: el contexto de Equipos (producto físico) tiene
  un ciclo de vida distinto al turístico; mantenerlos como contextos separados
  evita contaminar el modelo.
- Evolución de la oferta: cambios en cómo el negocio empaqueta experiencias
  (p. ej., combinar destinos) exigirían revisar el agregado Experiencia.

==================================================================
NOTA DE TRAZABILIDAD
==================================================================
Las entidades y reglas derivan de las User Stories y Acceptance Criteria, y los
contextos C1–C5 corresponden a los dominios D1–D5 de la arquitectura funcional,
conservando la cadena Oportunidad → Epic → Feature → User Story → Entidad/Regla.

Se mantiene la observación heredada del backlog sobre la doble numeración FT-037
(gestión de pedidos de equipos, modelada aquí como Pedido de Equipo en C5); se
recomienda renumerar esa feature a FT-038 para conservar IDs únicos.