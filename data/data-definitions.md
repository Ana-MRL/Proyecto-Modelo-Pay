# Definición de Bases de Datos

---

## Hoja: Creditos

Contiene los registros de líneas de crédito y ministraciones del producto Pay.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `Portfolio Id` | Numérico (ID) | Identificador del movimiento. Se conecta con la tabla de la hoja **Abonos**. |
| `Person Id` | Numérico (ID) | Identificador de la empresa acreditada. Se conecta con la hoja **Clientes** para extraer RFC y otros datos. |
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
| **Clientes** | `Person Id` | Permite obtener RFC y otros datos de la empresa acreditada. |

---

## Hoja: Abonos

Contiene los registros de pagos y abonos realizados por las empresas acreditadas.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `Portfolio Id` | Numérico (ID) | Identificador del movimiento. Se conecta con la hoja **Creditos** a nivel de ministración. |
| `Person Id` | Numérico (ID) | Identificador de la empresa acreditada. Se conecta con la hoja **Clientes**. |
| `Full Name` | Texto | Nombre de la empresa acreditada. |
| `Product` | Texto | Producto al que corresponde el abono. Se filtran únicamente los registros cuyo nombre contenga la palabra **PAY**. |
| `Application Date` | Fecha | Fecha en que se realizó el pago o abono. |
| `Amount` | Numérico | Monto total pagado. |
| `Capital` | Numérico | Porción del pago aplicada a capital. Ver nota de conversión de divisa abajo. |
| `Interest` | Numérico | Porción del pago aplicada a intereses. |
| `Penalty` | Numérico | Monto pagado en moratorios. Si es mayor a cero, indica que el cliente se atrasó en su pago — variable clave para el modelo. |
| `Payment Currency` | Texto | Divisa del pago. |
| `Agreed Exchange Rate` | Numérico | Tipo de cambio pactado. Se usa para convertir el capital de pagos en dólares a pesos. |

---

### Filtros a aplicar

| Filtro | Criterio |
|---|---|
| Producto | Conservar únicamente registros donde `Product` contenga la palabra **PAY** |

---

### Notas de transformación

#### Conversión de divisa

Los registros cuyo `Payment Currency` sea **DOLARES** deben convertirse antes de cualquier análisis:

```
Capital (MXN) = Capital × Agreed Exchange Rate
```

Los registros en pesos se usan directamente sin conversión.

#### Indicador de morosidad

`Penalty > 0` indica que el cliente pagó moratorios, lo que implica que se atrasó en al menos un pago. Esta columna es un insumo directo para construir la variable objetivo del modelo.

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **Creditos** | `Portfolio Id` | Permite asociar cada abono a su ministración correspondiente. |
| **Clientes** | `Person Id` | Permite obtener RFC y otros datos de la empresa acreditada. |

---

## Hoja: Clientes

Contiene el catálogo de personas registradas en el sistema. Para este modelo, se filtra únicamente a empresas (personas morales) con clasificación de cliente.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `Person Id` | Numérico (ID) | Identificador de la empresa. Llave de unión con las hojas **Creditos** y **Abonos**. |
| `Full Name` | Texto | Nombre de la empresa. |
| `Person Type` | Texto | Indica si es persona física o moral. Se filtran únicamente las **personas morales**. |
| `Taxpayer ID Number` | Texto | RFC del cliente. |
| `Person Classification` | Texto | Clasificación del registro (cliente, prospecto, obligado solidario, etc.). Se filtran únicamente los **clientes**. |

---

### Filtros a aplicar

| Filtro | Criterio |
|---|---|
| Tipo de persona | Conservar únicamente registros donde `Person Type` sea **persona moral** |
| Clasificación | Conservar únicamente registros donde `Person Classification` sea **cliente** |

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **Creditos** | `Person Id` | Permite asociar líneas y ministraciones a cada empresa. |
| **Abonos** | `Person Id` | Permite asociar pagos a cada empresa. |

---

## Hoja: csv_score

Contiene los resultados de consulta al Buró de Crédito (score PyME) por empresa.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `FECHA CARGA` | Fecha | Fecha en que se realizó la consulta al Buró de Crédito. |
| `PRIMERNOMBRE` | Texto | Nombre de la empresa consultada. |
| `VALORSCORE` | Numérico | Score PyME asignado por Buró de Crédito. Indica la calificación crediticia de la empresa. |

---

### Filtros a aplicar

Ninguno definido por el momento.

---

### Limitante: relación por nombre aproximado

Esta hoja **no tiene un ID compartido** con las demás hojas. La única columna de unión posible es el nombre de la empresa (`PRIMERNOMBRE`), pero los nombres no coinciden exactamente con los de las hojas **Creditos**, **Abonos** o **Clientes**.

La relación deberá hacerse mediante **coincidencia aproximada (fuzzy matching)**. Estrategias a evaluar:

| Estrategia | Descripción |
|---|---|
| Normalización de texto | Quitar tildes, mayúsculas, puntuación y palabras genéricas (S.A., DE C.V., etc.) antes de comparar |
| Similitud de cadenas | Usar métricas como Levenshtein o `rapidfuzz` en Python |
| Revisión manual | Para casos ambiguos, validar la asignación manualmente |

> Esta limitante puede afectar la cobertura del score PyME en el modelo. Se recomienda documentar los casos no emparejados.

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Tipo de join | Descripción |
|---|---|---|---|
| **Clientes** | `PRIMERNOMBRE` ↔ `Full Name` | Aproximado (fuzzy) | Permite asociar el score PyME a cada empresa en el catálogo de clientes. |

---

## Historial de cambios

| Fecha | Descripción |
|---|---|
| 2026-03-27 | Definición inicial de la hoja Creditos |
| 2026-03-27 | Agrega Person Id y relación con Clientes en Creditos |
| 2026-03-27 | Definición de la hoja Abonos |
| 2026-03-27 | Definición de la hoja Clientes |
| 2026-03-27 | Definición de la hoja csv_score (con nota de fuzzy matching) |
