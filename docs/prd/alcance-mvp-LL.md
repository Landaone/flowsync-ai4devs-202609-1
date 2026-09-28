# Alcance del MVP de FlowSync

## 1. El terreno que ya existe

Hoy FlowSync solo resuelve la cuenta de usuario: registrarse, iniciar sesión, cerrar sesión y ver el perfil propio.
El único dato de negocio es el usuario (email y contraseña), con sus sesiones abiertas.
No hay tareas, ni equipos, ni proyectos: el dominio de "saber quién está en qué" está por construir entero.
El MVP hereda la autenticación ya resuelta y no debe volver a especificarla.

## 2. El interrogatorio

Las cinco preguntas de la IA y sus respuestas, ya decididas por la ficha de hechos:

1. **¿Quién usa el estado y hay jerarquía?** Los pares, no un lead. Roles planos: todos ven y editan lo mismo. No hay reporte hacia arriba.
2. **¿Qué reemplaza y cuál es la unidad: la tarea o el estado de la persona?** La ronda de "¿en qué estás?" de la daily y del chat. No reemplaza la daily entera, porque los bloqueos siguen. La unidad es la tarea: el estado es de la tarea, no de la persona.
3. **¿Qué significa "tiempo real"?** Ver los cambios de estado de las tareas sin refrescar ni preguntar. Es un resumen que espera, no un aviso que interrumpe: sin notificaciones push, sin chat y sin presencia.
4. **¿Hasta dónde llega "menos rollo que Jira"?** Crear una tarea y cambiarle el estado en segundos, sin campos obligatorios. Fuera: sprints, estimaciones, épicas, backlog priorizado, informes, comentarios, integraciones y roles avanzados.
5. **¿Tamaño del equipo y si es uso real?** Un solo espacio compartido para un equipo remoto de 3 a 10 personas. Nada de varios equipos ni aislamiento entre ellos. El primer usuario es un caso de estudio: un equipo SaaS de 6 personas repartido en 3 husos horarios.

Supuestos que declaró la IA, donde la ficha no llegaba:

- Estados fijos y cortos: pendiente, en progreso y hecho. Propuso "bloqueada" como posible estado.
- Roles planos: cualquiera edita o reasigna cualquier tarea.
- Todo usuario registrado entra al espacio único, sin invitaciones ni gestión de miembros.
- Borrado simple de tareas.
- No se guarda el historial de cambios de estado.
- La fecha de vencimiento es opcional.
- Vista por defecto: una lista única del espacio con filtro por estado, no una lista por persona.
- Solo web, sin app móvil.

## 3. El alcance

### Problema

Los equipos remotos pequeños pierden tiempo sincronizándose sobre en qué está trabajando cada uno: media daily hablando de "en qué estás" en vez de bloqueos, e interrupciones sueltas por chat el resto del día. Esa ronda existe porque hoy nadie puede mirar y saberlo por su cuenta. El costo no es solo el tiempo de la reunión: es que la falta de visibilidad hace que dos personas empiecen a tocar lo mismo sin saberlo, y el choque se descubre días después, no en el momento.

### Usuarios

Un equipo remoto único de 3 a 10 personas, con roles planos: sin lead que reporte hacia arriba ni jerarquía de permisos. Todos son a la vez quien escribe su estado y quien consulta el de los demás — el mismo acto sirve a los dos papeles. El primer caso de estudio es un equipo de producto SaaS de 6 personas en 3 husos horarios, que hoy resuelve esto con un gestor de tareas pesado y una daily de 15 minutos por videollamada.

### Propuesta de valor

Una lista de tareas compartida donde el estado de cada tarea se actualiza en dos clics y se ve fresco sin preguntar: al entrar, cualquiera distingue lo que cambió desde su última visita de lo que sigue igual. No es presencia ("quién está conectado"), es frescura de la tarea. El mismo gesto que informa al equipo es el que la persona ya necesita hacer para llevar su propia cola de trabajo, así que se sostiene sin disciplina extra ni carga adicional. Si esto funciona, la ronda de "en qué estás" de la daily deja de tener motivo: el equipo la cancela porque ya lo sabe al entrar, no porque alguien se lo pida.

### Alcance

