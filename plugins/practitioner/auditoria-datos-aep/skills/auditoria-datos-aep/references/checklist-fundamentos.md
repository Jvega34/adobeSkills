# Fundamentos de datos en AEP: checklist de configuración, severidad y puntuación

Esta referencia cubre la **revisión de configuración**. La revisión de salud de datos tiene sus propios controles (Q1 a Q14) en `revision-salud-datos.md` y nunca se suma con esta.

Los controles marcados **\[B\]** son bloqueantes para activar con garantías; los **\[R\]** son recomendados. Los identificadores son estables para citarlos en los hallazgos. La columna "HC" indica el health check nativo de AEP que lo cubre, si existe; cuando lo hay, su resultado es la evidencia preferida.

## Guardrails de referencia

Valores de Experience League revisados el 25/11/2026. Antes de citar uno en un informe, comprueba que sigue vigente en la página de guardrails y cita la página. **Hard**: lo impone el sistema. **Soft**: recomendación de rendimiento; superarlo degrada, no bloquea.

| Límite | Valor | Tipo |
| --- | --- | --- |
| Identidades por grafo | 50 (los perfiles con más quedan fuera de segmentación, exports y lookups) | Hard |
| Datasets habilitados para perfil por clase (Individual Profile, ExperienceEvent) | 20 | Soft |
| Datasets de report suite de Analytics en perfil | 1 | Soft |
| Relaciones multi-entidad por schema | 5 | Soft |
| Profundidad JSON de la identidad primaria | 4 | Soft |
| Tamaño de un registro de perfil / de un ExperienceEvent | 100 KB / 10 KB (se descartan si los superan) | Hard |
| Tamaño almacenado de un perfil | 50 MB | Soft |
| ExperienceEvents por perfil | 5.000 (la segmentación solo usa los 5.000 más recientes) | Soft |
| Cardinalidad de arrays: perfil / ExperienceEvent | 500 / 10 | Soft |
| Merge policies con `_xdm.context.profile` por sandbox | 5 | Ver página |
| Batches de perfil o ExperienceEvent por día y sandbox | 90 | Soft |
| Batches por entidad de dimensión y día | 4 | Soft |
| Audiencias activas por sandbox: total / streaming / edge | 4.000 / 500 / 150 | Soft |
| Tamaño de una audiencia | 30% de los perfiles | Soft |
| Computed attributes por sandbox | 25 | Ver página |
| Sandboxes de desarrollo | 10% de los perfiles de la organización | Licencia |

## Checklist

### S. Modelo XDM (schemas)

| ID | Control | HC |
| --- | --- | --- |
| S1 \[B\] | Cada schema habilitado para perfil tiene una identidad primaria definida (campo marcado o `identityMap` con `primary` en la ingesta) y es única por schema | Primary identity uniqueness, Identity field validation |
| S2 \[B\] | La clase encaja con el dato: atributos en Individual Profile, comportamiento en ExperienceEvent, catálogos en clases de lookup o dimensión | Custom class datasets enabled for Profile |
| S3 \[R\] | Se usan field groups estándar antes que campos propios; no hay field groups custom duplicados entre schemas | Duplicate field groups across schemas |
| S4 \[R\] | Los campos propios están bajo el namespace del tenant, con nombre y descripción que explican su significado | — |
| S5 \[B\] | Tipos correctos: fechas como `date` o `date-time`, números como numéricos, valores cerrados como enum | — |
| S6 \[R\] | Los arrays tienen restricciones y respetan la cardinalidad (500 en perfil, 10 en eventos) | Arrays without constraints |
| S7 \[R\] | La identidad primaria está a profundidad 4 o menos | Primary identity field depth |
| S8 \[R\] | Como mucho 5 relaciones multi-entidad por schema y sin relaciones multinivel | Multi-entity relationship count, Multi-level relationships |
| S9 \[B\] | El schema de perfil incluye el field group de consentimiento cuando hay marketing activo | Consent field group missing |
| S10 \[R\] | Los schemas de eventos web llevan los field groups de auditoría y detección de bots | Missing audit field group, Bot detection field group missing |
| S11 \[R\] | Los campos de identidad no admiten cadenas vacías (longitud mínima 1) | Zero-length strings allowed |
| S12 \[R\] | Solo están habilitados para perfil los schemas que alimentan un caso de uso | — |

