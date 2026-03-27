# Definición de Bases de Datos

---

## Hoja: Creditos

Contiene los registros de líneas de crédito y ministraciones del producto Pay.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `Portfolio Id` | Numérico (ID) | Identificador del movimiento. Se conecta con la tabla de la hoja **Abonos**. |
| `External Code` | ID | ID del movimiento en la página web de Superia Capital. |
| `Line` | Numérico (ID) | Identificador numérico de la línea de crédito. |
| `Name` | Texto | Nombre de la empresa acreditada. |
| `Product` | Texto | Producto de la línea de crédito. Para este modelo, se filtran únicamente los registros cuyo nombre contenga la palabra **PAY**. |
| `Is Line` | Booleano | Indica si el registro corresponde a la **línea de crédito** (`True`) o a una **ministración** (`False`). |
| `Opening Date` | Fecha | Fecha de apertura de la línea o ministración. |
| `Due Date` | Fecha | Fecha de vencimiento de la línea o ministración. |
| `Amount` | Numérico | Monto de la línea o ministración. Ver nota de conversión de divisa abajo. |
| `Currency` | Texto | Divisa del registro. |
| `Agreed Exchange Rate` | Numérico | Tipo de cambio pactado. Se usa para convertir montos en dólares a pesos. |
| `Loan Status` | Texto | Estado de la línea. Solo indica si fue **liquidada** o sigue **activa** — no refleja atrasos. El cálculo de morosidad deberá construirse por nuestra parte. |

---

### Filtros a aplicar

| Filtro | Criterio |
|---|---|
| Producto | Conservar únicamente registros donde `Product` contenga la palabra **PAY** |

---

### Notas de transformación

#### Conversión de divisa

Los registros cuya `Currency` sea **DOLARES** deben convertirse a pesos antes de cualquier análisis:

```
Amount (MXN) = Amount × Agreed Exchange Rate
```

Los registros en pesos se usan directamente sin conversión.

#### Cálculo de morosidad

`Loan Status` no indica si un crédito está atrasado. La variable de incumplimiento deberá calcularse comparando `Due Date` contra la fecha de pago registrada en la hoja **Abonos** (o contra la fecha de corte del análisis si no hay pago registrado).

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **Abonos** | `Portfolio Id` | Permite asociar los abonos realizados a cada movimiento. |

---

## Hoja: Abonos

> Pendiente de definir.

---

## Historial de cambios

| Fecha | Descripción |
|---|---|
| 2026-03-27 | Definición inicial de la hoja Creditos |
