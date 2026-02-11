# 🏠 Airbnb Price Analysis & Machine Learning Pipeline

## 💡 Motivación del Proyecto: ¿Data Science para unas Vacaciones?

**El problema:** Un amigo tiene un viaje planeado a Barcelona en dos semanas. Al buscar alojamiento en Airbnb, se encontró con un mar de datos: miles de opciones, precios muy dispares para pisos similares y la eterna duda: *"¿Es esto una buena oferta o me están cobrando de más por ser turista?"*.

**La solución:** Me preguntó si podría pensar en algo para tomar una decisión basada en los datos. 
Construí un pipeline completo de Machine Learning para "hackear" mi búsqueda de alojamiento. El objetivo no era solo encontrar un sitio, sino desarrollar un modelo capaz de decirme el **precio justo** de cualquier propiedad basándose en sus características (ubicación, extras, tamaño) e identificar matemáticamente las **oportunidades de mercado (chollos)** que el ojo humano pasa por alto.


## 🛠️ Tech Stack
* **Lenguaje:** Python 3.9+
* **Librerías:** Pandas, NumPy, Scikit-Learn, Matplotlib.
* **Base de Datos:** PostgreSQL (pgAdmin 4).
* **Visualización:** Power BI Desktop (DAX).
* **Entorno:** Jupyter Notebook / VS Code.

---

## ⚙️ 1. ETL & Limpieza de Datos (Data Cleaning)
Los datos originales contenían ruido y formatos inconsistentes. Se realizaron las siguientes transformaciones en Python:

* **Limpieza de Baños (`bathrooms_text`):** La columna original mezclaba texto y números (ej: "1.5 shared baths"). Se utilizó Regex para extraer el valor numérico y estandarizarlo en una nueva columna `bathrooms_clean`.
* **Formato de Precio:** Eliminación de símbolos de moneda (`$`) y comas para convertir la columna `price` a tipo `float`.
* **Tratamiento de Nulos:** Imputación de valores faltantes en `bedrooms` y `beds` utilizando la mediana y la lógica de negocio.

## 🚀 2. Ingeniería de Características (Feature Engineering)
Para mejorar la precisión del modelo, se crearon nuevas métricas que aportan contexto de negocio:

### 📍 Métricas Geoespaciales
* **`distance_center_km`:** Cálculo de la distancia exacta desde cada piso hasta el centro de la ciudad (Km 0) utilizando la **Fórmula de Haversine** con las coordenadas de latitud y longitud.

### 💎 Métricas de Calidad y Lujo
* **`amenities_count`:** Conteo de la cantidad de servicios ofrecidos. Hipótesis: A mayor cantidad de extras, mayor precio.
* **Banderas Binarias (One-Hot Logic):** Creación de variables `0/1` para detectar lujos específicos mediante análisis de texto en la columna `amenities`:
    * `has_ac` (Aire acondicionado)
    * `has_pool` (Piscina)
    * `has_jacuzzi`
    * `has_parking`
    * `has_elevator`

### 📊 Métricas de Capacidad
* **`bathrooms_per_person`:** Ratio de comodidad (Baños / Capacidad).
* **`price_pp`:** Precio por persona (Métrica de análisis, no predictiva).

---

## 🐘 3. Integración con Base de Datos (PostgreSQL)
Se diseñó un Data Warehouse local para centralizar los datos limpios.

* **Schema Design:** Creación de la tabla `listings_clean` con tipos de datos optimizados (`NUMERIC` para dinero, `DOUBLE PRECISION` para coordenadas).
* **Estrategia de Carga:** * Resolución de conflictos de delimitadores en CSV mediante encapsulamiento estricto (`QUOTE_ALL`).
    * Importación masiva a PostgreSQL.

```sql
CREATE TABLE public.listings_clean (
    id INTEGER,
    listing_url VARCHAR,
    neighbourhood_cleansed VARCHAR,
    latitude double precision,
    longitude double precision,
    property_type VARCHAR,
    room_type VARCHAR,
    accommodates INTEGER,
    bedrooms FLOAT,
    beds FLOAT,
    amenities VARCHAR,
    price NUMERIC,
    minimum_nights INTEGER,
    availability_365 INTEGER,
    number_of_reviews INTEGER,
    review_scores_rating double precision,
    host_is_superhost INTEGER,
    price_pp NUMERIC, 
	bathrooms_per_person double precision, 
    distance_center_km double precision,   
    amenities_count integer,
	has_ac integer,
    has_elevator integer,
    has_pool integer,       
    has_jacuzzi integer,    
    has_parking integer     
);
```

## 🤖 4. Modelado Predictivo (Machine Learning)
Se implementó un modelo de regresión supervisada utilizando el algoritmo **Random Forest** para predecir el precio por noche. Este algoritmo fue seleccionado por su capacidad para manejar relaciones no lineales y su robustez frente al sobreajuste (overfitting).

