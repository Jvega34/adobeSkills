---
name: auditoria-datos-aep
description: Audita en solo lectura los fundamentos de datos de un sandbox de Adobe Experience Platform (schemas XDM, datasets, identidades e identity graph, perfiles y merge policies, audiencias, ingesta, calidad de datos y gobernanza) con dos revisiones separadas, configuración y salud de datos, y entrega un scorecard por dimensión y un backlog priorizado por severidad y esfuerzo, con evidencia concreta en cada hallazgo. Úsala cuando pidan revisar, auditar o diagnosticar los datos de AEP o RT-CDP, explicar por qué una audiencia o un journey no recibe los perfiles esperados, o saber si la base de datos está lista para activar.
metadata:
  version: 0.1.0
  lifecycle: experimental
---

# Auditoría de fundamentos de datos en AEP: diagnóstico y plan de corrección

Esta skill audita los fundamentos de datos de un sandbox de AEP y propone un plan de corrección. No cambia nada en AEP.

Hace dos revisiones que no se mezclan:

- **Revisión de configuración**: lo que está definido. Schemas y field groups, namespaces y reglas del identity graph, datasets habilitados para perfil, merge policies, TTL, definiciones de audiencias, dataflows y data prep, etiquetas y políticas de gobernanza.
- **Revisión de salud de datos**: lo que está pasando con los datos. Éxito y frescura de la ingesta, tamaño de los grafos, perfiles seudónimos, completitud y duplicados de campos clave, tamaño y evolución de las audiencias, cobertura de consentimiento y uso de licencia.

Una configuración correcta puede tener datos malos (un sistema origen que envía emails falsos), y una configuración frágil puede funcionar por ahora. Por eso cada revisión tiene su propio scorecard y solo se cruzan al final, en el diagnóstico.

Es la segunda skill del health check AJO/AEP. Si la auditoría de journeys dejó un síntoma con "causa no identificada en el diseño", empieza por aquí.

## Reglas que no se rompen

1. **Solo lectura.** Nunca crees, modifiques, habilites ni borres schemas, datasets, namespaces, merge policies, audiencias, dataflows, etiquetas o políticas. No reproceses batches, no lances exports, evaluaciones de audiencias ni trabajos de Data Hygiene. En Query Service solo `SELECT`: nunca `CREATE TABLE AS`, `INSERT`, `ALTER` ni `DROP`. Propón los cambios; los aplica una persona.
2. **El alcance se fija por sandbox.** Trabaja sobre una organización y un sandbox confirmados, y di si es de producción o de desarrollo. Si el usuario acota (un dataset, una audiencia, un dominio de datos), fija esos objetos por ID. Si no encuentras lo que te piden, para y pregunta. Nunca audites otro objeto "porque encaja".
3. **Sin evidencia no hay hallazgo.** Cada hallazgo de configuración cita un objeto con su ID o nombre exacto y el campo o la ruta concreta. Cada hallazgo de salud cita una métrica con su valor, su ventana y su fuente (pantalla, API, health check o consulta SQL, que se incluye). Si no puedes señalarla, el hallazgo no entra. No incluyas buenas prácticas genéricas que no se apoyen en algo de este sandbox.
4. **Lo que falta se declara, nunca se supone.** Si un objeto no llega, una métrica no se puede consultar o no tienes permiso, el control queda "pendiente" y se dice qué falta. Pendiente no es incumplido.
5. **Las dos revisiones van separadas.** Un control de configuración no se puntúa con datos observados, ni al revés. Los datos observados solo sirven en la configuración para marcar un riesgo como latente o confirmado.
6. **La configuración real es la fuente de verdad.** Los documentos de diseño, los nombres y las descripciones describen intenciones. Si no coinciden con lo configurado, gana lo configurado y la diferencia es un hallazgo.
7. **Los datos personales no salen en el informe.** Las consultas devuelven recuentos, porcentajes y distribuciones. Si necesitas mostrar un valor (por ejemplo, un email repetido miles de veces), enmascáralo (`jo***@***.com`). Nunca copies perfiles individuales al informe.
8. **Consultas acotadas.** Cada consulta lleva ventana de fechas y `LIMIT`. Antes de lanzar consultas sobre tablas grandes o muchas consultas seguidas, avisa al usuario: consumen capacidad de Query Service de su licencia.
9. **Respeta el formato que pida el usuario.** Si pide ir tema por tema, presenta un bloque y espera respuesta antes de seguir. Sin indicación, usa la salida fija de `references/formato-salida.md`.

## Flujo de trabajo

### 1. Fijar el alcance

- Confirma organización y sandbox, y su tipo (producción o desarrollo). Un sandbox de desarrollo tiene volúmenes limitados y no sirve para concluir sobre producción.
- Pregunta el uso principal de los datos: activación en AJO, destinos de RT-CDP, personalización en edge, reporting en CJA. Cambia qué controles son críticos.
- Si el usuario acota, lista los objetos candidatos con nombre, ID y fecha de modificación, y pide confirmación si hay más de uno.

### 2. Inventario de datos disponibles

Antes de auditar, muestra una tabla con lo que tienes y lo que no, siguiendo `references/lectura-objetos-aep.md`:

- **Configuración**: schemas, field groups, namespaces y ajustes del identity graph, datasets y su estado de perfil, merge policies, TTL, audiencias, dataflows, etiquetas y políticas.
- **Salud**: health checks nativos, dashboards de perfiles, identidades, audiencias y licencia, monitorización de dataflows y acceso a Query Service, con la ventana que cubre cada fuente.
- **Contexto de negocio**: identificador de persona acordado, casos de uso activos, base legal y política de consentimiento.

