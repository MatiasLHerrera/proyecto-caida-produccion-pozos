# Predicción de caída crítica de producción mensual en pozos no convencionales de petróleo y gas

## 1. Título

**Predicción de caída crítica de producción mensual en pozos no convencionales de petróleo y gas**

Proyecto final de Data Science I orientado a la construcción de un flujo reproducible de machine learning para anticipar caídas críticas de producción en pozos no convencionales.

## 2. Resumen ejecutivo

Este proyecto aborda un problema de clasificación binaria supervisada cuyo objetivo es predecir si un pozo no convencional tendrá una caída crítica de producción en el mes calendario siguiente. La unidad de análisis utilizada es pozo-mes, por lo que cada registro representa el estado productivo de un pozo específico en un mes determinado.

Se construyó una variable objetivo llamada `caida_critica`, definida como una caída del 30% o más en la producción principal del pozo durante el mes calendario siguiente. Para evitar errores metodológicos, se utilizó split temporal, se excluyeron variables futuras del dataset de modelado y todo el preprocesamiento se integró dentro de pipelines.

Se evaluaron dos modelos: Logistic Regression como baseline y Random Forest como modelo no lineal inicial. El mejor desempeño correspondió a Random Forest, con ROC AUC de 0,667 y recall de clase positiva de 0,553. El modelo constituye una primera aproximación académica útil y reproducible, pero no una herramienta lista para operación. Todavía presenta limitaciones, especialmente por la baja precisión de la clase positiva y la presencia de falsos positivos.

## 3. Contexto del problema y origen del dataset

La producción mensual de pozos no convencionales puede presentar caídas abruptas asociadas a condiciones operativas, técnicas o propias de la dinámica productiva del pozo. Anticipar estos eventos permite pensar el problema como un sistema de alerta temprana, donde el objetivo no es solo explicar lo ocurrido, sino detectar señales previas a una caída futura relevante.

El dataset utilizado proviene de datos abiertos oficiales de Argentina y corresponde al recurso **“Producción de pozos de gas y petróleo no convencional”**. A partir de esta fuente se construyeron datasets procesados, features históricas y un dataset final de modelado. No se incorporaron datasets externos en esta primera versión del trabajo.

