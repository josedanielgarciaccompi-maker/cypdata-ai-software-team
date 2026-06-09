# DOMAIN CATALOG — Rafting Peru & Kayak Adventures
Domain Architect (DDD) · CypData · Junio 2026
Entrada: domain-model.md
Catálogo estructurado de objetos de dominio, consumible por Backend, Database, API y QA.
Sin SQL, diseño físico de BD, código, APIs, arquitectura técnica, UI/UX ni priorización.

Convenciones del catálogo
- Tipos lógicos (agnósticos de tecnología): Id, Text, LongText, Int, Decimal, Money,
  Bool, Date, DateTime, Enum(<ref>), Ref(<Entidad>), List<...>.
- "Ref" = relación por identidad entre agregados/contextos (no composición).
- Contextos: C1 Contenido · C2 Captación · C3 Confianza · C4 Reservas · C5 Equipos · CX Transversal.
- Cardinalidad en Relationships: 1-1, 1-N, N-N, 1-0..1.

==================================================================
AGGREGATES
==================================================================

AGG-Destino (C1)
- Raíz: Destino
- Entidades internas: —
- Value Objects: —
- Invariantes: solo contratable/visible si estado = Publicado (RN-03).

AGG-Experiencia (C1)
- Raíz: Experiencia
- Entidades internas: Componente
- Value Objects: PerfilApto (referencia a PerfilCliente con requisitos)
- Invariantes: cada Componente es incluido u opcional sin ambigüedad (RN-04);
  expone PrecioReferencia y, si varía, BaseDeCalculo (RN-05); solo
  contratable/visible si estado = Publicado (RN-03).

AGG-Articulo (C1)
- Raíz: Articulo
- Invariantes: visible solo si estado = Publicado.

AGG-Consulta (C2)
- Raíz: Consulta
- Entidades internas: CambioEstadoConsulta (historial)
- Invariantes: registra Canal e Idioma; toda transición de estado se historiza (RN-01, RN-02).

AGG-SolicitudCotizacion (C2)
- Raíz: SolicitudCotizacion
- Entidades internas: Cotizacion (0..1), CambioEstadoCotizacion (historial)
- Invariantes: requiere organización, número de personas, destino y fecha para
  registrarse (RN-10); la Cotizacion emitida queda asociada a la solicitud (RN-11).

AGG-Guia (C3)
- Raíz: Guia
- Value Objects: — (Credencial se referencia)
- Invariantes: —

AGG-Credencial (C3)
- Raíz: Credencial
- Invariantes: identifica entidad emisora y titular (negocio o Guía) (RN-16).

AGG-Resena (C3)
- Raíz: Resena
- Invariantes: su ausencia no impide mostrar la página asociada (RN-17).

AGG-Faq (C3)
- Raíz: Faq
- Invariantes: —

AGG-Reserva (C4) — agregado transaccional
- Raíz: Reserva
- Entidades internas: Pago (1-1), Confirmacion (0..1), CambioEstadoReserva (historial)
- Invariantes: participantes ≤ cupo disponible (RN-06); Confirmada solo si Pago =
  Exitoso (RN-08); genera Confirmacion al confirmarse (RN-09).

AGG-Disponibilidad (C4)
- Raíz: Disponibilidad
- Invariantes: cupoDisponible ≤ cupoTotal y cupoDisponible ≥ 0; al confirmarse/
  cancelarse una Reserva, el cupo se ajusta de forma consistente (RN-07).

AGG-Equipo (C5)
- Raíz: Equipo
- Invariantes: condición de venta = bajo pedido/importación; visible solo si Publicado.

AGG-PedidoEquipo (C5)
- Raíz: PedidoEquipo
- Entidades internas: LineaPedidoEquipo, CambioEstadoPedido (historial)
- Invariantes: informa sujeción a disponibilidad de importación (RN-12).

AGG-ActorOperativo (CX)
- Raíz: ActorOperativo
- Invariantes: acciones de gestión requieren autenticación y rol autorizado (RN-15).

==================================================================
ENTITIES
==================================================================

