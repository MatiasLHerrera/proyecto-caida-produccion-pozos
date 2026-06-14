# Instrucciones para Codex/Copilot

Este proyecto corresponde a un problema de clasificacion binaria supervisada para predecir `caida_critica` en el mes siguiente a nivel pozo-mes.

## Reglas metodologicas

- No generar modelos antes de revisar calidad de datos, target, granularidad y split temporal.
- No hacer EDA profundo antes de validar estructura, duplicados, faltantes y consistencia temporal.
- Usar siempre paths relativos.
- No incluir datos raw, datasets pesados ni artefactos de modelos en Git.
- No usar informacion futura para construir features.
- No calcular variables con datos posteriores al mes observado.
- No definir el target con informacion disponible en el mismo momento de prediccion si eso implica fuga temporal.
- Priorizar split temporal sobre splits aleatorios.
- Mantener la unidad de analisis como pozo-mes.
- Documentar supuestos relevantes en `docs/project_context.md` o `reports/final_report_notes.md`.

## Estilo de trabajo

- Escribir codigo solo cuando sea necesario.
- Mantener notebooks ordenados y con pasos reproducibles.
- Separar exploracion en notebooks y logica reutilizable en `src/`.
- Evitar cambios grandes no solicitados.
- Preferir soluciones simples, explicables y defendibles para una entrega academica.

## Orden sugerido

1. Calidad de datos.
2. EDA descriptivo.
3. Feature engineering temporal.
4. Modelado.
5. Resultados para reporte.
