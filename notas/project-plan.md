# Plan de Trabajo — Modelo Predictivo de Riesgo Pay

## Decisiones de diseño

| Elemento | Definición |
|---|---|
| Variable objetivo | Probabilidad de incumplimiento (score continuo, 0 a 1) |
| Alcance | Clientes actuales + prospectos nuevos |
| Tipo de modelo | Clasificación con salida probabilística |
| Herramienta | Python + Jupyter Notebooks |

---

## Fases

### Fase 1 — Exploración de datos (EDA)
**Archivo:** `codigo/01-model-coding.ipynb`
**Estado:** Completada
**Fecha de inicio:** 2026-03-27
**Fecha de cierre:** 2026-04-07

- [x] Cargar y revisar la base de datos (`data/Base de datos.xlsx`)
- [x] Revisar estructura: columnas, tipos de datos, dimensiones
- [x] Identificar variables disponibles para clientes y prospectos
- [x] Analizar distribución de variables (estadísticas descriptivas)
- [x] Detectar valores nulos, duplicados y outliers
- [x] Visualizar correlaciones (Pearson + FIV)

---

### Fase 2 — Limpieza y preparación de datos
**Archivo:** `codigo/02-data-cleaning.ipynb`
**Estado:** En progreso
**Fecha de inicio:** 2026-04-07
**Fecha de cierre:** —

- [x] Construir variable objetivo (DPD > 30 por cliente)
- [x] Construir features por cliente (crediticias, financieras, Buró, SAT, empleados)
- [x] Tratar valores nulos (imputación con mediana)
- [ ] Codificar variables categóricas
- [x] Normalizar/estandarizar variables numéricas (StandardScaler)
- [x] Dividir en conjunto de entrenamiento y prueba (80/20 estratificado)
- [x] Guardar datasets procesados en `data/`

---

### Fase 3 — Modelado
**Archivo:** `codigo/03-modeling.ipynb`
**Estado:** Pendiente
**Fecha de inicio:** —
**Fecha de cierre:** —

- [ ] Definir features finales (compatibles con clientes y prospectos)
- [ ] Construir features agregadas de atraso a nivel cliente/línea desde `payc`:
  - `capital_vencido_total` — suma de `mvenc`
  - `pct_capital_vencido` — `mvenc / Amount`
  - `cuotas_en_atraso` — count donde `DPD > 0`
  - `max_dpd` — máximo DPD registrado
  - `capital_vencido_ultimo_mob` — capital vencido en el MOB más reciente
- [ ] Entrenar modelos candidatos:
  - [ ] Regresión logística (baseline)
  - [ ] Random Forest
  - [ ] Gradient Boosting (XGBoost o LightGBM)
- [ ] Validación cruzada
- [ ] Ajuste de hiperparámetros

---

### Fase 4 — Evaluación
**Archivo:** `codigo/04-evaluation.ipynb`
**Estado:** Pendiente
**Fecha de inicio:** —
**Fecha de cierre:** —

- [ ] Métricas: AUC-ROC, precisión, recall, F1
- [ ] Matriz de confusión
- [ ] Curva ROC
- [ ] Análisis de importancia de variables
- [ ] Selección del modelo final

---

### Fase 5 — Reporte
**Archivo:** `reporte/model-results.md`
**Estado:** Pendiente
**Fecha de inicio:** —
**Fecha de cierre:** —

- [ ] Resumen ejecutivo
- [ ] Descripción del modelo seleccionado
- [ ] Métricas de desempeño
- [ ] Variables más importantes
- [ ] Conclusiones y recomendaciones

---

## Historial de avance

| Fecha | Evento |
|---|---|
| 2026-03-26 | Estructura inicial del proyecto creada |
| 2026-03-27 | Convenciones y reglas definidas en `CLAUDE.md` |
| 2026-03-27 | Base de datos cargada en `data/` |
| 2026-03-27 | Plan de trabajo definido y aprobado |
| 2026-04-06 | Fase 1 EDA iniciada: carga de tablas, estructura, nulos, cobertura y estadísticas descriptivas |
| 2026-04-07 | Fase 1 EDA completada: duplicados, outliers (IQR), correlaciones (Pearson + FIV) |
| 2026-04-07 | Fase 2 iniciada: notebook 02-data-cleaning.ipynb creado |
