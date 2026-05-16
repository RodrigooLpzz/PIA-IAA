# PIA: Introducción al Aprendizaje Automático
**Modelo Predictivo de Goles Esperados (xG) en la Premier League**

## Sobre el Conjunto de Datos
Este proyecto utiliza el dataset `epl_2024_2025_match_shots.csv` (10MB) que contiene aproximadamente 9,000 registros correspondientes a **todos los tiros realizados en los partidos de la English Premier League (EPL)** durante la actual temporada.

### Origen de los Datos: Proyecto Personal (SportsAnalytics)
La obtención de estos datos no proviene de repositorios genéricos, sino de un esfuerzo de **Data Engineering (MLOps)** desarrollado inicialmente en el repositorio personal [SportsAnalytics](https://github.com/diegoaislasc/SportsAnalytics). 
La arquitectura técnica subyacente para construir este conjunto de datos consistió en:
1. **Scraping automatizado:** Uso de la librería de Python `ScraperFC` para extraer IDs de los partidos de la API oculta de **Sofascore**.
2. **Extracción y anidación:** Iteración sobre los IDs para extraer la telemetría masiva de cada tiro y jugada.
3. **Carga a Data Warehouse:** Consolidación de la información en una tabla centralizada en **Google BigQuery** (GCP).
De esa base de datos robusta de BigQuery, se extrajo una instantánea en CSV para trabajar localmente.

### Escalamiento Académico (Minería de Datos a Machine Learning)
Este dataset de telemetría deportiva pasó por distintas fases académicas para llegar a su madurez actual:

1. **Aportación de la materia de Minería de Datos:**
   Durante las prácticas de dicha materia, los datos brutos extraídos del proyecto personal fueron sometidos a procesos de transformación. Se utilizaron técnicas de *Data Cleaning* y manejo de calidad de datos (como el desempaquetado de las estructuras JSON de `playerCoordinates` para obtener columnas como `shot_x` y `shot_y`, y conversiones de variables de fecha/hora). Esto nos brindó un dataset estructurado.

2. **Aplicación para el PIA de Machine Learning (Propósito actual):**
   Gracias a que el dataset heredado cuenta con características tabulares de alta dimensionalidad espacial y categórica, resulta el recurso perfecto para aplicar las metodologías centrales de esta materia. Haber superado la etapa de recolección y limpieza nos permite enfocarnos al 100% en:
   * **Modelado Predictivo:** Uso de clasificadores para discernir fronteras no lineales.
   * **Estrategias de Evaluación:** Manejo estratégico del desbalance de clases (los goles representan ~10% de los datos).

---

## Metodología y Técnicas de Machine Learning (Fases del Proyecto)

El objetivo central del PIA es determinar la probabilidad de que un tiro se convierta en Gol (xG) basándose estrictamente en su telemetría. Para esto, se implementó un pipeline en las siguientes fases:

1. **Preprocesamiento y Feature Engineering:**
   * Binarización de la variable objetivo (`shotType` -> `is_goal`).
   * Cálculo de la distancia euclidiana hacia el centro de la portería (`distance_to_goal`).
   * One-Hot Encoding para las variables categóricas (`situation`, `bodyPart`).
   * Estandarización de las variables numéricas mediante `StandardScaler` para algoritmos sensibles a distancias.

2. **K-Nearest Neighbors (KNN) - El Modelo Base (Baseline):**
   Se implementó KNN para probar la agrupación de datos por cercanía geométrica. Aunque arrojó un Accuracy del 89%, su *Recall* para detectar goles fue de solo 19%, evidenciando la trampa de la exactitud global en datasets desbalanceados.

3. **Random Forest Classifier - Análisis de Importancia:**
   Entrenado con `class_weight='balanced'`. Random Forest demostró que variables como la *Distancia a la portería* y *Coordenada X/Y* dominan abrumadoramente el peso predictivo sobre la *Parte del cuerpo* o *Situación de juego*. Sin embargo, el modelo sufrió de "overfitting" hacia la clase mayoritaria (no goles).

4. **Support Vector Machine (SVM) - El Modelo Definitivo:**
   Implementado con un Kernel RBF (Radial Basis Function) y `class_weight='balanced'`. Al ser la probabilidad de un gol una frontera de decisión sumamente curva e irregular en la cancha, SVM logró mapear esa frontera no lineal con éxito, **aumentando el Recall de goles de un 19% (KNN) a un 66% (SVM)**.

---

## Resultados y Conclusiones Técnicas

El hallazgo más importante del proyecto demuestra que **predecir goles es un problema geométrico no lineal**, no un simple problema de árboles lógicos. Mientras los modelos tradicionales (KNN y Random Forest) priorizaron un Accuracy artificial alto ignorando los goles verdaderos, el algoritmo **SVM (Kernel RBF)** sacrificó su Accuracy general para convertirse en un detector mucho más sensible a las zonas reales de peligro en la cancha.

> **NOTA IMPORTANTE:** 
> Para conocer el detalle técnico profundo, la implementación paso a paso del código, las gráficas generadas (Feature Importance, Matrices de Confusión) y el razonamiento funcional exhaustivo de cada hallazgo, **es obligatorio revisar el archivo interactivo `pia_machine_learning.ipynb`**. Ese documento sirve como el reporte maestro donde se fusionan el código y la teoría.
