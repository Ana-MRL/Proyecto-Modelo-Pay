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
| `Is line` | Texto | Indica si el registro corresponde a la **línea de crédito** (`SI`) o a una **ministración** (`NO`). |
| `Loan Type` | Texto | Tipo de crédito. Se usa para filtrar únicamente las ministraciones (`MINISTRACION`). |
| `Period Number` | Numérico | Número de cuotas de la ministración. En el código se renombra a `Cuotas`. |
| `Cosecha` | Fecha | Mes de originación de la línea de crédito. En el código se renombra a `Originacion`. |
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
| `Installment` | Numérico | Número de cuota a la que corresponde el abono. En el código se renombra a `Cuota`. |
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
| `Full Name Or Business Name` | Texto | Nombre de la empresa. |
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

## Hoja: balance_sheet

Contiene los estados de situación financiera (balance general) anuales de las empresas acreditadas. Es una fuente clave de variables financieras para el modelo predictivo.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `id` | Numérico (ID) | Identificador único del registro. |
| `profile_id` | Numérico (ID) | Identificador de la empresa. Se conecta con `superia_entity` a través de `profile_entity_id`. |
| `year` | Numérico | Año al que corresponde el balance. |
| `total_assets` | Numérico | Activos totales de la empresa. |
| `current_assets` | Numérico | Activos circulantes (corto plazo). |
| `cash_equivalents` | Numérico | Efectivo y equivalentes de efectivo. |
| `accounts_receivable` | Numérico | Cuentas por cobrar. |
| `inventory` | Numérico | Inventario. |
| `lt_asset` | Numérico | Activos de largo plazo. |
| `property_plant_equipment` | Numérico | Propiedades, planta y equipo. |
| `intangible_assets` | Numérico | Activos intangibles. |
| `total_liabilities` | Numérico | Pasivos totales. |
| `common_stock` | Numérico | Capital social. |
| `sh_equity` | Numérico | Capital contable (patrimonio de los accionistas). |
| `retained_earnings` | Numérico | Utilidades retenidas. |
| `current_liabilities` | Numérico | Pasivos circulantes (corto plazo). |
| `lt_liabilities` | Numérico | Pasivos de largo plazo. |
| `current_debt` | Numérico | Deuda de corto plazo. |
| `lt_debt` | Numérico | Deuda de largo plazo. |
| `total_liabilities_sh_equity` | Numérico | Suma de pasivos totales y capital contable (equivale a activos totales). |
| `accounts_payable` | Numérico | Cuentas por pagar. |

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **superia_entity** | `profile_id` ↔ `profile_entity_id` | Permite asociar el balance a cada empresa y obtener su RFC. |

---

## Hoja: annual_invoices_comparisons

