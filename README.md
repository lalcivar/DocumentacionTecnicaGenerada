# Documentación técnica generada
# Resultados del Experimento: Generación Automática de Documentación Técnica con LLMs

Este repositorio contiene los resultados detallados de un experimento diseñado para evaluar la efectividad de una arquitectura híbrida en la generación automática de documentación técnica (Docstrings/Javadoc) a partir de código fuente, utilizando Modelos de Lenguaje de Gran Escala (LLMs).

## Descripción del Experimento

El estudio evaluó la capacidad de cuatro modelos comerciales de LLM para generar documentación para un corpus de código fuente bajo dos condiciones experimentales: con y sin contexto sintáctico extraído previamente.

*   **Corpus de Código Fuente:** 30 archivos seleccionados (5 por cada uno) de 6 repositorios de código abierto:
    *   **Python (3 repositorios):** `flask`, `requests`, `scikit-learn`
    *   **Java (3 repositorios):** `commons-lang`, `guava`, `junit5`
*   **Modelos de LLM Evaluados:**
    *   Google - Gemini
    *   Anthropic - Claude
    *   OpenAI - ChatGPT
    *   Meta - Llama
*   **Condiciones Experimentales:**
    1.  **Sin Contexto Sintáctico:** El LLM recibió el código fuente sin preprocesamiento estructural adicional.
    2.  **Híbrido AST/Regex:** El LLM recibió el código fuente junto con firmas y metadatos extraídos por un motor híbrido de análisis sintáctico (AST para Python, Regex para Java).
*   **Ejecuciones:** Se realizaron un total de 240 ejecuciones (30 archivos × 4 modelos × 2 condiciones), generando la documentación y métricas asociadas para cada combinación.
*   **Tipo de Documentación Generada:** Docstrings para Python y Javadoc para Java.

## Estructura de los Resultados Generados

La carpeta principal `ResultadoDelExperimento/` contiene los datos organizados por archivo original, modelo de LLM y condición experimental.

Dentro de `ResultadoDelExperimento/` encontrará una estructura de carpetas similar a la siguiente:

ResultadoDelExperimento/
├── init.Anthropic-Claude_Hibrido_AST_Regex/
│ ├── 01__init (Código fuente original)
│ ├── 02_documentacion_generada (Documentación generada por el LLM)
│ ├── 03_informe_de_metricas_ths (Reporte de Tasa de Alucinación Sintáctica)
│ └── 04_informe_de_metricas_nlp (Reporte de métricas NLP: BLEU, ROUGE, BERTScore)
├── init.Anthropic-_Claude_Sin_Contexto/
│ └── ...
├── init.Google-_Gemini_Hibrido_AST_Regex/
│ └── ...
├── ... (otras 237 carpetas con resultados por archivo/modelo/condición)
└── Tabla_Maestra_de_los_60_Artefactos.csv (Tabla con la selección de artefactos para evaluación experta)


Cada una de las 240 carpetas de resultados (`{nombre_archivo}_{modelo}_{condicion}/`) contiene los siguientes archivos clave para la evaluación de un artefacto:
*   `01_{nombre_archivo_original}`: El archivo de código fuente original (`.py` o `.java`).
*   `02_documentacion_generada`: El archivo de texto que contiene la documentación generada por el modelo bajo la condición específica.
*   `03_informe_de_metricas_ths`: Un archivo de texto con el resultado de la Tasa de Alucinación Sintáctica (THS) para este artefacto.
*   `04_informe_de_metricas_nlp`: Un archivo de texto con los resultados de las métricas de Procesamiento de Lenguaje Natural (BLEU, ROUGE, BERTScore) para este artefacto.

## Archivos de Resumen y Análisis

Además de las carpetas individuales, el directorio `ResultadoDelExperimento/` incluye los siguientes archivos de resumen clave:

*   `01_tabla_desempeno_modelos.csv`: Resultados agregados de rendimiento de los modelos.
*   `02_resultados_lote_masivo_GitHub.csv`: El log completo de las 240 ejecuciones.
*   `03_tabla_ablacion_motor_hibrido.csv`: Corresponde a la Tabla 7 del estudio, resumiendo el experimento de ablación.
*   `04_tablacaracterizacion_corpus.csv`: Corresponde a la Tabla 4 del estudio, caracterizando el corpus.
*   `07_tabla_metricas_por_perfil.csv`: Corresponde a la Tabla 9 del estudio, métricas automáticas por modelo, lenguaje y perfil.
*   `10_tabla_pruebas_estadisticas_friedman_wilcoxon.csv`: Corresponde a la Tabla 10 del estudio, resultados de pruebas estadísticas.
*   `11_tabla_valoracion_experta_de_la_utilidad.csv`: Corresponde a la Tabla 11 del estudio, valoración experta y THS.
*   `12_tabla_latencia_tokens_costo.csv`: Corresponde a la Tabla 12 del estudio, latencia y consumo de tokens/costo.
*   `13_tabla_costo_latencia_ths_recomendacion.csv`: Corresponde a la Tabla 13 del estudio, resumen de costos, latencia y THS.
*   `Tabla_Maestra_de_los_60_Artefactos.csv`: Archivo con la lista de los 60 artefactos seleccionados para la evaluación experta.
*   `ResultadoDeLaEvaluacionExperta.csv`: Archivo con los resultados consolidados de la evaluación por juicio experto.

## Cómo Utilizar

Para analizar los resultados:
1.  Explore las carpetas individuales para ver ejemplos de código fuente y documentación generada.
2.  Consulte los archivos CSV de resumen para obtener métricas agregadas y análisis estadísticos.

Para cualquier duda o aclaración sobre la metodología o los resultados, por favor, consulte el artículo de investigación asociado a este repositorio.
