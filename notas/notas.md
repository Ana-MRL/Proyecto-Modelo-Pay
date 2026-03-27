# Notas del proyecto

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

### Pendiente

- **Fase 1** — EDA: explorar `data/Base de datos.xlsx` e identificar variables disponibles (`codigo/01-eda.ipynb`)
- **Fase 2** — Limpieza y preparación de datos (`codigo/02-data-cleaning.ipynb`)
- **Fase 3** — Modelado: regresión logística, Random Forest, Gradient Boosting (`codigo/03-modeling.ipynb`)
- **Fase 4** — Evaluación y selección del modelo final (`codigo/04-evaluation.ipynb`)
- **Fase 5** — Reporte con resultados y recomendaciones (`reporte/model-results.md`)

