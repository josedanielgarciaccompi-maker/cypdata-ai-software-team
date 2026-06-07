# ACCEPTANCE CRITERIA — Rafting Peru & Kayak Adventures
Business Analyst · CypData · Junio 2026
Entrada: user-stories.md
Criterios de aceptación en formato Gherkin, trazados a cada User Story.
Sin nuevas User Stories, priorización, arquitectura, interfaces ni roadmap.

==================================================================
EP-001 · Captación y gestión de consultas
==================================================================

US-001 — Iniciar conversación por WhatsApp desde cualquier página

AC-001
Dado que estoy navegando en cualquier página del sitio
Cuando observo la pantalla
Entonces veo un acceso visible y persistente para contactar por WhatsApp

AC-002
Dado que el acceso a WhatsApp está visible
Cuando lo selecciono
Entonces se abre una conversación de WhatsApp con el número del negocio

AC-003
Dado que abro la conversación de WhatsApp
Cuando esto ocurre fuera del horario de atención
Entonces recibo un mensaje que indica el horario y el tiempo estimado de respuesta

---

US-002 — Mensaje de WhatsApp precargado con el destino visto

AC-001
Dado que estoy viendo la página de un destino o experiencia
Cuando inicio la conversación por WhatsApp
Entonces el mensaje viene precargado con la referencia a ese destino o experiencia

AC-002
Dado que inicio la conversación desde una página general sin destino específico
Cuando se abre WhatsApp
Entonces el mensaje precargado contiene un texto genérico de consulta

---

US-003 — Enviar consulta mediante formulario

AC-001
Dado que estoy en una página con formulario de consulta
Cuando completo los campos requeridos (nombre, contacto, destino de interés, mensaje) y envío
Entonces el sistema confirma que la consulta fue enviada

AC-002
Dado que intento enviar el formulario con campos requeridos vacíos o inválidos
Cuando presiono enviar
Entonces el sistema impide el envío y señala los campos a corregir

AC-003
Dado que mi consulta fue enviada correctamente
Cuando finaliza el envío
Entonces recibo una confirmación de recepción de la consulta

---

US-004 — Ver consultas centralizadas

AC-001
Dado que soy responsable comercial autenticado
Cuando accedo a la bandeja de consultas
Entonces veo todas las consultas recibidas por los distintos canales en un único listado

AC-002
Dado que estoy en el listado de consultas
Cuando reviso una consulta
Entonces veo sus datos asociados (canal de origen, datos de contacto, destino de interés, mensaje y fecha)

AC-003
Dado que ingresa una nueva consulta
Cuando se registra en el sistema
Entonces aparece en el listado sin necesidad de recargar manualmente la información existente

---

US-005 — Registrar y actualizar el estado de la consulta

AC-001
Dado que soy responsable comercial y abro una consulta
Cuando le asigno un estado (nueva, respondida, en proceso, cerrada)
Entonces el sistema guarda el estado seleccionado

AC-002
Dado que una consulta tiene un estado asignado
Cuando consulto su historial
Entonces veo los cambios de estado registrados

AC-003
Dado que filtro el listado por un estado
Cuando aplico el filtro
Entonces solo se muestran las consultas con ese estado

---

US-006 — Atención en el idioma del visitante

AC-001
Dado que soy un visitante con el sitio en inglés
Cuando inicio una consulta
Entonces los textos del flujo de consulta se presentan en inglés

AC-002
Dado que envío una consulta indicando un idioma de preferencia
Cuando el responsable comercial la recibe
Entonces la consulta muestra el idioma de preferencia del visitante

==================================================================
EP-002 · Transparencia y comunicación de la oferta
==================================================================

US-007 — Ver paquetes y experiencias por destino y dificultad

AC-001
Dado que estoy en la sección de paquetes
Cuando la visito
Entonces veo las experiencias disponibles con su destino, duración y nivel de dificultad

AC-002
Dado que estoy en la sección de paquetes
Cuando filtro por destino o por nivel de dificultad
Entonces el listado muestra solo los paquetes que cumplen el filtro

AC-003
Dado que un paquete no está disponible
Cuando reviso el listado
Entonces ese paquete no aparece como contratable

