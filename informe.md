# Informe — Clasificación de variables y población vs. muestra

**Para:** Gerencia de RetailAndina S.A. · **De:** Equipo de Business Intelligence
**Fecha:** 23/10/2026 · **Asunto:** Base de fidelización: tipos de variables y estimación por muestreo

---

## Resumen ejecutivo

Clasifiqué formalmente las variables del conjunto de datos de clientes (clientes.csv), evaluando su naturaleza estadística más allá del tipo de dato (dtype) que asigna por defecto el sistema. Para ello, analicé criterios lógicos como el sentido de las operaciones matemáticas o el orden inherente de las categorías, estructurando una base sólida para futuros análisis descriptivos y de toma de decisiones gerenciales basadas en muestras.

## Parte A — Clasificación de las variables

*Completa la tabla usando el **criterio** (¿tiene sentido la mitad del valor? ¿hay orden entre las categorías?), no el ejemplo. En «verificación» anota qué mostró el código: dtype, número de valores únicos y, si declaraste el orden, cómo lo comprobaste.*

| Variable | Tipo | Justificación (criterio) | Verificación en código |
|---|---|---|---|
| `cliente_id` | Cualitativa nominal | Funciona puramente como una etiqueta de identificación única; no aplican operaciones aritméticas. | `object` (Python) / `chr` (R), con 12 valores únicos (`C001` a `C012`). |
| `edad` | Cuantitativa discreta | Representa una cantidad numérica exacta de años cumplidos; es una magnitud contable de tiempo. | `int64` (Python) / `int` (R), con 12 valores únicos numéricos. |
| `numero_hijos` | Cuantitativa discreta | Cuenta elementos enteros y discretos; no existen decimales como "1.5 hijos" en el contexto. | `int64` (Python) / `int` (R), con 5 valores únicos de conteo (0, 1, 2, 3, 4). |
| `genero` | Categórica nominal | Agrupa a los registros en categorías cualitativas excluyentes sin orden inherente. | `object` (Python) / `chr` (R), con 3 valores únicos ("Femenino", "Masculino", "Otro"). |
| `nivel_educacion` | Categórica ordinal | Posee una jerarquía de menor a mayor grado. | `object` / `chr`, orden declarado explícitamente con `CategoricalDtype` / `factor(ordered=TRUE)`. |
| `ciudad` | Categórica nominal | Agrupa registros por ubicación geográfica sin importar jerarquías. | `object` (Python) / `chr` (R), con 3 valores únicos ("Quito", "Guayaquil", "Cuenca"). |
| `ingresos_mensuales` | Cuantitativa continua | Admite infinitos valores intermedios y operaciones de magnitud. | `int64` (Python) / `int` (R), con 12 valores únicos. |
| `satisfaccion` | Categórica ordinal | Mide una escala valorativa del 1 al 5, los números actúan como etiquetas ordenadas donde "4 no es el doble de 2". | `int64` / `int`, escala del 1 al 5 convertida a categoría ordenada para preservar su jerarquía lógica. |

**P1.** ¿Qué te dijo el dtype de `satisfaccion` y por qué no basta para clasificarla? ¿Qué pasa si una ordinal queda como texto sin declarar el orden?

El dtype me indicó que satisfaccion era un número entero, pero esto no basta para clasificarla porque el sistema solo ve el almacenamiento técnico y no que los números (1 al 5) actúan como etiquetas ordenadas donde el doble no tiene sentido estadístico; además, si dejo una variable ordinal como texto sin declarar su orden, el programa la organizaría alfabéticamente (por ejemplo, colocando "Posgrado" antes que "Primaria"), destruyendo por completo su jerarquía lógica natural.

## Parte B — Población vs. muestra

### La población y su parámetro

| Concepto | Valor |
|---|---|
| Población ($N$) | 400 clientes (N = 400) |
| Variable de interés | `ingresos_mensuales` |
| Parámetro poblacional (media) | 1,330.72 USD |

*¿Por qué se muestrea si aquí el parámetro existe y se calculó?*

Aunque en este ejercicio la población completa es de 400 clientes y pude calcular el parámetro exacto, en la práctica real se muestrea porque analizar a toda una población requiere tiempo y recursos excesivos; el muestreo me permite estimar esos mismos parámetros evaluando grupos de n = 20 con un margen de error controlable.

