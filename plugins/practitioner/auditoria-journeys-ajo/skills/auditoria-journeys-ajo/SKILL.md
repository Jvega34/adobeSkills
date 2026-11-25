---
name: auditoria-journeys-ajo
description: Audita en solo lectura un journey de Adobe Journey Optimizer (y los journeys enlazados por Jump) con dos revisiones separadas, diseño y rendimiento, y entrega un scorecard por dimensión y un backlog priorizado por severidad y esfuerzo, con evidencia concreta en cada hallazgo. Úsala cuando pidan revisar, auditar, diagnosticar u optimizar un journey, explicar por qué rinde mal o saber si está listo para publicar.
metadata:
  version: 0.2.0
  lifecycle: experimental
---

# Auditoría de journeys de AJO: diagnóstico y plan de optimización

Esta skill audita un journey y propone un plan de optimización. No cambia nada en AJO.

Hace dos revisiones que no se mezclan:

- **Revisión de diseño**: lo que dice la definición del journey. Reentrada, exit criteria, timeouts, ramas de error, waits, frecuencia, consentimiento, conflictos con otros journeys, tipo de entrada (unitario o audiencia) y flujo de datos.
- **Revisión de rendimiento**: lo que dice el reporting. Caídas por nodo, descartes, errores, salidas por motivo y conversión por rama.

Un journey bien diseñado puede rendir mal, y uno mal diseñado puede rendir bien por ahora. Por eso cada revisión tiene su propio scorecard y solo se cruzan al final, en el diagnóstico.

## Reglas que no se rompen

1. **Solo lectura.** Nunca crees, dupliques, pauses, cierres ni publiques un journey o una versión. Propón los cambios; los aplica una persona.
2. **El objeto se fija por ID.** Trabaja sobre un ID de journey y un ID de versión confirmados. Si no encuentras el journey que te piden, para y pregunta. Nunca audites otro journey "porque encaja".
3. **Sin evidencia no hay hallazgo.** Cada hallazgo cita un nodo, rama o campo con su ruta en la definición (diseño), o una métrica con su valor, su ventana y su fuente (rendimiento). Si no puedes señalarla, el hallazgo no entra en el informe. No incluyas buenas prácticas genéricas que no se apoyen en algo de este journey.
4. **Lo que falta se declara, nunca se supone.** Si un campo no llega, una métrica no se puede consultar o el tráfico no alcanza el mínimo, el control queda "pendiente" o "datos insuficientes" y se dice qué falta. Pendiente no es incumplido.
5. **Las dos revisiones van separadas.** Un control de diseño no se puntúa con datos de rendimiento, ni al revés. Los datos de ejecución solo sirven en el diseño para marcar un riesgo como latente o confirmado.
6. **La definición es la fuente de verdad del diseño.** Los comentarios, las etiquetas y los documentos describen intenciones, no comportamiento. Si no coinciden con el código, gana el código y la diferencia es un hallazgo (H10).
7. **Usa solo el contexto que corresponde.** Antes de tomar un documento como especificación del journey, confirma con el usuario que es de ese journey.
8. **Respeta el formato que pida el usuario.** Si pide ir tema por tema, presenta un bloque y espera su respuesta antes de seguir. Sin indicación, usa la salida fija de `references/formato-salida.md`.

## Flujo de trabajo

### 1. Fijar el objeto

- Busca el journey con coincidencia flexible (ignora mayúsculas y espacios dobles). Lista los candidatos con nombre, ID, versión, estado y fecha de despliegue, y pide confirmación si hay más de uno.
- Comprueba en qué sandbox consultas. Si no puedes cambiar de sandbox, dilo.
- Si el listado parece incompleto (el usuario ve un journey que tú no encuentras), dilo y pide el ID. No sigas con otro journey.

### 2. Inventario de datos disponibles

Antes de auditar, muestra una tabla con lo que tienes y lo que no:

- **Definición**: los campos mínimos de `references/lectura-json-ajo.md`.
- **Rendimiento**: las métricas mínimas de `references/revision-rendimiento.md`, con la ventana que cubre cada una.
- **Contexto de negocio**: objetivo, métrica de éxito, cadencia esperada.

Todo control que dependa de algo ausente queda pendiente, y se dice al principio del informe. Si no hay ningún dato de rendimiento (por ejemplo, el journey está en borrador), haz solo la revisión de diseño y dilo en el veredicto.

### 3. Leer los journeys enlazados

Por cada nodo Jump, lee el journey destino (`targetJourney`) y compara `targetJourneyVersion` con la versión desplegada. Si puedes, busca también qué journeys saltan a este. Origen y destino se auditan juntos (H5 a H7).

### 4. Establecer objetivo y tipo

Lee la descripción y los tags. Si el objetivo de negocio no está declarado, pregúntalo antes de puntuar la dimensión de intención y antes de leer la conversión por rama. En una ejecución sin nadie a quien preguntar, marca A1, A2 y R7 como pendientes.

### 5. Revisión de diseño

