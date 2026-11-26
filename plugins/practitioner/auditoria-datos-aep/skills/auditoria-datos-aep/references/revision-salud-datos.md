# Revisión de salud de datos

Esta referencia cubre la segunda revisión: qué está pasando con los datos. Se hace después de la revisión de configuración y se puntúa aparte. Los umbrales son orientativos: si el cliente tiene los suyos, usa los del cliente y dilo en el informe.

## Fuentes, en orden de preferencia

1. **Health checks nativos** (Run and operate > Health checks): ya calculados por AEP; cita el check y su fecha.
2. **Dashboards** de perfiles, identidades, audiencias, monitorización de dataflows y uso de licencia.
3. **Query Service**, solo `SELECT`, con las plantillas de abajo, para lo que no cubren los anteriores. Incluye en el informe la consulta exacta que sostiene cada hallazgo.

| Métrica | Fuente habitual | Controles |
| --- | --- | --- |
| Ejecuciones de dataflows y batches (éxito, fallo, registros fallidos) | Monitorización de dataflows, Catalog API | Q1, Q2 |
| Último dato por dataset | Datasets, health check Stale datasets, SQL | Q3 |
| Tamaño de los grafos | Dashboard de identidades, visor del identity graph | Q4 |
| Valores de identidad compartidos | SQL sobre el dataset de perfil | Q5 |
| Perfiles de una sola identidad o seudónimos | Dashboard de perfiles | Q6 |
| Completitud, formato y consentimiento | SQL sobre los datasets de perfil | Q7, Q9, Q10 |
| Eventos sin perfil conocido | SQL entre datasets de eventos y de perfil | Q8 |
| Tamaño y evolución de audiencias, tiempo de evaluación batch | Audience Portal, dashboard de audiencias, health checks de segmentación | Q11, Q12 |
| Uso de licencia | Dashboard de uso de licencia | Q13 |
| Perfiles por encima de guardrails | Health checks, avisos de perfil | Q14 |

Anota siempre la ventana (fechas y zona horaria), el sandbox y cualquier límite de la fuente (muestreo, fecha de actualización del dashboard).

## Umbrales mínimos de volumen

Por debajo, la métrica se marca **datos insuficientes**: se muestra el valor, pero no se puntúa ni genera hallazgo.

| Lectura | Mínimo orientativo |
| --- | --- |
| Tasas de un dataflow | 10 ejecuciones en la ventana |
| Tasas de un dataset (completitud, formato, consentimiento) | 1.000 registros |
| Evolución de una audiencia | 7 evaluaciones |
| Objeto recién creado | 7 días desde su creación |

## Controles de salud

| ID | Control | Sano | Con riesgos | Degradado |
| --- | --- | --- | --- | --- |
| Q1 | Ejecuciones correctas por dataflow (30 días) | ≥ 98% | 95–98% | < 95% |
| Q2 | Registros fallidos o descartados por dataflow, con su motivo principal | < 1% | 1–5% | > 5% |
| Q3 | Frescura: último dato frente a la cadencia esperada; datasets en perfil sin datos en 90 días | Al día | Retraso < 2 ciclos | Retraso ≥ 2 ciclos o dataset en perfil sin datos |
| Q4 | Perfiles con grafos de más de 50 identidades, o entre 40 y 50 | 0 | Solo entre 40 y 50 | Alguno de más de 50 |
| Q5 | Valores de email, teléfono u otro identificador compartible asociados a varios identificadores de persona | Casos aislados y explicables | Algún valor en más de 10 personas | Valores basura (`null`, `test@…`) o en más de 100 personas |
| Q6 | Proporción de perfiles seudónimos o de una sola identidad, leída junto al TTL de seudónimos | Coherente con el negocio y con TTL | Alta sin TTL | Creciendo sin TTL y acercándose a la licencia |
| Q7 | Completitud de los campos que usa cada caso de uso (email, teléfono, país, idioma, consentimiento) | ≥ 95% | 80–95% | < 80% |
| Q8 | Eventos cuyo identificador de persona no existe en el dataset de perfil | < 5% | 5–20% | > 20% |
| Q9 | Formato válido: emails, teléfonos, fechas (sin fechas futuras ni de 1970) | ≥ 98% | 95–98% | < 95% |
| Q10 | Cobertura de consentimiento: perfiles con valor explícito (no vacío ni desconocido) | ≥ 95% | 80–95% | < 80% |
| Q11 | Audiencias en uso con 0 perfiles, o por encima del 30% de los perfiles | Ninguna | Alguna explicada | Alguna en uso sin explicación |
| Q12 | Evolución de audiencias en uso (variación diaria) y tiempo de evaluación batch | Estable; evaluación dentro de la ventana habitual | Saltos > 30% explicados | Saltos > 30% sin explicar o evaluación que no termina a tiempo para los journeys |
| Q13 | Uso de licencia frente a lo autorizado (perfiles, richness, volumen) | < 80% | 80–100% | > 100% |
| Q14 | Perfiles por encima de guardrails de tamaño o de 5.000 eventos | 0 | Casos aislados | Volumen relevante |