- **Crear una tarea**: título y responsable obligatorios, fecha de vencimiento opcional. Sin flujo de configuración ni campos adicionales — valida que "dos clics" alcanza para que la gente realmente lo use.
- **Cambiar el estado de una tarea** entre pendiente, en progreso y hecho. Es la acción que reemplaza la ronda verbal, así que tiene que ser la más barata de toda la herramienta.
- **Ver la lista de tareas del espacio único**, filtrable por estado, para poder centrarse en lo pendiente sin scroll.
- **Distinguir qué tareas cambiaron desde la última visita de la persona**, dentro de esa misma lista. Es el caso de uso central: sin esto, la herramienta es una lista de tareas más, no un sustituto de la sincronización verbal.

### NO-alcance

- **Editar y borrar tareas** (recorte mío). No ayuda a validar ninguna de las dos hipótesis: ni que actualizar el estado cueste dos clics, ni que el equipo cancele la ronda de "¿en qué estás?". Una tarea equivocada se puede cerrar como hecha.
- **Cuenta de usuario** (recorte mío). No es alcance nuevo: ya existe y está descrita en el terreno. Contarla aquí inflaba el alcance.
- **Notificaciones push.** La propuesta de valor es frescura al mirar, no interrupción al momento; construir push validaría una hipótesis distinta ("avisar" en vez de "informar al entrar") que explícitamente se descartó.
- **Integración con Slack u otras herramientas.** FlowSync es la fuente de la tarea, no un espejo. Integrar valida "convivir con otra herramienta", que es justo el patrón de doble actualización que se quiere evitar.
- **Roles y permisos avanzados.** El equipo objetivo es plano y sin reporte hacia arriba; construir jerarquía respondería a una necesidad (control, reporting hacia un manager) que este usuario no tiene.
- **Analítica y reporting.** Nadie cobra valor de un reporte en este problema — el valor está en la lista viva, no en un histórico agregado. Construirlo no ayuda a validar si la gente deja de hacer la ronda de la daily.
- **Comentarios en tareas.** Añade una superficie de coordinación (discutir, no informar) que compite con la daily de bloqueos, la cual queda fuera del MVP a propósito.
- **Historial completo de cambios.** Solo hace falta saber qué cambió desde la última visita, no auditar quién cambió qué y cuándo. Guardar historial completo es esfuerzo de construcción que no aporta a esa hipótesis puntual.
- **Múltiples equipos o aislamiento entre espacios (multi-tenancy).** El caso de estudio es un único equipo. Resolver aislamiento entre equipos es un problema de plataforma que no informa si la propuesta de valor funciona para uno solo.
- **Gestión de miembros / invitaciones.** Cualquier usuario registrado entra al espacio único; construir invitaciones responde a un problema de administración, no al problema de sincronización que se está validando.
- **Sprints, estimaciones, épicas, backlog priorizado.** Son la maquinaria de planificación que "menos rollo que Jira" rechaza explícitamente; el usuario que las necesita no es el usuario de este MVP.
- **Resolución de bloqueos.** La daily no desaparece entera porque los bloqueos se quedan fuera a propósito: son una conversación, no un estado de tarea, y meterlos aquí diluiría la única cosa que el MVP tiene que probar.
- **Derivar el estado de señales externas (Git, PRs, CI, calendario).** Es otro producto, con OAuth e integraciones de terceros. El MVP valida si teclear el estado a mano, en segundos, ya es suficientemente barato — antes de gastar esfuerzo en automatizarlo.
- **App móvil.** El caso de uso es "entrar y mirar" al llegar a trabajar o volver de una reunión; el navegador de escritorio ya cubre ese momento sin coste adicional de construir y mantener otra plataforma.

## Las tres líneas

1. **Los dos números:** la IA propuso 6 cosas dentro del alcance; después de mi recorte quedaron 4.
2. **Tres cosas que dejé fuera y por qué:**
   - **Editar y borrar tareas:** no valida que actualizar cueste dos clics ni que el equipo cancele la ronda de "¿en qué estás?".
   - **Notificaciones push:** validan "avisar al momento", y la hipótesis es "enterarse al volver a mirar". Probarían otro producto.
   - **Historial completo de cambios:** la hipótesis solo necesita saber qué cambió desde mi última visita, no auditar quién cambió qué y cuándo.
3. **La exclusión de la que menos seguro estoy:** el filtro por estado. La ficha lo pide, pero con un equipo de 6 y una sola lista dudo que valide algo que no dé ya ver la lista entera más "qué cambió desde tu última visita". Se contradicen lo que me pedían y lo que veía. Entraría si en la primera semana la lista crece tanto que la gente deja de mirarla para encontrar lo pendiente. Por eso lo dejé dentro, pero es el primer candidato a salir.
