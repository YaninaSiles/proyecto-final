# Análisis Estratégico de Locación Comercial en CABA: Apertura de Kioscos

## Descripción del Proyecto
Este proyecto de Ingeniería de Datos tiene como objetivo determinar la ubicación óptima para la apertura de un nuevo kiosco en la Ciudad Autónoma de Buenos Aires (CABA). 

A través de un enfoque analítico y geoespacial, el modelo evalúa dos variables fundamentales:
1. **Zonas de Alta Demanda (Generadores de Tráfico):** Concentración de escuelas, universidades y hospitales.
2. **Saturación del Mercado (Competencia):** Densidad de kioscos existentes en la misma zona.

El resultado final es un modelo de datos analítico y un Dashboard interactivo que permite calcular el **"Índice de Oportunidad"** por barrio, facilitando la toma de decisiones comerciales basadas en datos.

## Arquitectura y Tecnologías
El pipeline de datos está construido sobre **Databricks** utilizando **Delta Lake** bajo la **Arquitectura Medallion** (Bronze, Silver, Gold).
* **Lenguaje:** SQL (Spark SQL).
* **Almacenamiento:** Databricks Volumes & Delta Tables.
* **Procesamiento Geoespacial:** Funciones espaciales nativas (`st_transform`, `st_contains`, `st_geomfromtext`).

## Modelado de Datos (Capa Gold)
![](/Workspace/Users/siles.yanina@gmail.com/proyecto-final/docs/modelo_de_datos.png)
Se implementó un **Modelo Estrella (Star Schema)** diseñado para optimizar el rendimiento de las consultas analíticas y facilitar la conexión con herramientas de visualización:

* **Fact Table (`fact_entidades`):** Contiene las métricas clave y las claves foráneas hacia las dimensiones. Incluye indicadores booleanos transformados en métricas numéricas (`es_generador_trafico`, `es_competencia`) para cálculos ágiles de densidad.
* **Dimensiones:** 
  * `dim_barrios` y `dim_comunas`: Contienen la jerarquía geográfica y polígonos WKT.
  * `dim_entidad`: Datos de contacto y ubicación exacta de cada punto.
  * `dim_categoria`: Clasificación semántica de los establecimientos.

## Dashboard Analítico
![](/Workspace/Users/siles.yanina@gmail.com/proyecto-final/viz/dashboard_analisis_estrategico_local_comercial.png)
Los indicadores clave (KPIs) presentados son:

1.  **Índice de Oportunidad Estratégica:** Ranking de barrios que poseen la mayor relación de demanda por cada local de competencia (`Demanda / Competencia`).
2.  **Concentración de Tráfico Puro:** Identificación de zonas con mayor volumen absoluto de instituciones (público masivo), ideal para estrategias de volumen.
3.  **Nivel de Saturación del Mercado:** Identificación de "Zonas Rojas" con exceso de competencia para prevenir inversiones de alto riesgo.
4.  **Perfil de Demanda por Barrio:** Desglose del tipo de público (escolar, universitario o de salud) para adaptar el inventario del local según la zona elegida.

## Origen de los Datos
Los datos provienen de fuentes públicas oficiales del Gobierno de la Ciudad y de extracción web (Web Scraping):

* **Hospitales:** Portal BA Data ([Link al dataset](https://data.buenosaires.gob.ar/dataset/hospitales/resource/955d9dad-e05c-4093-ae27-b8427f82e4bb))
* **Escuelas:** Portal BA Data ([Link al dataset](https://data.buenosaires.gob.ar/dataset/establecimientos-educativos/resource/ea2bd89f-c680-4b7d-b5fe-1b5b0ecc6a8a))
* **Universidades:** Portal BA Data ([Link al dataset](https://data.buenosaires.gob.ar/dataset/universidades/resource/juqdkmgo-2131-resource))
* **Kioscos:** Para obtener la competencia, se utilizó un [Scraper de Google Maps](https://github.com/Daniel1798-web/maps-local-business-scraper). Se relevaron un aproximado de 60 kioscos en 20 barrios representativos de CABA.

## Estructura del Repositorio
* `/00_EDA/`: Análisis Exploratorio de Datos preliminar para garantizar la calidad de origen.
* `/01_DDL/`: Scripts de creación de catálogos, esquemas y tablas (Bronze, Silver, Gold).
* `/02_Bronze/`: Scripts de ingesta de archivos CSV crudos (`read_files`).
* `/03_Silver/`: Scripts de limpieza, estandarización de tipos, deduplicación y conversiones espaciales.
* `/04_Gold/`: Construcción del Modelo Estrella (Dimensiones y Tabla de Hechos).
