# UI SPECIFICATION — Rafting Peru & Kayak Adventures
UX/UI Designer · CypData · Junio 2026
Entradas: architecture-v1.md · technical-architecture-v1.md · sprint-backlog.md · user-stories.md · acceptance-criteria.md
Especificación de experiencia y diseño funcional. Sin código, arquitectura técnica, diseño visual detallado, selección de colores, gráfica ni priorización.

==================================================================
RESUMEN UX
==================================================================

La experiencia se diseña alrededor de un único objetivo: convertir al visitante
en consulta o reserva con la menor fricción posible. Tres principios rectores,
derivados de los criterios de aceptación y la arquitectura:

1. Contacto siempre a un toque. El acceso a WhatsApp y a la consulta es
   persistente en toda la navegación (US-001), porque la captación es la
   palanca de mayor valor.
2. Transparencia que reduce la duda. La oferta comunica con claridad qué
   incluye y qué es opcional, precios de referencia y condiciones (US-008,
   US-010), y exhibe señales de confianza antes de pedir cualquier acción
   (US-011 a US-015).
3. Una sola experiencia, dos idiomas y muchos perfiles. El sitio es bilingüe
   (US-020) y permite que cada perfil (familia, principiante, adulto mayor,
   corporativo, practicante) encuentre rápido lo suyo (US-023).

El recorrido típico: el visitante descubre por un destino o por una búsqueda,
llega a una página de río o paquete, valida confianza y claridad, y convierte
por WhatsApp/formulario (R1) o, cuando exista, por reserva en línea (R3).

==================================================================
SITEMAP
==================================================================

Nivel público (capa A)
- Inicio
- Destinos (índice de los 4 ríos)
  - Destino: Urubamba (Cusco)
  - Destino: Lunahuaná (Lima)
  - Destino: Cotahuasi (Arequipa)
  - Destino: Marañón (norte)
- Experiencias / Paquetes (índice con filtros)
  - Detalle de experiencia / paquete
  - Comparador de paquetes
- Aventura para todos (navegación por perfil: familias, niños, adultos mayores, principiantes)
- Cursos (kayak / rafting)
- Seguridad y confianza
  - Certificaciones y licencias
  - Equipo de guías
  - Protocolos de seguridad
  - Preguntas frecuentes (FAQ)
- Contenido
  - Galería (fotos y video por destino)
  - Blog / guías
- Empresas y grupos (B2B)
  - Propuesta corporativa / team building
  - Solicitud de cotización para grupos
  - Material descargable para agencias
- Tienda de equipos
  - Catálogo de equipos
  - Detalle de equipo
  - Solicitud de cotización / pedido
- Contacto
- Selector de idioma (ES/EN, transversal)
- Acceso conversacional (WhatsApp, persistente y transversal)

Nivel transaccional (capa B, R3)
- Flujo de reserva (disponibilidad → datos → pago → confirmación)

Nivel back-office (capa C, acceso autenticado)
- Bandeja de consultas / leads
- Gestión de cotizaciones
- Gestión de reservas
- Gestión de pedidos de equipos

==================================================================
FLUJOS DE USUARIO
==================================================================

F1 · Captación por WhatsApp (US-001, US-002)
Visitante en cualquier página → toca el acceso persistente de WhatsApp →
se abre conversación con mensaje precargado (contextual al destino si venía de
una página de río/paquete; genérico si no) → continúa en WhatsApp.

F2 · Captación por formulario (US-003, US-006)
Visitante abre formulario de consulta → completa nombre, contacto, destino de
interés y mensaje → si hay campos inválidos, se señalan y no se envía → al
enviar, confirmación visible de recepción. El flujo respeta el idioma activo.

F3 · Exploración y decisión de oferta (US-007, US-008, US-009, US-010)
Visitante entra a Experiencias → filtra por destino/dificultad/perfil → abre
un detalle → revisa incluido/opcional, precio de referencia y condiciones →
opcional: agrega a comparación y contrasta paquetes → convierte por F1/F2.

