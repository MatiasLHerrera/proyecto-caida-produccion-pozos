# Contexto del proyecto

## Problema

El proyecto busca anticipar caidas criticas de produccion mensual en pozos no convencionales de petroleo y gas. La prediccion temprana puede ayudar a priorizar monitoreo, diagnostico operativo y analisis tecnico de pozos con mayor riesgo de deterioro productivo.

## Unidad de analisis

La unidad de analisis sera el registro pozo-mes. Cada fila deberia representar un pozo en un mes calendario o periodo mensual equivalente.

## Target preliminar

El target esperado es `caida_critica`.

Definicion preliminar:

- `caida_critica = 1` si el pozo presenta una caida de produccion considerada critica en el mes siguiente.
- `caida_critica = 0` si no presenta esa caida critica en el mes siguiente.

La definicion final debe especificar:

- variable de produccion usada como referencia;
- umbral porcentual o absoluto de caida;
- tratamiento de meses con produccion nula o faltante;
- tratamiento de pozos nuevos, cierres, workovers u otros eventos operativos.

## Riesgo de data leakage

Este problema tiene alto riesgo de fuga de informacion temporal. Las variables predictoras de una fila pozo-mes solo pueden usar informacion disponible hasta ese mes, nunca datos del mes siguiente ni de meses posteriores.

Evitar especialmente:

- usar produccion futura para construir features;
- calcular medias, tendencias o normalizaciones con datos posteriores al mes observado;
- mezclar registros del mismo pozo entre train y test sin respetar el orden temporal;
- imputar valores usando informacion del futuro;
- crear features derivadas directa o indirectamente del target.

## Split temporal

La evaluacion debe realizarse con separacion temporal. Una estrategia inicial recomendada es entrenar con meses historicos y reservar los meses mas recientes para validacion o test.

Antes de modelar, se debe definir:

- fecha de corte de entrenamiento;
- periodo de validacion;
- periodo de test final;
- criterio para manejar pozos que aparecen en varios periodos.

El split temporal es necesario para simular el uso real del modelo: predecir eventos futuros a partir de informacion historica.
