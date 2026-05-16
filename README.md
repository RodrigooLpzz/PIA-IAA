# PIA: Introducción al Aprendizaje Automático
**Modelo Predictivo de Goles Esperados (xG) en la Premier League**

- Rodrigo Lopez Escobedo 2049807
- Gustavo Arreola Almaguer 2074164
- David Elias Hernandez Arellano 1999703
- Diego Andre Islas Cadillo 2132810

## Sobre el Conjunto de Datos
Este proyecto utiliza el dataset `epl_2024_2025_match_shots.csv` (10MB) que contiene aproximadamente 9,000 registros correspondientes a **todos los tiros realizados en los partidos de la English Premier League (EPL)** durante la temporada **2024 - 2025**.

## Origen de los Datos
El dataset utilizado en esta práctica no proviene de repositorios genéricos, sino de un trabajo de **Data Engineering (MLOps)** desarrollado inicialmente en el repositorio personal [SportsAnalytics](https://github.com/diegoaislasc/SportsAnalytics). 
El trabajo técnico para construir este conjunto de datos consistió en:
1. Utilizar la librería de Python `ScraperFC` para extraer IDs de los partidos de la API oculta de **Sofascore**.
2. Iterar sobre los IDs para extraer la telemetría masiva de cada tiro y jugada.
3. Recopilar la información en una tabla centralizada en **Google BigQuery** (GCP).
4. A Partir de esa base de datos robusta de BigQuery, extraer una instantánea en CSV para trabajar localmente.

## Evolución del Dataset
Este dataset de telemetría deportiva pasó por distintas fases académicas para llegar a su madurez actual:

- Aportación de la materia de **Minería de Datos:**
   Durante las prácticas de dicha materia, los datos brutos extraídos del proyecto personal fueron sometidos a procesos de transformación. Se utilizaron técnicas de *Data Cleaning* y manejo de calidad de datos (como el desempaquetado de las estructuras JSON de `playerCoordinates` para obtener columnas como `shot_x` y `shot_y`, y conversiones de variables de fecha/hora). Esto nos brindó un dataset estructurado.

- Aplicación para el PIA de Introducción al **Aprendizaje Automático**:
  Al disponer de un dataset con características tabulares de alta dimensionalidad espacial y categórica, podemos aprovechar este recurso para aplicar las técnicas de modelado aprendidas a lo largo del curso. Al tener un dataset que ya ha sido recopilado y limpiado, podemos enfocarnos en el modelado predictivo y las estrategias de evaluación.

---

## Metodología y Técnicas de Aprendizaje Automático

El objetivo central del PIA es determinar la probabilidad de que un tiro se convierta en Gol basándose estrictamente en su telemetría. Para esto, se implementó un pipeline en las siguientes fases:

1. **Preprocesamiento de los datos:**
   * Binarización de la variable objetivo (`shotType` -> `is_goal`).
   * Cálculo de la distancia euclidiana hacia el centro de la portería (`distance_to_goal`).
   * One-Hot Encoding para las variables categóricas (`situation`, `bodyPart`).
   * Estandarización de las variables numéricas mediante `StandardScaler` para algoritmos sensibles a distancias.

2. **K-Nearest Neighbors (KNN):**
   Se implementó KNN para probar la agrupación de datos por cercanía geométrica. Aunque arrojó un Accuracy del 89%, su *Recall* para detectar goles fue de solo 19%, evidenciando la inexactitud al tener un dataset desbalanceado.

3. **Random Forest Classifier:**
   Entrenado con `class_weight='balanced'`. Random Forest demostró que variables como la *Distancia a la portería* y *Coordenada X/Y* dominan abrumadoramente el peso predictivo sobre la *Parte del cuerpo* o *Situación de juego*. Sin embargo, el modelo sufrió de "overfitting" hacia la clase mayoritaria (no goles).

4. **Support Vector Machine (SVM):**
   Implementado con un Kernel RBF (Radial Basis Function) y `class_weight='balanced'`. Al ser la probabilidad de un gol una frontera de decisión sumamente curva e irregular en la cancha, SVM logró mapear esa frontera no lineal con éxito, **aumentando el Recall de goles de un 19% (KNN) a un 66% (SVM)**.

---

## Resultados y Conclusiones Técnicas

El hallazgo más importante del proyecto demuestra que predecir goles es un problema geométrico no lineal, no un simple problema de árboles lógicos. Mientras los modelos tradicionales (KNN y Random Forest) priorizaron un Accuracy artificial alto ignorando los goles verdaderos, el algoritmo **SVM (Kernel RBF)** sacrificó su Accuracy general para convertirse en un detector mucho más sensible a las zonas reales de peligro en la cancha.