### I. Identidad

| ID | Control | HC |
| --- | --- | --- |
| I1 \[B\] | Hay un identificador de persona acordado (por ejemplo, ID de CRM) y está declarado como identidad en todos los schemas que lo transportan | — |
| I2 \[B\] | Los namespaces de persona y de no persona están bien tipados; ningún identificador de no persona se usa como identidad de persona ni en campos de relación | People and non-people identity configuration, Non-person identity on relationship field |
| I3 \[B\] | Las identity graph linking rules están configuradas: el identificador de persona está marcado como namespace único | Identity graph linking rules |
| I4 \[R\] | La prioridad de namespaces está ordenada a propósito (identificador de persona por encima de email, teléfono y dispositivos) | Identity graph linking rules |
| I5 \[B\] | Email, teléfono y otros identificadores que pueden compartirse no son la única unión entre perfiles conocidos | — |
| I6 \[R\] | Los namespaces propios tienen descripción | Custom identity namespace description |
| I7 \[R\] | No hay namespaces creados sin uso | Identity namespace not in use |
| I8 \[B\] | El namespace que usan los journeys, audiencias y destinos coincide con el que llevan los datos | — |
| I9 \[R\] | El data prep descarta o limpia valores de identidad no válidos (vacíos, `null`, `unknown`, valores de prueba) antes de ingerirlos | — |

### P. Perfil y merge policies

| ID | Control | HC |
| --- | --- | --- |
| P1 \[B\] | La merge policy por defecto está elegida a propósito (orden por timestamp o precedencia de datasets) y su lógica está documentada | Merge policies |
| P2 \[R\] | Como mucho 5 merge policies con `_xdm.context.profile` y ninguna sin uso | Merge policies |
| P3 \[B\] | Como mucho 20 datasets habilitados para perfil por clase | Profile dataset count |
| P4 \[R\] | No hay varios datasets con el mismo schema de perfil sin motivo | Multiple datasets per profile schema |
| P5 \[R\] | Hay TTL para perfiles seudónimos | Pseudonymous profile TTL |
| P6 \[B\] | Los datasets de ExperienceEvent tienen TTL y es mayor o igual que el lookback más largo de las audiencias que los usan | Experience Event datasets TTL, Segment lookback vs dataset TTL |
| P7 \[R\] | Computed attributes dentro del límite y todos con uso | Computed attributes limit |
| P8 \[R\] | La merge policy activa en edge es la que espera la personalización en edge (solo si hay casos de uso en edge) | — |

### U. Audiencias

| ID | Control | HC |
| --- | --- | --- |
| U1 \[B\] | El método de evaluación encaja con el uso: streaming para Audience Qualification y exit criteria de AJO, edge para personalización en la misma página, batch para el resto | — |
| U2 \[B\] | Audiencias activas dentro de los límites (4.000 total, 500 streaming, 150 edge) | Audience sandbox limit, Streaming audiences, Edge audiences |
| U3 \[B\] | Cada audiencia usa la merge policy esperada para su caso de uso | — |
| U4 \[R\] | Las audiencias con condiciones de tiempo tienen lookback definido | Segment lookback window required |
| U5 \[R\] | No hay audiencias sin uso (sin journey, destino ni dependencia) ni duplicadas por definición | — |
| U6 \[R\] | Nombre y descripción permiten saber qué contiene y quién la usa | — |
| U7 \[R\] | Las audiencias externas o importadas tienen la caducidad de pertenencia revisada | — |
| U8 \[R\] | Las audiencias que dependen de otras se evalúan en un orden compatible (una streaming no puede depender de una batch y esperar tiempo real) | — |