---

US-008 — Ver qué incluye y qué es opcional por paquete

AC-001
Dado que abro el detalle de un paquete
Cuando reviso su contenido
Entonces veo de forma explícita los componentes incluidos y los opcionales (alimentación, hospedaje, equipo, transporte, guía)

AC-002
Dado que un componente es opcional
Cuando lo reviso en el detalle
Entonces se distingue claramente de los componentes incluidos

---

US-009 — Comparar paquetes

AC-001
Dado que estoy viendo los paquetes
Cuando selecciono dos o más para comparar
Entonces veo sus características presentadas una al lado de la otra

AC-002
Dado que estoy comparando paquetes
Cuando reviso la comparación
Entonces puedo identificar las diferencias en componentes, duración y precio

---

US-010 — Ver precios de referencia y condiciones

AC-001
Dado que abro el detalle de un paquete
Cuando lo reviso
Entonces veo su precio de referencia y las condiciones aplicables

AC-002
Dado que un paquete tiene precio variable según número de personas o temporada
Cuando reviso el precio
Entonces se indica la base sobre la que se calcula

==================================================================
EP-003 · Confianza, seguridad y credibilidad
==================================================================

US-011 — Ver licencias y certificaciones del operador

AC-001
Dado que estoy en la sección de seguridad o nosotros
Cuando la visito
Entonces veo las licencias institucionales (MINCETUR, INDECI) y certificaciones del negocio

AC-002
Dado que reviso una certificación
Cuando la observo
Entonces se identifica la entidad que la otorga

---

US-012 — Conocer al equipo de guías y sus credenciales

AC-001
Dado que estoy en la sección del equipo de guías
Cuando la visito
Entonces veo los perfiles de los guías con su experiencia y certificaciones

AC-002
Dado que reviso el perfil de un guía
Cuando lo abro
Entonces veo al menos su nombre, experiencia y credenciales de seguridad o rescate

---

US-013 — Conocer protocolos y equipo de seguridad

AC-001
Dado que estoy en la sección de seguridad
Cuando la visito
Entonces veo los protocolos aplicados y el equipo de seguridad utilizado durante las actividades

AC-002
Dado que soy una familia interesada
Cuando reviso la información de seguridad
Entonces encuentro las medidas aplicables a niños y adultos mayores

---

US-014 — Leer reseñas y testimonios

AC-001
Dado que estoy en una página con reseñas
Cuando la visito
Entonces veo valoraciones y testimonios de clientes

AC-002
Dado que existen reseñas asociadas a un destino o experiencia
Cuando reviso ese destino o experiencia
Entonces veo las reseñas relacionadas

---

US-015 — Encontrar respuestas en preguntas frecuentes

AC-001
Dado que estoy en la sección de preguntas frecuentes
Cuando la visito
Entonces veo respuestas a dudas habituales sobre seguridad, requisitos, clima y políticas de cancelación

AC-002
Dado que estoy en la sección de preguntas frecuentes
Cuando busco un término
Entonces se muestran las preguntas relacionadas con ese término

==================================================================
EP-004 · Contenido y narrativa de marca
==================================================================

US-016 — Ver fotos y videos reales por río

AC-001
Dado que estoy en la página de un río
Cuando la visito
Entonces veo fotos y videos cortos propios de ese destino

AC-002
Dado que se está cargando contenido audiovisual
Cuando aún no termina de cargar
Entonces el contenido se presenta de forma progresiva sin bloquear la página

---

US-017 — Ver contenido de redes sociales en el sitio

AC-001
Dado que estoy en una página con contenido social integrado
Cuando la visito
Entonces veo publicaciones recientes de las redes sociales del negocio

AC-002
Dado que el contenido social no está disponible
Cuando visito la página
Entonces la página se muestra correctamente sin el bloque de redes

---

US-018 — Leer guías y artículos

AC-001
Dado que estoy en la sección de contenido editorial o blog
Cuando la visito
Entonces veo guías y artículos sobre los ríos y las actividades

AC-002
Dado que abro un artículo
Cuando lo leo
Entonces puedo acceder a las experiencias o destinos relacionados mencionados en él