Contiene comparaciones anuales de facturación de las empresas acreditadas, derivadas del SAT. Incluye ingresos, gastos, márgenes y métricas de liquidez operativa. Es una fuente importante de variables financieras para el modelo.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `id` | Numérico (ID) | Identificador único del registro. |
| `profile_id` | Numérico (ID) | Identificador de la empresa. Se conecta con `superia_entity` a través de `profile_entity_id`. |
| `period` | Numérico | Período (año) al que corresponde la comparación. |
| `total_income` | Numérico | Ingresos totales facturados. |
| `income_payment_pending` | Numérico | Ingresos con pago pendiente de cobro. |
| `total_income_cancelled` | Numérico | Ingresos de facturas canceladas. |
| `income_discounts` | Numérico | Descuentos aplicados a ingresos. |
| `income_credit_notes` | Numérico | Notas de crédito en ingresos. |
| `net_income` | Numérico | Ingresos netos. |
| `income_due_upon_receipt` | Numérico | Ingresos con vencimiento inmediato (pago de contado). |
| `income_due_over_time` | Numérico | Ingresos con vencimiento a plazo. |
| `collected_income_due_over_time` | Numérico | Porción ya cobrada de los ingresos a plazo. |
| `total_expenses` | Numérico | Gastos totales facturados. |
| `received_payment_pending` | Numérico | Gastos con pago pendiente. |
| `total_expenses_cancelled` | Numérico | Gastos de facturas canceladas. |
| `expenses_discounts` | Numérico | Descuentos en gastos. |
| `expenses_credit_notes` | Numérico | Notas de crédito en gastos. |
| `payroll` | Numérico | Nómina. |
| `net_expenses_plus_payroll` | Numérico | Gastos netos incluyendo nómina. |
| `margin` | Numérico | Margen operativo (ingresos netos − gastos netos). |
| `expenses_due_upon_receipt` | Numérico | Gastos con vencimiento inmediato. |
| `expenses_due_over_time` | Numérico | Gastos con vencimiento a plazo. |
| `paid_expenses_due_over_time` | Numérico | Porción ya pagada de los gastos a plazo. |
| `days_payable_outstanding` | Numérico | Días promedio que tarda la empresa en pagar a sus proveedores (DPO). |
| `days_sales_outstanding` | Numérico | Días promedio que tarda la empresa en cobrar a sus clientes (DSO). |
| `net_expenses` | Numérico | Gastos netos sin nómina. |
| `profit_or_loss` | Numérico | Utilidad o pérdida del período. |

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **superia_entity** | `profile_id` ↔ `profile_entity_id` | Permite asociar la facturación a cada empresa y obtener su RFC. |

---

## Hoja: employees

Contiene el historial del número de empleados por empresa y período. Sirve como indicador del tamaño y estabilidad operativa de la empresa acreditada.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `id` | Numérico (ID) | Identificador único del registro. |
| `profile_id` | Numérico (ID) | Identificador de la empresa. Se conecta con `superia_entity` a través de `profile_entity_id`. |
| `total` | Numérico | Total de empleados en el período. |
| `year` | Numérico | Año del registro. |
| `month` | Numérico | Mes del registro. |

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **superia_entity** | `profile_id` ↔ `profile_entity_id` | Permite asociar el número de empleados a cada empresa. |

---

## Hoja: cash_flow

Contiene el flujo de efectivo por empresa y período, derivado de información bancaria. Permite evaluar la liquidez real de la empresa acreditada.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `profile_id` | Numérico (ID) | Identificador de la empresa. Se conecta con `superia_entity` a través de `profile_entity_id`. |
| `start_date` | Fecha | Fecha de inicio del período analizado. |
| `amount` | Numérico | Monto total del período. |
| `INFLOW` | Numérico | Entradas de efectivo. |
| `OUTFLOW` | Numérico | Salidas de efectivo. |

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **superia_entity** | `profile_id` ↔ `profile_entity_id` | Permite asociar el flujo de efectivo a cada empresa. |

---

## Hoja: income_statement

Contiene los estados de resultados anuales de las empresas acreditadas. Permite evaluar la rentabilidad y estructura de costos de cada empresa.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `id` | Numérico (ID) | Identificador único del registro. |
| `profile_id` | Numérico (ID) | Identificador de la empresa. Se conecta con `superia_entity` a través de `profile_entity_id`. |
| `year` | Numérico | Año al que corresponde el estado de resultados. |
| `sales` | Numérico | Ventas totales. |
| `cost_sales` | Numérico | Costo de ventas. |
| `gross_income` | Numérico | Utilidad bruta (ventas − costo de ventas). |
| `operating_costs` | Numérico | Costos y gastos operativos. |
| `operating_income` | Numérico | Utilidad operativa. |
| `cost_finance` | Numérico | Costos financieros (intereses, comisiones, etc.). |
| `net_income` | Numérico | Utilidad neta. |

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **superia_entity** | `profile_id` ↔ `profile_entity_id` | Permite asociar el estado de resultados a cada empresa y obtener su RFC. |

---

## Hoja: csv_vc_pyme

