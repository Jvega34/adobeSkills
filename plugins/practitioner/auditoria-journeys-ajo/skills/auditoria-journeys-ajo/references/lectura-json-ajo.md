# Cómo leer la definición de un journey de AJO

Esta referencia describe la definición de autoría de una versión de journey, la misma que carga el lienzo de AJO. Los nombres de campo salen de definiciones reales, formato de autoría 2.0. Si la herramienta devuelve otra estructura, compárala con esta tabla y declara qué falta.

## Campos mínimos

Comprueba que llegan antes de auditar. Si falta alguno, los controles que dependen de él quedan pendientes.

| Campo | Para qué sirve | Controles |
| --- | --- | --- |
| `uid`, `journey`, `journeyVersion`, `name`, `state` | Fijar el objeto: ID de versión, ID de journey, número de versión, nombre exacto y estado | Todos |
| `description`, `tags`, `labels` | Objetivo e inventario | A1, B9 |
| `keyNamespace` | Namespace del journey | B2 |
| `reentrancePolicy`, `noReentrancePeriodInSecs` | Re-entrada y espera, en segundos (0 = sin espera) | B3 |
| `exitCriteria`, `segmentMembershipDefs` | Exit criteria (evento o audiencia) y las audiencias que usan | B4, B5, H8 |
| `timezone` | Zona horaria fija o del perfil | B6, H9 |
| `mergePolicy` | Merge policy del journey | C5 |
| `defaultActionExecutionTimeoutInSecs` | Timeout de las acciones | D6 |
| `priority` | Prioridad del journey (0 a 100) | D4 |
| `jumpTargetJourneys`, `hasJourneyJump` | Journeys destino de los Jump | H5 a H7 |
| `ui.nodes`, `ui.edges` | Nodos y conexiones | C, H |
| `metadata` | Autor, fechas de creación y de despliegue | Contexto |

## Nodos (`ui.nodes`)

Cada nodo tiene `id`, `type`, `label` y `data`. Los tipos que importan:

- **`event`**: escucha un evento.
  - `data.eventId` identifica el evento; `label` es su nombre.
  - Si existe `data.timeout` con `delay` (duración ISO 8601, por ejemplo `PT10M`) y `timeoutPath: true`, el nodo tiene rama de timeout.
  - El primer `event` tras `start` es la entrada del journey.
- **`condition`**: bifurca.
  - `data.label` es el nombre visible; `data.conditions` es la lista ordenada de ramas.
  - Cada rama tiene `uid`, `name` y `expression`.
  - Una rama con `conditionType: "opposite"` es la **rama por defecto** ("otros casos"): no tiene expresión y recoge todo lo que no cumple las anteriores.
  - Las ramas se evalúan en orden y gana la primera verdadera.
- **`action`** con `subtype: "custom"`: custom action.
  - `data.paramMappings` dice qué envía: cada mapeo tiene la ruta del payload (`paramPath`) y la expresión del valor (`stringExpression`).
  - Si la ruta es un evento XDM (`.../events/0/xdm/...`), la acción publica un evento de ingesta: la escritura en el perfil es asíncrona.
- **`action`** con `name: "updatePlatformAction"`: Update profile.
  - `data.fieldPath` es el atributo que escribe; `data.valuePicker.plainText`, el valor.
  - Escribe por ingesta, de forma asíncrona.
- **`action`** con `name: "jump"`: salto a otro journey.
  - `data.targetJourney` es el ID del journey destino y `data.targetJourneyVersion` el ID de la versión guardada.
  - `data.firstEventId` es el evento de entrada del destino y `data.fieldsMapping` lo que se le pasa.
- **`campaign`**: envío de mensaje.
  - `data.channel` es el canal y `data.label` el nombre del mensaje.
  - `data.channelOverrides` dice de dónde sale la dirección (por ejemplo, `personalEmail.address` o un campo del tenant como `_<tenant>.identidades.email`).
- **`timer`**: espera fija; `data.delay` en ISO 8601 (`PT10S`, `PT1H`).
- **`end`**: fin del camino.

`data.fallbackOnTimeoutOrError` indica si hay ruta alternativa ante error o timeout. Solo tiene efecto en acciones (custom, Update profile, Jump, mensajes) y en condiciones que consultan fuentes externas. En timers no tiene efecto.

## Conexiones (`ui.edges`)

Cada conexión tiene `source.elementId`, `target.elementId` y, a veces, `type`:

- `type: "condition"`: rama de una condición. El `id` de la conexión es igual al `uid` de la rama.
- `type: "opposite"`: rama por defecto. Su `id` es el `uid` de la rama `opposite`.
- `type: "timeout"`: rama de timeout de un evento.
- Sin `type`: continuación secuencial.

Para saber adónde va cada rama, busca la conexión cuyo `id` coincide con el `uid` de la rama. Que dos ramas lleguen al mismo `target.elementId` es convergencia, no duplicado.

## Expresiones

- `@event{<id de nodo o nombre de evento>.<ruta>}`: dato del evento. No cambia durante el journey.
- `#{ExperiencePlatform.ProfileFieldGroup.profile.<ruta>}`: atributo del perfil. Se lee en el momento de evaluar y puede no reflejar una escritura reciente.
- `inAudience("<nombre>")`: pertenencia a una audiencia. Compara si es batch o streaming con las de los exit criteria.
- `//` empieza un comentario. **Los comentarios no se ejecutan.** No saques tiempos, franjas ni pasos de ellos.
- `substr(toString(nowWithDelta(...)), 11, 13)` extrae la hora como texto. Depende del formato y de la zona horaria (control H9).

## Cómo seguir un camino

1. Empieza en `start` y sigue las conexiones.
2. En cada condición, sigue cada rama por separado, incluida la rama por defecto.
3. En cada evento con timeout, sigue las dos salidas: el evento llega o vence el timeout.
4. Anota en cada camino qué se escribe (custom actions, Update profile) y qué se lee (condiciones, direcciones de envío, el journey destino).
5. Entre una escritura y su lectura, anota el tiempo mínimo que pasa. Cero o pocos segundos es una carrera (control H1).
