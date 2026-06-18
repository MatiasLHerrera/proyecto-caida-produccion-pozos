# Notas finales para Google Docs

## Titulo del proyecto

**Prediccion de caida critica de produccion mensual en pozos no convencionales de petroleo y gas**

## Conclusion ejecutiva breve

El proyecto construye una primera solucion reproducible de machine learning para anticipar caidas criticas de produccion mensual en pozos no convencionales. El problema fue formulado como clasificacion binaria a nivel pozo-mes. El mejor modelo evaluado fue Random Forest, con ROC AUC de 0,667 y recall de la clase positiva de 0,553. El modelo es util como primera aproximacion academica y tecnica, aunque todavia requiere mejoras antes de pensarse como herramienta operativa, especialmente por la baja precision de la clase positiva y la presencia de falsos positivos.

## Objetivo

El objetivo del proyecto es predecir si un pozo no convencional tendra una caida critica de produccion en el mes siguiente. La unidad de analisis es pozo-mes, por lo que cada registro representa la situacion de un pozo en un mes determinado.

El problema se definio como una tarea de clasificacion binaria supervisada:

- `caida_critica = 1`: el pozo presenta una caida critica en el mes siguiente.
- `caida_critica = 0`: el pozo no presenta una caida critica en el mes siguiente.

## Dataset

Se trabajo con el dataset de produccion mensual de pozos no convencionales, procesado hasta generar el archivo final:

`data/processed/model_dataset.csv`

El dataset final contiene:

- Registros: 319.453.
- Columnas: 77.
- Pozos unicos: 4.681.
- Rango temporal modelable: enero de 2006 a marzo de 2026.
- Unidad de analisis: pozo-mes.

El dataset final incluye variables productivas, temporales, geograficas, tecnicas, categoricas e historicas. Las columnas de identificacion, como `idpozo` y `fecha_data`, se conservaron para trazabilidad, pero no se usaron como features del modelo.

## Variable objetivo

La variable objetivo del proyecto es `caida_critica`.

Se definio de la siguiente manera:

- Para pozos petroliferos, se uso `prod_pet` como produccion principal.
- Para pozos gasiferos, se uso `prod_gas` como produccion principal.
- Se calculo la variacion porcentual de la produccion principal contra el mes siguiente.
- `caida_critica = 1` cuando la produccion principal cae un 30% o mas en el mes siguiente.
- `caida_critica = 0` cuando la caida futura es menor al 30%.

Para mejorar la consistencia temporal, el target se calculo solo cuando el registro siguiente observado correspondia exactamente al mes calendario siguiente. Las variables futuras usadas para construir el target no fueron utilizadas como features.

Distribucion final del target:

| Clase | Significado | Cantidad | Porcentaje |
|---|---|---:|---:|
| 0 | Sin caida critica futura | 281.723 | 88,19% |
| 1 | Con caida critica futura | 37.730 | 11,81% |

## EDA

El analisis exploratorio mostro una fuerte concentracion geografica y productiva en Neuquen y en la Cuenca Neuquina. Tambien se observaron diferencias claras entre pozos petroliferos y gasiferos, lo que justifico construir el target usando una variable de produccion principal por tipo de pozo.

Principales hallazgos:

- Las variables `prod_pet`, `prod_gas` y `prod_agua` presentan distribuciones fuertemente sesgadas.
- Existen outliers relevantes en las variables de produccion.
- Hay presencia importante de ceros, que pueden deberse a causas operativas, pozos parados, abandono, estudio, ausencia de extraccion efectiva o pozos cuyo producto principal no coincide con esa variable.
- Los valores negativos detectados fueron casos aislados y deben tratarse con cuidado en iteraciones futuras.
- Existen pozos con historial corto y otros con historial largo, lo que afecta la construccion de lags y medias moviles.
- Se observo que 2026 es un anio parcial dentro del dataset.

## Feature engineering

El feature engineering se realizo respetando la temporalidad del problema y evitando data leakage. Las features usan informacion del mes actual o de meses anteriores, nunca informacion del mes siguiente.

Variables creadas:

- Variables temporales: anio, mes, trimestre, semestre, anio-mes y codificacion ciclica del mes.
- Antiguedad del pozo en meses.
- Ratios productivos, como gas/petroleo, agua/petroleo y agua/produccion principal.
- Lags de produccion por pozo.
- Variaciones historicas porcentuales.
- Medias moviles y desvios moviles.
- Acumulados productivos.
- Flags de produccion cero y tipo de pozo.

Las variables futuras o auxiliares usadas para construir el target fueron excluidas del dataset de modelado, por ejemplo:

- `produccion_principal_next`
- `fecha_siguiente_observada`
- `gap_meses_hasta_siguiente`
- `var_futura_principal_pct`
- `prod_pet_next`
- `prod_gas_next`

## Modelos

Se evaluaron dos modelos con el mismo split temporal y el mismo conjunto de features:

1. **Logistic Regression**
   - Se uso como baseline interpretable.
   - Incluyo preprocesamiento dentro de un Pipeline.
   - Uso `class_weight="balanced"` por el desbalance de clases.

2. **Random Forest**
   - Se uso como modelo no lineal inicial.
   - Incluyo el mismo preprocesamiento dentro de un Pipeline.
   - Uso parametros conservadores para reducir riesgo de overfitting:
     - `n_estimators=200`
     - `max_depth=12`
     - `min_samples_leaf=10`
     - `random_state=42`
     - `n_jobs=-1`
     - `class_weight="balanced"`

El split utilizado fue temporal, no aleatorio. Esto es importante porque el problema busca predecir eventos futuros y un split aleatorio podria mezclar meses antiguos y recientes, generando una evaluacion demasiado optimista.