Contiene las variables califica del reporte de Buró de Crédito PyME por empresa. Complementa el score de `csv_score` con variables individuales del comportamiento crediticio.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `FECHA CARGA` | Fecha | Fecha en que se realizó la consulta al Buró de Crédito. |
| `PRIMERNOMBRE` | Texto | Nombre de la empresa consultada. **Ojo:** los nombres no coinciden exactamente con los de la hoja **Creditos** — se requiere fuzzy matching para empatar. |
| `5` | Numérico continuo (0 a 1) | Porcentaje de pagos en tiempo con instituciones financieras **bancarias**. |
| `6` | Numérico continuo (0 a 1) | Porcentaje de pagos en tiempo con instituciones financieras **no bancarias**. |
| `14` | Binaria (0 o 1) | Presencia de quitas, castigos y reestructuras con instituciones financieras bancarias en los últimos 12 meses. `0` = no existe, `1` = sí existe. |

---

### Limitante: relación por nombre aproximado

Esta hoja **no tiene un ID compartido** con las demás hojas. La unión debe hacerse por nombre de empresa (`PRIMERNOMBRE`), que no coincide exactamente con los nombres en otras hojas. Se requiere **fuzzy matching** (ver estrategias en la hoja `csv_score`).

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Tipo de join | Descripción |
|---|---|---|---|
| **Clientes** | `PRIMERNOMBRE` ↔ `Full Name` | Aproximado (fuzzy) | Permite asociar las variables califica a cada empresa del catálogo de clientes. |

---

## Hoja: superia_entity

Contiene el catálogo de entidades registradas en Superia Capital. Para este modelo, se usa únicamente como puente de unión entre el RFC y los datos de crédito.

### Columnas relevantes

| Columna | Tipo | Descripción |
|---|---|---|
| `profile_entity_id` | Numérico (ID) | Identificador único de cada cliente en el sistema. |
| `rfc` | Texto | RFC de la entidad. Llave de unión con la hoja **Creditos** a través de la columna `Taxpayer ID Number`. |

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **Creditos** | `rfc` ↔ `Taxpayer ID Number` | Permite asociar cada entidad con sus créditos registrados. |

---

## Hoja: xls_registro

Contiene el registro de evaluación crediticia de cada cliente. Combina datos generales, información de Buró de Crédito, resultados del modelo interno Cub y estados financieros en tres períodos distintos.

### Prefijos de estados financieros

| Prefijo | Descripción |
|---|---|
| `p_` | Penúltimo período anual |
| `u_` | Último período anual |
| `i_` | Último registro disponible (puede ser parcial o anual) |

---

### Columnas relevantes

#### Generales

| Columna | Tipo | Descripción |
|---|---|---|
| `id` | Numérico (ID) | Identificador único del registro. |
| `rfc` | Texto | RFC de la empresa. Llave de unión con otras tablas. |
| `reg_date` | Fecha | Fecha de registro del cliente. |
| `client_name` | Texto | Nombre del cliente. |
| `buro_rev_date` | Fecha | Fecha de consulta al Buró de Crédito. |
| `buro_active_loans` | Numérico | Número de créditos activos en Buró. |
| `buro_active_amount` | Numérico | Monto total de créditos activos en Buró. |
| `buro_largest_paid` | Numérico | Monto del crédito más grande ya pagado en Buró. |
| `buro_late_loans` | Numérico | Número de créditos con atraso en Buró. |
| `buro_late_amount` | Numérico | Monto total de créditos con atraso en Buró. |
| `loan_start_date` | Fecha | Fecha de inicio del crédito con Superia. |
| `operation_quantity` | Numérico | Número de operaciones con Superia. |
| `late_quantity` | Numérico | Número de atrasos registrados con Superia. |
| `latest_late_date` | Fecha | Fecha del último atraso. |
| `latest_late_amount` | Numérico | Monto del último atraso. |
| `loan_utilization` | Numérico | Porcentaje de utilización de la línea de crédito. |
| `max_late_days` | Numérico | Máximo de días de atraso registrado. |
| `cub_q_score` | Numérico | Score cualitativo del modelo interno Cub. |
| `cub_c_score` | Numérico | Score cuantitativo del modelo interno Cub. |
| `cub_score` | Numérico | Score global del modelo interno Cub. |
| `cub_p_default` | Numérico | Probabilidad de default según Cub. |
| `cub_expected_shortfall` | Numérico | Pérdida esperada en escenario adverso según Cub. |
| `cub_expected_loss` | Numérico | Pérdida esperada según Cub. |
| `cub_rating` | Texto | Calificación crediticia según Cub. |