Link de referencia de la fuente oficial: [http://datos.energia.gob.ar/dataset/c846e79c-026c-4040-897f-1ad3543b407c/archivo/b5b58cdc-9e07-41f9-b392-fb9ec68b0725]

## 4. Descripción del dataset

La fuente original corresponde a datos abiertos oficiales de producción de pozos no convencionales de Argentina, específicamente al dataset **“Producción de pozos de gas y petróleo no convencional”**. Luego de las etapas de calidad de datos, EDA y feature engineering, el archivo final utilizado para modelado fue:

`data/processed/model_dataset.csv`

El dataset final contiene:

| Característica | Valor |
|---|---:|
| Registros | 319.453 |
| Columnas | 77 |
| Pozos únicos | 4.681 |
| Rango temporal modelable | Enero de 2006 a marzo de 2026 |
| Unidad de análisis | Pozo-mes |

El dataset incluye variables productivas, temporales, geográficas, técnicas, categóricas e históricas. La unidad de análisis es pozo-mes: cada fila representa un pozo en un mes calendario. Las columnas `idpozo` y `fecha_data` se conservaron para trazabilidad y split temporal, pero no se utilizaron como variables predictoras.

## 5. Variable objetivo

La variable objetivo del proyecto es `caida_critica`.

Su construcción se basó en la producción principal del pozo:

- Para pozos petrolíferos, se utilizó `prod_pet`.
- Para pozos gasíferos, se utilizó `prod_gas`.
- Se calculó la variación porcentual entre la producción principal del mes actual y la del mes calendario siguiente.
- `caida_critica = 1` si la producción principal cae un 30% o más en el mes calendario siguiente.
- `caida_critica = 0` si la caída futura es menor al 30%.

Para mejorar la consistencia temporal, el target se calculó únicamente cuando el siguiente registro observado correspondía exactamente al mes calendario siguiente. Las variables utilizadas para mirar el mes siguiente se usaron solo para construir el target y fueron excluidas como features del modelo.

Distribución final del target:

| Clase | Significado | Cantidad | Porcentaje |
|---|---|---:|---:|
| 0 | Sin caída crítica futura | 281.723 | 88,19% |
| 1 | Con caída crítica futura | 37.730 | 11,81% |

Esta distribución muestra un desbalance de clases: la clase positiva es minoritaria y representa el 11,81% de los registros. Por esta razón, durante el modelado se consideraron métricas como recall, precisión y F1-score, además de ROC AUC, y no se evaluó el desempeño únicamente con accuracy.

## 6. EDA

El análisis exploratorio de datos permitió identificar patrones relevantes antes del modelado. Se observó una fuerte concentración geográfica y productiva en Neuquén y en la Cuenca Neuquina. También se detectaron diferencias claras entre pozos petrolíferos y gasíferos, lo que justificó definir una variable de producción principal por pozo.

Principales hallazgos del EDA:

- Las variables `prod_pet`, `prod_gas` y `prod_agua` presentan distribuciones fuertemente sesgadas.
- Existen outliers importantes en las variables de producción.
- Hay presencia significativa de valores iguales a cero.
- Los ceros pueden responder a situaciones operativas, pozos parados, abandono, estudio, falta de extracción efectiva o pozos cuyo producto principal no coincide con esa variable.
- Los valores negativos detectados fueron casos aislados y deben revisarse cuidadosamente en futuras iteraciones.
- Existen pozos con historial corto y pozos con historial largo, lo cual impacta en la construcción de lags, medias móviles y variables históricas.
- El año 2026 aparece como año parcial dentro del dataset.

En calidad de datos, no se detectaron duplicados exactos ni duplicados por pozo-mes. Las columnas `vida_util` y `observaciones` presentaban alta nulidad y no se usaron como features. Los valores negativos en producción fueron pocos casos aislados, mientras que los ceros fueron relevantes y se analizaron por variable productiva y tipo de pozo.

Estos resultados mostraron que no era conveniente definir `caida_critica` directamente solo sobre petróleo o solo sobre gas para todos los pozos. Por esa razón, se utilizó la producción principal según el tipo de pozo.

Figura 1. Cobertura mensual del dataset.

![Cobertura mensual del dataset](reports/figures/eda_cobertura_mensual.png)

La figura resume la disponibilidad temporal de registros y permite identificar la evolución de la cobertura mensual.

Figura 2. Producción mensual total y pozos activos.

![Producción mensual total y pozos activos](reports/figures/eda_produccion_mensual_total.png)

La figura muestra la evolución agregada de la producción mensual y el crecimiento de pozos activos a lo largo del tiempo.

Figura 3. Producción por provincia.

![Producción por provincia](reports/figures/eda_produccion_por_provincia.png)

La figura permite observar la concentración geográfica de la producción, especialmente en las provincias más relevantes del dataset.

Figura 4. Producción por cuenca.

![Producción por cuenca](reports/figures/eda_produccion_por_cuenca.png)

La figura evidencia la concentración productiva por cuenca, con predominio de la Cuenca Neuquina.

Figura 5. Distribución logarítmica de variables productivas.

![Distribución logarítmica de producción](reports/figures/eda_10_hist_produccion_log1p.png)

La figura utiliza escala logarítmica para visualizar distribuciones fuertemente sesgadas y con outliers.

Figura 6. Producción igual a cero por variable.

![Producción igual a cero por variable](reports/figures/eda_10_ceros_por_variable.png)

La figura resume la frecuencia de ceros en las principales variables productivas.

Figura 7. Evaluación de umbrales de caída crítica.

![Evaluación de umbrales de caída crítica](reports/figures/eda_tabla_umbrales_caida.png)

La figura documenta la evaluación exploratoria de distintos umbrales para definir caídas críticas.

Figura 8. Umbrales sobre producción principal.

![Umbrales sobre producción principal](reports/figures/eda_umbrales_produccion_principal.png)

La figura apoya la decisión de construir el target usando la producción principal del pozo según su tipo.

## 7. Preprocesamiento y feature engineering

El feature engineering se desarrolló respetando la temporalidad del problema y evitando data leakage. Las variables predictoras fueron construidas usando información del mes actual o de meses anteriores, nunca información del mes siguiente.

Se crearon las siguientes familias de variables:

- Variables temporales: año, mes, trimestre, semestre, año-mes y codificación cíclica del mes.
- Antigüedad del pozo en meses.
- Ratios productivos, como gas/petróleo, agua/petróleo y agua/producción principal.
- Lags de producción por pozo.
- Variaciones históricas porcentuales.
- Medias móviles y desvíos móviles.
- Producciones acumuladas.
- Flags de producción igual a cero y tipo de pozo.

La prevención de data leakage se realizó de tres maneras principales: se usó split temporal en lugar de split aleatorio, se calcularon features históricas agrupando por pozo y ordenando por fecha, y se excluyeron explícitamente variables futuras o auxiliares utilizadas para construir el target, entre ellas:

- `produccion_principal_next`
- `fecha_siguiente_observada`
- `gap_meses_hasta_siguiente`
- `var_futura_principal_pct`
- `prod_pet_next`
- `prod_gas_next`

La imputación, el escalado y la codificación de variables categóricas se realizaron dentro de pipelines durante el modelado, para que las transformaciones se ajustaran con el conjunto de entrenamiento y luego se aplicaran al conjunto de test.

## 8. Modelos utilizados

Se evaluaron dos modelos sobre el mismo split temporal y el mismo conjunto de features:

**Logistic Regression**

Se utilizó como modelo baseline interpretable. El pipeline incluyó imputación, escalado de variables numéricas y codificación One Hot de variables categóricas. Se usó `class_weight="balanced"` debido al desbalance de clases.

**Random Forest**

Se utilizó como modelo no lineal inicial. También se integró dentro de un pipeline con el mismo preprocesamiento. Se usaron parámetros conservadores para reducir el riesgo de overfitting:

- `n_estimators=200`
- `max_depth=12`
- `min_samples_leaf=10`
- `random_state=42`
- `n_jobs=-1`
- `class_weight="balanced"`

El split utilizado fue temporal y no aleatorio. Esto es metodológicamente importante porque el objetivo del proyecto es predecir eventos futuros. Un split aleatorio podría mezclar meses antiguos y recientes, generando una evaluación demasiado optimista por contaminación temporal entre entrenamiento y test.

## 9. Evaluación del modelo

La comparación de modelos se realizó con las mismas métricas y sobre el mismo conjunto de test temporal.

| Modelo | ROC AUC | Accuracy | Precisión | Recall | F1-score |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 0,632 | 0,625 | 0,179 | 0,548 | 0,270 |
| Random Forest | 0,667 | 0,656 | 0,196 | 0,553 | 0,290 |

Además de la tabla agregada, se revisaron métricas equivalentes al `classification report`, especialmente precisión, recall y F1-score por clase. Dado el desbalance del target, se priorizó la lectura de las métricas de la clase positiva (`caida_critica = 1`). La accuracy por sí sola no alcanza para evaluar el modelo, ya que un clasificador podría obtener un valor aparentemente aceptable favoreciendo la clase mayoritaria.

La métrica principal utilizada para comparar modelos fue ROC AUC, complementada con recall de la clase positiva y F1-score. El recall de la clase positiva es especialmente relevante porque los falsos negativos representan caídas críticas reales que el modelo no logra anticipar.

El mejor modelo fue **Random Forest**, ya que obtuvo mejores valores de ROC AUC, accuracy, precisión y F1-score, manteniendo además un recall positivo levemente superior al baseline.

Figura 9. Curva ROC del modelo Random Forest.

![Curva ROC del modelo Random Forest](reports/figures/model_rf_roc_curve.png)

La curva ROC resume la capacidad del modelo para ordenar correctamente casos positivos y negativos según su probabilidad estimada.

## 10. Interpretación de errores

Para el modelo Random Forest, la matriz de confusión en test fue:

| Clase real / predicha | Predice 0 | Predice 1 |
|---|---:|---:|
| Real 0 | 89.056 | 43.621 |
| Real 1 | 8.605 | 10.658 |

Los falsos positivos corresponden a casos en los que el modelo predijo una caída crítica, pero esta no ocurrió. En una aplicación operativa, estos casos podrían generar alertas innecesarias.

Los falsos negativos corresponden a casos en los que sí ocurrió una caída crítica, pero el modelo no la anticipó. Estos errores son especialmente relevantes para el problema, porque implican no detectar a tiempo un evento productivo negativo. Por este motivo, el recall de la clase positiva es una métrica prioritaria.

El modelo Random Forest logra detectar algo más de la mitad de las caídas críticas reales, pero todavía genera muchas falsas alarmas. Esto se refleja en una precisión baja para la clase positiva.

Figura 10. Matriz de confusión del modelo Random Forest.

![Matriz de confusión del modelo Random Forest](reports/figures/model_rf_confusion_matrix.png)

La matriz de confusión permite visualizar los aciertos y errores del mejor modelo, incluyendo falsos positivos y falsos negativos.

## 11. Interpretabilidad

La interpretabilidad del modelo se analizó mediante la importancia de variables del Random Forest. Las variables más importantes incluyeron señales productivas, históricas, variabilidad reciente, acumulados, variables técnicas y contexto geográfico.

Entre las variables destacadas aparecieron:

- `tef`
- `produccion_principal_std_3`
- `coordenaday`
- `produccion_principal_std_6`
- `anio`
- `prod_gas_acum`
- `var_principal_1m_pct`
- `coordenadax`
- `produccion_principal_acum`
- `var_gas_1m_pct`

La variable `tef` aparece como una de las variables importantes del modelo, pero su interpretación debe hacerse con cautela y de acuerdo con el diccionario de datos original. No debe interpretarse de forma aislada ni como una explicación causal directa.

Esta composición es coherente con el objetivo del proyecto, ya que una caída crítica futura puede estar relacionada con el comportamiento reciente del pozo, su nivel de producción, su madurez, su localización y sus condiciones técnicas. Sin embargo, la importancia de variables no implica causalidad: indica asociación predictiva dentro del modelo entrenado, no una relación causal demostrada.

Figura 11. Importancia de variables del modelo Random Forest.

![Importancia de variables del modelo Random Forest](reports/figures/rf_feature_importance_top20.png)

La figura muestra las veinte variables con mayor importancia relativa dentro del Random Forest. Su lectura debe realizarse como interpretación predictiva y no causal.

## 12. Conclusiones

El proyecto logró construir un flujo completo y reproducible para anticipar caídas críticas de producción mensual en pozos no convencionales. Se definió correctamente la unidad de análisis pozo-mes, se construyó un target consistente con la temporalidad del problema y se evitaron variables con data leakage.

Random Forest obtuvo el mejor desempeño general y fue seleccionado como modelo final de esta primera iteración. El resultado es útil como aproximación académica y técnica, ya que valida el proceso completo desde la calidad de datos hasta el modelado y la interpretación de resultados.

No obstante, el modelo no debe interpretarse como una solución operativa final. La baja precisión de la clase positiva y la presencia de falsos positivos muestran que todavía se requieren mejoras antes de un uso práctico.

## 13. Limitaciones

Las principales limitaciones del proyecto son:

- La clase positiva representa solo el 11,81% del dataset final, por lo que existe desbalance.
- La precisión del mejor modelo es baja.
- El modelo genera una cantidad importante de falsos positivos.
- El umbral de caída crítica del 30% fue elegido como criterio inicial y debería validarse con conocimiento experto.
- La importancia de variables del Random Forest no implica causalidad.
- No se realizó una optimización exhaustiva de hiperparámetros.
- No se implementó validación temporal por múltiples ventanas.
- No se incorporaron datasets externos en esta primera versión.

## 14. Recomendaciones futuras

Para próximas iteraciones se recomienda:

- Ajustar el umbral de decisión del modelo según el costo relativo de falsos positivos y falsos negativos.
- Probar modelos adicionales, como Gradient Boosting, XGBoost, LightGBM o CatBoost.
- Implementar validación temporal por ventanas.
- Analizar el desempeño por segmentos, como tipo de pozo, cuenca, provincia, empresa o tipo de recurso.
- Evaluar técnicas de calibración de probabilidades.
- Revisar la importancia de variables con métodos complementarios, como permutation importance o SHAP.
- Validar la definición de caída crítica con expertos del dominio.
- Monitorear drift temporal si el modelo se utilizara en un entorno productivo.

## 15. Figuras sugeridas

Las figuras principales ya fueron insertadas en las secciones correspondientes. Para el armado final en Google Docs, se recomienda conservar especialmente las siguientes:

1. `reports/figures/eda_cobertura_mensual.png`
   - Cobertura temporal y cantidad de registros por mes.

2. `reports/figures/eda_produccion_mensual_total.png`
   - Evolución mensual de producción total y pozos activos.

3. `reports/figures/eda_produccion_por_provincia.png`
   - Concentración geográfica por provincia.

4. `reports/figures/eda_produccion_por_cuenca.png`
   - Concentración productiva por cuenca.

5. `reports/figures/eda_10_hist_produccion_log1p.png`
   - Distribuciones sesgadas de variables productivas.

6. `reports/figures/eda_10_ceros_por_variable.png`
   - Presencia de producción igual a cero.

7. `reports/figures/eda_tabla_umbrales_caida.png`
   - Evaluación exploratoria de umbrales de caída crítica.

8. `reports/figures/eda_umbrales_produccion_principal.png`
   - Comparación para justificar el uso de producción principal por pozo.

9. `reports/figures/model_logreg_confusion_matrix.png`
   - Matriz de confusión del baseline.

10. `reports/figures/model_rf_confusion_matrix.png`
    - Matriz de confusión del mejor modelo.

11. `reports/figures/model_logreg_roc_curve.png`
    - Curva ROC del baseline.

12. `reports/figures/model_rf_roc_curve.png`
    - Curva ROC del mejor modelo.

13. `reports/figures/rf_feature_importance_top20.png`
    - Top 20 variables más importantes del Random Forest.

No se identificaron figuras pendientes entre las solicitadas para esta versión del documento.

## 16. Checklist de cumplimiento de consigna

| Requisito | Cumple | Comentario | Acción pendiente |
|---|---|---|---|
| Documento autocontenido | Sí | El documento incluye contexto, datos, target, metodología, modelos, métricas, errores, limitaciones y recomendaciones. | Ninguna. |
| Contexto del problema | Sí | Se explica la necesidad de anticipar caídas críticas de producción mensual en pozos no convencionales. | Ninguna. |
| Origen del dataset | Sí | Se indica que la fuente proviene de datos abiertos oficiales de producción de pozos no convencionales de Argentina. | Ninguna. |
| Descripción del dataset | Sí | Se informa archivo final, cantidad de registros, columnas, pozos únicos, rango temporal y unidad de análisis. | Ninguna. |
| Variable objetivo | Sí | Se define `caida_critica` como caída del 30% o más en el mes calendario siguiente sobre producción principal. | Ninguna. |
| Balance del target | Sí | Se reporta la distribución: 88,19% clase 0 y 11,81% clase 1. | Ninguna. |
| EDA con visualizaciones | Sí | Se resumen hallazgos del EDA y se insertan las figuras principales con rutas relativas a `reports/figures/`. | Ninguna. |
| Missingness y calidad de datos | Sí | Se documenta que no hubo duplicados exactos ni duplicados pozo-mes, que `vida_util` y `observaciones` tenían alta nulidad, que los negativos fueron pocos casos aislados y que los ceros fueron relevantes. | Ninguna. |
| Preprocesamiento | Sí | Se documenta imputación, escalado y codificación categórica dentro de pipelines. | Ninguna. |
| Feature engineering | Sí | Se describen variables temporales, ratios, lags, variaciones históricas, medias móviles, acumulados y flags. | Ninguna. |
| Pipeline | Sí | Se aclara que el preprocesamiento y los modelos se integraron en pipelines reproducibles. | Ninguna. |
| Modelos utilizados | Sí | Se utilizaron Logistic Regression como baseline y Random Forest como modelo no lineal inicial. | Ninguna. |
| Métricas de clasificación | Sí | Se reportan ROC AUC, accuracy, precisión, recall y F1-score para ambos modelos. | Ninguna. |
| Matriz de confusión | Sí | Se incluye la matriz de confusión del Random Forest en tabla y figura embebida. | Ninguna. |
| Curva ROC | Sí | Se reporta ROC AUC y se inserta la curva ROC del Random Forest. | Ninguna. |
| Conclusiones | Sí | Se incluye una sección de conclusiones con alcance académico y lectura general del resultado. | Ninguna. |
| Data leakage | Sí | Se explica la exclusión de variables futuras, el uso de split temporal y el cálculo de features históricas ordenadas por pozo y fecha. | Ninguna. |
| Desbalance de clases | Sí | Se explicita que la clase positiva representa el 11,81% y que por eso no alcanza con evaluar accuracy. | Ninguna. |
| Overfitting o limitaciones del tuning | Sí | Se mencionan parámetros conservadores, falta de optimización exhaustiva de hiperparámetros y necesidad de validación temporal adicional. | Ninguna. |
| Temporalidad y split temporal | Sí | Se aclara que se usó split temporal, no aleatorio, para evitar contaminación entre pasado y futuro. | Ninguna. |