F4 · Validación de confianza (US-011 a US-015)
Desde un detalle o el menú → Seguridad y confianza → revisa certificaciones,
guías, protocolos, reseñas y FAQ → vuelve al punto de conversión con la duda
resuelta.

F5 · Descubapacidad por destino (US-019, US-021, US-022)
Visitante llega desde buscador/IA/ficha local a la página del río → contenido
específico del destino + experiencias disponibles + vía de contacto.

F6 · Exploración segmentada por perfil (US-023, US-024, US-025)
Visitante elige su perfil en "Aventura para todos" → ve solo experiencias
adecuadas con edad mínima y nivel de esfuerzo → recibe recomendaciones → abre
detalle.

F7 · Cotización de grupos / B2B (US-026, US-027, US-028, US-029)
Cliente corporativo entra a Empresas y grupos → revisa propuesta → completa
solicitud de cotización (organización, personas, destino, fecha) → confirmación
de recepción. (Lado interno: responsable comercial gestiona y responde.)

F8 · Reserva y pago en línea (US-030 a US-033) — R3
Cliente en un detalle reservable → consulta disponibilidad/calendario →
selecciona fecha con cupo y número de participantes → ingresa datos → paga →
recibe confirmación automática con el detalle. Si el pago falla, la reserva no
se confirma y se informa el resultado.

F9 · Catálogo y pedido de equipos (US-035, US-036, US-037) — R3
Practicante/club entra a Tienda de equipos → filtra por tipo de cliente → abre
ficha (con condición de venta bajo pedido) → solicita cotización/pedido → se le
informa que está sujeto a disponibilidad de importación → confirmación.

F10 · Operación interna (US-004, US-005, US-027, US-034, US-038)
Actor operativo autenticado → bandeja correspondiente (consultas, cotizaciones,
reservas o pedidos) → abre un ítem → actualiza estado → el cambio queda
registrado e historizado.

==================================================================
PANTALLAS
==================================================================

Públicas
- P01 Inicio: propuesta de valor, accesos a destinos/experiencias, señales de
  confianza resumidas, acceso de contacto persistente.
- P02 Índice de Destinos: los 4 ríos con entrada a cada uno.
- P03 Página de Destino/Río: contenido propio del destino, experiencias
  disponibles, galería, vía de contacto (US-019, US-016).
- P04 Índice de Experiencias/Paquetes: listado con filtros (destino, dificultad,
  perfil) y estado de disponibilidad (US-007).
- P05 Detalle de Experiencia/Paquete: incluido/opcional, precio de referencia y
  condiciones, acción de consulta/reserva, acceso a comparación (US-008, US-010).
- P06 Comparador de Paquetes: comparación lado a lado (US-009).
- P07 Aventura para todos: navegación por perfil con requisitos visibles
  (US-023, US-024, US-025).
- P08 Seguridad y Confianza: certificaciones, guías, protocolos (US-011, US-012,
  US-013).
- P09 FAQ: con búsqueda de términos (US-015).
- P10 Reseñas y Testimonios: valoraciones, eventualmente por destino/experiencia
  (US-014).
- P11 Galería de contenido y Blog: multimedia por destino y artículos (US-016,
  US-017, US-018).
- P12 Empresas y Grupos (B2B): propuesta de valor + acceso a cotización +
  material descargable (US-028, US-029).
- P13 Solicitud de Cotización de Grupos: formulario (US-026).
- P14 Catálogo de Equipos: listado con filtro por tipo de cliente (US-035,
  US-037).
- P15 Detalle de Equipo: ficha + condición bajo pedido + solicitud (US-036).
- P16 Solicitud de Cotización/Pedido de Equipo: formulario (US-036).
- P17 Contacto / Consulta: formulario general de consulta (US-003).

Transaccionales (R3)
- P18 Disponibilidad/Calendario de una experiencia (US-030).
- P19 Reserva: selección de fecha y participantes + datos (US-031).
- P20 Pago (US-032).
- P21 Confirmación de Reserva (US-033).

