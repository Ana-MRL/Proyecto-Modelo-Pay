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