(Formato: Entidad (Contexto) · [rol] · atributos: nombre: Tipo)

Destino (C1) · [AR]
- id: Id
- nombre: Text
- sede: Ref(Sede)
- descripcion: LongText
- idiomas: List<Enum(Idioma)>
- estadoPublicacion: Enum(EstadoPublicacion)

Sede (C1) · [Entity]
- id: Id
- nombre: Text
- destino: Ref(Destino)
- referenciaFichaLocal: Text (identificador externo de la ficha local)

Experiencia (C1) · [AR]
- id: Id
- nombre: Text
- destino: Ref(Destino)
- duracion: Text
- dificultad: Enum(NivelDificultad)
- precioReferencia: Money
- baseCalculoPrecio: Enum(BaseCalculoPrecio)
- condiciones: LongText
- perfilesAptos: List<Enum(PerfilCliente)>
- edadMinima: Int (opcional)
- nivelEsfuerzo: Enum(NivelEsfuerzo) (opcional)
- estadoPublicacion: Enum(EstadoPublicacion)

Componente (C1) · [Entity interna de Experiencia]
- id: Id
- experiencia: Ref(Experiencia)
- nombre: Text
- tipo: Enum(TipoComponente)
- condicion: Enum(CondicionComponente)

ActivoMultimedia (C1) · [Entity]
- id: Id
- tipo: Enum(TipoMultimedia)
- referenciaAlmacenamiento: Text
- destino: Ref(Destino) (opcional)
- experiencia: Ref(Experiencia) (opcional)
- idioma: Enum(Idioma) (opcional)
- caption: Text (opcional)

Articulo (C1) · [AR]
- id: Id
- titulo: Text
- cuerpo: LongText
- idioma: Enum(Idioma)
- relacionados: List<Ref(Destino|Experiencia)>
- estadoPublicacion: Enum(EstadoPublicacion)

Consulta (C2) · [AR]
- id: Id
- nombreContacto: Text
- medioContacto: Text
- destinoInteres: Ref(Destino) (opcional)
- mensaje: LongText
- canal: Enum(Canal)
- idioma: Enum(Idioma)
- fecha: DateTime
- estado: Enum(EstadoConsulta)
- historial: List<CambioEstadoConsulta>

CambioEstadoConsulta (C2) · [Entity interna]
- id: Id
- estadoAnterior: Enum(EstadoConsulta)
- estadoNuevo: Enum(EstadoConsulta)
- fecha: DateTime
- actor: Ref(ActorOperativo) (opcional)

SolicitudCotizacion (C2) · [AR]
- id: Id
- organizacion: Text
- numeroPersonas: Int
- destino: Ref(Destino)
- fechaSolicitada: Date
- datosContacto: Text
- estado: Enum(EstadoCotizacion)
- cotizacion: Cotizacion (0..1)
- historial: List<CambioEstadoCotizacion>

Cotizacion (C2) · [Entity dependiente]
- id: Id
- solicitud: Ref(SolicitudCotizacion)
- contenido: LongText
- fechaEmision: DateTime
- autor: Ref(ActorOperativo)

Guia (C3) · [AR]
- id: Id
- nombre: Text
- experiencia: LongText
- credenciales: List<Ref(Credencial)>

Credencial (C3) · [AR]
- id: Id
- tipo: Text
- entidadEmisora: Text
- titularTipo: Enum(TitularCredencial)
- titularRef: Ref(Guia) (opcional; null = titular negocio)

Resena (C3) · [AR]
- id: Id
- autor: Text
- valoracion: Int
- texto: LongText
- objetivo: Ref(Destino|Experiencia) (opcional)
- fecha: DateTime

Faq (C3) · [AR]
- id: Id
- pregunta: Text
- respuesta: LongText
- categoria: Text
- idioma: Enum(Idioma)

Reserva (C4) · [AR]
- id: Id
- experiencia: Ref(Experiencia)
- disponibilidad: Ref(Disponibilidad)
- fecha: Date
- numeroParticipantes: Int
- datosCliente: Text
- estado: Enum(EstadoReserva)
- pago: Pago (1-1)
- confirmacion: Confirmacion (0..1)
- historial: List<CambioEstadoReserva>

