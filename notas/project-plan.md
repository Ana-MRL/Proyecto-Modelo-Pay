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
**Archivo:** `codigo/01-eda.ipynb`
**Estado:** Pendiente
**Fecha de inicio:** —
**Fecha de cierre:** —

- [ ] Cargar y revisar la base de datos (`data/Base de datos.xlsx`)
- [ ] Revisar estructura: columnas, tipos de datos, dimensiones
- [ ] Identificar variables disponibles para clientes y prospectos
- [ ] Analizar distribución de variables
- [ ] Detectar valores nulos, duplicados y outliers
- [ ] Visualizar correlaciones

---

### Fase 2 — Limpieza y preparación de datos
**Archivo:** `codigo/02-data-cleaning.ipynb`
**Estado:** Pendiente
**Fecha de inicio:** —
**Fecha de cierre:** —

- [ ] Tratar valores nulos (imputación o eliminación)
- [ ] Codificar variables categóricas
- [ ] Normalizar/estandarizar variables numéricas
- [ ] Dividir en conjunto de entrenamiento y prueba
- [ ] Guardar datasets procesados en `data/`

---

### Fase 3 — Modelado
**Archivo:** `codigo/03-modeling.ipynb`
**Estado:** Pendiente
**Fecha de inicio:** —
**Fecha de cierre:** —

- [ ] Definir features finales (compatibles con clientes y prospectos)
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
