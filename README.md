# Machine Learning y Econometría - Panel EPS Chile

## Descripción del Proyecto

Serie de tres tareas que aplican técnicas econométricas y de machine learning sobre datos panel longitudinales de la **Encuesta de Protección Social (EPS)** de Chile, con aproximadamente ~77.000 observaciones. El objetivo transversal es modelar la **expectativa de vida subjetiva** (`expectancy`) en función de variables socioeconómicas, laborales y de salud.

***

## Estructura del Repositorio

| Notebook | Descripción |
|----------|-------------|
| `Tarea-1-Alan-Wilson.ipynb` | EDA completo + modelos de conteo (Poisson, Binomial Negativa, Probit, Logit) |
| `TAREA_2_ML_Alan_Wilson.ipynb` | Imputación avanzada del panel + modelos de regresión (Pooled OLS) |
| `ML-TAREA_3_FINAL.ipynb` | Modelos de datos panel: Efectos Fijos (FE), Efectos Aleatorios (RE) y CRE |

***

## Tarea 1 — EDA y Modelos de Conteo

**Dataset:** datos climáticos y de sensores con variable dependiente binaria `FailureToday` (fallo detectado por sensor).  
**Observaciones:** ~117.793

### Metodología

1. **Limpieza y EDA:** mapa de calor de valores nulos, estadísticas descriptivas, matriz de correlaciones (variables no binarias), boxplots de outliers.
2. **Modelo de Probabilidad Lineal (OLS):** R² = 0,275. Variables significativas: `MinTemp`, `P1Speed`, `P39am`, `P49am`, `P43pm` (positivas); `Evaporation`, `P33pm`, `P59am` (negativas).
3. **Modelo Probit:** Pseudo R² = 0,304. Mismas variables robustas, con efectos marginales más grandes.
4. **Modelo Logit:** Pseudo R² = 0,306. Resultados similares al Probit; permite interpretación en términos de *odds ratio*.
5. **Modelo Poisson (datos agregados mensualmente):** Pseudo R² CS = 0,855. Variables robustas: `P1Speed`, `P43pm`, `P49am`, `P33pm`, `P59am`.
6. **Diagnóstico de sobredispersión** mediante regresión auxiliar (α estimado positivo y significativo → sobredispersión presente).
7. **Modelo Binomial Negativa:** menor ajuste que Poisson, pero coeficientes muy similares → Poisson es más parsimonioso en este caso.

### Conclusión metodológica

El modelo **Logit** es el más adecuado para la variable binaria (fallo diario), por su interpretabilidad vía *odds ratios*. Para conteos mensuales, el modelo **Poisson** es preferible dado que la sobredispersión es leve.

***

## Tarea 2 — Imputación Avanzada y Pooled OLS

**Dataset:** `paneleps.csv` — datos panel EPS Chile (~96.846 filas en carga inicial).  
**Variable dependiente:** `expectancy` (expectativa de vida subjetiva, escala 0–150).

### Preprocesamiento

- Eliminación de columnas completamente vacías (`fondoa`–`fondoe`) y variables redundantes (`selfemp`).
- Imputación de `sistema` con reglas basadas en cotización, activos previsionales e informalidad.
- `expectancy`: valores > 150 convertidos a NaN; imputación con siguiente/anterior valor del mismo individuo.
- Variables laborales (`size`, `occupation`, `wage`, `hours`, `informal`, `publicemp`): imputadas con 0 para inactivos.
- Variables de salud (`illness`, `cronica`, `nocronica`, `mental`): relleno hacia adelante por individuo (`bfill`).
- `assets`: interpolación lineal por individuo.
- `region`: relleno hacia adelante y hacia atrás por individuo.
- Eliminación de outliers extremos (percentil 95 superior en `hours`).
- Creación de variables dummy para `status`, `situation`, `region`, año y `children` (agrupado).
- Estructura panel indexada por `folion20` y `time`.

### Modelos estimados

- **Pooled OLS** con errores estándar robustos (HC0).

***

## Tarea 3 — Modelos de Datos Panel

**Variable dependiente:** `expectancy`  
**Identificador de individuo:** `folion20` · **Período:** `time`

### Modelos estimados y resultados

| Modelo | R² within | R² between | R² overall | Observaciones |
|--------|:---------:|:----------:|:----------:|---------------|
| Pooled OLS | — | — | — | Baseline sin efectos individuales |
| Efectos Fijos (FE) | ~0.08 | — | — | Controla heterogeneidad no observada constante |
| Efectos Aleatorios (RE) | — | — | — | Asume incorrelación entre efectos y regresores |
| **CRE (Correlated Random Effects)** | — | — | **0.505** | **Modelo preferido** |

### Conclusión metodológica

El modelo **CRE** es el preferido: combina la consistencia de FE con la eficiencia de RE, permite incluir variables invariantes en el tiempo y captura la correlación entre los efectos individuales no observados y los regresores. R² overall = **0,505**.

