# 🏠 Airbnb Price Analysis & Machine Learning Pipeline

## 📋 Descripción del Proyecto
Este proyecto analiza el mercado de alquiler vacacional (Airbnb) para identificar oportunidades de inversión y determinar el "precio justo" de mercado. Se ha desarrollado un flujo de trabajo completo (ETL) que va desde la limpieza de datos crudos hasta la implementación de un modelo de Machine Learning (Random Forest) y la visualización interactiva en Power BI.

El objetivo final es detectar activos infravalorados ("chollos") y entender qué características (ubicación, extras, capacidad) influyen más en el precio.

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
