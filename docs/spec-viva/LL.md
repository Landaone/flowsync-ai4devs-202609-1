# Spec: API y pantallas de registro, login, sesión y perfil

## Purpose

Esta spec describe el comportamiento observable de la autenticación y la
sesión de usuario de FlowSync, tanto a nivel de API como de interfaz.

A nivel de API, cubre el registro de cuenta (signup), el inicio de sesión
(login), el cierre de sesión (logout) y la consulta del perfil del usuario
autenticado: las peticiones que un cliente de la API puede enviar a cada
endpoint, las validaciones que se aplican sobre esos datos, y las respuestas
(código de estado, campos y mensajes de error) que ese cliente puede
recibir.

A nivel de interfaz, cubre las pantallas de acceso (inicio de sesión y
registro), el estado de sesión que percibe la persona que usa la aplicación
mientras navega, y la protección de las rutas según haya o no una sesión
activa: qué ve la persona en cada pantalla, qué puede hacer en ella, y a
qué pantalla llega o es redirigida en cada caso.

No cubre la gestión de tareas ni la actividad de equipo, ni el contenido o
diseño detallado de la pantalla de perfil más allá de la acción de cerrar
sesión, que son funcionalidades o aspectos independientes de este alcance.

## Requirements

### Requirement: Registro de cuenta

El sistema SHALL permitir crear una cuenta nueva mediante una petición
`POST /api/v1/auth/signup`, validando los datos de entrada y devolviendo
un token de acceso cuando la cuenta se crea correctamente.

#### Scenario: Registro con datos válidos crea la cuenta y devuelve un token
- **WHEN** un cliente envía `POST /api/v1/auth/signup` con un `email` no
  registrado previamente, una `password` y una `passwordConfirmation`
  idénticas entre sí (de entre 8 y 32 caracteres), y opcionalmente un
  `fullName`
- **THEN** el sistema responde con los datos del usuario creado (incluyendo
  `id`, `fullName`, `email`, `createdAt`, `updatedAt` e `initials`) y un
  token de acceso, envueltos en un campo `data`

#### Scenario: El email es obligatorio y debe tener formato válido
- **WHEN** un cliente envía `POST /api/v1/auth/signup` sin el campo `email`,
  o con un valor que no tiene formato de dirección de correo, o que supera
  los 254 caracteres
- **THEN** el sistema responde con un error de validación indicando el
  campo `email` como inválido, sin crear ninguna cuenta

#### Scenario: El email debe ser único entre las cuentas existentes
- **WHEN** un cliente envía `POST /api/v1/auth/signup` con un `email` que
  ya pertenece a una cuenta existente
- **THEN** el sistema responde con un error de validación sobre el campo
  `email` indicando que ya está en uso, sin crear ninguna cuenta nueva

#### Scenario: La contraseña debe cumplir una longitud mínima y máxima
- **WHEN** un cliente envía `POST /api/v1/auth/signup` con una `password`
  de menos de 8 caracteres o de más de 32 caracteres
- **THEN** el sistema responde con un error de validación sobre el campo
  `password`, sin crear ninguna cuenta

#### Scenario: La confirmación de contraseña debe coincidir con la contraseña
- **WHEN** un cliente envía `POST /api/v1/auth/signup` con una
  `passwordConfirmation` que no es idéntica al valor de `password`
- **THEN** el sistema responde con un error de validación sobre el campo
  `passwordConfirmation`, sin crear ninguna cuenta

#### Scenario: El nombre completo es opcional
- **WHEN** un cliente envía `POST /api/v1/auth/signup` sin el campo
  `fullName` (o con valor nulo), junto con un `email`, `password` y
  `passwordConfirmation` válidos
- **THEN** el sistema crea la cuenta sin exigir el campo `fullName`

### Requirement: Inicio de sesión

El sistema SHALL permitir autenticar una cuenta existente mediante una
petición `POST /api/v1/auth/login` con `email` y `password`, devolviendo un
token de acceso cuando las credenciales son correctas.

#### Scenario: Login con credenciales correctas devuelve un token
- **WHEN** un cliente envía `POST /api/v1/auth/login` con el `email` y la
  `password` de una cuenta existente, coincidiendo exactamente con los
  datos registrados
- **THEN** el sistema responde con los datos del usuario y un nuevo token
  de acceso, envueltos en un campo `data`

#### Scenario: Login con email no registrado o contraseña incorrecta es rechazado
- **WHEN** un cliente envía `POST /api/v1/auth/login` con un `email` que no
  corresponde a ninguna cuenta, o con una `password` que no coincide con la
  de la cuenta asociada a ese `email`
- **THEN** el sistema responde con un error indicando que el email o la
  contraseña no son correctos, sin revelar cuál de los dos campos fue el
  incorrecto, y sin emitir ningún token