==================================================================
EP-005 · Presencia digital y descubrimiento
==================================================================

US-019 — Encontrar página dedicada por río

AC-001
Dado que busco un río específico (Urubamba, Lunahuaná, Cotahuasi, Marañón)
Cuando accedo a su página
Entonces veo contenido propio y específico de ese destino

AC-002
Dado que estoy en la página de un río
Cuando la reviso
Entonces encuentro las experiencias disponibles en ese destino y una vía de contacto o consulta

---

US-020 — Navegar el sitio en inglés

AC-001
Dado que selecciono el idioma inglés
Cuando navego el sitio
Entonces el contenido principal se presenta en inglés

AC-002
Dado que cambié el idioma a inglés
Cuando navego a otra página
Entonces se mantiene el idioma seleccionado

---

US-021 — Aparecer en buscadores y asistentes de IA

AC-001
Dado que una página de destino o experiencia es publicada
Cuando es rastreada por buscadores
Entonces expone la información estructurada y los metadatos que la describen

AC-002
Dado que el contenido del negocio es relevante para una consulta
Cuando un buscador o asistente lo procesa
Entonces la información del negocio está disponible para ser referenciada

---

US-022 — Encontrar la ficha local por sede

AC-001
Dado que busco al operador en un destino o cerca de una sede
Cuando aparece su ficha local
Entonces veo información de contacto, ubicación y reseñas de esa sede

AC-002
Dado que la información de una sede cambia
Cuando se actualiza la ficha
Entonces la ficha refleja la información vigente

==================================================================
EP-006 · Propuesta segmentada por tipo de cliente
==================================================================

US-023 — Explorar la oferta filtrada por perfil

AC-001
Dado que estoy en la sección de experiencias
Cuando selecciono mi perfil (familia, principiante, adulto mayor, etc.)
Entonces veo solo las experiencias adecuadas para ese perfil

AC-002
Dado que apliqué un filtro por perfil
Cuando ningún paquete cumple el criterio
Entonces se muestra un mensaje indicando que no hay experiencias para ese perfil

---

US-024 — Conocer edad mínima y nivel de esfuerzo

AC-001
Dado que reviso una actividad
Cuando abro su detalle
Entonces veo la edad mínima y el nivel de esfuerzo físico requerido

AC-002
Dado que una actividad tiene requisitos de participación
Cuando reviso su detalle
Entonces los requisitos están indicados de forma visible

---

US-025 — Recibir recomendaciones por perfil

AC-001
Dado que indico mi perfil o preferencias
Cuando solicito recomendaciones
Entonces recibo una selección de experiencias acordes a ese perfil

AC-002
Dado que recibo recomendaciones
Cuando las reviso
Entonces puedo acceder al detalle de cada experiencia recomendada

==================================================================
EP-007 · Atención a grupos, corporativos y canal institucional (B2B)
==================================================================

US-026 — Solicitar cotización para grupos

AC-001
Dado que soy un cliente corporativo o institucional
Cuando completo el formulario de cotización con organización, número de personas, destino y fecha
Entonces el sistema registra la solicitud y confirma su recepción

AC-002
Dado que envío una solicitud de cotización con datos incompletos
Cuando intento enviarla
Entonces el sistema señala los campos faltantes y no registra la solicitud

---

US-027 — Registrar y responder cotizaciones

AC-001
Dado que soy responsable comercial
Cuando accedo a las solicitudes de cotización
Entonces veo las solicitudes recibidas con sus datos

AC-002
Dado que atiendo una solicitud de cotización
Cuando registro y emito la respuesta
Entonces la solicitud queda asociada a su respuesta y a un estado de atención

---

US-028 — Conocer la propuesta de valor corporativa

AC-001
Dado que estoy en la sección para empresas e instituciones
Cuando la visito
Entonces veo la propuesta de valor para corporativos (team building, condiciones por volumen, coordinación)

AC-002
Dado que estoy en la sección corporativa
Cuando deseo avanzar
Entonces encuentro una vía para solicitar una cotización o contacto

---

US-029 — Descargar material informativo B2B