### N. Ingesta

| ID | Control | HC |
| --- | --- | --- |
| N1 \[B\] | Cada dataflow activo tiene alertas de fallo suscritas por alguien que actúa | Sources |
| N2 \[B\] | El mapeo de data prep cubre la identidad y convierte tipos y formatos (fechas, mayúsculas en emails, prefijos telefónicos) | — |
| N3 \[R\] | La frecuencia de batches a perfil está dentro de 90 por día y sandbox, y la de dimensiones dentro de 4 por entidad y día | Batches per day, Dimension batch frequency |
| N4 \[R\] | La ingesta parcial y su umbral de error están decididos a propósito en cada dataflow | — |
| N5 \[R\] | Los endpoints de streaming usan autenticación | — |
| N6 \[R\] | Los upserts a perfil tienen volúmenes acotados (por debajo de 2 millones de registros por carga) | — |

### G. Gobernanza y consentimiento

| ID | Control | HC |
| --- | --- | --- |
| G1 \[B\] | Los datasets o schemas con datos personales tienen etiquetas de uso (contrato, identidad, sensibles), heredadas o directas | — |
| G2 \[B\] | Hay políticas de uso de datos activas y los destinos tienen marketing actions asignadas, de forma que las políticas se apliquen | — |
| G3 \[B\] | El consentimiento de marketing se almacena en el field group de consentimiento y es el que leen los journeys y destinos | — |
| G4 \[R\] | Si la licencia lo incluye, hay políticas de consentimiento activas | — |
| G5 \[R\] | Hay una política de retención (TTL, Data Hygiene) para datos que no deben guardarse indefinidamente | Stale datasets |
| G6 \[R\] | El acceso a datos sensibles está restringido por roles o etiquetas | — |

## Dimensiones y pesos (configuración)

| Dimensión | Peso | Controles |
| --- | --- | --- |
| Modelo XDM | 15% | S1 a S12 |
| Identidad | 20% | I1 a I9 |
| Perfil y merge policies | 15% | P1 a P8 |
| Audiencias | 15% | U1 a U8 |
| Ingesta | 15% | N1 a N6 |
| Gobernanza y consentimiento | 20% | G1 a G6 |

Si el sandbox no usa algún área (por ejemplo, edge), sus controles son "no aplica".

## Cómo se puntúa

- Cada control es "cumple" (1), "parcial" (0,5), "no cumple" (0), "no aplica" o "pendiente".
- **Pendiente no es incumplido.** Se excluye del cálculo y se dice qué falta.
- La puntuación de una dimensión es su peso por el porcentaje de cumplimiento sobre los controles verificables. El total se expresa sobre los puntos puntuables, no sobre 100.
- Con menos de tres controles verificables, la dimensión se marca "confianza baja".
- Un control que se evalúa sobre muchos objetos (por ejemplo, S1 en 30 schemas) es "parcial" si falla en algunos; el hallazgo lista los objetos que fallan.

## Severidad

- **Crítico**: un control \[B\] incumplido con un mecanismo concreto, que se puede señalar, de mensaje a la persona equivocada (grafos fusionados), uso de datos contra la política o el consentimiento, o perfiles excluidos de la activación (grafos de más de 50 identidades, namespace que no coincide).
- **Alto**: pérdida probable de alcance, de frescura o de medición (audiencias que no se actualizan a tiempo, datasets sin TTL que agotan la licencia, ingesta que falla sin alerta).
- **Medio**: riesgo operativo o deuda de mantenimiento con impacto acotado.
- **Bajo**: higiene sin impacto en el comportamiento.
- **Informativo**: mejora opcional.

Reglas:

- Un riesgo sin efecto observado en los datos se marca **latente**: conserva la severidad del daño potencial, pero no se presenta como un fallo activo.
- Un hallazgo que necesita datos para confirmarse se marca **pendiente de verificar**, con la comprobación concreta.
- La severidad se justifica en una frase con el mecanismo, no con la categoría del control.

## Veredicto de configuración

- **Lista para activar**: sin críticos ni altos.
- **Activar con riesgos**: sin críticos, con altos; se listan los casos de uso afectados.
- **Requiere corrección antes de activar**: algún crítico.
- **Requiere rediseño**: algún crítico cuya corrección exige un schema nuevo, cambiar la identidad de datos ya ingeridos o reingestar (ver la escala de esfuerzo L).

## Formato de cada hallazgo

| Campo | Contenido |
| --- | --- |
| Control | Identificador (por ejemplo, I3) |
| Severidad | Crítico, Alto, Medio, Bajo o Informativo, y si es latente |
| Confianza | Verificado en la configuración, confirmado con datos o pendiente de verificar |
| Evidencia | Objeto con nombre e ID, campo o ruta; o resultado del health check |
| Por qué importa | Una frase con el mecanismo y su efecto en perfiles o activación |
| Corrección | Acción concreta en AEP |
| Efecto en datos existentes | Qué pasa con los datos y perfiles ya ingeridos al aplicar la corrección |
| Restricción | Si exige schema nuevo, reingesta, recalcular audiencias o coordinarse con otro equipo |
| Impacto en activación | Journeys, audiencias o destinos afectados, si se pueden ver |
| Fuente | Página de documentación o sección de esta guía |

## Recomendaciones que nunca se dan sin análisis de impacto

- Quitar o renombrar campos de un schema habilitado para perfil: solo admite cambios no destructivos; la corrección suele exigir un schema nuevo y migrar datasets.
- Deshabilitar un dataset para perfil o borrarlo: retira sus datos de los perfiles y de las audiencias que los usan.
- Cambiar la identidad primaria, el namespace de un campo, los namespaces únicos o su prioridad: afecta a cómo se unen los datos nuevos y puede no corregir los grafos ya formados sin reingesta. Revisa la guía de implementación de linking rules y pruébalo con la simulación de grafos.
- Cambiar la merge policy por defecto: cambia a la vez todas las audiencias y destinos que la usan.
- Bajar un TTL o lanzar Data Hygiene: borra datos de forma irreversible.
- Pasar una audiencia de batch a streaming: cuenta contra el límite de 500 y cambia cuándo entran los perfiles en los journeys.
- Cualquier cambio sin decir cómo se valida (simulación de grafos, sandbox de desarrollo, previsualización de audiencia).

## Fuentes

- [Guardrails de Real-Time Customer Profile y segmentación](https://experienceleague.adobe.com/en/docs/experience-platform/profile/guardrails)
- [Guardrails de ingesta](https://experienceleague.adobe.com/en/docs/experience-platform/ingestion/guardrails)
- [Health checks de AEP](https://experienceleague.adobe.com/en/docs/experience-platform/run-and-operate/health-checks/overview) y [schemas e identidades](https://experienceleague.adobe.com/en/docs/experience-platform/run-and-operate/health-checks/schemas-and-identities)
- [Identity graph linking rules](https://experienceleague.adobe.com/en/docs/experience-platform/identity/features/identity-graph-linking-rules/overview), [prioridad de namespaces](https://experienceleague.adobe.com/en/docs/experience-platform/identity/features/identity-graph-linking-rules/namespace-priority) y [guía de implementación](https://experienceleague.adobe.com/en/docs/experience-platform/identity/features/identity-graph-linking-rules/implementation-guide)
- [Buenas prácticas de licencia de Data Management](https://experienceleague.adobe.com/en/docs/experience-platform/landing/license/data-management-best-practices)
- [Audiencias en AJO](https://experienceleague.adobe.com/en/docs/journey-optimizer/using/audiences-profiles-identities/audiences/about-audiences)