Cada resultado se ubica en un objeto concreto: dataflow, dataset, campo, namespace o audiencia, con su ID.

## Dimensiones y pesos (salud)

| Dimensión | Peso | Controles |
| --- | --- | --- |
| Ingesta y frescura | 25% | Q1, Q2, Q3 |
| Identidad observada | 25% | Q4, Q5, Q6 |
| Calidad de campos | 25% | Q7, Q8, Q9, Q10 |
| Audiencias | 15% | Q11, Q12 |
| Capacidad y licencia | 10% | Q13, Q14 |

Sano = 1, con riesgos = 0,5, degradado = 0. "Confianza baja" con menos de dos controles evaluables en la dimensión.

Veredicto de salud: **Sano** (sin degradados), **Con riesgos** (algún degradado en dimensiones de peso 15% o menos), **Degradado** (algún degradado en ingesta, identidad o calidad), **Sin datos suficientes**.

Severidad: un degradado en Q4, Q5 o Q10 que afecta a perfiles activados es Crítico; un degradado en el resto es Alto; con riesgos es Medio; el resto, Bajo o Informativo.

## Plantillas SQL (solo lectura)

Sustituye los marcadores `<…>` por los nombres reales de tabla y campo (el nombre de tabla de un dataset está en sus detalles). Todas devuelven recuentos, nunca registros individuales.

**Q3. Frescura de un dataset de eventos**

```sql
SELECT MAX(timestamp) AS ultimo_evento,
       SUM(CASE WHEN timestamp >= current_date - INTERVAL 1 DAY THEN 1 ELSE 0 END) AS eventos_ultimas_24h,
       COUNT(*) AS eventos_30d
FROM <tabla_eventos>
WHERE timestamp >= current_date - INTERVAL 30 DAY;
```

**Q5. Identificadores compartidos entre personas**

```sql
SELECT COUNT(*) AS valores_compartidos,
       MAX(n) AS max_personas_por_valor,
       SUM(CASE WHEN n > 10 THEN 1 ELSE 0 END) AS valores_en_mas_de_10
FROM (
  SELECT LOWER(TRIM(<campo_email>)) AS v, COUNT(DISTINCT <campo_id_persona>) AS n
  FROM <tabla_perfil>
  WHERE <campo_email> IS NOT NULL AND TRIM(<campo_email>) <> ''
  GROUP BY LOWER(TRIM(<campo_email>))
  HAVING COUNT(DISTINCT <campo_id_persona>) > 1
) t;
```

Para ver los peores casos sin exponer datos personales:

```sql
SELECT CONCAT(SUBSTR(v, 1, 2), '***') AS valor_enmascarado, n AS personas
FROM (
  SELECT LOWER(TRIM(<campo_email>)) AS v, COUNT(DISTINCT <campo_id_persona>) AS n
  FROM <tabla_perfil>
  WHERE <campo_email> IS NOT NULL
  GROUP BY LOWER(TRIM(<campo_email>))
) t
ORDER BY n DESC
LIMIT 10;
```

