# Falsos positivos conocidos

Antes de reportar un hallazgo que encaje con alguno de estos casos, haz la comprobación indicada. Las lecturas engañosas de datos observados están en `revision-salud-datos.md`.

| Hallazgo tentativo | Por qué suele ser falso | Comprobación |
| --- | --- | --- |
| "El schema no tiene identidad primaria" | La identidad puede llegar en `identityMap` en la ingesta (Web SDK con ECID) o el schema no está habilitado para perfil | Mira si está habilitado para perfil y cómo llega la identidad en un batch reciente |
| "El schema no tiene identidad" en una clase de lookup o dimensión | Esas clases se relacionan por un campo de relación, no por identidad de persona | Confirma la clase antes de aplicar S1 |
| "Los datasets no tienen etiquetas" | Las etiquetas pueden estar en el schema y heredarse | Revisa las etiquetas del schema y las heredadas del dataset |
| "Audiencia batch usada en un journey" | En Read Audience es correcto; solo es problema en Audience Qualification y exit criteria | Mira en qué actividad del journey se usa |
| "Namespace sin uso" | Puede usarlo un destino, una fuente prevista o un journey | Busca en destinos y journeys antes de proponer borrarlo |
| "Dataset habilitado para perfil sin uso" | Puede alimentar computed attributes, lookups o audiencias indirectas | Busca dependencias antes de proponer deshabilitarlo |
| "La merge policy por timestamp es incorrecta" | Es la opción correcta cuando todas las fuentes son igual de fiables y el dato más reciente gana | Pregunta qué fuente es la verdad para cada atributo antes de juzgarla |
| "Email marcado como no único = riesgo" | Lo normal es que el email no sea único; el riesgo es que sea la única unión entre perfiles conocidos | Comprueba I3 (identificador de persona único) e I5 |
| "Hay más de 5 merge policies" | El límite cuenta solo las que usan `_xdm.context.profile` | Separa las merge policies por clase |
| "Faltan field groups de bots o auditoría" | Solo aplica a schemas de eventos web | Confirma el tipo de datos del schema |
| "El TTL es demasiado corto" | Puede estar alineado con la política de retención legal | Pregunta la política antes de proponer alargarlo |
| "Valores repetidos de email = grafos fusionados" | Si el email no es identidad, no une perfiles | Comprueba si ese campo está declarado como identidad |
| "La configuración de linking rules no existe" | La herramienta puede no exponerla | Si no llega, es pendiente; pide una captura de Identities > Settings |
| Documento de diseño tomado como configuración | Puede describir otra fase o un sandbox distinto | Confirma con el usuario que corresponde al sandbox auditado |
| "El objeto no existe" | El listado de la herramienta puede estar paginado o filtrado | Busca por ID y con coincidencia flexible; nunca sustituyas el objeto |