### 🧠 Flujo de Trabajo (Pipeline)
1.  **Preprocesamiento:**
    * Eliminación de variables no predictivas (IDs, URLs).
    * **One-Hot Encoding:** Transformación de variables categóricas (`neighbourhood`, `room_type`) en variables numéricas.
    * **Filtrado de Outliers:** Se excluyeron propiedades con precios > 500€ para estabilizar el entrenamiento.
2.  **Entrenamiento:**
    * División del dataset: 80% Train / 20% Test.
    * **Hyperparameter Tuning:** Optimización de parámetros (`n_estimators=300`, `max_depth=20`) mediante `RandomizedSearchCV` para reducir el error.
3.  **Resultados:**
    * El modelo generó el precio sugerido (`price_suggested`) y la diferencia porcentual (`price_diff`).
    * **Métrica de Evaluación:** Se priorizó el **MAE (Error Absoluto Medio)** sobre el RMSE para obtener una interpretación directa en euros. Obtuvimos un valor de **18,86$**.
  
## 📊 5. Visualización Interactiva (Power BI)
Se construyó un cuadro de mando ejecutivo (Dashboard) para traducir las predicciones del modelo en decisiones de inversión. El informe utiliza DAX (Data Analysis Expressions) para cálculos dinámicos y segmentación avanzada.

### 📄 Estructura del Informe

#### Página 1: Radar de Oportunidades
Mapa geoespacial interactivo que destaca en verde los activos infravalorados (Chollos) y en rojo los sobrevalorados. Incluye KPIs de rentabilidad potencial como cantidad de pisos infravalorados y promedio de ahorro. 

![informe1](https://github.com/Nachoide100/An-lisis-Airbnb/blob/64495816b5fb1861f93160b050cf3b52daf1b3cc/visualizations/Captura%20de%20pantalla%202026-02-11%20100144.png)

#### Página 2: Drivers de Valor
Análisis de qué factores influyen en el precio (Impacto de la distancia al centro, curva de capacidad y prima por equipamiento), además de una validación visual de la precisión de predicción del modelo. ini

![informe2](https://github.com/Nachoide100/An-lisis-Airbnb/blob/64495816b5fb1861f93160b050cf3b52daf1b3cc/visualizations/Captura%20de%20pantalla%202026-02-11%20100154.png)

*Enlace para descargar el informe interactivo* -> [Informe](https://drive.google.com/file/d/1E45f9VDB352jjaxyWNUOr8qqnrM6hEaO/view?usp=drive_link)

#### 🧮 Métricas DAX Implementadas
Se crearon medidas y columnas calculadas para enriquecer la visualización:

**Nivel de Equipamiento:** clasifica los alojamientos en 4 niveles según la cantidad de "extras" detectados. 
```dax
Nivel_Equipamiento = 
SWITCH(
    TRUE(),
    'listings_clean'[amenities_count] < 10, "1. Básico",
    'listings_clean'[amenities_count] >= 10 && 'listings_clean'[amenities_count] < 20, "2. Estándar",
    'listings_clean'[amenities_count] >= 20, "3. Premium",
    "Desconocido"
)
```
**Mean Absolute Error (MAE):** calcula el error promedio en euros entre el precio real y el predicho por nuestro modelo. 
```dax
MAE (Error Medio) = 
AVERAGEX(
    'listings_clean', 
    ABS('listings_clean'[price] - 'listings_clean'[price_predecido])
)
```
**Coeficiente de correlación de Pearson:** fórmula estadística para validad la linealidad entre la predicción y la realidad. 
```dax
Correlacion Pearson = 
VAR MediaReal = AVERAGE('listings_clean'[price])
VAR MediaPred = AVERAGE('listings_clean'[price_predecido])
VAR Numerador = SUMX(
    'listings_clean', 
    ('listings_clean'[price] - MediaReal) * ('listings_clean'[price_predecido] - MediaPred)
)
VAR Denominador = SQRT(
    SUMX('listings_clean', ('listings_clean'[price] - MediaReal)^2) * SUMX('listings_clean', ('listings_clean'[price_predecido] - MediaPred)^2)
)
RETURN
DIVIDE(Numerador, Denominador)
```
---

## 🏁 Conclusión: De los Datos a la Decisión

Este proyecto demuestra que la Ciencia de Datos no es solo teoría abstracta, sino una herramienta práctica para la toma de decisiones cotidianas. Gracias a este pipeline:

1.  **Eliminé la incertidumbre:** Ya no dependemos de la intuición para saber si un precio es justo.
2.  **Identifiqué oportunidades reales:** El Dashboard de Power BI ha localizado activos en Barcelona con un descuento teórico significativo (chollos) basándose en las predicciones del modelo Random Forest.
3.  **Validé un flujo End-to-End:** Desde la limpieza de datos crudos hasta la visualización ejecutiva, demostrando la capacidad de transformar *raw data* en *actionable insights*.

Espero que mi amigo disfrute sus vacaciones!

--- 

*Autor:* José Ignacio Rubio

*Contacto:* [Perfil Linkledin](https://www.linkedin.com/in/jos%C3%A9-ignacio-rubio-194471308/)
