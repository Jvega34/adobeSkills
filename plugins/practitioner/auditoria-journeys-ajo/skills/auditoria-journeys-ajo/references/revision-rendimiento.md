# Revisión de rendimiento de un journey

Esta referencia cubre la segunda revisión: qué dice el reporting del journey. Se hace después de la revisión de diseño y se puntúa aparte. Los umbrales son orientativos: si el cliente tiene los suyos, usa los del cliente y dilo en el informe.

## Métricas mínimas

Comprueba qué llega y con qué ventana antes de puntuar. Lo que falte deja sus controles como pendientes.

| Métrica | Granularidad | Dónde suele estar | Controles |
| --- | --- | --- | --- |
| Entradas | Journey y día | Reporte del journey | R1 |
| Descartes y motivo | Journey | Reporte, alerta Profile Discard Rate, step events | R2 |
| Errores y motivo | Nodo | Reporte por nodo, step events (motivo de fallo) | R3 |
| Perfiles que llegan a cada nodo | Nodo | Reporte del journey, análisis de fallout | R4, R6 |
| Salidas por motivo | Journey | Reporte: exit criteria, End, timeout, error | R5 |
| Perfiles en curso o en espera | Nodo | Reporte en tiempo real | R5, R10 |
| Conversión (métrica de éxito) | Journey, rama, mensaje | Reporting de AJO o CJA | R7 |
| Entregas, rebotes, bajas, quejas | Mensaje | Reporte de canal | R8 |
| Errores, capping y latencia de custom actions | Acción | Reporte, alertas de custom action | R9 |
| Lift frente a holdout | Journey | CJA o reporte de holdout | R11 |

Anota siempre la ventana (fechas y zona horaria), la versión del journey a la que corresponde y cualquier límite de la herramienta (ventana máxima, muestreo, tiempo de consulta).

## Umbrales mínimos de tráfico

Por debajo de estos volúmenes la métrica se marca **datos insuficientes**: se muestra el valor, pero no se puntúa ni genera hallazgo.

| Lectura | Mínimo orientativo |
| --- | --- |
| Tasas del journey (descartes, errores, salidas) | 1.000 entradas en la ventana |
| Tasas por nodo o por rama | 100 perfiles que llegan a ese nodo o rama |
| Conversión por rama o comparación entre ramas | 30 conversiones en cada rama comparada |
| Bajas o quejas por mensaje | 1.000 entregas del mensaje |
| Lift frente a holdout | Holdout de al menos 1.000 perfiles y un ciclo completo del journey |
| Journey recién desplegado | 72 horas desde el despliegue de la versión |

Si la ventana mezcla versiones, separa los datos por versión. Si no se puede, dilo y baja la confianza.

## Controles de rendimiento

Cada control se evalúa como "cumple", "parcial", "no cumple", "no aplica", "pendiente" o "datos insuficientes". Cada hallazgo cita la métrica, su valor, la ventana y la fuente.