#### Scenario: El email y la contraseña son obligatorios para iniciar sesión
- **WHEN** un cliente envía `POST /api/v1/auth/login` sin el campo `email`
  o sin el campo `password`
- **THEN** el sistema responde con un error de validación indicando el
  campo faltante, sin emitir ningún token

### Requirement: Autenticación mediante token de acceso

El sistema SHALL identificar al usuario autor de una petición a un endpoint
protegido a partir de un token de acceso enviado por el cliente, y SHALL
rechazar el acceso cuando ese token falta o no es válido.

#### Scenario: Una petición a un endpoint protegido sin token es rechazada
- **WHEN** un cliente envía una petición a `GET /api/v1/account/profile` o
  a `POST /api/v1/account/logout` sin incluir un token de acceso
- **THEN** el sistema responde con un error de autenticación y no da acceso
  al recurso solicitado

#### Scenario: Una petición con un token inválido, revocado o desconocido es rechazada
- **WHEN** un cliente envía una petición a un endpoint protegido
  incluyendo un token de acceso que no corresponde a ninguna sesión
  vigente (por ejemplo, porque ya fue invalidado mediante logout)
- **THEN** el sistema responde con un error de autenticación y no da acceso
  al recurso solicitado

#### Scenario: Un token de acceso válido no caduca por el paso del tiempo
- **WHEN** un cliente envía una petición a un endpoint protegido
  incluyendo un token de acceso emitido por un signup o login anteriores
  que no ha sido invalidado mediante logout
- **THEN** el sistema da acceso al recurso solicitado, sin importar cuánto
  tiempo haya transcurrido desde la emisión del token

> Duda: no he podido determinar si este comportamiento (tokens sin
> expiración por tiempo) es intencional para este proyecto de práctica o un
> valor por defecto pendiente de configurar.

### Requirement: Consulta del perfil propio

El sistema SHALL permitir a un cliente autenticado consultar los datos de
su propia cuenta mediante `GET /api/v1/account/profile`.

#### Scenario: Un cliente autenticado consulta su perfil
- **WHEN** un cliente autenticado con un token de acceso válido envía
  `GET /api/v1/account/profile`
- **THEN** el sistema responde con los datos de la cuenta asociada a ese
  token (`id`, `fullName`, `email`, `createdAt`, `updatedAt` e `initials`),
  envueltos en un campo `data`

### Requirement: Cierre de sesión

El sistema SHALL permitir a un cliente autenticado invalidar el token de
acceso con el que realizó la petición, mediante
`POST /api/v1/account/logout`.

#### Scenario: Un cliente autenticado cierra sesión
- **WHEN** un cliente autenticado con un token de acceso válido envía
  `POST /api/v1/account/logout`
- **THEN** el sistema invalida ese token de acceso y responde confirmando
  el cierre de sesión

#### Scenario: Un token ya invalidado no puede volver a usarse
- **WHEN** un cliente envía una petición a un endpoint protegido
  reutilizando un token de acceso que ya fue invalidado mediante un logout
  anterior
- **THEN** el sistema responde con un error de autenticación y no da acceso
  al recurso solicitado

### Requirement: Formato de las respuestas de error de validación

El sistema SHALL devolver los errores de validación de los datos de
entrada en un formato uniforme que identifique, por cada campo inválido,
la regla incumplida.

#### Scenario: Un error de validación identifica el campo y la regla incumplida
- **WHEN** un cliente envía a `POST /api/v1/auth/signup` o a
  `POST /api/v1/auth/login` datos que incumplen alguna de las
  validaciones definidas (campo obligatorio ausente, formato de email
  inválido, longitud de contraseña fuera de rango, confirmación de
  contraseña que no coincide, o email ya registrado)
- **THEN** el sistema responde con una lista de errores, cada uno con al
  menos un mensaje, el nombre del campo afectado y la regla de validación
  incumplida

> Duda: no he podido determinar si existe un código de estado HTTP
> específico y documentado para estos errores de validación, más allá del
> comportamiento por defecto del framework subyacente; tampoco si ese
> código está garantizado a futuro como parte del contrato de la API.

### Requirement: Pantalla de inicio de sesión

El sistema SHALL mostrar en `/login` una pantalla de inicio de sesión con
un campo de email, un campo de contraseña y un botón para enviarlos, y
SHALL informar en esa misma pantalla cuando el intento de acceso no se
complete.

#### Scenario: La pantalla de inicio de sesión muestra el formulario
- **WHEN** una persona sin sesión activa visita `/login`
- **THEN** ve el título "Inicia sesión", una descripción invitando a
  entrar, un campo "Email", un campo "Contraseña", un botón "Entrar" y un
  enlace para ir a crear una cuenta