### Los tres diseños

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---|---|---|---|
| Conveniencia (`head`) | 20 | $1,348.50 | `{'Quito': 20}` | +17.78 |
| Aleatorio simple (`sample`) | 20 | $1,336.00 | `{'Cuenca': 8, 'Quito': 8, 'Guayaquil': 4}` | +5.28 |
| Estratificado (`groupby` + 5 %) | 20 | $1,287.50 | `{'Quito': 10, 'Guayaquil': 6, 'Cuenca': 4}` | -43.22 |

*Anota también las medias de 5 sorteos distintos del aleatorio simple (semillas 1 a 5):*

| Semilla | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Media | $1,336.00 | $1405.00| $1235.00  | $1491.50 | $1288.00|

#### P2. ¿Cuánto vale el parámetro poblacional y cuánto estimó cada diseño? ¿Cuál quedó más cerca y por qué eso no alcanza para declarar un ganador?
El parámetro poblacional con N =400 es **1,330.72 USD**. En esta ejecución con la semilla 1, el diseño **Aleatorio Simple** quedó más cerca con **1,336.00 USD** (diferencia de +5.28 USD), frente a el espacio de Conveniencia (1,348.50 USD) y con Estratificado (1,287.50 USD). Pero, un solo número no basta para declarar un ganador porque esa cercanía es producto del azar de una semilla específica; necesito evaluar la distribución y variabilidad a través de múltiples muestras para determinar cuál diseño es consistentemente mejor.

#### P3. La muestra por conveniencia también tiene 20 filas. ¿Qué defecto tiene y cómo lo delata en la composición por ciudad? ¿Se corrige tomando más filas?
El defecto es el **sesgo de selección**. Su composición por ciudad lo delata inmediatamente al mostrar que el 100 % de los registros pertenecen a Quito (`{'Quito': 20}`), omitiendo por completo a Guayaquil y Cuenca. Este problema no se corrige tomando más filas mediante un simple `.head()`, ya que solo seguiría capturando datos bajo el mismo orden original de la base, perpetuando la falta de representación territorial.

#### P4. El sorteo aleatorio dio un número distinto en cada semilla. ¿Qué implica esa variabilidad para una empresa que va a decidir con un solo estudio?
Implica un riesgo alto por error de muestreo. Si RetailAndina S.A. toma una decisión gerencial basada en una única muestra (una sola semilla), asume el riesgo de operar sobre un valor que se desvió por pura suerte del parámetro real. Esto demuestra que las decisiones estratégicas no deben basarse en un estudio aislado sin medir la variabilidad o intervalos de confianza.

#### P5. ¿Cuándo usarías cada diseño en la práctica? Relaciona cada uno con un uso real (piloto, subgrupos que deben aparecer, conclusión sobre toda la población).
* **Conveniencia:** Lo usaría exclusivamente para un estudio piloto o pruebas operativas rápidas y de bajo costo donde no se busquen inferencias poblacionales.
* **Estratificado:** Lo aplicaría cuando existan subgrupos que deben aparecer obligatoriamente en proporciones reales (como las ciudades de Quito, Guayaquil y Cuenca) para garantizar comparaciones válidas.
* **Aleatorio simple:** Lo utilizaría para obtener una conclusión sobre toda la población cuando esta sea homogénea y cada elemento deba tener exactamente la misma probabilidad de selección.


## Conclusiones

1. Al finalizar el trabajao de laboratorio, aprendí que elegir muestras por simple conveniencia (`head`) destruye la representatividad de los datos, ya que un criterio no probabilístico puede excluir por completo a regiones enteras como ocurrió al dejar fuera a Guayaquil y Cuenca, demostrando que un mal diseño arruina cualquier análisis posterior.
2. Ademàs, aprendí que confiar en una sola muestra o en una única semilla (`random_state`) es un error metodológico grave, ya que la variabilidad aleatoria hace que cada sorteo fluctúe; por eso, evaluar múltiples escenarios es indispensable para saber qué tan estable es el resultado.
3. Finalmente a la gerencia le recomendaría que, antes de aceptar o solicitar un estudio basado en muestras, exija total transparencia sobre cómo se recolectó la información y si se verificó la estabilidad de las estimaciones, evitando así tomar decisiones estratégicas de alto impacto basadas en datos frágiles o sesgados.
