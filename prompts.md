# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Sonnet 5 (medium effort)
**Herramienta:** Claude Code v2.1.277
**Hora de inicio:** 00:27

```
crea una spec, enfocandote en las funcionalidades de registro, login,  sesion y perfil, validaciones y todo lo que conlleva, escribe solamente la aprte de la API en docs/spec-viva/LL.md, la spec debe tener el siguiente formato: 

## Purpose, 
## Requirements, 
### Requirement: con SHALL, 
#### Scenario: con WHEN/THEN

las dudas que no puedas resolver analizando el codigo marcalas como duda en lugar de inventar solucion a menos que la solucion tenga una unica opcion

cada requisito debe llevar al menos un escenario y que la precondicion vaya dentro del  WHEN porque no hay casilla para GIVEN.

invariante: sin ADDED/MODIFIED/REMOVED, sin nombres de clase, archivo ni ruta de código, y sin tocar el código. las URLs de la API (por ejemplo /api/v1/auth/login) sí se pueden escribir, porque forman parte de la petición; lo prohibido es mencionar archivos, clases o carpetas del código.

describe solo el comportamiento observable desde fuera: lo que un cliente de la API envía (URL, método, datos) y lo que recibe (código de estado, campos y mensajes de error). No describas cómo está implementado por dentro.

todo deb ser escrito en castellano neutral
```

**Nota:** me apoyé en Claude (otra sesión) para revisar este prompt; añadí a mi versión la regla de lo observable y la aclaración sobre las URLs de la API.

**Qué salió:** escribió la parte de la API: 6 requisitos, 16 escenarios y 2 dudas marcadas (tokens sin caducidad y código de estado de los errores de validación). Sin nombres de archivo ni de clase. Casi no da códigos de estado concretos aunque los pedí.

## Prompt 2

**Modelo:** Sonnet 5 (medium effort)
**Herramienta:** Claude Code v2.1.277

```
Añade en el mismo archivo las pantallas de acceso, estado de sesion y proteccion de rutas.

solo las pantallas de acceso, el estado de sesión y la protección de rutas; nada de tareas ni de otras partes de la aplicación.

las dudas que no puedas resolver analizando el codigo marcalas como duda en lugar de inventar solucion a menos que la solucion tenga una unica opcion

cada requisito debe llevar al menos un escenario y que la precondicion vaya dentro del  WHEN porque no hay casilla para GIVEN.

invariante: sin ADDED/MODIFIED/REMOVED, sin nombres de clase, archivo ni ruta de código, y sin tocar el código. las URLs de la API (por ejemplo /api/v1/auth/login) sí se pueden escribir, porque forman parte de la petición; lo prohibido es mencionar archivos, clases o carpetas del código.

describe solo el comportamiento observable desde fuera: lo que una persona ve en pantalla y lo que puede hacer (textos, campos, botones, mensajes de error, a qué pantalla llega o es redirigida). No describas cómo está implementado por dentro ni las llamadas a la API. Las direcciones de las pantallas (por ejemplo /login) sí se pueden escribir, porque forman parte de lo que la persona ve en el navegador.

todo deb ser escrito en castellano neutral
```

**Nota:** en mi primera versión copié el párrafo de lo observable del Prompt 1, que habla de peticiones y respuestas de la API; con ayuda de Claude (otra sesión) lo cambié por uno pensado para pantallas y añadí la línea de alcance.

**Qué salió:** añadió la parte de interfaz: 5 requisitos más (11 en total), 34 escenarios en total y 3 dudas en total. Sin nombres de archivo, de clase ni llamadas a la API en las pantallas.