Disponibilidad (C4) · [AR]
- id: Id
- experiencia: Ref(Experiencia)
- fecha: Date
- cupoTotal: Int
- cupoDisponible: Int

Pago (C4) · [Entity dependiente de Reserva]
- id: Id
- reserva: Ref(Reserva)
- monto: Money
- estado: Enum(EstadoPago)
- referenciaPasarela: Text (token; sin datos sensibles de tarjeta)
- fecha: DateTime

Confirmacion (C4) · [Entity dependiente de Reserva]
- id: Id
- reserva: Ref(Reserva)
- detalle: LongText
- fechaEmision: DateTime

Equipo (C5) · [AR]
- id: Id
- nombre: Text
- descripcion: LongText
- tipo: Text
- segmentosAplicables: List<Enum(PerfilCliente)>
- condicionVenta: Enum(CondicionVentaEquipo)
- estadoPublicacion: Enum(EstadoPublicacion)

PedidoEquipo (C5) · [AR]
- id: Id
- lineas: List<LineaPedidoEquipo>
- tipoCliente: Enum(PerfilCliente)
- datosContacto: Text
- estadoImportacion: Enum(EstadoPedidoEquipo)
- fecha: DateTime
- historial: List<CambioEstadoPedido>

LineaPedidoEquipo (C5) · [Entity interna de PedidoEquipo]
- id: Id
- equipo: Ref(Equipo)
- cantidad: Int

ActorOperativo (CX) · [AR]
- id: Id
- nombre: Text
- rol: Enum(RolOperativo)
(credenciales de acceso fuera del modelo de negocio: contexto de identidad)

CambioEstadoCotizacion / CambioEstadoReserva / CambioEstadoPedido (varios) · [Entity interna]
- id: Id
- estadoAnterior: Enum(correspondiente)
- estadoNuevo: Enum(correspondiente)
- fecha: DateTime
- actor: Ref(ActorOperativo) (opcional)

==================================================================
VALUE OBJECTS
==================================================================

- Money: { monto: Decimal, moneda: Enum(Moneda) }
- DatosContacto: { nombre: Text, email: Text (opcional), telefono: Text (opcional) }
  (usado por Consulta, SolicitudCotizacion, PedidoEquipo, Reserva)
- RequisitosParticipacion: { edadMinima: Int, nivelEsfuerzo: Enum(NivelEsfuerzo) }
  (asociado a Experiencia / PerfilApto)
- PerfilApto: { perfil: Enum(PerfilCliente), requisitos: RequisitosParticipacion (opcional) }
- PeriodoVigencia: { desde: Date, hasta: Date (opcional) }
  (uso potencial en Disponibilidad por temporada — marcado como evolución)
- ReferenciaContenido: { tipo: Enum(TipoContenidoRef), id: Id }
  (para relaciones opcionales Destino|Experiencia en Articulo/Resena/Multimedia)

Nota: Money se trata como VO inmutable; las cantidades monetarias del dominio
(precioReferencia, monto de Pago) lo utilizan.

==================================================================
ENUMS
==================================================================