## Metricas

Tabla final de metricas:

| Modelo | ROC AUC | Accuracy | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 0,632 | 0,625 | 0,179 | 0,548 | 0,270 |
| Random Forest | 0,667 | 0,656 | 0,196 | 0,553 | 0,290 |

La metrica principal para comparar modelos fue ROC AUC, complementada con recall de la clase positiva y F1-score. El recall de clase positiva es especialmente relevante porque los falsos negativos representan caidas criticas reales que el modelo no logra anticipar.

## Mejor modelo

El mejor modelo fue **Random Forest**.

Resultados principales:

- ROC AUC: 0,667.
- Accuracy: 0,656.
- Precision: 0,196.
- Recall: 0,553.
- F1-score: 0,290.

Random Forest mejora a Logistic Regression en ROC AUC, accuracy, precision y F1-score, y mantiene un recall positivo levemente superior. Por ese motivo fue seleccionado como mejor pipeline y guardado en:

`models/modelo_clasificacion_caida_critica.joblib`

## Interpretacion de falsos positivos y falsos negativos

Para el modelo Random Forest, la matriz de confusion en test fue:

| Clase real / predicha | Predice 0 | Predice 1 |
|---|---:|---:|
| Real 0 | 89.056 | 43.621 |
| Real 1 | 8.605 | 10.658 |

Interpretacion:

- **Falsos positivos:** 43.621 casos fueron clasificados como caida critica, pero no tuvieron caida critica real. En una aplicacion operativa, esto implicaria generar alertas innecesarias.
- **Falsos negativos:** 8.605 casos tuvieron caida critica real, pero el modelo los clasifico como no criticos. Estos casos son especialmente importantes porque representan eventos no anticipados.

En este problema, el recall de la clase positiva es una metrica prioritaria, ya que perder una caida critica puede ser mas costoso que generar una falsa alarma. Sin embargo, la baja precision indica que el modelo todavia genera muchas alertas falsas.

## Conclusiones

El proyecto logro construir un flujo completo y reproducible de machine learning para anticipar caidas criticas de produccion en pozos no convencionales. Se definio correctamente la unidad de analisis pozo-mes, se construyo un target temporalmente consistente y se evitaron variables con leakage.

El modelo Random Forest obtuvo el mejor desempeno general y funciona como una primera aproximacion razonable. Las variables mas importantes del modelo incluyen senales productivas, historicas, variabilidad reciente, acumulados, variables tecnicas y contexto geografico. Esto es coherente con el problema, ya que las caidas de produccion suelen estar relacionadas con el comportamiento previo del pozo.

Aun asi, el desempeno no debe interpretarse como definitivo. La precision de la clase positiva es baja y existe desbalance de clases, por lo que el modelo requiere mejoras antes de una aplicacion practica.

## Limitaciones

Principales limitaciones del trabajo:

- La clase positiva representa solo el 11,81% del dataset, por lo que existe desbalance.
- La precision del mejor modelo es baja, con presencia importante de falsos positivos.
- El umbral de caida critica del 30% fue elegido como criterio inicial y deberia validarse con conocimiento experto.
- La importancia de variables del Random Forest no implica causalidad.
- No se realizo optimizacion exhaustiva de hiperparametros.
- No se implemento validacion temporal por multiples ventanas.
- No se incorporaron datasets externos en esta primera version.

## Proximos pasos

Recomendaciones para futuras iteraciones:

- Ajustar el umbral de decision del modelo segun el costo relativo de falsos positivos y falsos negativos.
- Probar modelos adicionales, como Gradient Boosting, XGBoost, LightGBM o CatBoost.
- Realizar validacion temporal por ventanas.
- Analizar desempeno por segmentos: tipo de pozo, cuenca, provincia, empresa o tipo de recurso.
- Evaluar tecnicas de calibracion de probabilidades.
- Revisar importancia de variables con metodos complementarios, como permutation importance o SHAP.
- Validar la definicion de caida critica con expertos del dominio.
- Monitorear drift temporal si el modelo se usara en produccion.

## Figuras recomendadas para Google Docs

Figuras sugeridas para insertar en el informe:

1. `reports/figures/eda_cobertura_mensual.png`
   - Muestra la cobertura temporal y cantidad de registros por mes.

2. `reports/figures/eda_produccion_mensual_total.png`
   - Resume la evolucion mensual de produccion total y pozos activos.

3. `reports/figures/eda_produccion_por_provincia.png`
   - Muestra concentracion geografica por provincia.

4. `reports/figures/eda_produccion_por_cuenca.png`
   - Muestra concentracion por cuenca.

5. `reports/figures/eda_10_hist_produccion_log1p.png`
   - Ilustra la distribucion sesgada de variables productivas.

6. `reports/figures/eda_10_ceros_por_variable.png`
   - Resume presencia de produccion igual a cero.

7. `reports/figures/eda_tabla_umbrales_caida.png`
   - Documenta la evaluacion exploratoria de umbrales de caida critica.

8. `reports/figures/eda_umbrales_produccion_principal.png`
   - Justifica el uso de produccion principal por pozo.

9. `reports/figures/model_logreg_confusion_matrix.png`
   - Matriz de confusion del baseline.

10. `reports/figures/model_rf_confusion_matrix.png`
    - Matriz de confusion del mejor modelo.

11. `reports/figures/model_logreg_roc_curve.png`
    - Curva ROC del baseline.

12. `reports/figures/model_rf_roc_curve.png`
    - Curva ROC del mejor modelo.

13. `reports/figures/rf_feature_importance_top20.png`
    - Top 20 variables mas importantes del Random Forest.