1. Construye el **mapa de ramas** y el **mapa de datos** siguiendo `references/lectura-json-ajo.md`.
   - Mapa de ramas: por cada condición, sus ramas en orden, la expresión de cada una, la rama por defecto si existe y el nodo destino. Dos ramas con el mismo destino no son un duplicado.
   - Mapa de datos: una fila por dato relevante (atributos de perfil, datos de evento): dónde se escribe, dónde se lee y qué hay entre la escritura y la lectura en cada camino. Incluye lo que lee el journey destino de cada Jump.
2. Aplica el checklist de `references/guia-buenas-practicas.md`, fases A a H.
   - H1 y H4: usa el mapa de datos.
   - H2: enumera el dominio de cada condición sobre rangos (por ejemplo, las horas 0 a 23) y comprueba que cada valor cae en exactamente una rama o en la rama por defecto.
   - H6: compara los `eventId` que escucha el destino con los que dispara el origen.
   - A5 y D4 (conflictos con otros journeys): usa el análisis de conflictos de AJO si está disponible; si no, pendiente.
3. Puntúa con las dimensiones de diseño de la guía (sección "Criterios de evaluación").

### 6. Revisión de rendimiento

Sigue `references/revision-rendimiento.md`.

1. Fija la ventana de análisis y anótala. Compara con la fecha del último despliegue: si la ventana mezcla versiones, sepáralas o dilo.
2. Comprueba los umbrales mínimos de tráfico. Una métrica por debajo del mínimo se marca "datos insuficientes", no se interpreta.
3. Ejecuta los controles R1 a R11 con la métrica, el valor y la fuente de cada uno.
4. Ubica cada problema en un nodo o una rama concreta. "La conversión es baja" no es un hallazgo; "la rama `Sin compra 24h` (uid …) convierte un 0,4% frente al 2,1% de la rama `Compra`" sí lo es.
5. No interpretes los perfiles que esperan un evento como abandono sin revisar antes cuántos salen por exit criteria.

### 7. Diagnóstico cruzado

Solo aquí se juntan las dos revisiones:

- Para cada hallazgo de rendimiento, busca si un hallazgo de diseño lo explica (por ejemplo, salidas en una condición ↔ hueco de rango H2). Si lo hay, enlázalos por ID.
- Para cada hallazgo de diseño, usa los datos para marcarlo **latente** (sin errores observados) o **confirmado con datos**.
- Un síntoma de rendimiento sin causa en la definición se queda como "causa no identificada en el diseño" y apunta a datos, contenido o configuración externa. No inventes la causa.

### 8. Verificar cada hallazgo antes de reportarlo

- Pasa cada hallazgo de diseño por `references/falsos-positivos.md` y cada uno de rendimiento por la sección "Lecturas engañosas" de `references/revision-rendimiento.md`.
- Vuelve a leer el fragmento exacto de la definición o la métrica que lo sostiene.
- Asigna la confianza: verificado en la definición, confirmado con datos o pendiente de verificar.
- Si no puedes señalar la evidencia, el hallazgo no se reporta.

### 9. Construir el backlog

Convierte los hallazgos en acciones con `references/formato-salida.md`:

- Severidad según la guía; esfuerzo según la escala S/M/L; prioridad P1 a P4 con la matriz.
- Cada acción dice qué cambia, qué pasa con los perfiles que ya están dentro, si exige nueva versión, duplicar el journey o desplegar junto con otro, de qué otras acciones depende y cómo se valida.
- No recomiendes, sin ese análisis, ninguno de los cambios de "Recomendaciones que nunca se dan sin análisis de impacto" de la guía.

### 10. Entregar el informe

Usa la plantilla fija de `references/formato-salida.md`. Al final, ofrece el bloque de "estado de la auditoría" para pegar en una sesión futura.

## Herramientas

Usa las herramientas de AJO que tenga Coworker en esta organización para leer la definición, el reporting, los step events y las trazas. Si existen análisis nativos de fallout, conflictos, custom actions, anomalías o comparación de versiones, úsalos como fuente de datos y cita su salida como evidencia. Esta skill no los sustituye: aporta los controles, la separación de revisiones, la verificación y el formato de salida.

Si una herramienta no devuelve un campo que sí existe en la definición de autoría (por ejemplo, exit criteria o merge policy), dilo en el inventario y marca el control como pendiente.

## Referencias

- `references/guia-buenas-practicas.md`: buenas prácticas, checklist de diseño (51 controles, fases A a H), severidad y dimensiones de diseño.
- `references/lectura-json-ajo.md`: cómo leer la definición de un journey (campos, nodos, ramas, expresiones).
- `references/falsos-positivos.md`: errores de lectura de la definición y cómo comprobarlos.
- `references/revision-rendimiento.md`: métricas mínimas, umbrales de tráfico, controles R1 a R11, dimensiones de rendimiento y lecturas engañosas.
- `references/formato-salida.md`: plantilla del informe, scorecards, escala de esfuerzo, matriz de prioridad y bloque de estado.

## Validación de la skill

Antes de usarla con un cliente, y después de cada cambio en la skill o en las referencias, ejecútala sobre un journey ya revisado a mano y compara el resultado con `evals/evals.json` del plugin. Cada caso describe los hallazgos que debe encontrar, lo que no debe reportar y lo que no debe recomendar. Los casos con identificadores de un cliente se guardan fuera del plugin.
