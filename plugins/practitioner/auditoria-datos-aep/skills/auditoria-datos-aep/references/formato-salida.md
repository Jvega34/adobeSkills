# Formato de salida fijo

El informe sigue siempre esta estructura, en este orden. Si una sección no tiene contenido, se deja con "Sin elementos" y el motivo; no se elimina. Ningún dato personal aparece en el informe: solo recuentos, porcentajes y valores enmascarados.

## 1. Alcance auditado y datos disponibles

- Organización, sandbox y tipo (producción o desarrollo), fecha de la auditoría.
- Objetos en alcance: todo el sandbox o la lista acotada con sus IDs.
- Uso principal de los datos (AJO, destinos, edge, CJA).
- Tabla de inventario: objeto o métrica, disponible (sí, no, parcial), ventana, controles afectados.
- Lista explícita de **datos faltantes** y qué controles quedan pendientes por cada uno.

## 2. Veredicto

Una línea por revisión y una frase que lo justifique:

- **Configuración**: "Lista para activar", "Activar con riesgos", "Requiere corrección antes de activar" o "Requiere rediseño".
- **Salud de datos**: "Sano", "Con riesgos", "Degradado" o "Sin datos suficientes".

Matriz de lectura:

| | Datos sanos | Datos degradados |
| --- | --- | --- |
| **Configuración sólida** | Mantener y vigilar | Buscar la causa en el sistema origen o en el data prep |
| **Configuración con críticos** | Riesgo latente: corregir antes de que se manifieste | Corregir configuración primero; luego volver a medir |

## 3. Lo que funciona y conviene conservar

Solo elementos con evidencia. Sirve para que ninguna corrección rompa lo que ya funciona.

## 4. Mapas

- Mapa de identidad (`lectura-objetos-aep.md`).
- Mapa de perfil (`lectura-objetos-aep.md`).

Si son largos, resume y ofrece la tabla completa aparte.

## 5. Scorecards

Dos tablas, una por revisión. Nunca se suman en un total único.

**Scorecard de configuración**

| Dimensión | Peso | Controles evaluables / totales | Cumplimiento | Puntos | Confianza |
| --- | --- | --- | --- | --- | --- |
| Modelo XDM | 15% | | | | |
| Identidad | 20% | | | | |
| Perfil y merge policies | 15% | | | | |
| Audiencias | 15% | | | | |
| Ingesta | 15% | | | | |
| Gobernanza y consentimiento | 20% | | | | |
| **Total sobre puntos puntuables** | | | | x / y | |

**Scorecard de salud de datos** (ventana: …)

| Dimensión | Peso | Controles evaluables / totales | Cumplimiento | Puntos | Confianza |
| --- | --- | --- | --- | --- | --- |
| Ingesta y frescura | 25% | | | | |
| Identidad observada | 25% | | | | |
| Calidad de campos | 25% | | | | |
| Audiencias | 15% | | | | |
| Capacidad y licencia | 10% | | | | |
| **Total sobre puntos puntuables** | | | | x / y | |

## 6. Hallazgos

Dos listas, configuración y salud, cada una ordenada por severidad. Cada hallazgo usa los campos de "Formato de cada hallazgo" del checklist más:

| Campo | Contenido |
| --- | --- |
| ID | C-01, C-02… para configuración; Q-01, Q-02… para salud |
| Revisión | Configuración o salud |
| Evidencia | Configuración: objeto, ID y campo, o health check. Salud: métrica, valor, ventana, fuente y consulta SQL si la hay |
| Relacionado con | IDs del otro lado del diagnóstico cruzado |

## 7. Diagnóstico cruzado e impacto en activación

| Hallazgo de salud | Causa en la configuración | Estado | Journeys, audiencias o destinos afectados |
| --- | --- | --- | --- |
| Q-01 … | C-03 … | Causa confirmada / probable / no identificada en la configuración | |

## 8. Backlog priorizado

| Prioridad | Acción | Resuelve | Severidad | Esfuerzo | Restricción | Depende de | Validación |
| --- | --- | --- | --- | --- | --- | --- | --- |

### Escala de esfuerzo

- **S**: cambio de configuración sin efecto en datos existentes (descripción, alerta, etiqueta, orden de una merge policy no usada por defecto). Validable en el momento.
- **M**: cambio con efecto en datos o audiencias existentes que no exige reingesta (TTL, merge policy por defecto, método de evaluación, data prep para datos nuevos), con validación en un sandbox de desarrollo o con previsualización.
- **L**: exige schema nuevo, reingesta, cambiar identidades de datos ya ingeridos, corrección en el sistema origen u otro equipo (datos, legal, Adobe).

### Matriz de prioridad

| Severidad \ Esfuerzo | S | M | L |
| --- | --- | --- | --- |
| Crítico | P1 | P1 | P1 |
| Alto | P1 | P2 | P2 |
| Medio | P2 | P3 | P3 |
| Bajo | P3 | P4 | P4 |
| Informativo | P4 | P4 | P4 |

Reglas del backlog:

- Dentro de la misma prioridad, lo que desbloquea otras acciones va antes (por ejemplo, la limpieza de identidades en data prep antes de activar el namespace único).
- Una acción "latente" conserva su prioridad, pero se marca como tal.
- Una acción que depende de un dato pendiente lleva "condicionada a confirmar …".
- Agrupa las acciones que exigen la misma reingesta o el mismo schema nuevo.

## 9. Pendientes de verificar

Cada uno con la comprobación concreta que lo resolvería y quién puede hacerla.

## 10. Preguntas para negocio

Solo las que cambian una puntuación o una prioridad (identificador de persona acordado, fuente de verdad por atributo, política de retención, base legal).

## 11. Límites del análisis

Objetos que no se pudieron leer, ventanas usadas, sandbox de desarrollo, umbrales de volumen no alcanzados, consultas no lanzadas y herramientas que no devolvieron campos.

## Bloque de estado de la auditoría

```
Auditoría datos AEP · <fecha>
Organización: <org> · sandbox <sandbox> (<prod|dev>)
Alcance: <todo el sandbox | IDs>
Ventana de salud: <desde> a <hasta> (<zona horaria>)
Scorecard configuración: x/y · salud: x/y
Hallazgos: C-01 <sev> <estado> · Q-01 <sev> <estado> · …
Backlog: P1 <n> · P2 <n> · P3 <n> · P4 <n>
Pendientes: <lista corta>
```