Idioma: ES | EN
Canal: WHATSAPP | FORMULARIO | OTRO
EstadoPublicacion: BORRADOR | PUBLICADO | NO_DISPONIBLE
NivelDificultad: II | III | IV | MIXTO   (clases de aguas bravas; MIXTO si abarca varias)
NivelEsfuerzo: BAJO | MEDIO | ALTO
TipoComponente: ALIMENTACION | HOSPEDAJE | EQUIPO | TRANSPORTE | GUIA
CondicionComponente: INCLUIDO | OPCIONAL
BaseCalculoPrecio: FIJO | POR_PERSONA | POR_TEMPORADA
PerfilCliente: FAMILIA | NINO | ADULTO_MAYOR | PRINCIPIANTE | CORPORATIVO | PRACTICANTE | TURISTA_NACIONAL | TURISTA_INTERNACIONAL
TipoMultimedia: FOTO | VIDEO
TipoContenidoRef: DESTINO | EXPERIENCIA
EstadoConsulta: NUEVA | EN_PROCESO | RESPONDIDA | CERRADA
EstadoCotizacion: RECIBIDA | EN_PROCESO | COTIZACION_EMITIDA | CERRADA
EstadoReserva: INICIADA | PAGADA | CONFIRMADA | NO_CONFIRMADA | CANCELADA | COMPLETADA
EstadoPago: INICIADO | EXITOSO | FALLIDO | RECHAZADO
EstadoDisponibilidad: DISPONIBLE | PARCIAL | AGOTADO   (derivable de cupoDisponible)
EstadoPedidoEquipo: RECIBIDO | EN_COTIZACION | CONFIRMADO | EN_IMPORTACION | ENTREGADO | CANCELADO
CondicionVentaEquipo: BAJO_PEDIDO | IMPORTACION
TitularCredencial: NEGOCIO | GUIA
RolOperativo: RESPONSABLE_COMERCIAL | GESTOR_OPERACIONES
Moneda: PEN | USD

==================================================================
DOMAIN EVENTS
==================================================================

(Formato: Evento · agregado origen · payload conceptual)

- ConsultaRecibida · Consulta · { consultaId, canal, idioma, destinoInteres? }
- ConsultaCambioEstado · Consulta · { consultaId, estadoNuevo }
- SolicitudCotizacionRecibida · SolicitudCotizacion · { solicitudId, destino }
- CotizacionEmitida · SolicitudCotizacion · { solicitudId, cotizacionId }
- SolicitudCotizacionCambioEstado · SolicitudCotizacion · { solicitudId, estadoNuevo }
- DisponibilidadConsultada · Disponibilidad · { experienciaId, fecha }
- ReservaIniciada · Reserva · { reservaId, experienciaId, fecha, numeroParticipantes }
- PagoIniciado · Reserva · { reservaId, pagoId, monto }
- PagoConfirmado · Reserva · { reservaId, pagoId }
- PagoRechazado · Reserva · { reservaId, pagoId, motivo? }
- ReservaConfirmada · Reserva · { reservaId } (desencadena ConfirmacionEmitida)
- ConfirmacionEmitida · Reserva · { reservaId, confirmacionId }
- ReservaCancelada · Reserva · { reservaId }
- CupoActualizado · Disponibilidad · { disponibilidadId, cupoDisponible }
- PedidoEquipoRecibido · PedidoEquipo · { pedidoId }
- PedidoEquipoCambioEstadoImportacion · PedidoEquipo · { pedidoId, estadoNuevo }
- ContenidoPublicado · (Destino|Experiencia|Articulo|Equipo) · { tipo, id }
- ContenidoDespublicado · (Destino|Experiencia|Articulo|Equipo) · { tipo, id }

==================================================================
RELATIONSHIPS
==================================================================

(Formato: Origen — cardinalidad — Destino · tipo)

- Destino — 1-N — Experiencia · composición lógica por contexto
- Destino — 1-1 — Sede · asociación
- Experiencia — 1-N — Componente · composición (interna del agregado)
- Experiencia — N-N — PerfilCliente · asociación (perfilesAptos)
- Destino — 1-N — ActivoMultimedia · asociación
- Experiencia — 1-N — ActivoMultimedia · asociación
- Destino — N-N — Articulo · asociación
- Consulta — N-1 — Destino · ref. opcional
- SolicitudCotizacion — 1-0..1 — Cotizacion · composición
- SolicitudCotizacion — N-1 — Destino · ref.
- Guia — N-N — Credencial · asociación
- Negocio — 1-N — Credencial · titularidad (titularTipo = NEGOCIO)
- Resena — N-1 — Destino|Experiencia · ref. opcional
- Experiencia — 1-N — Disponibilidad · asociación
- Reserva — N-1 — Experiencia · ref.
- Reserva — N-1 — Disponibilidad · ref. (por experiencia+fecha)
- Reserva — 1-1 — Pago · composición (agregado transaccional)
- Reserva — 1-0..1 — Confirmacion · composición
- Equipo — N-N — PerfilCliente · asociación (segmentosAplicables)
- PedidoEquipo — 1-N — LineaPedidoEquipo · composición
- LineaPedidoEquipo — N-1 — Equipo · ref.
- ActorOperativo — 1-N — Consulta|SolicitudCotizacion|Reserva|PedidoEquipo · operativa (gestión)