#### Estados financieros (`p_`, `u_`, `i_`)

Las siguientes columnas existen con los tres prefijos. Ejemplo: `p_revenue`, `u_revenue`, `i_revenue`.

| Columna (sin prefijo) | Tipo | Descripción |
|---|---|---|
| `result_date` | Fecha | Fecha del estado financiero. |
| `revenue` | Numérico | Ingresos totales. |
| `gross_profit` | Numérico | Utilidad bruta. |
| `ebit` | Numérico | Utilidad antes de intereses e impuestos. |
| `interest_expense` | Numérico | Gastos financieros. |
| `net_profit` | Numérico | Utilidad neta. |
| `current_assets` | Numérico | Activos circulantes. |
| `cash_equivalents` | Numérico | Efectivo y equivalentes. |
| `inventory` | Numérico | Inventario. |
| `account_receivable` | Numérico | Cuentas por cobrar. |
| `ppe` | Numérico | Propiedades, planta y equipo. |
| `lt_assets` | Numérico | Activos de largo plazo. |
| `total_assets` | Numérico | Activos totales. |
| `current_liabilities` | Numérico | Pasivos circulantes. |
| `accounts_payable` | Numérico | Cuentas por pagar. |
| `st_debt` | Numérico | Deuda de corto plazo. |
| `total_debt` | Numérico | Deuda total. |
| `lt_liabilities` | Numérico | Pasivos de largo plazo. |
| `total_liabilities` | Numérico | Pasivos totales. |
| `common_stock` | Numérico | Capital social. |
| `sh_equity` | Numérico | Capital contable. |
| `retained_earnings` | Numérico | Utilidades retenidas. |
| `total_liabilities_sh_equity` | Numérico | Pasivos totales + capital contable. |
| `depreciation_amortization` | Numérico | Depreciación y amortización. |
| `ebitda` | Numérico | EBITDA. |
| `cf_operations` | Numérico | Flujo de efectivo de operaciones. |
| `capex` | Numérico | Inversión en activos fijos. |
| `fcf` | Numérico | Flujo de caja libre. |

#### Fiscales (`p_` y `u_` únicamente)

| Columna | Tipo | Descripción |
|---|---|---|
| `p_fiscal_revenue` | Numérico | Ingresos fiscales declarados del penúltimo período anual. |
| `p_fiscal_net_profit` | Numérico | Utilidad neta fiscal declarada del penúltimo período anual. |
| `u_fiscal_revenue` | Numérico | Ingresos fiscales declarados del último período anual. |
| `u_fiscal_net_profit` | Numérico | Utilidad neta fiscal declarada del último período anual. |

---

### Relaciones con otras hojas

| Hoja | Columna de unión | Descripción |
|---|---|---|
| **Creditos** | `rfc` ↔ `Taxpayer ID Number` | Permite asociar el registro de evaluación a las operaciones de crédito del cliente. |

---

## Historial de cambios

| Fecha | Descripción |
|---|---|
| 2026-03-27 | Definición inicial de la hoja Creditos |
| 2026-03-27 | Agrega Person Id y relación con Clientes en Creditos |
| 2026-03-27 | Definición de la hoja Abonos |
| 2026-03-27 | Definición de la hoja Clientes |
| 2026-03-27 | Definición de la hoja csv_score (con nota de fuzzy matching) |