| ID | Control | Señal de problema (orientativa) | Evidencia exigida |
| --- | --- | --- | --- |
| R1 | Las entradas están en línea con lo esperado (dry run, tamaño de audiencia o volumen del evento) | Desviación superior al 30% sin explicación, o días a cero | Entradas por día frente a la referencia |
| R2 | Los descartes son bajos y tienen motivo conocido | Más del 5% de descartes, o un motivo dominante (re-entrada, namespace, throughput, cola de eventos) | Tasa y desglose por motivo |
| R3 | Ningún nodo tiene errores sistémicos | Más del 1% de error en un nodo, o un motivo de error repetido | Nodo (id y etiqueta), tasa, motivo |
| R4 | No hay caídas inesperadas entre nodos consecutivos | La mayor caída no la explica una rama, un exit criteria ni un timeout | Par de nodos, perfiles en cada uno, caída en % |
| R5 | La salida por objetivo domina sobre las salidas por timeout, error o perfiles detenidos | Salidas por error o detención mayores que por exit criteria; perfiles detenidos en un nodo sin rama de salida | Salidas por motivo, en número y % |
| R6 | La distribución por rama encaja con lo que el diseño espera | Una rama sin tráfico (posible rama muerta, cruzar con H3), o la rama por defecto recogiendo más del 20% | Perfiles por rama de cada condición |
| R7 | La conversión por rama o por mensaje justifica cada paso | Un mensaje o rama sin conversión incremental frente al paso anterior o frente a la rama sin envío | Métrica de éxito por rama o mensaje |
| R8 | La salud del canal es aceptable por mensaje | Rebote duro > 2%, bajas > 0,5% o quejas > 0,1% por envío | Mensaje, entregas y tasas |
| R9 | Las custom actions responden sin errores ni capping | Error > 1%, capping activado o latencia cercana al timeout | Acción, tasa de error, capping, latencia |
| R10 | Los tiempos reales en el journey encajan con los waits y timeouts | Perfiles acumulados en un nodo más allá del tiempo que el diseño prevé | Nodo, perfiles en espera y antigüedad |
| R11 | El journey muestra lift frente a holdout | Lift no significativo o negativo tras un ciclo completo | Lift, intervalo, tamaño de cada grupo |

## Dimensiones y pesos

| Dimensión | Peso | Controles |
| --- | --- | --- |
| Volumen y entrada | 20% | R1, R2 |
| Fiabilidad de ejecución | 25% | R3, R9 |
| Avance y salida | 25% | R4, R5, R6, R10 |
| Resultado de negocio | 20% | R7, R11 |
| Salud de canal | 10% | R8 |

La puntuación se calcula como en la revisión de diseño: peso por el porcentaje de cumplimiento sobre los controles evaluables. Se excluyen los pendientes y los de datos insuficientes. Con un solo control evaluable en una dimensión, su puntuación lleva "confianza baja".

## Severidad de un hallazgo de rendimiento

Usa la escala de la guía (Crítico a Informativo) con este criterio:

- **Crítico**: envíos erróneos observados (a quien no debía, contenido roto, fuera de consentimiento) o un nodo que detiene a la mayoría de los perfiles.
- **Alto**: pérdida medible de envíos o conversión (caída sin explicación, errores sistémicos, rama por defecto masiva).
- **Medio**: degradación acotada (un mensaje con bajas altas, una custom action lenta).
- **Bajo** o **Informativo**: señales a vigilar o hipótesis a testear.

## Lecturas engañosas

Comprueba esto antes de reportar un hallazgo de rendimiento.

| Lectura tentativa | Por qué puede ser falsa | Comprobación |
| --- | --- | --- |
| "Los perfiles en espera son abandono" | Muchos salen por exit criteria mientras esperan | Revisa las salidas por exit criteria en la misma ventana |
| "Caída fuerte entre dos nodos" | Una condición o un exit criteria filtra a propósito | Mira el mapa de ramas y los exit criteria antes de llamarlo caída |
| "La rama A convierte más que la B" | Las ramas segmentan públicos distintos; no es un experimento | Compara solo ramas con asignación aleatoria (Optimize) o dilo como correlación |
| "El journey convierte bien" | Atribución de último toque de 7 días sin holdout | Pide el lift frente a holdout o márcalo como no demostrado |
| "Pocas entradas" | La ventana incluye días antes del despliegue o un cambio de versión | Ajusta la ventana a la versión auditada |
| "Errores altos en un nodo" | Los errores de un periodo de incidencia ya resuelto inflan la tasa | Mira la serie diaria, no solo el total |
| "Una rama no tiene tráfico" | La ventana es demasiado corta o el caso es raro por diseño | Aplica el mínimo de tráfico y pregunta si la rama es de excepción |
| "Alertas sin disparar, todo bien" | Las alertas no se disparan en test mode ni si no están suscritas | Confirma F2 en la revisión de diseño |
