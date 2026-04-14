# Variables Candidatas para el Modelo

Inventario de todas las variables disponibles en `features_master.csv` una vez ejecutada la Fase 2. Se indica la fuente, tipo y recomendación de uso para reducir multicolinealidad.

**Convención de recomendación:**
- `✓ Preferida` — ratio o variable independiente, baja colinealidad esperada
- `⚠ Usar con cuidado` — puede ser colineal con otras variables de la misma fuente
- `✗ Excluir del modelo` — variable de identificación o redundante con otra preferida

---

## 1. Comportamiento crediticio (por línea de crédito)

Fuente: `payc` agregado a nivel `Line`.

| Variable | Descripción | Recomendación |
|---|---|---|
| `n_ministraciones` | Número de ministraciones activas de la línea | `✓ Preferida` |
| `tasa_interes` | Tasa de interés anual promedio de la línea | `✓ Preferida` |
| `cuotas_promedio` | Número promedio de cuotas por ministración | `✓ Preferida` |
| `pct_cuotas_atraso` | % de cuotas con DPD > 0 | `✓ Preferida` |
| `dpd_max` | Máximo de días de atraso observado | `✓ Preferida` |
| `pct_capital_vencido` | Capital vencido / monto total desembolsado | `✓ Preferida` |
| `mob_max` | MOB máximo observado en la línea | `✓ Preferida` |
| `monto_total` | Monto total desembolsado en la línea | `⚠ Usar con cuidado` — colineal con `monto_promedio` |
| `monto_promedio` | Monto promedio por ministración | `⚠ Usar con cuidado` — colineal con `monto_total` |
| `dpd_promedio` | DPD promedio de todas las cuotas | `⚠ Usar con cuidado` — colineal con `dpd_max` |
| `mvenc_total` | Capital vencido absoluto (pesos) | `⚠ Usar con cuidado` — colineal con `pct_capital_vencido` |

---

## 2. Balance general

Fuente primaria: `balance_sheet`. Fallback: `xls_registro` (prefijo `u_`).

### Ratios (preferidos)

| Variable | Fórmula | Dimensión | Recomendación |
|---|---|---|---|
| `deuda_activos` | pasivos totales / activos totales | Solvencia | `✓ Preferida` |
| `liquidez_corriente` | activos CP / pasivos CP | Liquidez | `✓ Preferida` |
| `prueba_acida` | (activos CP − inventario) / pasivos CP | Liquidez estricta | `✓ Preferida` |
| `razon_efectivo` | efectivo / pasivos CP | Liquidez inmediata | `✓ Preferida` |
| `pct_deuda_cp` | deuda CP / deuda total | Estructura de deuda | `✓ Preferida` |
| `pct_activos_corrientes` | activos CP / activos totales | Estructura de activos | `✓ Preferida` |
| `apalancamiento` | pasivos totales / capital contable | Solvencia | `⚠ Usar con cuidado` — derivado de `deuda_activos` y `capital_activos` |
| `capital_activos` | capital contable / activos totales | Solvencia | `⚠ Usar con cuidado` — `= 1 − deuda_activos` (colinealidad perfecta) |

> **Nota:** `deuda_activos + capital_activos ≈ 1`. Usar solo una de las dos.
> Las tres métricas de liquidez (`liquidez_corriente`, `prueba_acida`, `razon_efectivo`) son versiones progresivamente restrictivas. Evaluar correlación antes de incluir las tres.

### Variables brutas

| Variable | Descripción | Recomendación |
|---|---|---|
| `total_assets` | Activos totales | `⚠ Usar con cuidado` — indicador de tamaño útil, pero colineal con variables de income |
| `deuda_total` | Deuda CP + LP | `⚠ Usar con cuidado` — colineal con `deuda_activos` si se incluye `total_assets` |
| `total_liabilities`, `sh_equity`, `current_assets`, `current_liabilities`, `current_debt`, `lt_debt`, `cash_equivalents`, `accounts_receivable`, `inventory` | Componentes del balance | `✗ Excluir del modelo` — están capturados por los ratios |

---

## 3. Estado de resultados

Fuente primaria: `income_statement`. Fallback: `xls_registro` (prefijo `u_`).

### Ratios (preferidos)