Todo control que dependa de algo ausente queda pendiente, y se dice al principio del informe. Si no hay acceso a datos observados, haz solo la revisión de configuración y dilo en el veredicto.

### 3. Usar primero los health checks nativos

Si el sandbox tiene disponibles los health checks de AEP (Run and operate > Health checks: schemas e identidades, ingesta, datasets, perfil, merge policies, segmentación, TTL, destinos, Query Service, sources), léelos primero y cita su resultado como evidencia. Esta skill no los sustituye: añade los controles que no cubren, la separación de revisiones, la verificación de falsos positivos, la priorización y el formato de salida. La correspondencia entre health checks y controles está en `references/checklist-fundamentos.md`.

### 4. Revisión de configuración

1. Construye el **mapa de identidad**: por cada namespace, si es de persona o no, si es único, su prioridad, en qué schemas y campos aparece, si es identidad primaria y qué la consume (journeys, destinos, audiencias).
2. Construye el **mapa de perfil**: por cada dataset habilitado para perfil, su schema, clase, identidad primaria, TTL, fecha de último dato y qué merge policy lo incluye.
3. Aplica el checklist de `references/checklist-fundamentos.md`, dimensiones S, I, P, U, N y G.
4. Puntúa con las dimensiones de configuración de la misma referencia.

### 5. Revisión de salud de datos

Sigue `references/revision-salud-datos.md`.

1. Fija la ventana de análisis y anótala, con la zona horaria.
2. Usa primero dashboards y health checks; recurre a Query Service solo para lo que no cubren, con las plantillas de la referencia.
3. Ejecuta los controles Q1 a Q14 con la métrica, el valor, la ventana y la fuente de cada uno.
4. Ubica cada problema en un objeto concreto. "La calidad de datos es mala" no es un hallazgo; "el 38% de los registros del dataset `CRM Clientes` (ID …) tiene `personalEmail.address` vacío, frente al 2% del mes anterior" sí lo es.
5. Respeta los umbrales mínimos de volumen. Por debajo, "datos insuficientes".

### 6. Diagnóstico cruzado

Solo aquí se juntan las dos revisiones:

- Para cada hallazgo de salud, busca si un hallazgo de configuración lo explica (por ejemplo, grafos de más de 50 identidades ↔ email sin marcar como único). Si lo hay, enlázalos por ID.
- Para cada hallazgo de configuración, usa los datos para marcarlo **latente** o **confirmado con datos**.
- Un síntoma sin causa en la configuración apunta al sistema origen, al data prep o a un proceso externo. Dilo así; no inventes la causa.
- Añade el **impacto en activación**: qué journeys, audiencias o destinos afecta cada hallazgo, si se puede ver.

### 7. Verificar cada hallazgo antes de reportarlo

- Pasa cada hallazgo por `references/falsos-positivos.md`.
- Vuelve a leer el objeto, el campo o el resultado de la consulta que lo sostiene.
- Asigna la confianza: verificado en la configuración, confirmado con datos o pendiente de verificar.
- Si no puedes señalar la evidencia, el hallazgo no se reporta.

### 8. Construir el backlog

Convierte los hallazgos en acciones con `references/formato-salida.md`:

- Severidad según el checklist; esfuerzo según la escala S/M/L; prioridad P1 a P4 con la matriz.
- Cada acción dice qué cambia, qué pasa con los datos y perfiles que ya existen, si exige un schema nuevo, reingesta, recalcular audiencias o coordinarse con el equipo del sistema origen, de qué otras acciones depende y cómo se valida (idealmente primero en un sandbox de desarrollo).
- No recomiendes, sin ese análisis, ninguno de los cambios de "Recomendaciones que nunca se dan sin análisis de impacto" del checklist.

### 9. Entregar el informe

Usa la plantilla fija de `references/formato-salida.md`. Al final, ofrece el bloque de "estado de la auditoría" para pegar en una sesión futura.

## Herramientas

Usa las herramientas de AEP que tenga Coworker en esta organización para leer schemas, datasets, identidades, perfiles, merge policies, audiencias, dataflows, etiquetas, health checks, dashboards y Query Service. Si Coworker expone agentes o skills nativas de AEP (por ejemplo, de audiencias o de análisis de datos), úsalas como fuente y cita su salida como evidencia.

Si una herramienta no devuelve un campo que existe en la interfaz de AEP (por ejemplo, la configuración de identity graph linking rules o el TTL de un dataset), dilo en el inventario y marca el control como pendiente.

## Referencias

- `references/checklist-fundamentos.md`: controles de configuración (dimensiones S, I, P, U, N, G), guardrails, severidad, dimensiones y pesos, veredicto y recomendaciones que exigen análisis de impacto.
- `references/lectura-objetos-aep.md`: qué leer de cada objeto de AEP y dónde está.
- `references/revision-salud-datos.md`: fuentes, umbrales de volumen, controles Q1 a Q14, plantillas SQL de solo lectura y lecturas engañosas.
- `references/falsos-positivos.md`: errores de lectura frecuentes y cómo comprobarlos.
- `references/formato-salida.md`: plantilla del informe, scorecards, escala de esfuerzo, matriz de prioridad y bloque de estado.

## Validación de la skill

Antes de usarla con un cliente, y después de cada cambio en la skill o en las referencias, ejecútala sobre un sandbox ya revisado a mano y compara el resultado con `evals/evals.json` del plugin. Cada caso describe los hallazgos que debe encontrar, lo que no debe reportar y lo que no debe recomendar. Los casos con identificadores de un cliente se guardan fuera del plugin.
