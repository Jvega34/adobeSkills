---
name: auditoria-journeys-ajo
description: Audita en solo lectura un journey de Adobe Journey Optimizer y los journeys enlazados por Jump, con la guía de buenas prácticas y controles de flujo de datos, y devuelve hallazgos verificados con evidencia, severidad y pendientes. Úsala cuando pidan revisar, auditar u optimizar un journey, o saber si está listo para publicar.
---

# Auditoría de journeys de AJO

Esta skill audita un journey y propone optimizaciones. No cambia nada en AJO.

## Reglas que no se rompen

1. **Solo lectura.** Nunca crees, dupliques, pauses, cierres ni publiques un journey o una versión. Propón los cambios; los aplica una persona.
2. **El objeto se fija por ID.** Trabaja sobre un ID de journey y un ID de versión confirmados. Si no encuentras el journey que te piden, para y pregunta. Nunca audites otro journey "porque encaja".
3. **La definición es la fuente de verdad.** Los comentarios de las expresiones, las etiquetas y los documentos de contexto describen intenciones, no comportamiento. Si no coinciden con el código, gana el código y la diferencia es un hallazgo.
4. **Lo que no ves es pendiente, nunca incumplido.** Si un campo no te llega o un dato no se puede consultar, márcalo "pendiente de confirmar" y di qué falta.
5. **Cada hallazgo cita su evidencia:** nodo, rama o campo, con su ruta en la definición.
6. **Respeta el formato que pida el usuario.** Si pide revisar tema por tema, presenta un bloque y espera su respuesta antes de seguir.
7. **Usa solo el contexto que corresponde.** Antes de tomar un documento como especificación del journey, confirma con el usuario que es de ese journey.

## Flujo de trabajo

### 1. Fijar el objeto

- Busca el journey con coincidencia flexible: ignora mayúsculas y espacios dobles. Lista los candidatos con nombre, ID, versión, estado y fecha de despliegue, y pide confirmación si hay más de uno.
- Comprueba en qué sandbox estás consultando. Si no puedes cambiar de sandbox, dilo.
- Si el listado parece incompleto (por ejemplo, el usuario ve en la interfaz un journey que tú no encuentras), dilo y pide el ID. No sigas con otro journey.

### 2. Leer la definición completa y comprobar qué ha llegado

Lee la definición de autoría de la versión, la misma que carga el lienzo. Antes de auditar, muestra una tabla con los campos de `references/lectura-json-ajo.md` (sección "Campos mínimos") y si los tienes o no. Todo control que dependa de un campo ausente queda pendiente, y lo dices al principio del informe.

### 3. Leer los journeys enlazados

Por cada nodo Jump, lee el journey destino (`targetJourney`) y compara `targetJourneyVersion` con la versión desplegada. Si puedes, busca también qué journeys saltan a este. Origen y destino se auditan juntos (controles H5 a H7).

### 4. Establecer objetivo y tipo

Lee la descripción y los tags. Si el objetivo de negocio no está declarado, pregúntalo antes de puntuar la dimensión de intención. En una ejecución sin nadie a quien preguntar, marca los controles A como pendientes.

### 5. Construir dos mapas

Sigue `references/lectura-json-ajo.md`.

- **Mapa de ramas.** Por cada condición: sus ramas en orden, la expresión de cada una, la rama por defecto si existe y el nodo destino de cada una. Dos ramas con el mismo destino no son un duplicado.
- **Mapa de datos.** Una fila por dato relevante (atributos de perfil, datos de evento): dónde se escribe, dónde se lee y qué hay entre la escritura y la lectura en cada camino (nada, una espera, un evento con timeout). Incluye lo que lee el journey destino de cada Jump.

### 6. Ejecutar los controles

- Aplica el checklist de `references/guia-buenas-practicas.md`, fases A a H.
- Para H1 y H4, usa el mapa de datos.
- Para H2, enumera el dominio de cada condición sobre rangos (por ejemplo, las horas 0 a 23) y comprueba que cada valor cae en exactamente una rama, o en la rama por defecto.
- Para H6, compara los `eventId` que escucha el destino con los que dispara el origen.

### 7. Datos de ejecución, si están disponibles

- Consulta entradas, errores por nodo, salidas por motivo (exit criteria, End, error) y perfiles en espera.
- Anota la ventana que cubren y cualquier límite (timeouts, ventanas máximas, errores de acceso).
- Usa los datos para:
  - marcar como **latente** un riesgo sin errores observados;
  - confirmar o descartar hipótesis, como salidas en una condición sin rama.
- No interpretes los perfiles que esperan un evento como abandono sin revisar antes cuántos salen por exit criteria.

### 8. Verificar cada hallazgo antes de reportarlo

- Pasa cada hallazgo por `references/falsos-positivos.md`.
- Vuelve a leer el fragmento exacto de la definición que lo sostiene.
- Asigna la confianza: verificado en la definición, confirmado con datos o pendiente de verificar.
- Si no puedes señalar la evidencia, el hallazgo no se reporta.

### 9. Redactar recomendaciones seguras

Cada corrección dice:
- qué cambia;
- qué pasa con los perfiles que ya están dentro;
- si exige nueva versión, duplicar el journey o desplegar junto con otro;
- en qué orden va respecto a otras correcciones;
- cómo se valida.

No recomiendes, sin ese análisis, ninguno de los cambios de la lista "Recomendaciones que nunca se dan sin análisis de impacto" de la guía.

### 10. Entregar el informe

En este orden:

1. Objeto auditado (nombre, IDs, versión, sandbox) y campos que no llegaron.
2. Veredicto y una frase que lo justifique.
3. Lo que funciona bien y conviene conservar.
4. Puntuación por dimensión, con el número de controles verificables de cada una.
5. Hallazgos ordenados por severidad, en el formato de la guía (sección "Formato de cada hallazgo").
6. Pendientes de verificar, con la comprobación concreta de cada uno.
7. Preguntas para negocio.
8. Límites del análisis: datos que no se pudieron consultar y ventanas usadas.

Al final, ofrece un bloque breve de "estado de la auditoría" que el usuario pueda pegar en una sesión futura: IDs, versión, hallazgos con su estado y datos de ejecución obtenidos.

## Referencias

- `references/guia-buenas-practicas.md`: guía de buenas prácticas, checklist de 51 controles y criterios de evaluación.
- `references/lectura-json-ajo.md`: cómo leer la definición de un journey (campos, nodos, ramas, expresiones).
- `references/falsos-positivos.md`: errores de lectura conocidos y cómo comprobarlos antes de reportar.

## Validación de la skill

Antes de usarla con un cliente, y después de cada cambio en la skill o en la guía, ejecútala sobre un journey que ya se haya revisado a mano y compara el resultado. El caso de prueba describe los hallazgos que debe encontrar, lo que no debe reportar y lo que no debe recomendar. Cada cliente tiene su propio caso de prueba, que se guarda fuera de la skill porque contiene identificadores de su instancia.
