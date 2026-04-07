# Documentación del código — `01-model-coding.ipynb`

---

## Celda 1 — Importación de librerías y rutas de archivos

Se importan las librerías necesarias para manipulación de datos (`pandas`, `numpy`), visualización (`matplotlib`, `seaborn`), cálculo de fechas (`datetime`, `dateutil`) y análisis de supervivencia (`lifelines`).

Se definen las rutas absolutas a los dos archivos de entrada:
- `Base de datos.xlsx` — fuente principal con créditos, abonos y clientes
- `Cierre LDC.xlsx` — cierres de líneas de crédito

---

## Celda 2 — Carga y preparación de datos

Se cargan las 4 tablas necesarias del archivo Excel: **Créditos**, **Abonos**, **Clientes** y **Cierres**.

El análisis se restringe a **personas morales** únicamente, ya que el producto PAY está dirigido a ese segmento.

De los créditos de personas morales se extraen dos subconjuntos:

- **Líneas de Crédito (LDC):** los contratos marco que autorizan el crédito. Se les agrega RFC, fecha de originación y se determina si están activas o vencidas según su fecha de vencimiento.
- **Ministraciones:** los desembolsos individuales que se hacen contra cada línea. Cada ministración hereda la fecha de originación de su LDC para poder agruparla en la cosecha correcta.

Finalmente, se filtra al **producto PAY** específicamente. A partir de aquí todo el análisis es sobre PAY.

---

## Celda 3 — Construcción de la tabla de cuotas, DPD y capital vencido

Este es el paso más importante del análisis. El objetivo es construir una tabla donde cada fila representa una cuota específica de cada ministración PAY, con su estado de pago calculado.

**Unificación de pagos parciales:** una misma cuota puede haberse pagado en varias transacciones. Se consolidan en un solo registro por cuota, sumando el capital y los moratorios pagados.

**Expansión de cuotas esperadas:** se genera una fila por cada cuota que debería existir según el plazo pactado en cada ministración, independientemente de si fue pagada o no. Esto permite identificar cuotas impagas como registros explícitos.

**Cruce con pagos reales:** las cuotas esperadas se cruzan con los pagos registrados. Si una cuota no tiene pago, se trata como impaga activa (su fecha de pago se marca como hoy).

**Filtro de cuotas vencidas:** solo se analizan cuotas cuya fecha de vencimiento ya pasó. Las cuotas futuras se excluyen porque aún no son exigibles y no deben afectar el indicador de morosidad.

**MOB (Months on Books):** se calcula cuántos meses han pasado desde la originación de la cosecha hasta el vencimiento de la cuota. Es el eje horizontal de la matriz de cosechas y refleja la madurez de la cartera.

**DPD (Days Past Due):** se calculan los días de atraso de cada cuota. La lógica distingue entre créditos liquidados y activos, y usa la presencia de moratorios como indicador de que hubo atraso real (no solo un retraso administrativo).

**Capital vencido (`mvenc`):** si una cuota tiene DPD mayor a cero, su capital se considera vencido. De lo contrario es cero. Este es el numerador principal del indicador de morosidad.

**Período de análisis:** el usuario define un rango de fechas de originación. Solo se incluyen las cosechas originadas dentro de ese rango.

**Matriz de cosechas:** se construye la tabla bidimensional donde las filas son los meses de originación, las columnas son los MOB, y los valores son el capital vencido de cada combinación, normalizado por el monto desembolsado total de esa cosecha. El resultado es el **porcentaje de capital vencido por cosecha y madurez**.

---

## Celda 4 — Visualización: heatmap de cosechas

Se genera un heatmap que muestra la matriz de cosechas. El color va de blanco (sin morosidad) a rojo (alta morosidad), lo que permite identificar visualmente qué cosechas tienen mayor deterioro y en qué momento de su vida comenzaron a presentar atrasos. El tamaño de la figura se ajusta dinámicamente al número de cosechas y meses analizados.

---

# Fase 1 — Exploración de Datos (EDA)

---

## Celda 5 — Carga de tablas adicionales

Se cargan todas las fuentes de datos que se usarán como variables predictoras en el modelo, además de las ya cargadas en la sección de cosechas:

| Variable | Hoja | Contenido |
|---|---|---|
| `data_balance` | `balance_sheet` | Estados de situación financiera anuales |
| `data_income` | `income_statement` | Estados de resultados anuales |
| `data_cashflow` | `cash_flow` | Flujo de efectivo por período |
| `data_invoices` | `annual_invoices_comparisons` | Comparaciones anuales de facturación SAT |
| `data_employees` | `employees` | Número de empleados por período |
| `data_entity` | `superia_entity` | Catálogo de entidades con RFC y `profile_entity_id` |
| `data_score` | `csv_score` | Score PyME de Buró de Crédito |
| `data_vc_pyme` | `csv_vc_pyme` | Variables califica del reporte de Buró |
| `data_registro` | `xls_registro` | Registro de evaluación crediticia con datos financieros y Buró |

---

## Celda 6 — Estructura de cada tabla

Para cada tabla se imprime el número de filas, número de columnas y los nombres de todas las columnas.

**Cómo interpretar:** permite verificar que las tablas se cargaron correctamente y tener una vista rápida de qué información contiene cada una antes de entrar al análisis. Si alguna tabla tiene 0 filas o columnas inesperadas, hay un problema en la carga.

---

## Celda 7 — Valores nulos por tabla

Para cada tabla se calcula el número y porcentaje de valores nulos por columna. Solo se muestran las columnas que tienen al menos un nulo.

**Cómo interpretar:** un porcentaje alto de nulos en una variable candidata al modelo indica que puede tener cobertura limitada o que requiere imputación. Variables con más del 50% de nulos deben evaluarse con cuidado antes de incluirlas. Si una tabla no tiene nulos, se imprime "Sin valores nulos."

---

## Celda 8 — Construcción de la tabla maestra de clientes PAY

Se parte de las ministraciones PAY para extraer los clientes únicos y construir una tabla con un registro por cliente. La cadena de unión es:

```
pay (Person Id) → data_clientes (Person Id / RFC) → data_entity (rfc / profile_entity_id)
```

El `profile_entity_id` es la llave que conecta al cliente con todas las tablas financieras (`balance_sheet`, `income_statement`, etc.).

**Cómo interpretar:** se imprime cuántos clientes PAY únicos existen, cuántos tienen `profile_entity_id` encontrado y cuántos no. Los clientes sin `profile_entity_id` no podrán conectarse con las tablas financieras. Si el número de no encontrados es alto, puede haber inconsistencias en los RFC entre sistemas.

---

## Celda 9 — Cobertura de variables por cliente PAY

Para cada fuente de datos se calcula cuántos clientes PAY tienen información disponible y cuántos no, listando por nombre a los clientes sin datos.

Para las tablas financieras (`balance_sheet`, `income_statement`, etc.) la unión se hace por `profile_entity_id`. Para `xls_registro` se usa el RFC directamente. Para las tablas de Buró (`csv_score`, `csv_vc_pyme`) se hace una coincidencia por nombre normalizado, ya que no tienen un ID compartido.

**Normalización de nombres:** antes de comparar, los nombres pasan por un proceso de limpieza que elimina tildes, convierte a mayúsculas, quita puntuación y remueve sufijos legales comunes (`SA DE CV`, `SAPI DE CV`, `SRL`, etc.). Esto mejora el match sin llegar a fuzzy matching completo.

**Cómo interpretar:**
- **% cobertura alto (>80%):** la fuente es confiable para el modelo
- **% cobertura medio (40–80%):** usar con precaución; puede sesgar el modelo si los clientes sin datos tienen perfil diferente
- **% cobertura bajo (<40%):** la fuente puede no ser viable como variable predictora o requiere estrategia de imputación
- **Lista de no encontrados:** útil para investigar si el problema es de datos faltantes reales o de inconsistencias en nombres/RFC entre sistemas

---

## Celda 10 — Estadísticas descriptivas de variables clave

Se calculan las estadísticas descriptivas (conteo, media, desviación estándar, mínimo, cuartiles y máximo) de las variables numéricas más relevantes de cada fuente.

**Cómo interpretar por tabla:**

