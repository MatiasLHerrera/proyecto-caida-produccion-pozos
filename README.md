# Predicción de caída crítica de producción mensual en pozos no convencionales de petróleo y gas

Proyecto final del curso **Data Science I**, orientado a construir una solución de machine learning aplicada a datos de producción hidrocarburífera no convencional.

El objetivo es desarrollar una base de trabajo profesional, reproducible y extensible, utilizando buenas prácticas de análisis exploratorio, preprocesamiento, feature engineering, modelado y evaluación.

## Estado del proyecto

Proyecto en etapa inicial.

Actualmente se está preparando la estructura del repositorio, la documentación base y la metodología de trabajo. En las próximas etapas se incorporarán los datos, se realizará el análisis de calidad, el EDA y la construcción del dataset modelable.

## Objetivo

Predecir si un pozo no convencional tendrá una **caída crítica de producción en el mes siguiente**, utilizando información histórica a nivel **pozo-mes**.

La idea es formular el problema como una tarea de clasificación binaria, similar a un problema de churn, donde el evento positivo representa una caída relevante de producción.

## Tipo de problema

* Tipo de aprendizaje: supervisado.
* Tipo de problema: clasificación binaria.
* Unidad de análisis: pozo-mes.
* Variable objetivo esperada: `caida_critica`.

La variable `caida_critica` se definirá a partir de la variación porcentual de producción entre el mes actual y el mes siguiente. El umbral final de caída se seleccionará luego del análisis exploratorio de datos.

Ejemplo conceptual:

```text
caida_critica = 1 si la producción del mes siguiente cae más de X%
caida_critica = 0 si no ocurre esa caída
```

## Dataset

El dataset será incorporado en la carpeta `data/raw/`.

En esta etapa todavía resta documentar:

* fuente del dataset;
* período temporal cubierto;
* cantidad de registros;
* cantidad de pozos únicos;
* variables disponibles;
* granularidad real de los datos;
* diccionario de columnas;
* posibles limitaciones de calidad.

Los archivos de datos crudos no serán versionados en Git debido a su tamaño y/o naturaleza operativa.

## Estructura del repositorio

```text
proyecto-caida-produccion-pozos/
├── data/
│   ├── raw/
│   ├── processed/
│   └── external/
├── notebooks/
│   ├── 01_data_quality.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_feature_engineering.ipynb
│   ├── 04_modeling.ipynb
│   └── 05_results_for_report.ipynb
├── src/
│   ├── data/
│   ├── features/
│   ├── models/
│   └── visualization/
├── models/
├── reports/
│   ├── figures/
│   └── final_report_notes.md
├── docs/
│   └── project_context.md
├── prompts/
│   └── prompts_usados.md
├── requirements.txt
├── README.md
├── .gitignore
└── copilot-instructions.md
```

## Metodología general

1. Definir claramente el problema de negocio y la variable objetivo `caida_critica`.
2. Validar que el target represente una caída futura y no una condición del mismo mes.
3. Revisar calidad de datos, granularidad, duplicados, faltantes y consistencia temporal.
4. Realizar análisis exploratorio de datos sin utilizar información futura.
5. Analizar la distribución de variaciones mensuales para definir un umbral razonable de caída crítica.
6. Construir variables predictoras con rezagos, ventanas históricas y agregaciones disponibles hasta cada mes de observación.
7. Separar entrenamiento y evaluación mediante un split temporal.
8. Entrenar modelos de clasificación binaria.
9. Evaluar resultados con métricas adecuadas para clasificación.
10. Documentar resultados, limitaciones, supuestos e insights para el reporte final.

## Métricas previstas

Las métricas principales serán:

* ROC AUC;
* accuracy;
* precision;
* recall;
* F1-score;
* matriz de confusión;
* curva ROC.

Dado que el evento positivo representa una caída crítica de producción, se prestará especial atención al **recall de la clase positiva**, ya que un falso negativo implicaría no detectar un pozo con riesgo de caída relevante.

## Consideraciones metodológicas

Este proyecto prestará especial atención a tres riesgos:

### Data leakage

Las variables predictoras deberán construirse únicamente con información disponible hasta el mes actual. La información del mes siguiente solo podrá utilizarse para construir la variable objetivo.

### Temporalidad

Al tratarse de datos mensuales, la evaluación del modelo deberá respetar el orden temporal. Por eso, se priorizará un split temporal en lugar de una partición aleatoria tradicional.

### Desbalance de clases

Si la clase `caida_critica = 1` representa una proporción baja del dataset, se evaluará el desempeño del modelo con métricas más informativas que accuracy, como recall, precision, F1-score y ROC AUC.

## Próximos pasos

1. Incorporar los datos en `data/raw/` sin versionarlos en Git.
2. Documentar fuente, período y diccionario de datos.
3. Completar `01_data_quality.ipynb` con validaciones iniciales.
4. Confirmar la granularidad real del dataset.
5. Analizar la distribución de variaciones mensuales.
6. Definir formalmente el umbral de `caida_critica`.
7. Diseñar el split temporal antes de cualquier modelado.
8. Construir el dataset modelable en `data/processed/`.