**Q7 y Q9. Completitud y formato**

```sql
SELECT COUNT(*) AS total,
       ROUND(100.0 * SUM(CASE WHEN <campo_email> IS NULL OR TRIM(<campo_email>) = '' THEN 1 ELSE 0 END) / COUNT(*), 2) AS pct_email_vacio,
       ROUND(100.0 * SUM(CASE WHEN <campo_email> IS NOT NULL AND NOT <campo_email> RLIKE '^[^@[:space:]]+@[^@[:space:]]+\\.[^@[:space:]]+$' THEN 1 ELSE 0 END) / COUNT(*), 2) AS pct_email_invalido,
       ROUND(100.0 * SUM(CASE WHEN <campo_pais> IS NULL THEN 1 ELSE 0 END) / COUNT(*), 2) AS pct_pais_vacio,
       ROUND(100.0 * SUM(CASE WHEN <campo_fecha> > current_date OR <campo_fecha> < DATE '1900-01-01' THEN 1 ELSE 0 END) / COUNT(*), 2) AS pct_fecha_imposible
FROM <tabla_perfil>;
```

**Q8. Eventos sin perfil conocido**

```sql
SELECT COUNT(*) AS eventos,
       ROUND(100.0 * SUM(CASE WHEN p.id IS NULL THEN 1 ELSE 0 END) / COUNT(*), 2) AS pct_sin_perfil
FROM <tabla_eventos> e
LEFT JOIN (SELECT DISTINCT <campo_id_persona> AS id FROM <tabla_perfil>) p
  ON e.<campo_id_persona_evento> = p.id
WHERE e.timestamp >= current_date - INTERVAL 30 DAY
  AND e.<campo_id_persona_evento> IS NOT NULL;
```

**Q10. Cobertura de consentimiento**

```sql
SELECT COALESCE(<campo_consentimiento>, 'vacío') AS valor, COUNT(*) AS registros
FROM <tabla_perfil>
GROUP BY COALESCE(<campo_consentimiento>, 'vacío')
ORDER BY registros DESC
LIMIT 20;
```

Para el field group estándar de consentimiento, el campo suele ser `consents.marketing.<canal>.val` (valores como `y`, `n`, `u`).

## Lecturas engañosas

| Lectura tentativa | Por qué suele ser errónea | Comprobación |
| --- | --- | --- |
| `COUNT(*)` de un dataset = número de perfiles | El data lake guarda todas las versiones de cada registro (cargas completas repetidas, upserts); el perfil las combina por identidad | Cuenta identificadores distintos, o usa el dashboard de perfiles, y di qué cuentas |
| Comparar el dashboard de perfiles con SQL | El dashboard se actualiza periódicamente y algunas cifras son estimaciones | Anota la fecha de actualización del dashboard y no mezcles fuentes en la misma tasa |
| Usar el snapshot de perfiles para encontrar grafos de más de 50 identidades | Los perfiles por encima del límite se excluyen de exports y segmentación, así que pueden no aparecer | Usa el dashboard o el visor de identidades, o el health check |
| Un batch fallido = datos perdidos | Puede haberse reintentado con éxito | Mira la ejecución siguiente del mismo dataflow |
| Muchos perfiles seudónimos = problema | Es normal con tráfico web anónimo | Léelo junto al TTL de seudónimos (P5) y la licencia (Q13) |
| Audiencia a 0 perfiles = rota | Las audiencias batch recién creadas esperan a la siguiente evaluación diaria | Comprueba su fecha de creación y de última evaluación |
| Audiencia por encima del 30% = error | Audiencias amplias como "todos los clientes" pueden ser intencionadas | Pregunta el uso; si es intencionada, informativo |
| Conclusiones en un sandbox de desarrollo | Volumen limitado al 10% de los perfiles y datos de prueba | Dilo en los límites y no extrapoles a producción |
| Campo vacío = mala calidad | Puede ser opcional o aplicar solo a un tipo de registro | Mide la completitud sobre los registros donde el caso de uso lo necesita |
