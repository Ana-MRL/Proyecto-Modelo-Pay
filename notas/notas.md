# Notas del proyecto

---

## Avance del proyecto — 2026-04-06

### Estado actual

- `2026-04-06` — Revisión general del estado del proyecto
- `2026-04-06` — `01-eda.ipynb` eliminado del repo; reemplazado por `01-model-coding.ipynb` como notebook de trabajo principal
- `2026-04-06` — `CosechasPAY.ipynb` agregado al repo como notebook original de referencia/respaldo
- `2026-04-06` — `data/Base de datos.xlsx` actualizada con datos más recientes de la fuente externa (actualización periódica esperada)
- `2026-04-06` — Creado `codigo/codigo.md` con documentación explicativa del notebook `01-model-coding.ipynb`
- `2026-04-06` — `data/data-definitions.md` ampliado: correcciones en Creditos, Abonos y Clientes; definición de 8 hojas nuevas (balance_sheet, annual_invoices_comparisons, employees, cash_flow, income_statement, superia_entity, csv_vc_pyme, xls_registro)
- `2026-04-06` — Fase 1 EDA agregada al notebook `01-model-coding.ipynb`:
  - Carga de tablas adicionales: `balance_sheet`, `income_statement`, `cash_flow`, `annual_invoices_comparisons`, `employees`, `superia_entity`, `xls_registro`, `csv_score`, `csv_vc_pyme`
  - Resumen de estructura (dimensiones y columnas) por tabla
  - Análisis de valores nulos por tabla y columna
  - Construcción de tabla maestra de clientes PAY via RFC / `profile_id`
  - Cobertura por fuente: muestra clientes encontrados y lista los no encontrados por nombre
  - Normalización de nombres para match con tablas de Buró (quita tildes, puntuación y sufijos legales)
  - Estadísticas descriptivas de variables clave de todas las fuentes
  - Columnas `5`, `6`, `14` de `csv_vc_pyme` referenciadas como texto
- `2026-04-06` — Cambios commiteados y push a GitHub (`main`)

---

## Avance del proyecto — 2026-03-27

### Estado actual

- `2026-03-26` — Estructura inicial del proyecto creada: `data/`, `codigo/`, `notas/`, `reporte/`, `docs/`
- `2026-03-27` — Convenciones definidas en `CLAUDE.md`: nombres de archivos en inglés, contenido en español
- `2026-03-27` — Repositorio git inicializado en la rama `main`

- `2026-03-27` — Base de datos cargada en `data/` (`Base de datos.xlsx`)

- `2026-03-27` — Definición del modelo: score continuo de probabilidad de incumplimiento para clientes actuales y prospectos
- `2026-03-27` — Plan de trabajo aprobado con 5 fases: EDA → Limpieza → Modelado → Evaluación → Reporte
- `2026-03-27` — Herramienta definida: Python con Jupyter Notebooks

### Decisiones de diseño

- **Variable objetivo:** probabilidad de incumplimiento (valor continuo entre 0 y 1)
- **Alcance:** aplica tanto para clientes actuales como para prospectos nuevos
- **Tipo de modelo:** clasificación con salida probabilística (score de riesgo)
- **Herramienta:** Python + Jupyter Notebooks

### Notebooks del proyecto

- `codigo/01-model-coding.ipynb` — Notebook de trabajo principal del proyecto (análisis de cosechas PAY como insumo complementario al modelo)
- `codigo/CosechasPAY.ipynb` — Notebook original de referencia / respaldo del análisis de cosechas, previo al inicio del proyecto

### Análisis de cosechas

El análisis de cosechas (vintages) para el producto PAY complementa el modelo predictivo. Calcula el % de capital vencido por mes de originación y MOB (Months on Books), y genera una matriz de cosechas visualizada como heatmap.

### Pendiente

- **Fase 1** — EDA: explorar `data/Base de datos.xlsx` e identificar variables disponibles (`codigo/01-model-coding.ipynb`)
- **Fase 2** — Limpieza y preparación de datos (`codigo/02-data-cleaning.ipynb`)
- **Fase 3** — Modelado: regresión logística, Random Forest, Gradient Boosting (`codigo/03-modeling.ipynb`)
- **Fase 4** — Evaluación y selección del modelo final (`codigo/04-evaluation.ipynb`)
- **Fase 5** — Reporte con resultados y recomendaciones (`reporte/model-results.md`)

