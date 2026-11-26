# Cómo leer los objetos de AEP

Qué leer de cada objeto, dónde suele estar y qué controles alimenta. Usa la herramienta de Coworker que lo exponga; si no lo expone, anota el objeto como no disponible en el inventario y deja sus controles pendientes. Los nombres de API son orientativos para reconocer el dato, no una instrucción de llamarlas.

## Inventario mínimo

| Objeto | Campos mínimos | Dónde suele estar | Controles |
| --- | --- | --- | --- |
| Schemas | Nombre, ID, clase, field groups, campos de identidad (namespace, primaria), relaciones, si está habilitado para perfil | Schemas (UI), Schema Registry API | S1 a S12, I1, I2 |
| Namespaces | Código, tipo (persona, dispositivo, no persona), propio o estándar, descripción | Identities > Namespaces, Identity Service API | I1, I2, I6, I7 |
| Ajustes del identity graph | Namespaces únicos, prioridad de namespaces | Identities > Settings | I3, I4 |
| Datasets | Nombre, ID, schema, habilitado para perfil, TTL, etiquetas, último batch, número de registros | Datasets (UI), Catalog API | P3, P4, P6, G1, G5 |
| Merge policies | Nombre, ID, por defecto o no, método (timestamp o precedencia de datasets), orden de datasets, identity stitching, activa en edge | Profiles > Merge policies | P1, P2, P8, U3 |
| TTL de perfiles seudónimos | Activado, días, namespaces afectados | Configuración de perfil (solicitud a Adobe en muchas organizaciones) | P5 |
| Computed attributes | Número, estado, uso | Profiles > Computed attributes | P7 |
| Audiencias | Nombre, ID, origen, método de evaluación, merge policy, lookback, dependencias, estado, uso en journeys y destinos | Audiences (Audience Portal), Segmentation API | U1 a U8 |
| Dataflows | Fuente, dataset destino, frecuencia, ingesta parcial y umbral, alertas, mapeo de data prep | Sources > Dataflows, Flow Service API | N1 a N6 |
| Etiquetas y políticas | Etiquetas por schema, dataset y campo; políticas activas; marketing actions de los destinos | Privacy > Policies, Destinations | G1, G2, G4, G6 |
| Health checks | Resultado por check, objeto señalado, fecha | Run and operate > Health checks | Ver columna HC del checklist |

## Mapa de identidad

Una fila por namespace:

| Namespace | Tipo | Único | Prioridad | Schemas y campos donde aparece | Primaria en | Lo consume |
| --- | --- | --- | --- | --- | --- | --- |

Qué mirar:

- El identificador de persona aparece en todos los schemas de perfil que lo transportan y está marcado como único.
- Email y teléfono no son únicos por defecto: dos personas pueden compartirlos. Si son la única unión entre perfiles conocidos, hay riesgo de grafos fusionados (I5).
- Un namespace de dispositivo (ECID, IDFA, GAID) nunca es identidad de persona.
- El namespace de los journeys (propiedad `namespace` del journey), de los exit criteria y de los destinos está en la columna "Lo consume". Si ese namespace no aparece en los datos, es I8.

## Mapa de perfil

Una fila por dataset habilitado para perfil:

| Dataset | ID | Schema | Clase | Identidad primaria | TTL | Último dato | Merge policies que lo incluyen |
| --- | --- | --- | --- | --- | --- | --- | --- |

Qué mirar:

- Recuento por clase frente al límite de 20 (P3).
- Datasets con el mismo schema (P4) y datasets sin datos en 90 días (Q3, salud).
- En merge policies por precedencia de datasets, un dataset nuevo que no está en el orden queda al final; comprueba que el orden refleja la fuente de verdad acordada.

## Audiencias

Para cada audiencia en alcance:

- Método de evaluación y si está en uso: busca los journeys que la usan en Read Audience, Audience Qualification o exit criteria, y los destinos donde se activa. Audience Qualification y exit criteria necesitan audiencias streaming (U1).
- Merge policy de la audiencia frente a la merge policy por defecto (U3).
- Condiciones con tiempo: lookback definido (U4) y TTL de los datasets de eventos que lee (P6).
- Dependencias: audiencias que se basan en otras audiencias y su método (U8).

## Campos que a menudo no llegan

Algunas herramientas no devuelven todos los campos que ve la interfaz. Los que más faltan: ajustes de linking rules, TTL de perfiles seudónimos, TTL de datasets, alertas suscritas, umbral de ingesta parcial y etiquetas heredadas. Si no llegan, son pendientes; nunca se deducen de su ausencia.
