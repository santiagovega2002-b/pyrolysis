Biomass Pyrolysis — Liquid Yield Prediction
Qué es el proyecto

Modelo de regresión para predecir el rendimiento líquido (Liquid phase, %) en pirólisis de biomasa lignocelulósica a partir de composición elemental, análisis próximo y variables de proceso.

El problema

El objetivo fue predecir el porcentaje de fase líquida generado durante la pirólisis. Este problema es complejo porque el rendimiento líquido depende simultáneamente de la composición estructural de la biomasa, del contenido de volátiles y carbono fijo y de parámetros operativos como temperatura final y tasa de calentamiento. Además, el dataset proviene de múltiples estudios con protocolos heterogéneos, lo que introduce variabilidad experimental real e inconsistencias esperables en unidades o bases de reporte.

Los datos

Fuente: Kaggle — Biomass Pyrolysis Data
https://www.kaggle.com/datasets/mustafakeser4/biomass-pyrolysis-data
![Gráfico de Van Krevelen](image-2.png)

Compilación basada en Dong et al. (Beijing Forestry University), revisión de múltiples estudios de pirólisis de biomasa lignocelulósica.

Dimensión original: 751 filas × 17 columnas
Dimensión final limpia: 512 filas × 18 columnas

Variables incluidas: composición elemental (C, H, O, N), análisis próximo (M, Ash, VM, FC), tamaño de partícula (PS), temperatura final (FT), heating rate (HR), flujo de gas (FR) y rendimientos de fase (Solid, Liquid, Gas).
Variable target: Liquid phase (%)

Lo que hice

Primero realicé una auditoría estructural completa del dataset. Se analizaron tipos de datos, valores faltantes y coherencia fisicoquímica. Se evaluó el cierre de masa entre fases sólido/líquido/gas y se identificaron inconsistencias atribuibles a la naturaleza compilada del dataset.

En segundo lugar, se aplicó una limpieza basada en criterios físicos y no solo estadísticos. Se eliminaron registros con humedad negativa y balances de fase extremos, reduciendo el dataset de 751 a 512 observaciones. Se recalculó VM cuando fue necesario mediante balance de masa y se creó la variable FR_disponible para preservar información sobre mediciones imputadas, considerando la heterogeneidad de protocolos experimentales.

Por último, se construyeron variables derivadas con fundamento termoquímico (ratios elementales y variables de interacción relevantes) y se entrenaron dos modelos: una regresión lineal como baseline y un XGBoostRegressor como modelo principal. Se utilizó train/test split y validación cruzada para evaluar estabilidad y riesgo de sobreajuste.

Resultados

Baseline — Regresión Lineal
R² = 0.496
MAE = 6.207

Modelo final — XGBoost
Test R² = 0.879
Test MAE = 2.560
Train R² = 0.994
![Predicho vs Real — XGBoost](image.png)
Validación cruzada (5 folds)
R² promedio = 0.835 ± 0.033

Features más importantes: VM (volatile matter), N (nitrógeno) y FC (fixed carbon).
FR_disponible apareció en el top 5, validando su inclusión como variable de trazabilidad.
![Importancia de los Features](image-1.png)

El modelo no lineal capturó interacciones entre composición y condiciones de proceso que la regresión lineal no pudo representar adecuadamente.

Cómo correr el código

Orden recomendado de ejecución:

01_EDA_baseline.ipynb

02_cleaning.ipynb

03_features.ipynb

04_modeling.ipynb

El dataset limpio se exporta desde 02_cleaning.ipynb.
El modelo final se entrena y guarda en 04_modeling.ipynb.


Biomasa “óptima” — interpretación

El modelo converge hacia una biomasa con H/C = 1.170, lo que químicamente implica mayor insaturación y carácter aromático, condición que en principio favorecería formación de char más que de bio-oil. Sin embargo, esa desventaja estructural se ve compensada por un VM = 78.1%, que incrementa la fracción susceptible de despolimerización primaria y generación de vapores condensables. La solución refleja una tensión fisicoquímica real: la composición elemental no es ideal para líquido, pero la alta fracción volátil domina el balance global de fases.

La temperatura óptima estimada (557 °C) se ubica muy cerca del máximo teórico previamente calculado (568 °C) mediante un ajuste parabólico independiente. La convergencia de ambos enfoques constituye una validación cruzada robusta del rango térmico óptimo para maximizar líquidos.

Es importante enfatizar que esta biomasa es una solución matemática dentro del espacio del modelo; no implica que exista en la naturaleza exactamente con esa combinación composicional. Debe interpretarse como una guía para diseño experimental y screening dirigido, no como una receta directa de formulación