| Variable | Fórmula | Dimensión | Recomendación |
|---|---|---|---|
| `margen_neto` | utilidad neta / ventas | Rentabilidad final | `✓ Preferida` |
| `margen_operativo` | EBIT / ventas | Rentabilidad operativa | `✓ Preferida` |
| `cobertura_intereses` | EBIT / costos financieros | Capacidad de pago deuda | `✓ Preferida` |
| `roa` | utilidad neta / activos totales | Rentabilidad sobre activos | `✓ Preferida` |
| `rotacion_activos` | ventas / activos totales | Eficiencia de activos | `✓ Preferida` |
| `carga_financiera` | costos financieros / ventas | Peso de deuda financiera | `⚠ Usar con cuidado` — inverso de `cobertura_intereses` |
| `margen_bruto` | utilidad bruta / ventas | Rentabilidad bruta | `⚠ Usar con cuidado` — colineal con `margen_operativo` y `margen_neto` |
| `roe` | utilidad neta / capital contable | Rentabilidad sobre capital | `⚠ Usar con cuidado` — `roa × apalancamiento` (DuPont); colineal si se incluyen ambos |

> **Nota DuPont:** `roe = margen_neto × rotacion_activos × apalancamiento`. Si se incluyen las tres componentes más `roe`, hay colinealidad perfecta.

### Variables brutas

| Variable | Descripción | Recomendación |
|---|---|---|
| `sales` | Ventas totales | `⚠ Usar con cuidado` — útil como escala, colineal con `total_assets` vía `rotacion_activos` |
| `gross_income`, `operating_income`, `net_income`, `cost_finance` | Componentes del estado de resultados | `✗ Excluir del modelo` — capturados por los ratios |

---

## 4. Facturación SAT

Fuente: `annual_invoices_comparisons`.

| Variable | Descripción | Recomendación |
|---|---|---|
| `sat_margin` | Margen SAT (ingresos netos − gastos netos) | `✓ Preferida` |
| `days_sales_outstanding` | Días promedio para cobrar (DSO) | `✓ Preferida` |
| `days_payable_outstanding` | Días promedio para pagar a proveedores (DPO) | `✓ Preferida` |
| `sat_net_income` | Ingresos netos SAT | `⚠ Usar con cuidado` — colineal con `sat_total_income` |
| `sat_total_income` | Ingresos totales SAT | `⚠ Usar con cuidado` — colineal con `sales` del income statement |

---

## 5. Empleados

Fuente: `employees`.

| Variable | Descripción | Recomendación |
|---|---|---|
| `empleados` | Total de empleados (último período) | `✓ Preferida` — proxy de tamaño operativo |

---

## 6. Buró de Crédito

Fuente: `csv_score`, `csv_vc_pyme`, `xls_registro`.

| Variable | Descripción | Recomendación |
|---|---|---|
| `buro_score` | Score PyME Buró de Crédito | `✓ Preferida` |
| `buro_pct_pago_bancario` | % pagos en tiempo con instituciones bancarias | `✓ Preferida` |
| `buro_pct_pago_no_bancario` | % pagos en tiempo con instituciones no bancarias | `✓ Preferida` |
| `buro_quitas_12m` | Presencia de quitas/reestructuras últimos 12 meses (binaria) | `✓ Preferida` |
| `loan_utilization` | % de utilización de la línea | `✓ Preferida` |
| `max_late_days` | Máximo días de atraso histórico en Buró | `✓ Preferida` |
| `cub_score` | Score modelo interno Cub | `✓ Preferida` |
| `cub_p_default` | Probabilidad de default según Cub | `⚠ Usar con cuidado` — derivado del `cub_score`; evaluar si incluir ambos |
| `buro_late_loans` | Número de créditos con atraso en Buró | `⚠ Usar con cuidado` — colineal con `buro_late_amount` |
| `buro_late_amount` | Monto en atraso en Buró | `⚠ Usar con cuidado` — colineal con `buro_late_loans` |
| `buro_active_loans` | Número de créditos activos en Buró | `⚠ Usar con cuidado` — colineal con `buro_active_amount` |
| `buro_active_amount` | Monto total activo en Buró | `⚠ Usar con cuidado` — colineal con `buro_active_loans` |
| `operation_quantity` | Número de operaciones con Superia | `⚠ Usar con cuidado` — colineal con `late_quantity` |
| `late_quantity` | Número de atrasos con Superia | `⚠ Usar con cuidado` — colineal con `pct_cuotas_atraso` (feat_cred) |

---

## Resumen ejecutivo

| Fuente | Variables preferidas | Variables a evaluar | Variables a excluir |
|---|---|---|---|
| Comportamiento crediticio | 7 | 4 | 0 |
| Balance — ratios | 6 | 2 | 9 |
| Income — ratios | 5 | 3 | 4 |
| Facturación SAT | 3 | 2 | 0 |
| Empleados | 1 | 0 | 0 |
| Buró | 6 | 8 | 0 |
| **Total** | **28** | **19** | **13** |

> La selección final se hace en Fase 3 con análisis de correlación, VIF y/o importancia de variables según el modelo elegido.