#### Scenario: Enviar el formulario sin completar un campo obligatorio muestra el error debajo de ese campo
- **WHEN** una persona envía el formulario de inicio de sesión dejando
  vacío el campo "Email" o el campo "Contraseña"
- **THEN** la pantalla muestra, debajo del campo correspondiente, un
  mensaje indicando que falta rellenarlo

#### Scenario: Enviar credenciales incorrectas muestra un aviso general de error
- **WHEN** una persona envía el formulario de inicio de sesión con un
  email y una contraseña que no coinciden con ninguna cuenta existente
- **THEN** la pantalla muestra un aviso indicando que el email o la
  contraseña no son correctos, sin señalar ningún campo como incorrecto
  en particular, y permanece en `/login`

#### Scenario: El botón de envío indica que la petición está en curso
- **WHEN** una persona envía el formulario de inicio de sesión con datos
  completos
- **THEN** el botón cambia su texto a "Entrando…" y queda deshabilitado
  mientras dura el intento

#### Scenario: Un inicio de sesión correcto abandona la pantalla de acceso
- **WHEN** una persona envía el formulario de inicio de sesión con un
  email y una contraseña que corresponden a una cuenta existente
- **THEN** la persona deja de ver la pantalla de inicio de sesión

### Requirement: Pantalla de registro de cuenta

El sistema SHALL mostrar en `/register` una pantalla de registro con los
campos necesarios para crear una cuenta nueva, marcando cuáles son
obligatorios y cuáles opcionales, y SHALL informar en esa misma pantalla
de cualquier dato inválido antes o después de contactar con el servidor.

#### Scenario: La pantalla de registro muestra el formulario
- **WHEN** una persona sin sesión activa visita `/register`
- **THEN** ve el título "Crea tu cuenta", un campo "Nombre completo"
  marcado como opcional, un campo "Email", un campo "Contraseña" con la
  indicación de que debe tener entre 8 y 32 caracteres, un campo "Repite
  la contraseña", un botón "Crear cuenta" y un enlace para ir a iniciar
  sesión

#### Scenario: Repetir la contraseña de forma distinta a la original se avisa sin esperar al servidor
- **WHEN** una persona envía el formulario de registro con un valor en
  "Repite la contraseña" distinto al escrito en "Contraseña"
- **THEN** la pantalla muestra de inmediato, debajo del campo "Repite la
  contraseña", un mensaje indicando que las contraseñas no coinciden, sin
  mostrar ningún otro aviso

#### Scenario: Registrar un email ya existente muestra el error sobre ese campo
- **WHEN** una persona envía el formulario de registro con un email que
  ya pertenece a una cuenta existente, junto con el resto de campos
  válidos
- **THEN** la pantalla muestra, debajo del campo "Email", un mensaje
  indicando que ese email ya está registrado y sugiriendo iniciar sesión
  en su lugar

#### Scenario: El botón de envío indica que la cuenta se está creando
- **WHEN** una persona envía el formulario de registro con datos válidos
- **THEN** el botón cambia su texto a "Creando cuenta…" y queda
  deshabilitado mientras dura la creación de la cuenta

#### Scenario: Un registro correcto abandona la pantalla de registro
- **WHEN** una persona envía el formulario de registro con datos válidos
  que el servidor acepta
- **THEN** la persona deja de ver la pantalla de registro

### Requirement: Estado de sesión al abrir la aplicación

El sistema SHALL distinguir, mientras la persona navega la aplicación,
entre que la sesión todavía se está comprobando, que no hay sesión activa
y que la sesión está activa, y SHALL mostrar un indicador de carga
mientras dura esa comprobación en vez de dar por buena o por inválida la
sesión de forma prematura.

#### Scenario: Abrir la aplicación con una sesión guardada muestra un indicador de carga mientras se comprueba
- **WHEN** una persona abre la aplicación en un navegador donde ya había
  iniciado sesión antes y no la había cerrado
- **THEN** ve brevemente un indicador de carga antes de que se muestre la
  pantalla que corresponda a si esa sesión sigue siendo válida o no

#### Scenario: Una sesión guardada que ya no es válida devuelve a la pantalla de inicio de sesión con una explicación
- **WHEN** una persona abre la aplicación con una sesión guardada que el
  servidor ya no reconoce como válida
- **THEN** la persona llega a `/login` y ve un aviso indicando que su
  sesión ha caducado y que vuelva a iniciarla

#### Scenario: Un corte de conexión al comprobar la sesión no cierra la sesión guardada
- **WHEN** una persona abre la aplicación con una sesión guardada y, al
  comprobarla, no se obtiene una respuesta de sesión inválida sino un
  error de conexión o del servidor
- **THEN** la persona llega a `/login` y ve un aviso explicando que no se
  pudo completar la comprobación, sin que esto se le presente como una
  sesión caducada