Back-office (autenticado)
- P22 Bandeja de Consultas/Leads (US-004, US-005).
- P23 Gestión de Cotizaciones (US-027).
- P24 Gestión de Reservas (US-034).
- P25 Gestión de Pedidos de Equipos (US-038).
- P26 Acceso / Autenticación interna.

Transversal
- Selector de idioma ES/EN en todas las pantallas públicas (US-020).
- Acceso de contacto persistente (WhatsApp) en todas las pantallas públicas.

==================================================================
COMPONENTES
==================================================================

De navegación y estructura
- Encabezado con navegación principal y selector de idioma.
- Pie de página con accesos a confianza, contacto y secciones clave.
- Botón/acceso flotante de WhatsApp persistente (con contexto de destino).
- Migas de pan en secciones profundas (destinos, experiencias, equipos).

De contenido y oferta
- Tarjeta de destino (resumen del río con acceso al detalle).
- Tarjeta de experiencia/paquete (destino, duración, dificultad, precio de
  referencia, estado de disponibilidad).
- Bloque "incluido / opcional / no incluye" con distinción visual entre
  categorías (sin definir el tratamiento gráfico).
- Tabla/panel comparador de paquetes.
- Filtros (por destino, dificultad y perfil de cliente).
- Selector/segmentador de perfil de cliente.
- Indicador de requisitos (edad mínima, nivel de esfuerzo).
- Galería multimedia con carga progresiva.
- Bloque embebido de redes sociales (con degradación elegante).
- Listado de artículos/guías.

De confianza
- Bloque de certificaciones/licencias (con entidad emisora).
- Tarjeta de perfil de guía.
- Bloque de protocolos de seguridad.
- Componente de reseñas/testimonios.
- Acordeón de FAQ con búsqueda.

De captación y transacción
- Formulario de consulta.
- Formulario de cotización de grupos.
- Formulario de cotización/pedido de equipos.
- Calendario/selector de disponibilidad (R3).
- Resumen de reserva (R3).
- Componente de pago (R3, delegado a pasarela).
- Mensaje/pantalla de confirmación.

De back-office
- Bandeja/listado con filtros por estado.
- Vista de detalle de ítem (consulta, cotización, reserva, pedido).
- Control de cambio de estado.
- Indicador de historial de estados.

Transversales de feedback
- Notificaciones/avisos (éxito, error, información).
- Indicadores de carga.
- Mensajes de estado vacío.
- Selector de idioma.

==================================================================
FORMULARIOS
==================================================================

FORM-1 · Consulta general (US-003)
Campos: nombre, medio de contacto (correo o teléfono), destino de interés
(selección), mensaje. Idioma según selección activa.
Reglas visibles: campos requeridos marcados; el envío se bloquea si hay
requeridos vacíos o con formato inválido, señalando cada campo; al enviar con
éxito, confirmación visible de recepción.

FORM-2 · Solicitud de cotización de grupos (US-026)
Campos: organización, número de personas, destino, fecha; datos de contacto.
Reglas visibles: requeridos marcados; si faltan datos, se señalan y no se
registra; al completar, confirmación de recepción.

FORM-3 · Solicitud de cotización/pedido de equipo (US-036)
Campos: equipo (precargado desde la ficha), cantidad, tipo de cliente
(practicante/club/escuela), datos de contacto.
Reglas visibles: aviso explícito de "sujeto a disponibilidad de importación";
confirmación de recepción al enviar.

FORM-4 · Reserva (US-031, US-032) — R3
Campos: experiencia (precargada), fecha (de disponibilidad), número de
participantes, datos del cliente, datos de pago (gestionados por la pasarela).
Reglas visibles: no permite reservar más participantes que los cupos
disponibles, informando el cupo; si el pago falla, no confirma y muestra el
resultado.

FORM-5 · Acceso interno (back-office)
Campos: identificación de usuario y credencial. Reglas: acceso solo a roles
autorizados.

