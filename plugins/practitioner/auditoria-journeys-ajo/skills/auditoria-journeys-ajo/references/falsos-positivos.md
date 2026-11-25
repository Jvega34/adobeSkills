# Falsos positivos conocidos

Antes de reportar un hallazgo que encaje con alguno de estos casos, haz la comprobación indicada. Todos salieron de auditorías reales en las que el hallazgo resultó falso o estaba mal diagnosticado.

| Hallazgo tentativo | Por qué suele ser falso | Comprobación |
| --- | --- | --- |
| "Rama duplicada" en una condición | Una rama concreta y la rama por defecto convergen en el mismo nodo | Compara las expresiones de cada `uid`. Si una es `conditionType: opposite`, no es duplicado. Solo es duplicado si dos ramas tienen la misma expresión |
| "Falta rama por defecto" | La rama por defecto no tiene expresión y es fácil pasarla por alto | Busca `conditionType: "opposite"` en `data.conditions` y su conexión `type: "opposite"` |
| "No hay exit criteria" | Los exit criteria están en la raíz de la definición, no en los nodos | Lee `exitCriteria`. Si el campo no te ha llegado, es pendiente, no incumplido |
| "El evento espera sin timeout" | El timeout está dentro del nodo de evento | Lee `data.timeout.delay` y `timeoutPath` |
| Cadencia o franjas horarias sacadas de comentarios | Los comentarios suelen describir versiones anteriores | Calcula los tiempos con los timers y los timeouts reales, y las franjas con las expresiones |
| "Los timers no tienen ruta alternativa" | El flag no tiene efecto en timers | Limita C2 a acciones y condiciones con fuentes externas |
| "La espera de N s protege la lectura del perfil" | La espera puede no estar en el camino que lee el dato, o el dato se escribió mucho antes | Usa el mapa de datos: qué se escribe justo antes de la lectura en cada camino y cuánto tiempo pasa |
| "La prioridad 0 impide el capping de entrada" | El capping depende de la prioridad del namespace de identidad, no de la del journey | Separa la prioridad del journey (`priority`) de la del namespace |
| "Los perfiles en espera son abandonos" | Muchos salen por exit criteria (conversión) mientras esperan | Revisa las salidas por exit criteria antes de interpretar la espera |
| "Crítico" por falta de ruta alternativa | Un perfil detenido sin envío no es un envío erróneo | Aplica la definición de Crítico de la guía y marca como latente si no hay errores observados |
| "La custom action llama a un endpoint del cliente" | Algunas custom actions publican un evento de ingesta en AEP | Mira `paramMappings`: rutas `.../events/0/xdm/...` indican ingesta |
| "Bajar el timeout a 1-5 s" | Sin ruta alternativa, un timeout más corto detiene más perfiles | Recomiéndalo solo después de la ruta alternativa y si el flujo es realmente en tiempo real |
| "Cambiar la re-entrada en la próxima versión" | La regla de re-entrada no puede cambiar entre versiones | La corrección exige duplicar el journey |
| Documento adjunto tomado como especificación | Puede ser de otro journey o de otra fase del proyecto | Confirma con el usuario que el documento corresponde al journey auditado |
| "El journey no existe" | El listado de la herramienta puede ser incompleto o el nombre tener espacios dobles | Busca con coincidencia flexible y pide el ID; nunca sustituyas el objeto |
