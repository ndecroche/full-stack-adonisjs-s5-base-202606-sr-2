# Resumen de ajustes — retrospectiva S4 (FlowSync MVP)
# ¿Siguen teniendo sentido las historias tal como las generaste? 
Las historias siguen teniendo sentido, pero realmente les falta una base de arquitectura que se le dijo explicitamente que no tenga en cuenta.
Creo que, al ya conocer la arquitectura que se debe utilziar, lo mejor es que lo tome cómo contexto, por más que los cambios no sean significativos.
¿El alcance sigue ceñido al MVP del PRD, o se coló alguna que la IA "inventó" fuera de scope?
El alcance sigue los puntos requeridos en la definición. No hay tareas que la IA haya agregado por fuera del scope.


# ¿Hay historias cuyos criterios de aceptación ahora ves incompletos o poco verificables?
Hay varias historias, que al refiniarlas con Poke-holes terminan variando en los criterios de aceptación. También es cierto que le pedí explícitamente
a la IA que genere entre 3 y 5 criterios de aceptación, y eso seguramente la limitó a escribir más.
Entonces hay varias historias con criterios incompletos, por ej. con el tema de las fechas y los null.
También hay criterios que no son del todo verificables, por ej. en ningún caso se dice en qué idiomas se deben mostrar los mensajes. Esto termina en criterios
poco verificables o mal verificados.


# ¿Hay historias que han cambiado de naturaleza desde entonces? (porque descubriste una dependencia, porque la spec evolucionó, porque entiendes mejor el dominio).
Muchas tareas depeden de la tarea: "US-03: Crear tarea". Sin esta, todas las historias de modificación, borrado, ordenamiento y demás, no se pueden realizar.
No es que haya cambiado su naturaleza, pero hay que tenerlo en cuenta a la hora de planificar porque puede afectar todo el proyecto.
La otra tarea que si evolucionó es la tarea: "US-10: Sincronizar tareas con fecha hacia Google Calendar", esto tiene mucha dependencia con la zona horaria 
en qué se encuentre quién está pidiendo la sincronización, por lo tanto hay qué modificar los criterios de aceptación par que lo tenga en cuenta.


# ¿Hay historias nuevas que no aparecieron cuando lo generaste y que ahora sí deberían estar?
El "ver el listado de las tareas" debería estar, porque es un get que es básico para el trabajo requerido y tiene criterios de aceptación importantes para la solución.
Sigo pensando que debería haber tareas para verificar el exito del MVP.

Al contrastar con el backlog que el mentor construyó en el directo de S4 sobre Linear: ¿qué priorizaste distinto tú? ¿Quién acertó y por qué?
Priorizamos las mismas cosas, solo agregué las parte del listado y las tareas para verficar el exito.
Ambos acertamos, las historias son realizables y dividen bien el proyecto para llevarlo a cabo.

Cómo acotación final, acá no se tuvo en cuenta la cantidad de desarrolladores ni la velocidad del equipo, pero esto se puede agregar en un
paso posterior al momento de asignar las tareas.

# Ajustes y motivos
---

**Ajuste:** Incorporar la arquitectura/stack definido (AdonisJS 7, React 19, SQLite, Google Calendar API, etc.) como contexto al refinar y planificar las historias, aunque el alcance funcional no cambie de forma significativa.

**Motivo:** En S4 el prompt prohibía explícitamente proponer arquitectura; las historias siguen siendo válidas, pero les falta esa base. Al conocer ya el stack y las specs, conviene usarlo como marco para AC más concretos y decisiones técnicas implícitas (endpoints, modelos, validación).

---

**Ajuste:** Ampliar y endurecer los criterios de aceptación en historias refinadas con poke-holes, especialmente en fechas, valores nulos y mensajes al usuario.

**Motivo:** El límite de 3–5 AC por historia dejó varios criterios incompletos (p. ej. exportación CSV sin descripción/fecha, comportamiento con `null`). Otros no son verificables: no se define idioma de los mensajes ni criterios medibles para "mensaje comprensible". El refinamiento reveló huecos que el formato inicial no capturó.

---

**Ajuste:** Tratar **US-03: Crear tarea** como dependencia crítica del plan (bloqueante para edición, borrado, estados, filtrado, ordenamiento, exportación y sync).

**Motivo:** No cambió la naturaleza de las demás historias, pero sin US-03 el resto del módulo CRUD y organización no es ejecutable. Afecta el orden y la planificación del sprint, aunque la descomposición en historias siga siendo correcta.

---

**Ajuste:** Incorporar historias o tareas de **verificación del éxito del MVP** (flujo end-to-end, métricas de la sección 6 del PRD).

**Motivo:** Los criterios de éxito del MVP (flujo completo sin ayuda, ≥40% conectan Google, <5% fallos no recuperables de sync) no tienen cobertura en el backlog funcional. Son necesarios para validar que el MVP cumple su hipótesis, no solo que las features individuales funcionan.

---