Convención transversal de formularios: etiquetas claras, indicación de campos
requeridos, validación con mensajes específicos por campo, preservación de los
datos ya ingresados ante un error, y confirmación explícita de resultado.

==================================================================
ESTADOS DE ERROR
==================================================================

- Validación de formulario: mensaje por campo, foco al primer campo con error,
  datos ingresados preservados, envío bloqueado hasta corregir (FORM-1 a FORM-4).
- Cupo insuficiente en reserva: se impide confirmar y se informa el cupo
  disponible (US-031).
- Pago fallido o rechazado: la reserva no se confirma; se muestra el resultado y
  se ofrece reintentar (US-032).
- Contenido externo no disponible (redes sociales): la página se muestra
  correctamente sin el bloque, sin romper el diseño (US-017).
- Contacto fuera de horario (WhatsApp): mensaje con horario y tiempo estimado de
  respuesta (US-001).
- Sin resultados / sin oferta para un perfil o filtro: ver Estados Vacíos.
- Error general de carga: aviso no intrusivo con opción de reintentar.

==================================================================
ESTADOS VACÍOS
==================================================================

- Filtro/perfil sin coincidencias: mensaje indicando que no hay experiencias
  para ese perfil o criterio, con sugerencia de ajustar filtros (US-023, US-007).
- Calendario sin cupos: las fechas sin disponibilidad se muestran como no
  disponibles; si no hay ninguna, mensaje orientando a consultar por otros medios
  (US-030).
- Bandejas de back-office sin ítems: mensaje de "sin consultas/cotizaciones/
  reservas/pedidos" según corresponda.
- Sección de reseñas sin contenido aún: mensaje neutro, sin romper la página
  (US-014).
- Catálogo/destino sin elementos publicados: mensaje informativo de contenido en
  preparación.

==================================================================
ESTADOS DE CARGA
==================================================================

- Contenido multimedia: carga progresiva, sin bloquear el resto de la página
  (US-016).
- Listados (experiencias, catálogo, bandejas): indicador de carga y, cuando
  aplique, presentación incremental de resultados.
- Consulta de disponibilidad (R3): indicador mientras se resuelve el calendario.
- Envío de formularios: indicador de envío en progreso y bloqueo de reenvío
  hasta obtener respuesta.
- Procesamiento de pago (R3): indicador de operación en curso; el usuario no
  puede duplicar el intento mientras se procesa.

==================================================================
CONSIDERACIONES RESPONSIVE
==================================================================

- Prioridad móvil: el comportamiento del cliente es mayoritariamente móvil; las
  pantallas se conciben primero para pantallas pequeñas y luego se amplían.
- Acceso de contacto persistente: el acceso a WhatsApp/consulta permanece
  visible y alcanzable con una mano en móvil, sin obstruir el contenido.
- Filtros y comparador: en móvil, los filtros se presentan en panel desplegable;
  el comparador adopta una vista apilada/desplazable en lugar de columnas
  paralelas.
- Formularios: campos a una columna en móvil, con teclados y controles
  adecuados al tipo de dato.
- Galería y multimedia: adaptación a ancho disponible con carga progresiva.
- Navegación: menú colapsable en móvil; selector de idioma siempre accesible.
- Back-office: prioriza tablet/escritorio por densidad de información, sin
  excluir el uso móvil para acciones de seguimiento.

==================================================================
NOTA DE TRAZABILIDAD
==================================================================
Cada pantalla, flujo y formulario referencia las User Stories que satisface,
conservando la cadena Oportunidad → Epic → Feature → User Story → Pantalla/Flujo.
No se modificaron User Stories ni prioridades. El diseño visual detallado
(colores, tipografía, tratamiento gráfico) queda fuera del alcance de este
documento, conforme a las restricciones del rol.

Se mantiene la observación heredada del backlog sobre la doble numeración
FT-037 (US-038 / gestión de pedidos de equipos); se recomienda renumerar esa
feature a FT-038 para conservar IDs únicos.