> Duda: en este último caso, el aviso que ve la persona varía según el
> motivo exacto del fallo (por ejemplo, sin conexión con el servidor
> frente a un error del propio servidor); no hay un único mensaje fijo
> que describir como el resultado de este escenario.

### Requirement: Cierre de sesión desde la aplicación

El sistema SHALL permitir a una persona con sesión activa cerrarla desde
la aplicación, y SHALL dar la sesión por cerrada localmente aunque el
servidor no confirme el cierre.

#### Scenario: Cerrar sesión indica que la petición está en curso
- **WHEN** una persona con sesión activa pulsa el botón de cerrar sesión
- **THEN** el botón cambia su texto a "Cerrando sesión…" y queda
  deshabilitado mientras dura la operación

#### Scenario: Cerrar sesión devuelve a la pantalla de inicio de sesión
- **WHEN** una persona con sesión activa pulsa el botón de cerrar sesión
- **THEN** la persona deja de tener acceso a las pantallas que requieren
  sesión activa y llega a `/login`

### Requirement: Protección de las pantallas de acceso y de las pantallas que requieren sesión

El sistema SHALL impedir que una persona sin sesión activa llegue a una
pantalla que requiere sesión, redirigiéndola a la pantalla de inicio de
sesión, y SHALL impedir que una persona con sesión activa llegue a las
pantallas de inicio de sesión o de registro, redirigiéndola a la pantalla
de perfil.

#### Scenario: Visitar una pantalla que requiere sesión sin tener una activa redirige al inicio de sesión
- **WHEN** una persona sin sesión activa visita `/profile`
- **THEN** es redirigida a `/login` sin llegar a ver el contenido de
  `/profile`

#### Scenario: Visitar las pantallas de acceso con una sesión ya activa redirige al perfil
- **WHEN** una persona con sesión activa visita `/login` o `/register`
- **THEN** es redirigida a `/profile` sin llegar a ver el formulario
  correspondiente

#### Scenario: Visitar una dirección desconocida lleva a la pantalla que corresponda según la sesión
- **WHEN** una persona visita una dirección de la aplicación que no
  corresponde a ninguna de las pantallas anteriores
- **THEN** si tiene sesión activa llega a `/profile`, y si no tiene
  sesión activa llega a `/login`

## Las tres listas

Reloj: de las 00:27 a las 01:12.

### 1. Los dos números

El agente escribió **11** requisitos. Yo comprobé **8**.

Cómo los comprobé: los 6 de la API, ejecutando peticiones contra la API local; y 2 de la interfaz (la protección de pantallas y el cierre de sesión), probándolos a mano en el navegador. Ninguno abriendo el código.

Sin comprobar: el límite de 254 caracteres del email (mi prueba falló por otro motivo), los formularios de inicio de sesión y de registro (textos, avisos y botones), y del estado de sesión solo comprobé que recargar mantiene la sesión; no comprobé el indicador de carga, la sesión caducada ni el corte de conexión.

### 2. Incoherencias que aparecieron

- **El nombre completo no es opcional como dice la spec.** Registrar sin enviar `fullName` devuelve un error de validación ("must be defined"); solo se acepta enviarlo vacío o nulo, y vacío se guarda como nulo. Se ve en `POST /api/v1/auth/signup`.
- **El error de longitud de la contraseña sale también sobre la confirmación.** Con 7 o 33 caracteres el error aparece sobre `password` y sobre `passwordConfirmation`, y la spec solo habla del primero. Se ve en `POST /api/v1/auth/signup`.
- **El mensaje de credenciales incorrectas está en inglés.** La API responde "Invalid user credentials", mientras que la spec lo describe como "el email o la contraseña no son correctos", y toda la interfaz está en castellano. Se ve en `POST /api/v1/auth/login`; no comprobé cómo lo muestra la pantalla.
- **La duda del agente sobre el código de estado de los errores se podía resolver y no lo hizo.** Los errores de validación devuelven 422, las credenciales incorrectas 400 y la falta de token 401; la spec no da ningún código aunque se los pedí.

### 3. Lo que no supe decidir si era un bug o el contrato

- **El email distingue mayúsculas.** `ok@...` y `OK@...` crean dos cuentas distintas y el inicio de sesión también las distingue. Una lectura: el email es un texto exacto y así se acordó. Otra: la unicidad del email está rota, porque dos personas pueden tener "la misma" dirección.
- **Los tokens no caducan por el paso del tiempo.** Una lectura: es un valor por defecto del proyecto de práctica, pendiente de configurar. Otra: es la decisión de que la sesión dure hasta cerrarla. No se puede comprobar sin esperar.
- **Credenciales incorrectas devuelven 400 y no 401.** Mientras la falta de token en las pantallas privadas sí devuelve 401. Una lectura: 400 es un dato de entrada incorrecto. Otra: es una incoherencia del mismo tema (no estás autorizado).
