# Journeys en AJO: guía de buenas prácticas y checklist de imprescindibles

Nov 12, 2026 · @Johan Vega · revisada el 17/11/2026 con lo aprendido en la primera auditoría asistida de un journey en producción

## Resumen ejecutivo

Un journey rinde cuando cinco decisiones están bien tomadas antes de abrir el canvas: un disparador que capture intención real, una salida ligada al objetivo, un control de presión a nivel de cartera, una ruta para cuando algo falla y una medición incremental. Todo lo demás (copy, diseño, número de pasos) mueve el resultado bastante menos.

La evidencia externa empuja en la misma dirección:

- **El momento pesa más que el mensaje.** En el benchmark de Klaviyo 2025, un flujo de carrito abandonado genera 3,07 USD por destinatario frente a 0,10 USD de una campaña masiva, unas 30 veces más.
- **La frecuencia es la primera causa de baja.** En la encuesta de Litmus (2025, n=1.000), el 67% se da de baja de un retailer por exceso de envíos y el 46% por contenido irrelevante.
- **Personalizar no siempre suma.** McKinsey sitúa el uplift típico en un 10 a 15% de ingresos, pero Gartner (2025) encontró que el 53% vivió la personalización como negativa, con 3,2 veces más probabilidad de arrepentirse de la compra.
- **Medir clics no demuestra valor.** Adobe recomienda medir el lift del journey contra un holdout y con métricas de conversión, no con aperturas ni clics.

Traducido a AJO, los imprescindibles son: tipo de journey elegido por la naturaleza del dato (evento streaming, audiencia streaming o batch), re-entrada y espera definidas a propósito, exit criteria ligados al objetivo, rule sets de frecuencia, horas de silencio y capping de journeys, rutas alternativas en acciones y condiciones, una sola métrica de éxito de negocio con holdout, validación por simulación, test mode y dry run, y alertas suscritas desde el día uno.

A estos se suma una capa que no se ve mirando nodo a nodo: **la integridad del flujo de datos**. Qué se escribe, cuándo y quién lo lee después, si las condiciones cubren todos los casos y si los journeys enlazados por Jump hablan del mismo evento. En la primera auditoría en producción, los cuatro fallos de mayor impacto estaban en esta capa y no en la configuración de catálogo.

Las secciones de checklist y criterios de evaluación convierten estas ideas en controles verificables, pensados para que una skill de Adobe Coworker pueda auditar un journey y devolver hallazgos con severidad y evidencia.

## Alcance y fuentes

La guía cruza tres capas de evidencia: la documentación oficial de Adobe Journey Optimizer (guardrails, diseño de journeys, reglas de negocio, experimentación y reporting), estudios y benchmarks de la industria sobre lifecycle marketing y personalización (consultoras, analistas y proveedores de plataformas de engagement), y los aprendizajes ya documentados en este proyecto (implementaciones AJO desde cero, journeys post visita, migraciones desde SFMC, Campaign y Braze).

La evidencia de proveedores (Braze, Salesforce, Klaviyo, Iterable, etc.) se usa por sus datos agregados, no por su posicionamiento comercial; cuando un dato procede de un estudio de proveedor se indica.

El foco es funcional: qué decisiones de diseño, orquestación y medición separan un journey que rinde de uno que solo funciona. Lo técnico (esquemas, identidades, configuración de canales) aparece solo cuando condiciona el diseño.

## Qué mueve realmente el rendimiento

Ocho palancas explican la mayor parte de la diferencia entre un journey que rinde y uno que solo funciona. Las tres primeras (intención, momento y salida) actúan sobre la conversión; las dos siguientes (presión y relevancia) sobre la retención del canal; las tres últimas hacen que el resultado sea sostenible y demostrable.

&#91;embedded content: árbol de palancas · 3 grupos, 8 palancas\]

Las tres palancas de conversión se deciden dentro de cada journey; las otras cinco se gobiernan a nivel de cartera, datos y medición.

