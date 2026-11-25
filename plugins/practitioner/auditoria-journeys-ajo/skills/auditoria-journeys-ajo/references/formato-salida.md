# Formato de salida fijo

El informe sigue siempre esta estructura, en este orden. Si una sección no tiene contenido, se deja con "Sin elementos" y el motivo; no se elimina.

## 1. Objeto auditado y datos disponibles

- Nombre exacto, ID de journey, ID y número de versión, estado, sandbox, fecha de despliegue.
- Journeys enlazados por Jump auditados junto con este.
- Tabla de inventario: campo o métrica, disponible (sí, no, parcial), ventana, controles afectados.
- Lista explícita de **datos faltantes** y qué controles quedan pendientes por cada uno.

## 2. Veredicto

Una línea por revisión y una frase que lo justifique:

- **Diseño**: "Apto para publicar", "No apto para publicar", "Requiere versión correctiva" o "Requiere duplicar el journey", con las reglas de la guía.
- **Rendimiento**: "Sano", "Con riesgos", "Degradado" o "Sin datos suficientes".

Matriz de lectura:

| | Rendimiento sano | Rendimiento degradado |
| --- | --- | --- |
| **Diseño sólido** | Mantener y optimizar contenido | Buscar la causa en datos, contenido o configuración externa |
| **Diseño con críticos** | Riesgo latente: corregir antes de que se manifieste | Corregir diseño primero; luego volver a medir |

## 3. Lo que funciona y conviene conservar

Solo elementos con evidencia (nodo, campo o métrica). Sirve para que ninguna corrección rompa lo que ya funciona.

## 4. Scorecards

Dos tablas, una por revisión. Nunca se suman en un total único.

**Scorecard de diseño**

| Dimensión | Peso | Controles evaluables / totales | Cumplimiento | Puntos | Confianza |
| --- | --- | --- | --- | --- | --- |
| Intención y diseño | 15% | | | | |
| Configuración y robustez | 20% | | | | |
| Flujo de datos | 20% | | | | |
| Presión y cumplimiento | 15% | | | | |
| Medición | 15% | | | | |
| Operación | 15% | | | | |
| **Total sobre puntos puntuables** | | | | x / y | |

**Scorecard de rendimiento** (ventana: …, versión: …)

| Dimensión | Peso | Controles evaluables / totales | Cumplimiento | Puntos | Confianza |
| --- | --- | --- | --- | --- | --- |
| Volumen y entrada | 20% | | | | |
| Fiabilidad de ejecución | 25% | | | | |
| Avance y salida | 25% | | | | |
| Resultado de negocio | 20% | | | | |
| Salud de canal | 10% | | | | |
| **Total sobre puntos puntuables** | | | | x / y | |

"Confianza baja" cuando la dimensión tiene menos de tres controles evaluables (diseño) o menos de dos (rendimiento).

## 5. Hallazgos

Dos listas, diseño y rendimiento, cada una ordenada por severidad. Cada hallazgo usa los campos de la guía ("Formato de cada hallazgo") más:

| Campo | Contenido |
| --- | --- |
| ID | D-01, D-02… para diseño; R-01, R-02… para rendimiento |
| Revisión | Diseño o rendimiento |
| Evidencia | Diseño: nodo, rama o campo con su ruta. Rendimiento: métrica, valor, ventana y fuente |
| Relacionado con | IDs del otro lado del diagnóstico cruzado, si los hay |

## 6. Diagnóstico cruzado

Tabla síntoma → causa:

| Hallazgo de rendimiento | Causa en el diseño | Estado |
| --- | --- | --- |
| R-01 … | D-03 … | Causa confirmada / probable / no identificada en el diseño |

## 7. Backlog priorizado

Una fila por acción. Varias acciones pueden resolver un mismo hallazgo, y una acción puede resolver varios.

| Prioridad | Acción | Resuelve | Severidad | Esfuerzo | Restricción de versión | Depende de | Validación |
| --- | --- | --- | --- | --- | --- | --- | --- |

### Escala de esfuerzo

- **S**: cambio de configuración o de un nodo en una nueva versión, sin cambios de lógica. Validable con test mode.
- **M**: cambio de lógica (ramas, rutas alternativas, esperas) o de contenido en varios mensajes, con simulación de todas las ramas.
- **L**: exige duplicar el journey, desplegar de forma coordinada con otro journey, o depende de otro equipo (datos, esquemas, endpoint, legal).

### Matriz de prioridad

| Severidad \ Esfuerzo | S | M | L |
| --- | --- | --- | --- |
| Crítico | P1 | P1 | P1 |
| Alto | P1 | P2 | P2 |
| Medio | P2 | P3 | P3 |
| Bajo | P3 | P4 | P4 |
| Informativo | P4 | P4 | P4 |

Reglas del backlog:

- Dentro de la misma prioridad, ordena por dependencias: lo que desbloquea otras acciones va antes (por ejemplo, la ruta alternativa antes de bajar un timeout).
- Una acción "latente" conserva su prioridad, pero se marca como tal.
- Una acción que depende de un dato pendiente lleva "condicionada a confirmar …".
- Agrupa en un solo despliegue las acciones que exigen la misma nueva versión.

## 8. Pendientes de verificar

Cada uno con la comprobación concreta que lo resolvería y quién puede hacerla.

## 9. Preguntas para negocio

Solo las que cambian una puntuación o una prioridad.

## 10. Límites del análisis

Datos que no se pudieron consultar, ventanas usadas, versiones mezcladas, umbrales de tráfico no alcanzados y herramientas que no devolvieron campos.

## Bloque de estado de la auditoría

Al final, un bloque breve para pegar en una sesión futura:

```
Auditoría AJO · <fecha>
Journey: <nombre> · ID <journey> · versión <n> (<uid versión>) · sandbox <sandbox>
Enlazados: <IDs>
Ventana de rendimiento: <desde> a <hasta> (<zona horaria>)
Scorecard diseño: x/y · rendimiento: x/y
Hallazgos: D-01 <sev> <estado> · R-01 <sev> <estado> · …
Backlog: P1 <n> · P2 <n> · P3 <n> · P4 <n>
Pendientes: <lista corta>
```