==================================================================
BUSINESS RULES MAPPING
==================================================================

(Formato: Regla · agregado/entidad que la hace cumplir · origen US/AC)

- RN-01 · Consulta · US-003, US-004, US-006
- RN-02 · Consulta, SolicitudCotizacion, Reserva, PedidoEquipo (historial) · US-005, US-027, US-034, US-038
- RN-03 · Destino, Experiencia (estadoPublicacion) · US-007
- RN-04 · Componente (condicion) · US-008
- RN-05 · Experiencia (precioReferencia, baseCalculoPrecio) · US-010
- RN-06 · Reserva ↔ Disponibilidad (invariante) · US-031
- RN-07 · Disponibilidad (cupoDisponible) · US-031
- RN-08 · Reserva ↔ Pago (invariante) · US-032
- RN-09 · Reserva → Confirmacion · US-033
- RN-10 · SolicitudCotizacion (validación de campos) · US-026
- RN-11 · Cotizacion ↔ SolicitudCotizacion · US-027
- RN-12 · Equipo, PedidoEquipo (condicionVenta) · US-036
- RN-13 · Equipo (segmentosAplicables) · US-037
- RN-14 · Contenido (Destino/Experiencia/Articulo) idioma · US-020
- RN-15 · ActorOperativo (autorización) · US-004, US-034
- RN-16 · Credencial (entidadEmisora, titular) · US-011, US-012
- RN-17 · Resena (opcionalidad) · US-014

==================================================================
GLOSSARY MAPPING
==================================================================

(Formato: Término del Glosario [domain-model] → Objeto del catálogo)

- Destino (Río) → Destino (AGG-Destino)
- Sede → Sede
- Experiencia (Paquete) → Experiencia (AGG-Experiencia)
- Componente → Componente
- Perfil de Cliente → Enum(PerfilCliente) + VO PerfilApto
- Visitante → (actor externo; no entidad persistida)
- Consulta (Lead) → Consulta (AGG-Consulta)
- Solicitud de Cotización → SolicitudCotizacion (AGG-SolicitudCotizacion)
- Cotización → Cotizacion
- Reserva → Reserva (AGG-Reserva)
- Disponibilidad (Cupo) → Disponibilidad (AGG-Disponibilidad)
- Pago → Pago
- Confirmación → Confirmacion
- Guía → Guia (AGG-Guia)
- Credencial (Certificación/Licencia) → Credencial (AGG-Credencial)
- Reseña → Resena (AGG-Resena)
- Artículo → Articulo (AGG-Articulo)
- Activo Multimedia → ActivoMultimedia
- Equipo → Equipo (AGG-Equipo)
- Pedido de Equipo → PedidoEquipo (AGG-PedidoEquipo)
- Actor Operativo → ActorOperativo (AGG-ActorOperativo)
- Canal → Enum(Canal)
- Idioma → Enum(Idioma)
- FAQ → Faq (AGG-Faq)

==================================================================
NOTA DE TRAZABILIDAD
==================================================================
El catálogo es una reexpresión estructurada del domain-model.md (mismos
contextos, entidades y reglas), preparada para consumo por Backend, Database,
API y QA. No introduce entidades ni reglas nuevas; solo formaliza tipos lógicos,
enums y mapeos.

Observación heredada: la capacidad de gestión de pedidos de equipos (PedidoEquipo)
proviene de la feature con doble numeración FT-037; se recomienda renumerarla a
FT-038 para conservar IDs únicos en la trazabilidad.