| Palanca | Qué dice la evidencia | Qué implica en AJO |
| --- | --- | --- |
| 1. Disparo por intención | Carrito abandonado: 6,77 USD por destinatario de media en email y 13,70 USD en el top 10%; bienvenida: 5,75 USD ([Klaviyo 2026](https://www.klaviyo.com/blog/abandoned-cart-benchmarks)) | Reservar los journeys unitarios (evento streaming) para momentos de alta intención; lo que no ocurre en tiempo real va por Read Audience |
| 2. Momento y cadencia | Secuencia habitual de carrito: recordatorio a las 2 a 4 h, seguimiento a las 24 h y cierre a las 48 h ([Klaviyo 2026](https://www.klaviyo.com/blog/abandoned-cart-benchmarks)) | Esperas y timeouts de evento calibrados por objetivo; un journey por evento puede tardar hasta 5 min en ejecutar la primera acción ([Adobe guardrails](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/get-started/essentials/guardrails)) |
| 3. Salida ligada al objetivo | Adobe define los exit criteria como el mecanismo para que quien ya convirtió no reciba más mensajes del journey ([propiedades](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/journey-properties)) | Exit criteria por evento unitario o audiencia streaming, evaluados en cada paso con lógica OR |
| 4. Presión de contacto | El 67% se da de baja de un retailer por exceso de frecuencia ([Litmus 2025](https://www.litmus.com/blog/why-your-best-marketing-channel-is-driving-customers-away)) | Rule sets de canal (frecuencia, horas de silencio) y de journey (entrada y concurrencia), más prioridad y detección de conflictos |
| 5. Relevancia útil | Uplift típico del 10 al 15% ([McKinsey](https://www.mckinsey.com/featured-insights/destacados/el-valor-de-entender-bien-o-mal-la-personalizacion-se-multiplica/es)); la personalización pasiva genera 3,2 veces más arrepentimiento ([Gartner 2025](https://www.gartner.com/en/newsroom/press-releases/2025-06-03-gartner-survey-reveals-personalization-can-triple-the-likelihood-of-customer-regret-at-key-journey-points)) | Personalizar con contexto del evento y datos declarados; en los puntos de decisión, ayudar a decidir en vez de empujar una oferta |
| 6. Orquestación multicanal | Retención a 30 días del 82% con mensajería híbrida (push y pull) frente al 15% sin mensajes ([Braze 2025](https://www.braze.com/resources/articles/building-a-retention-machine-6-data-driven-insights)) | Combinar canales salientes (email, push, SMS, WhatsApp) con entrantes (in-app, web, code-based) y fallback por canal |
| 7. Identidad | Los usuarios conocidos retienen 2,3 veces más a 30 días que los anónimos ([Braze 2025](https://www.braze.com/resources/articles/building-a-retention-machine-6-data-driven-insights)) | Namespace del journey coherente con el evento y la audiencia; identidad de persona en el propio evento |
| 8. Medición incremental | Holdout mínimo recomendado del 5% y 1.000 perfiles, con métricas de conversión ([Adobe](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/journey-properties)); Braze: control de al menos 1.000 usuarios y no más del 10% ([Braze docs](https://www.braze.com/docs/user_guide/engagement_tools/testing/global_control_group)) | Holdout de journey (Limited Availability), experimentos de ruta con Optimize y una métrica de éxito de negocio |

Cómo leer estas cifras: proceden mayoritariamente de proveedores, con sesgo hacia ecommerce y mercado estadounidense, y varias son correlacionales (quien recibe un carrito abandonado ya tenía alta intención). Sirven como órdenes de magnitud y para priorizar, no como objetivo contractual. La única cifra válida para un cliente concreto es el lift medido contra su propio holdout.

## Elementos críticos según el objetivo

El objetivo decide el tipo de entrada, la cadencia, la salida y la métrica; si cualquiera de los cuatro no se puede justificar desde el objetivo, el journey está mal planteado. Las cadencias son hipótesis de partida para testear, no reglas.

| Objetivo | Entrada en AJO | Cadencia orientativa | Salida (exit) | Métrica de éxito | Error típico |
| --- | --- | --- | --- | --- | --- |
| Bienvenida y onboarding | Evento de alta o audiencia streaming de cliente nuevo | 3 a 5 toques en 7 a 14 días, cada uno empujando un hito (app, primera compra, activación) | Al completar el hito de cada fase | Tasa de activación o de primer hito | Leer el perfil justo tras crearlo (latencia de ingesta) y seguir enviando a quien ya activó |
| Carrito o navegación abandonada | Evento streaming de carrito o checkout | Recordatorio a las 2 a 4 h, seguimiento a las 24 h, cierre a las 48 h | Compra (exit criteria por evento) | Ingresos o compras por perfil entrado | Wait fijo en lugar de esperar el evento de compra con timeout; descuento a todos los carritos |
| Reactivación y win-back | Read Audience recurrente con lectura incremental (la inactividad es ausencia de eventos) | 2 a 3 toques en 2 a 4 semanas, con escalado de incentivo | Vuelta a la actividad; al final, supresión o sunset de quien no responde | Reactivados frente a holdout | Regalar incentivo a quien habría vuelto solo; sin holdout no se ve |
| Transaccional y servicio | Evento unitario o campaña API-triggered | Un mensaje, inmediato | Fin del flujo | Entrega, latencia, contactos evitados al call center | Mezclar promoción en mensajes transaccionales y usar configuración de canal de marketing |
| Cross-sell y siguiente mejor oferta | Evento de compra o hito, con Decisioning para elegir la oferta | Tras un periodo de cortesía; 1 a 2 toques | Compra del producto ofertado o pérdida de elegibilidad | Conversión y margen incremental de la oferta | Ofrecer lo que ya tiene; empujar oferta en pleno momento de decisión |
| Retención y prevención de churn | Audiencia por propensión o señales de riesgo, vía Read Audience | Mensajes de valor y ayuda, no solo descuento | Renovación, cancelación o mejora de la señal | Churn frente a holdout | Medir aperturas en lugar de bajas evitadas |
| Post-compra y feedback | Evento de entrega confirmada | Solicitud a los pocos días; un recordatorio | Feedback enviado o 10 días sin respuesta (ejemplo de Adobe) | Tasa de respuesta, recompra | Pedir valoración antes de que llegue el producto |

La referencia de Adobe para estos patrones está en la [guía de entrada y salida](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/journey-use-cases/entry-exit-criteria-guide), que incluye los ejemplos de bienvenida, carrito, fidelización y feedback. Los tiempos de carrito proceden del [benchmark de Klaviyo 2026](https://www.klaviyo.com/blog/abandoned-cart-benchmarks).

Dos reglas transversales:

- **Un objetivo por journey.** Si el journey persigue a la vez la descarga de app y la primera compra, se parte en sub-journeys por fase, como recomienda Adobe en su [guía de journeys avanzados](https://experienceleague.adobe.com/en/perspectives/best-practices-for-advanced-journeys-in-journey-optimizer).
- **La ausencia de comportamiento no dispara nada.** Reglas como "no ha vuelto en Y días" se resuelven con un timeout de evento, un Wait o una evaluación programada (Read Audience), nunca esperando un evento que no llegará.

## Buenas prácticas funcionales en AJO

Las prácticas se agrupan en siete decisiones de diseño. Cada una incluye la regla del producto que la condiciona, porque en AJO muchas buenas prácticas son en realidad la forma de no chocar con un guardrail.

### 1. Elegir el tipo de journey por la naturaleza del dato

- **Evento unitario** para reaccionar a un comportamiento individual. Solo disparan eventos enviados en streaming; los ingestados en batch, los creados con Query Service y los de datasets internos de AJO no sirven ([guardrails](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/get-started/essentials/guardrails)).
- **Audience Qualification** solo con audiencias streaming. Desde septiembre de 2026 AJO bloquea la publicación de journeys nuevos, borradores o duplicados con audiencia batch en Audience Qualification o en exit criteria ([propiedades](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/journey-properties)).
- **Read Audience** para todo lo que depende de agregados históricos o de ausencia de actividad. Programar con lectura incremental y separar los arranques de distintos Read Audience al menos 5 a 10 minutos.
- **Campaña (action o API-triggered)** cuando no hay lógica multi-paso: newsletters, promociones puntuales, transaccionales simples. Adobe reserva los journeys para orquestación 1:1 con condiciones ([introducción a journeys](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/journey)).
- **Paquete Select**: los journeys unitarios solo admiten una acción, sin condiciones ni Wait. Comprobar la licencia antes de diseñar.

### 2. Entrada y re-entrada

- La re-entrada viene activada por defecto. Desactivarla en journeys de un solo uso; si se mantiene, fijar el periodo de espera a propósito (5 minutos por defecto, máximo 90 días).
- Un perfil no puede estar dos veces a la vez en el mismo journey y solo re-entra tras salir del todo. El bloqueo de re-entrada no se garantiza más allá de 91 días, porque AJO borra la instancia.
- Usar identificadores suplementarios cuando la re-entrada debe ser por contexto (por ejemplo, un journey por pedido o por reserva).
- La regla de re-entrada debe ser la misma en todas las versiones del journey. Cambiarla exige duplicar el journey: hay que planificar qué pasa con los perfiles que siguen dentro de la versión actual.
- Filtrar en la entrada: consentimiento del propósito, canal alcanzable (email o token presente) y perfil identificado si el journey lo necesita.
- Usar en la exclusión de entrada las mismas audiencias que en los exit criteria, en streaming. Si la entrada usa una audiencia batch y la salida una streaming, un perfil que cualificó antes de entrar no aparece aún en la batch y tampoco dispara la salida.

### 3. Lógica del canvas

- **Esperar el evento, no el reloj.** Para "si no compra en 4 h", usar una actividad de evento con timeout y ruta de timeout, no un Wait de 4 h seguido de una condición.
- **Rutas alternativas siempre.** Si una acción, condición o custom action falla sin ruta alternativa, el perfil se detiene en ese punto ([actividades](https://experienceleague.adobe.com/docs/journey-optimizer/using/orchestrate-journeys/about-journey-building/about-journey-activities.html?lang=en)).
- **Timeouts realistas.** El timeout de custom action y data source es de 30 s por defecto y Adobe recomienda entre 1 y 30 s; corto si el journey es sensible al tiempo. El reintento ante incidencias internas es de hasta 72 h: acortarlo si un mensaje tardío ya no tiene sentido.
- **Margen tras crear un perfil.** La ingesta al perfil tarda menos de 1 minuto en el percentil 95; si el journey nace con el alta, añadir un Wait o personalizar con los datos del propio evento.
- **Custom actions idempotentes.** AJO reintenta tres veces cualquier error HTTP salvo 401, 403 y 404; el endpoint debe soportar al menos 200 TPS.
- **Coherencia de merge policy.** Usar en `inAudience` audiencias con otra merge policy bloquea la publicación; usarlas en personalización no da aviso, así que hay que revisarlo a mano.
- **Sin lookup de experience events.** Para contadores o recencia, usar computed attributes, audiencias o Data Distiller.
- **Etiquetas con significado**: "Exit - Compra completada", no "Exit 1". Las etiquetas aparecen en reporting y en los logs. Cada nodo End lleva el motivo de salida.
- **No leer lo que se acaba de escribir.** Update profile y las custom actions que publican un evento de ingesta escriben en el perfil de forma asíncrona. Una condición que lee ese atributo en el nodo siguiente ve el valor anterior. Usar el dato que ya se tiene (del evento o el valor que se acaba de escribir) o enrutar directamente sin releer.
- **Condiciones que cubren todos los casos.** Toda condición lleva rama por defecto ("otros casos"). Los rangos numéricos y horarios se comprueban valor a valor: un `>6` donde debía ir `>5` deja una hora entera sin rama, y el perfil sale del journey. Las ramas se evalúan en orden y gana la primera verdadera.
- **Dos ramas con el mismo destino no son un duplicado.** Es habitual que una rama concreta y la rama por defecto converjan en el mismo nodo. Eliminar la rama por defecto deja sin camino a los perfiles que no cumplen ninguna otra.
- **Una sola evaluación por dato inmutable.** Los datos del evento de entrada no cambian durante el journey: evaluarlos dos veces crea ramas que nunca se pueden tomar.
- **Los datos que se necesitan después se garantizan en todos los caminos.** Si un nodo posterior, o el journey destino de un Jump, necesita un atributo (la dirección de envío, un identificador), todos los caminos que llegan a él deben escribirlo o comprobarlo.
- **Fechas y horas con funciones de fecha.** No extraer la hora recortando el texto de una fecha: depende del formato y de la zona horaria, y se desplaza con el cambio de hora. Usar funciones de fecha con zona horaria explícita y validar la hora real de envío.
- **Comentarios fieles al código.** Los comentarios y las etiquetas de rama se leen como documentación. Si no coinciden con la expresión (por ejemplo, una cadencia que ya no existe), inducen a error a quien mantiene o audita el journey.
- **Ruta alternativa donde tiene efecto.** El flag de ruta alternativa ante error o timeout importa en custom actions, Update profile, Jump, acciones de mensaje y condiciones que consultan fuentes externas. En timers no tiene efecto.

### 4. Salida y fin del journey

- Definir exit criteria por evento unitario o por audiencia streaming; varios criterios se evalúan con OR en cada paso. Los basados en audiencia pueden tardar hasta 10 minutos en surtir efecto.
- Cerrar a nuevas entradas no es lo mismo que exit criteria: lo primero deja terminar a quien ya está dentro.
- Usar fecha de fin en journeys estacionales. Recordar el timeout global de 91 días: un perfil solo entra en un Wait si le queda tiempo para completarlo.
- Ante un incidente, pausar el journey y usar el exit criteria por atributo de perfil (uno por journey, evaluado en los nodos de acción).

### 5. Gobierno de la presión

- **Rule sets de canal**: frecuencia y horas de silencio. Los periodos de capping son de calendario en UTC y el perfil que supera el límite se excluye del siguiente envío, no se pospone. Hay 10 rule sets de canal y 10 de journey por sandbox ([rule sets](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/configuration/frequency-rules)).
- **Rule sets de journey**: capping de entradas y de concurrencia, con arbitraje por prioridad (0 a 100). El capping de entrada solo funciona si el namespace del journey es el de mayor prioridad del sandbox, y con varias Audience Qualification simultáneas los recuentos no son exactos ([conflictos](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/conflict-prioritization/gs-conflict-prioritization)).
- **Revisar conflictos antes de publicar**: solape de audiencia, fechas, canal y configuración de canal desde las propiedades del journey.
- **Zona horaria** del perfil cuando la hora de recepción importa; fija solo si la audiencia es de un único huso.

### 6. Contenido, canal y personalización

- Elegir canal por objetivo y alcanzabilidad, con fallback: si no hay token push, email; si no hay consentimiento de email, inbound. Las reaction events deben ir justo después de la acción que observan.
- Personalizar con el contexto del evento y con datos declarados; poner valores por defecto en todos los campos dinámicos.
- Para ofertas, usar Decisioning (Experience Decisioning); Decision Management está marcado como legacy.
- Separar configuración de canal transaccional y de marketing; nunca mezclar promoción en un transaccional.
- Un in-app vive lo que vive el journey del perfil: al salir, deja de mostrarse.

### 7. Organización y mantenimiento

- Documentar entrada, ruta y salida en un diagrama antes de construir, y alinear con analítica.
- Partir journeys largos en sub-journeys por hito con Jump. El límite es de 50 actividades y 2 MB de payload, y la edición se degrada antes de llegar.
- Usar tags y categorías para metadatos (equipo, objetivo, fechas) y dejar el nombre para el hito que se persigue.
- Revisar los journeys activos cada trimestre. Límites a vigilar: 200 journeys activos en producción y 100 en desarrollo, y un mismo evento usado como máximo por 25 journeys.
- Separar el journey que detecta y registra del que comunica. Una espera mínima antes del primer mensaje absorbe la latencia de ingesta al perfil y la de los exit criteria por audiencia (hasta 10 minutos).
- Retirar versiones cerradas y copias de QA que ya no se usan. Evitar versiones paralelas de un mismo evento (v1, v2, v3, v4) que distintos journeys escuchan sin saberlo.

### 8. Journeys enlazados por Jump

- **Se auditan juntos.** Origen y destino forman un solo flujo: lo que el origen escribe en el perfil es lo que el destino usa para enviar.
- **El mismo comportamiento, el mismo evento.** Si el destino escucha un evento para saber que el usuario ha vuelto, debe ser el mismo evento, y la misma versión, que dispara el origen.
- **El Jump guarda la versión de destino.** Al redesplegar el destino, comprobar a qué versión llegan los perfiles y desplegar origen y destino de forma coordinada.
- **Datos garantizados.** Todo camino del origen que llega al Jump garantiza los atributos que el destino necesita.
- **Exit criteria alineados.** El destino comparte con el origen los exit criteria que representan la conversión o la exclusión.
- **Re-entrada coherente.** La espera de re-entrada del origen controla cuántas veces se reinicia la secuencia del destino.

## Medición, experimentación y monitoreo

Un journey se mide con una métrica de negocio contra un holdout y se opera con alertas y step events; las aperturas y los clics sirven para diagnosticar el contenido, no para declarar éxito.

### Métrica de éxito

- AJO admite **una sola métrica de éxito por journey**. Para que existan métricas de conversión (compras, checkouts, instalaciones, page views) hay que añadir en Reporting un dataset con los field groups estándar Commerce Details, Web y Mobile; sin él solo hay clics, CTR y tasa de apertura ([métricas](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/success-metrics)).
- Solo con licencia de AJO, la atribución es último toque con ventana máxima de 7 días. Con CJA se pueden crear métricas propias y cambiar la atribución: imprescindible cuando el ciclo de conversión es más largo que una semana (banca, seguros, B2B).
- El reporting de la interfaz muestra unos 91 días. Para series más largas, CJA o exportación de datasets.

### Holdout y experimentos

- **Holdout de journey** (Limited Availability): excluye un porcentaje aleatorio y determinista de la entrada. Adobe recomienda al menos un 5% y 1.000 perfiles, una métrica de fondo de embudo y no usar aperturas ni clics; los cambios de porcentaje en una nueva versión solo afectan a los nuevos entrantes ([propiedades](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/journey-properties)).
- **Experimentos de ruta** con la actividad Optimize (A/B o multi-armed bandit): cada actividad asigna de forma independiente, así que dos experimentos en serie se cruzan ([experimentos de ruta](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/about-journey-building/optimize-activity/path-experimentation)).
- **Experimentos de contenido**: aislar una sola variable y dejar correr el test el tiempo suficiente; AJO usa estadística "anytime valid", que permite mirar resultados sin invalidar el test ([content experiments](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/content-management/content-experiment/get-started-experiment)).
- No cambiar contenido, audiencia ni tamaño del grupo de control mientras el test está en marcha, y dejar que dure al menos un ciclo completo del journey.

### Cuadro de mando operativo

| Señal | Qué indica | Dónde se ve |
| --- | --- | --- |
| Entradas frente a lo esperado | Problemas de evento, namespace o audiencia | Reporte del journey; dry run como línea base |
| Tasa de descartes de perfil | Re-entrada bloqueada, identidad incompatible, throughput superado | Alerta Profile Discard Rate Exceeded |
| Tasa de perfiles en error | Fallos de ejecución sistémicos | Alerta Profile Error Rate Exceeded; step events |
| Errores y capping de custom actions | Endpoint caído, credenciales caducadas, límite de llamadas | Alertas Custom Action Error Rate Exceeded y Capping Activated |
| Read Audience sin ejecutar | Audiencia vacía o error de programación | Alerta Read Audience trigger unsuccessful |
| Salida por objetivo frente a timeout | Si el journey convierte o solo acompaña | Reporte del journey y exit criteria etiquetados |
| Bajas y quejas por mensaje | Fatiga o irrelevancia en un paso concreto | Reporte de canal y feedback de entregabilidad |
| Lift frente a holdout | Valor real del journey | Reporte de lift en CJA |

Las alertas de journey evalúan la proporción sobre los últimos 5 minutos, permiten ajustar umbrales y se suscriben por sandbox o por journey; solo se disparan en journeys live, no en test mode ([alertas](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/monitor/alerts)). Para diagnosticar, consultar los step events (campo de motivo de fallo) y usar las propiedades técnicas del journey, como el último código de error, para enrutar errores a otro sistema ([propiedades técnicas](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/building-advanced-conditions-journeys/syntax/journey-properties)).

Cadencia recomendada: revisión diaria durante las primeras 72 horas y tras cada nueva versión, semanal en régimen estable y trimestral a nivel de cartera (solapes, journeys obsoletos, capping).

## Guardrails que condicionan el diseño

Estos límites, vigentes según la documentación de Adobe consultada en noviembre de 2026, son los que más a menudo obligan a rediseñar un journey; algunos varían por contrato o paquete. Fuentes: [guardrails](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/get-started/essentials/guardrails), [propiedades](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/journey-properties), [métricas](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/success-metrics) y [rule sets](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/configuration/frequency-rules).

| Límite | Valor | Implicación de diseño |
| --- | --- | --- |
| Actividades por journey | 50, no ampliable | Partir en sub-journeys con Jump mucho antes del límite |
| Payload del journey | 2 MB (aviso al 90%) | Simplificar expresiones y mapeos; vigilar el indicador en propiedades |
| Journeys activos a la vez | 200 en producción, 100 en desarrollo | Retirar journeys obsoletos; cuentan live, cerrados, pausados y dry run |
| Instancia de perfil | 1 MB | Evitar arrastrar respuestas grandes de custom actions |
| Payload de evento | 64 KB | Eventos mayores se descartan sin disparar |
| Cola de eventos por perfil y versión | 10 pendientes | El resto se descarta (maxInstanceStackEventsReached) |
| Throughput de entrada | 5.000 eventos/s unitarios y 5.000 de Read Audience, en todas las sandboxes | Hacer el evento de entrada lo más selectivo posible |
| Primera acción tras evento | Hasta 5 minutos | No prometer inmediatez de segundos en journeys por evento |
| Uso de un evento | Máximo 25 journeys | Diseñar eventos por propósito, no un evento genérico para todo |
| Audience Qualification | Hasta 10 min en activarse; 300 por sandbox | Contar también las usadas como exit criteria |
| Audiencias batch | Bloqueadas en Audience Qualification y exit criteria desde septiembre de 2026 | Batch solo por Read Audience |
| Re-entrada | 5 min por defecto, máximo 90 días | Fijarla según la frecuencia máxima deseada del mensaje |
| Timeout global | 91 días por perfil | Ningún journey puede durar más; los Wait largos se recortan |
| Custom actions | 300.000 llamadas/min por host y sandbox (150.000 cada 30 s si el endpoint tarda más de 0,75 s); 3 reintentos | Endpoints rápidos, idempotentes y de al menos 200 TPS |
| Métrica de éxito | 1 por journey; atribución de 7 días sin CJA | Elegirla con el negocio antes de construir |
| Rule sets | 10 de canal y 10 de journey por sandbox | Diseñar una política de presión común, no un rule set por journey |
| Versiones | Tipo de entrada, audiencia de Audience Qualification, namespace y regla de re-entrada fijos | Decidirlos bien en la v1; si cambian, duplicar |
| Datasets de sistema de AJO | TTL de 90 días en perfil y 13 meses en data lake | Adobe ha aplazado el despliegue en sandboxes existentes (previsto para el 1 de octubre de 2026): confirmar el estado en cada cliente |

## Antipatrones frecuentes

Casi todos los journeys que rinden mal repiten alguno de estos patrones, y la mayoría se detectan leyendo la definición del journey sin necesidad de datos de ejecución.

| Antipatrón | Síntoma | Cómo detectarlo | Corrección |
| --- | --- | --- | --- |
| Sin salida ligada al objetivo | "Sigo recibiendo mensajes después de comprar" | Exit criteria vacíos o sin relación con la métrica de éxito | Exit criteria por el evento de conversión |
| Wait fijo seguido de condición | Mensajes a quien ya convirtió durante la espera | Secuencia Wait y Condition evaluando un evento | Actividad de evento con timeout y ruta de timeout |
| Re-entrada por defecto en journeys comerciales | La misma serie llega varias veces | Re-entrada activa con espera de 5 minutos | Desactivar o fijar una espera acorde a la frecuencia deseada |
| Acciones sin ruta alternativa | Perfiles detenidos en silencio | Acciones, condiciones o custom actions sin "alternative path" | Añadir ruta de error y timeout con fallback o registro |
| Journey monolito | QA imposible, aviso de payload, ediciones lentas | Más de 30 o 40 actividades, varios hitos en un canvas | Sub-journeys por hito unidos con Jump |
| Audiencia batch donde no toca | Publicación bloqueada desde septiembre de 2026 | Audiencia batch en Audience Qualification o exit criteria | Read Audience, o audiencia streaming |
| Leer el perfil justo tras el alta | Personalización vacía o rama equivocada | Primera acción o condición sin Wait tras el evento de registro | Wait corto o datos del propio evento |
| Sin política de presión | Varios mensajes el mismo día, bajas al alza | Sin rule set de canal ni de journey; conflictos sin revisar | Rule sets comunes, prioridad y capping de entrada |
| Éxito medido con aperturas | Journey "exitoso" sin impacto en negocio | Métrica de éxito de apertura o clic; sin holdout | Métrica de conversión y holdout |
| Incentivo para todos | Margen erosionado; clientes entrenados a esperar descuento | Oferta sin condición de valor ni holdout | Incentivo solo en segmentos de alto valor, medido contra control |
| Custom action frágil | Duplicados en el sistema destino, rutas de timeout masivas | Endpoint no idempotente; timeout por defecto en un caso de tiempo real | Idempotencia, timeout de 1 a 5 s, alerta de errores |
| Test mode con datos reales | Envíos a clientes durante QA | Perfiles de test con direcciones de clientes | Perfiles de test con direcciones controladas; simulación |
| Promoción dentro de transaccionales | Problemas de entregabilidad y de base legal | Configuración de canal transaccional con contenido comercial | Separar mensajes y configuraciones |
| Nombres como metadatos | Inventario ilegible, filtros que fallan | Nombres largos con equipo, objetivo y fechas; sin tags | Tags y categorías; nombre igual al hito |
| Leer lo que se acaba de escribir | Rama equivocada tras un reinicio o una actualización | Condición sobre un atributo de perfil justo después del nodo que lo escribe | Usar el valor que ya se tiene o enrutar sin releer |
| Condición con huecos | Perfiles que salen sin mensaje a ciertas horas o con ciertos valores | Rangos no contiguos; sin rama por defecto | Corregir los límites y añadir rama por defecto |
| Dato no garantizado | El perfil termina sin envío aunque el evento traía el dato | Un camino llega al filtro o al Jump sin pasar por el nodo que guarda el dato | Guardar el dato en todos los caminos o leerlo del evento |
| Eventos desalineados entre journeys | El usuario que vuelve sigue recibiendo mensajes | El destino escucha otra versión del evento que dispara el origen | Escuchar el mismo evento o usar un exit criterion |
| Hora calculada como texto | Envíos fuera de la franja tras el cambio de hora | `substr` sobre una fecha convertida a texto | Funciones de fecha con zona horaria explícita |
| Comentarios desactualizados | Auditorías y mantenimiento basados en lógica que no existe | Comentarios que describen pasos, tiempos o valores distintos del código | Actualizar los comentarios con cada versión |

## Checklist de imprescindibles

El checklist tiene 51 controles en ocho fases; los marcados como **\[B\]** son bloqueantes (no se publica sin ellos) y los **\[R\]** son recomendados. Los identificadores son estables para que la skill de Coworker pueda citarlos en sus hallazgos.

### A. Estrategia (antes del canvas)

- [ ] **A1 \[B\]** El journey tiene un único objetivo de negocio escrito en una frase.
- [ ] **A2 \[B\]** Hay una métrica de éxito de conversión acordada con negocio (no apertura ni clic).
- [ ] **A3 \[B\]** Entrada, rutas y salida están documentadas en un diagrama antes de construir.
- [ ] **A4 \[R\]** Hay una hipótesis de cadencia y de canal por objetivo, pendiente de testear.
- [ ] **A5 \[B\]** Se revisaron los journeys y campañas activos que comparten audiencia o canal.

### B. Configuración del journey

- [ ] **B1 \[B\]** El tipo de journey encaja con el dato: evento streaming, audiencia streaming, Read Audience o campaña.
- [ ] **B2 \[B\]** El namespace es coherente con el evento, la audiencia y los exit criteria.
- [ ] **B3 \[B\]** La re-entrada está decidida a propósito (activa o no, y con qué espera).
- [ ] **B4 \[B\]** Hay exit criteria ligados al objetivo, por evento unitario o audiencia streaming.
- [ ] **B5 \[B\]** Ninguna audiencia batch se usa en Audience Qualification ni en exit criteria.
- [ ] **B6 \[R\]** Zona horaria del perfil cuando la hora de recepción importa.
- [ ] **B7 \[R\]** Fecha de fin en journeys estacionales o de campaña.
- [ ] **B8 \[B\]** La métrica de éxito está asignada en propiedades y el dataset de reporting tiene los field groups necesarios.
- [ ] **B9 \[R\]** Tags y categorías aplicados; el nombre describe el hito.

### C. Lógica y contenido

- [ ] **C1 \[B\]** Las esperas de comportamiento usan actividad de evento con timeout, no Wait más condición.
- [ ] **C2 \[B\]** Todas las acciones, condiciones y custom actions tienen ruta alternativa de error y timeout.
- [ ] **C3 \[B\]** Si el journey nace con un alta, hay margen antes de leer el perfil o se usan datos del evento.
- [ ] **C4 \[R\]** Menos de 30 a 40 actividades; si no, sub-journeys con Jump.
- [ ] **C5 \[B\]** Merge policy coherente entre audiencias de condiciones y de personalización.
- [ ] **C6 \[B\]** Todos los campos personalizados tienen valor por defecto.
- [ ] **C7 \[R\]** Hay fallback de canal cuando el principal no es alcanzable.
- [ ] **C8 \[R\]** Las ofertas se resuelven con Decisioning y reglas de elegibilidad, no con lógica fija en el canvas.
- [ ] **C9 \[R\]** Etiquetas descriptivas en actividades y salidas.

### D. Gobierno y cumplimiento

- [ ] **D1 \[B\]** La entrada filtra por consentimiento del propósito y por canal alcanzable.
- [ ] **D2 \[B\]** Hay rule set de frecuencia por canal aplicado (o el global por defecto revisado).
- [ ] **D3 \[R\]** Horas de silencio para canales intrusivos (SMS, push, WhatsApp).
- [ ] **D4 \[R\]** Prioridad y capping de entrada o concurrencia definidos frente a journeys solapados.
- [ ] **D5 \[B\]** Configuración de canal transaccional o de marketing correcta según el contenido.
- [ ] **D6 \[R\]** Custom actions idempotentes, con timeout ajustado y endpoint de al menos 200 TPS.

### E. Validación

- [ ] **E1 \[B\]** El journey valida sin errores en el panel de alertas.
- [ ] **E2 \[B\]** Simulación o test mode recorre cada rama, incluidas las de timeout y error.
- [ ] **E3 \[B\]** Los perfiles de test usan direcciones controladas por el equipo.
- [ ] **E4 \[R\]** Dry run contra la audiencia real para confirmar volúmenes por rama.
- [ ] **E5 \[B\]** Proofs de contenido revisados en dispositivos y clientes de correo.

### F. Lanzamiento

- [ ] **F1 \[R\]** Holdout configurado (5% o más y al menos 1.000 perfiles) o justificación de por qué no.
- [ ] **F2 \[B\]** Alertas de journey suscritas con umbrales revisados.
- [ ] **F3 \[R\]** Volumen inicial controlado (waves o audiencia acotada) en journeys de gran alcance.

### G. Monitoreo continuo

- [ ] **G1 \[B\]** Revisión diaria de entradas, descartes y errores durante las primeras 72 horas.
- [ ] **G2 \[R\]** Revisión semanal de salida por objetivo frente a timeout y de bajas por mensaje.
- [ ] **G3 \[R\]** Lectura de lift frente a holdout al cerrar cada ciclo de medición.
- [ ] **G4 \[R\]** Revisión trimestral de cartera: solapes, journeys obsoletos, límites de journeys activos.

### H. Flujo de datos y journeys enlazados

- [ ] **H1 \[B\]** Ninguna condición lee del perfil un atributo que el propio journey acaba de escribir sin una espera suficiente; en su lugar se usa el valor conocido o se enruta sin releer.
- [ ] **H2 \[B\]** Toda condición tiene rama por defecto o cubre todos los valores posibles; los rangos numéricos y horarios son contiguos y sin huecos.
- [ ] **H3 \[R\]** No hay condiciones repetidas sobre datos que no cambian (ramas que nunca se pueden tomar).
- [ ] **H4 \[B\]** Los datos que necesita un nodo posterior o el journey destino se garantizan en todos los caminos que llegan a él.
- [ ] **H5 \[B\]** Los journeys enlazados por Jump se auditan juntos, origen y destino.
- [ ] **H6 \[B\]** El destino escucha el mismo evento, en la misma versión, que el origen para el mismo comportamiento.
- [ ] **H7 \[R\]** La versión de destino del Jump es la desplegada y los redespliegues se coordinan.
- [ ] **H8 \[R\]** Las audiencias de exclusión de entrada coinciden con las de exit criteria y son streaming.
- [ ] **H9 \[R\]** Los cálculos de fecha y hora usan funciones de fecha con zona horaria explícita, no recortes de texto.
- [ ] **H10 \[R\]** Los comentarios y las etiquetas describen lo que hace el código.

## Criterios de evaluación

La skill audita, no corrige: puntúa el journey, devuelve hallazgos con severidad y evidencia, y separa lo que verificó en la definición de lo que necesita confirmar una persona. El MCP de AJO es de solo lectura sobre journeys, campañas y configuraciones de canal, así que ningún cambio se aplica desde la skill.

### Dimensiones y pesos

| Dimensión | Peso | Controles del checklist | Se verifica en la definición | Se pregunta |
| --- | --- | --- | --- | --- |
| Intención y diseño | 15% | A1, A3, A4, B1, C1, C4 | Tipo de entrada, esperas por evento, número de actividades | Objetivo de negocio, hipótesis de cadencia |
| Configuración y robustez | 20% | B2, B3, B4, B5, B7, C2, C3, C5, C6, C9, D6, E1 | Namespace, re-entrada, exit criteria, rutas alternativas, tipo de audiencia, etiquetas | Comportamiento del endpoint de custom actions |
| Flujo de datos | 20% | H1 a H10 | Escrituras y lecturas, cobertura de condiciones, journeys enlazados, eventos, versiones | Intención de rutas ambiguas (por ejemplo, adónde debe ir un valor nulo) |
| Presión y cumplimiento | 15% | A5, B6, D1, D2, D3, D4, D5 | Rule sets aplicados, zona horaria, condición de consentimiento en entrada | Base legal del propósito, política de frecuencia de la marca |
| Medición | 15% | A2, B8, F1, G3 | Métrica de éxito asignada, holdout activo | Si la métrica refleja el objetivo; dataset de reporting |
| Operación | 15% | B9, C7, C8, E2, E3, E4, E5, F2, F3, G1, G2, G4 | Tags, fallback de canal, Decisioning | Validaciones hechas, alertas suscritas, perfiles de test |

### Cómo se puntúa

- Cada control es "cumple" (1), "parcial" (0,5), "no cumple" (0), "no aplica" o "pendiente de confirmar".
- **Pendiente no es incumplido.** Un control cuyo dato no llegó a la skill, o que depende de algo que hay que preguntar, se marca pendiente y se excluye del cálculo. Nunca se deduce que no se cumple porque no se ve.
- La puntuación de una dimensión es el peso por el porcentaje de cumplimiento sobre sus controles verificables. El total se expresa sobre los puntos puntuables, no sobre 100.
- Cada dimensión muestra cuántos controles se pudieron verificar. Con menos de tres, su puntuación se marca como "confianza baja".

### Severidad

- **Crítico**: un control \[B\] incumplido con un mecanismo concreto, que se puede señalar en la definición, de envío erróneo, incumplimiento legal o publicación bloqueada.
- **Alto**: pérdida probable de envíos, de conversión o de medición (perfiles que se quedan sin mensaje, sin exit por objetivo, sin holdout).
- **Medio**: riesgo operativo o deuda de mantenimiento con impacto acotado.
- **Bajo**: higiene sin impacto en el comportamiento.
- **Informativo**: mejora opcional o hipótesis a testear.

Reglas para asignarla:

- Un riesgo sin errores observados en ejecución se marca **latente**: conserva la severidad del daño potencial, pero no se presenta como un fallo activo.
- Un hallazgo que necesita datos para confirmarse se marca **pendiente de verificar**, con la comprobación concreta que lo confirmaría.
- La severidad se justifica en una frase con el mecanismo, no con la categoría del control.

### Veredicto

- Journey en borrador: cualquier crítico implica "No apto para publicar".
- Journey ya desplegado: cualquier crítico implica "Requiere versión correctiva". Si la corrección afecta a algo que no puede cambiar entre versiones (tipo de entrada, audiencia de Audience Qualification, namespace, regla de re-entrada), "Requiere duplicar el journey".

### Formato de cada hallazgo

| Campo | Contenido |
| --- | --- |
| Control | Identificador del checklist (por ejemplo, H2) |
| Severidad | Crítico, Alto, Medio, Bajo o Informativo, y si es latente |
| Confianza | Verificado en la definición, confirmado con datos de ejecución o pendiente de verificar |
| Evidencia | Nodo, rama o campo concreto, con su ruta en la definición |
| Por qué importa | Una frase con el mecanismo y su efecto en los perfiles |
| Corrección | Acción concreta en AJO |
| Efecto en perfiles en curso | Qué pasa con quien ya está dentro al aplicar la corrección |
| Restricción de versión | Si exige nueva versión, duplicar el journey o desplegar junto con otro |
| Fuente | Página de documentación o sección de esta guía |

### Recomendaciones que nunca se dan sin análisis de impacto

- Eliminar una rama, sobre todo la rama por defecto.
- Reducir un timeout antes de que exista ruta alternativa para el error.
- Cambiar la re-entrada, el namespace o el tipo de entrada "en una nueva versión": exige duplicar.
- Cualquier cambio en un journey desplegado sin decir cómo se valida.

### Preguntas abiertas con Adobe

- Qué campos de la definición del journey recibe el MCP de AJO. En la primera auditoría la herramienta dio como "no expuestos" los exit criteria y la merge policy, que sí están en la definición de autoría.
- Si un MCP declarado en el plugin hereda la autenticación del usuario de Coworker.
- Límites de las herramientas de ejecución: ventana máxima de las estadísticas en tiempo real y tiempo máximo de las consultas de traza.

## Fuentes

Documentación de Adobe Journey Optimizer (Experience League):

- [Get started with journeys](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/journey)
- [Best practices for advanced journeys in Journey Optimizer](https://experienceleague.adobe.com/en/perspectives/best-practices-for-advanced-journeys-in-journey-optimizer)
- [Get started with journey activities](https://experienceleague.adobe.com/docs/journey-optimizer/using/orchestrate-journeys/about-journey-building/about-journey-activities.html?lang=en)
- [Set your journey properties](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/journey-properties)
- [Work with journey entry and exit criteria](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/journey-use-cases/entry-exit-criteria-guide)
- [Guardrails and limitations](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/get-started/essentials/guardrails)
- [Configure and track your journey metrics](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/success-metrics)
- [Work with rule sets](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/configuration/frequency-rules)
- [Conflict management and prioritization](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/conflict-prioritization/gs-conflict-prioritization)
- [Use path experimentation](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/about-journey-building/optimize-activity/path-experimentation)
- [Get started with content experiments](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/content-management/content-experiment/get-started-experiment)
- [Choose a validation method](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/create-journey/validate-journey/choose-validation-method)
- [Access and subscribe to system alerts](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/monitor/alerts)
- [Journey properties attributes](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/orchestrate-journeys/building-advanced-conditions-journeys/syntax/journey-properties)

Estudios y benchmarks:

- [Klaviyo, 2026 Abandoned Cart Benchmark Report](https://www.klaviyo.com/blog/abandoned-cart-benchmarks) y [Klaviyo 2025 Benchmark Report](https://resources.klaviyo.com/2025-benchmark-report-us/p/1)
- [McKinsey, El valor de entender bien o mal la personalización se multiplica (Next in Personalization 2021)](https://www.mckinsey.com/featured-insights/destacados/el-valor-de-entender-bien-o-mal-la-personalizacion-se-multiplica/es)
- [Gartner, Personalization Can Triple the Likelihood of Customer Regret (junio de 2025)](https://www.gartner.com/en/newsroom/press-releases/2025-06-03-gartner-survey-reveals-personalization-can-triple-the-likelihood-of-customer-regret-at-key-journey-points)
- [Litmus, Why Your Best Marketing Channel Is Driving Customers Away (junio de 2025)](https://www.litmus.com/blog/why-your-best-marketing-channel-is-driving-customers-away)
- [Braze, Building a retention machine: 6 data-driven insights (2025 Global Customer Engagement Review)](https://www.braze.com/resources/articles/building-a-retention-machine-6-data-driven-insights)
- [Braze, Global Control Group](https://www.braze.com/docs/user_guide/engagement_tools/testing/global_control_group)

Base de conocimiento del proyecto: hallazgos de implantación de AJO desde cero, journeys post visita con reglas de páginas e inactividad, migración desde SFMC, Campaign y Braze, y guía de plugins y skills de Coworker.