AC-001
Dado que soy una agencia u operadora turística
Cuando accedo al material descargable
Entonces puedo descargar el documento informativo de la oferta

AC-002
Dado que el material informativo es actualizado
Cuando lo descargo
Entonces obtengo la versión vigente

==================================================================
EP-008 · Reserva y pago en línea
==================================================================

US-030 — Consultar disponibilidad y calendario

AC-001
Dado que estoy en una experiencia reservable
Cuando consulto su disponibilidad
Entonces veo las fechas y cupos disponibles

AC-002
Dado que una fecha no tiene cupos
Cuando reviso el calendario
Entonces esa fecha se muestra como no disponible

---

US-031 — Reservar en línea

AC-001
Dado que selecciono una experiencia, una fecha con cupo y un número de participantes
Cuando confirmo la reserva
Entonces el sistema registra la reserva y reduce los cupos disponibles

AC-002
Dado que intento reservar más participantes que los cupos disponibles
Cuando confirmo
Entonces el sistema impide la reserva e informa los cupos disponibles

---

US-032 — Pagar la reserva en línea

AC-001
Dado que tengo una reserva pendiente de pago
Cuando realizo el pago en línea correctamente
Entonces la reserva queda marcada como pagada

AC-002
Dado que un pago no se completa o es rechazado
Cuando finaliza el intento
Entonces la reserva no queda confirmada y se informa el resultado del pago

---

US-033 — Recibir confirmación automática de reserva

AC-001
Dado que mi reserva quedó confirmada y pagada
Cuando se completa el proceso
Entonces recibo automáticamente una confirmación con el detalle de la reserva

AC-002
Dado que recibo la confirmación
Cuando la reviso
Entonces incluye experiencia, fecha, número de participantes y datos de la reserva

---

US-034 — Administrar reservas

AC-001
Dado que soy gestor de operaciones
Cuando accedo a las reservas
Entonces veo las reservas registradas con su estado

AC-002
Dado que una reserva cambia de situación
Cuando actualizo su estado
Entonces el sistema guarda el nuevo estado de la reserva

==================================================================
EP-009 · Comercialización de equipos especializados
==================================================================

US-035 — Ver el catálogo de equipos

AC-001
Dado que estoy en la sección de equipos
Cuando la visito
Entonces veo el catálogo de equipos con su información

AC-002
Dado que abro un equipo del catálogo
Cuando reviso su ficha
Entonces veo su información descriptiva y su condición de venta bajo pedido

---

US-036 — Solicitar cotización o pedido de equipos

AC-001
Dado que estoy viendo un equipo
Cuando solicito su cotización o pedido indicando los datos requeridos
Entonces el sistema registra la solicitud y confirma su recepción

AC-002
Dado que el equipo se comercializa bajo importación
Cuando solicito la cotización
Entonces se me informa que está sujeto a disponibilidad de importación

---

US-037 — Ver la oferta de equipos organizada por tipo de cliente

AC-001
Dado que estoy en la sección de equipos
Cuando selecciono mi tipo de cliente (practicante, club, escuela)
Entonces veo la oferta y las condiciones que aplican a ese tipo de cliente

AC-002
Dado que no existe oferta específica para un tipo de cliente
Cuando lo selecciono
Entonces se muestra la oferta general disponible

---

US-038 — Registrar y dar seguimiento a pedidos de equipos

AC-001
Dado que soy gestor de operaciones
Cuando accedo a los pedidos de equipos
Entonces veo los pedidos registrados con su estado de importación

AC-002
Dado que un pedido cambia su estado de importación
Cuando actualizo el estado
Entonces el sistema guarda el nuevo estado y queda disponible para informar al cliente

==================================================================
NOTA DE TRAZABILIDAD
==================================================================
Cada AC corresponde a la User Story bajo la que aparece (US-001 a US-038),
y conserva el vínculo con su Feature y Epic según user-stories.md.

Observación: en user-stories.md, US-038 referencia "FT-038", mientras que
features.md numera esa funcionalidad como FT-037 (existiendo dos FT-037).
Se recomienda renumerar la última feature de EP-009 a FT-038 para mantener
IDs únicos y conservar la trazabilidad US-038 → FT-038 → EP-009.