| Tabla | Qué observar |
|---|---|
| `pay` | Distribución de montos (`Amount`) y plazos (`Cuotas`). Valores muy altos en `Amount` pueden indicar outliers. |
| `payc` | Distribución de `DPD` — si la mayoría es 0 la cartera está sana. `mvenc` elevado indica capital en riesgo. `MOB` indica la madurez promedio del portafolio analizado. |
| `balance_sheet` | Revisar si hay valores negativos en activos (posible error) o pasivos mayores a activos (empresa con patrimonio negativo). |
| `income_statement` | `net_profit` negativo indica empresas con pérdidas. Alta varianza en `sales` puede reflejar empresas de muy distinto tamaño. |
| `cash_flow` | `OUTFLOW` mayor que `INFLOW` de forma sostenida es señal de alerta de liquidez. |
| `annual_invoices_comparisons` | `margin` negativo indica que los gastos superan a los ingresos. `days_sales_outstanding` muy alto indica problemas de cobranza. |
| `employees` | Empresas con 0 o 1 empleado pueden ser empresas fantasma o holdings. Alta varianza indica heterogeneidad del portafolio. |
| `csv_vc_pyme` | `pct_pagos_bancarias` ('5') y `pct_pagos_no_bancarias` ('6') cercanos a 1 indican buen historial de pago. `quitas_castigos_12m` ('14') con valor 1 es señal de alerta crítica. |

---

## Celda 11 — Duplicados y outliers

### Duplicados

Para cada tabla se cuenta el número de filas completamente duplicadas (misma combinación de valores en todas las columnas).

**Cómo interpretar:** un duplicado en tablas transaccionales (créditos, abonos) puede indicar un error de carga doble. En tablas con un registro por empresa-año (balance, income), un duplicado es más grave porque distorsionaría el análisis financiero. Si se encuentran duplicados relevantes, deben eliminarse antes de la Fase 2.

### Outliers (método IQR)

Para 13 variables numéricas clave se aplica el método de rango intercuartílico (IQR) con factor 1.5. Un valor se considera outlier si cae fuera del rango:

```
[Q1 − 1.5 × IQR,  Q3 + 1.5 × IQR]
```

Las variables analizadas son:

| Variable | Tabla | Qué representa |
|---|---|---|
| `Amount` | `pay` | Monto de cada ministración PAY |
| `Cuotas` | `pay` | Plazo en meses de cada ministración |
| `DPD` | `payc` | Días de atraso por cuota |
| `mvenc` | `payc` | Capital vencido por cuota |
| `MOB` | `payc` | Meses en libros |
| `total_assets` | `balance_sheet` | Activos totales |
| `total_liabilities` | `balance_sheet` | Pasivos totales |
| `sh_equity` | `balance_sheet` | Capital contable |
| `sales` | `income_statement` | Ventas totales |
| `net_income` | `income_statement` | Utilidad neta |
| `total_income` | `annual_invoices_comparisons` | Ingresos totales facturados |
| `margin` | `annual_invoices_comparisons` | Margen operativo |
| `total` | `employees` | Número de empleados |

**Cómo interpretar:**

- **% de outliers bajo (<5%):** normal en datos financieros; los casos extremos probablemente son reales (empresas grandes o con comportamiento atípico). Considerar transformaciones logarítmicas en la Fase 2.
- **% de outliers medio (5–15%):** revisar si corresponden a errores de captura o a empresas genuinamente atípicas. Evaluar si se deben excluir o tratar por separado.
- **% de outliers alto (>15%):** señal de que la distribución es muy sesgada o que hay problemas de calidad de datos. Requiere atención antes de modelar.
- **Rango normal:** se imprime el límite inferior y superior del IQR. Valores fuera de ese rango son los candidatos a revisar manualmente.

> Nota: el método IQR es conservador — no elimina outliers, solo los identifica. La decisión de tratamiento se toma en la Fase 2.

---

## Celda 12 — Correlaciones entre variables clave

Se construye una tabla de una fila por cliente PAY combinando variables financieras y de comportamiento crediticio, y se visualiza su correlación de Pearson.

### Lógica de fuentes (prioridad)

Para las variables de balance e ingresos se aplica una jerarquía de fuentes:

| Prioridad | Fuente | Variables |
|---|---|---|
| 1 (primaria) | `balance_sheet` | `total_assets`, `total_liabilities`, `sh_equity`, `current_assets`, `current_liabilities`, `current_debt`, `lt_debt` |
| 1 (primaria) | `income_statement` | `sales`, `gross_income`, `operating_income`, `net_income` |
| 2 (fallback) | `xls_registro` | Columnas `u_*` equivalentes — solo para clientes sin datos en las fuentes primarias |

`xls_registro` se usa exclusivamente como respaldo para no perder clientes con datos financieros disponibles en ese registro pero ausentes en las fuentes primarias. La unión con `xls_registro` se hace por RFC.

> Nota: en el fallback, `u_lt_liabilities` (pasivos de largo plazo) se usa como proxy de `lt_debt` (deuda de largo plazo). Ambas miden deuda a largo plazo pero no son idénticas.

Al ejecutar la celda se imprime un resumen de cuántos clientes se completaron desde cada fuente.

### Variables incluidas en la matriz de correlación

| Grupo | Variables |
|---|---|
| Balance | `total_assets`, `total_liabilities`, `sh_equity`, `current_assets`, `current_liabilities`, `current_debt`, `lt_debt` |
| Resultados | `sales`, `gross_income`, `operating_income`, `net_income` |
| Comportamiento crediticio | `dpd_max` (máximo DPD del cliente), `mvenc_total` (capital vencido acumulado), `mob_max` (MOB máximo) |

### Cómo interpretar el heatmap

El heatmap muestra el triángulo inferior de la matriz de correlación de Pearson. Los valores van de −1 a 1:

| Valor | Interpretación |
|---|---|
| Cercano a **+1** | Relación positiva fuerte — ambas variables crecen juntas |
| Cercano a **−1** | Relación negativa fuerte — cuando una sube, la otra baja |
| Cercano a **0** | Sin relación lineal |

**Qué buscar específicamente:**

- **Correlación entre variables financieras y `dpd_max` / `mvenc_total`:** indica si el tamaño o salud financiera de la empresa se relaciona con su comportamiento de pago. Una correlación negativa entre `sh_equity` y `dpd_max` sugiere que empresas con mayor capital propio pagan mejor.
- **Multicolinealidad:** correlaciones muy altas (>0.9) entre variables predictoras (ej. `total_assets` y `sales`) indican que aportan información redundante. En la Fase 3 se deberá seleccionar una o combinarlas.
- **Variables con baja correlación con todo:** pueden ser poco informativas para el modelo o tener una relación no lineal que Pearson no captura.
- **`mob_max` y `dpd_max`:** se espera correlación positiva — a más tiempo en libros, mayor probabilidad de haber tenido algún atraso en algún momento.

---

## Celda 13 — Factor de Inflación de la Varianza (FIV)

Complementa la correlación de Pearson para detectar multicolinealidad. Mientras que Pearson analiza pares de variables, el FIV mide cuánto se puede explicar una variable a partir de todas las demás en conjunto — lo que permite detectar colinealidad múltiple que Pearson no captura.

Se usa `variance_inflation_factor` de `statsmodels`. El cálculo requiere datos completos, por lo que se eliminan filas con nulos antes de correr el análisis.

**Cómo interpretar:**

| FIV | Nivel | Acción recomendada en Fase 2 |
|---|---|---|
| < 5 | Aceptable ✓ | Incluir en el modelo sin cambios |
| 5 – 10 | Moderada ~ | Evaluar si aporta información adicional; considerar eliminar o combinar |
| > 10 | Severa ⚠ | Eliminar del modelo o reemplazar por una variable compuesta (ej. ratio) |

**Qué esperar:** variables del mismo estado financiero tienden a tener FIV alto entre sí — por ejemplo `total_assets`, `total_liabilities` y `sh_equity` están relacionadas por identidad contable (`Activos = Pasivos + Capital`), por lo que es normal que tengan FIV elevado. En ese caso, la solución es construir ratios (ej. `apalancamiento = total_liabilities / sh_equity`) en lugar de incluir las tres variables por separado.
