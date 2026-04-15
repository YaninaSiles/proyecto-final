# Análisis Estratégico de Locación Comercial en CABA

## Descripción y Objetivos
Este proyecto construye un pipeline de datos end-to-end integrando fuentes publicas y web scraping para determinar la ubicación óptima de un nuevo kiosco en CABA.
El pipeline sigue una arquitectura Medallion(Broze->Silver->Gold) y permite explorar oportunidades de negocio analizando la demanda potencial(escuelas, hospitales y universidades) contra la competencia existente mediante un dashboard interactivo.

## Arquitectura

* **Capa Bronze:** Ingesta de datos crudos desde archivos CSV alojados en Databricks Volumes(Datos de BA y Google maps).
* **Capa Silver:** Limpieza, estandarización de texto, manejo de nulos, cruces espaciales. 
* **Capa Gold:** Creación del modelo dimensional y tablas de agregación para el consumo desde el dashboard.

## Tecnologías
* **Motor de Procesamiento:** Databricks / Apache Spark
* **Lenguaje:** SQL (Spark SQL)
* **Almacenamiento:** Delta Lake & Databricks Volumes
* **Orquestación:** Databricks Workflows (Schedule diario activado bajo la regla *1 Notebook = 1 Task*)
* **Control de Versiones:** Git / GitHub

## Modelado de Datos
Para la capa de consumo analítico (Gold) se implementó un **Modelo Estrella** optimizado para BI con actualización SCD Type 1:

* **Tabla de hechos (`fact_entidades`):** Diseñada con granularidad a nivel de entidad/punto geográfico. Contiene claves foráneas, coordenadas (latitud/longitud) y métricas de negocio (`es_generador_trafico`, `es_competencia`).
* **Tablas de dimensiones:**.
  * `dim_barrios` y `dim_comunas`: Jerarquía geográfica y polígonos.
  * `dim_entidad`: Datos de contacto y ubicación exacta.
  * `dim_categoria`: Clasificación de los establecimientos.

## Dashboard Analítico

KPIs:
- Índice de Oportunidad (Relación Demanda / Competencia por barrio).
- Top mejores Barrios.
- Nivel de Saturación del Mercado.

Visualizaciones:
- Mapa Geoespacial: Muestra la ubicación exacta de los puntos de atracción (verde) y la competencia (rojo).
- Distribución de Demanda: Desglose del tipo de público (escolar, salud, universitario) por barrio.
- Ranking de Rentabilidad: Ordena las zonas más viables para invertir minimizando el riesgo.

## Origen de los Datos
Los datos provienen de fuentes públicas oficiales del Gobierno de la Ciudad y de extracción web (Web Scraping):
* **Hospitales, Escuelas y Universidades:** Portal oficial *BA Data* del Gobierno de la Ciudad.
* **Kioscos (Competencia):** Datos obtenidos mediante extracción web (Web Scraping) de Google Maps sobre una muestra de 20 barrios estratégicos de CABA.

## Estructura del Repositorio
El repositorio sigue las mejores prácticas de organización lógica de procesos:
* `notebooks/00_EDA/`: Análisis Exploratorio de Datos sistemático (6 pasos) justificando las reglas de limpieza posteriores.
* `notebooks/01_DDL/`: Scripts de creación de esquemas, volúmenes y tablas Delta para las 3 capas.
* `notebooks/02_Bronze/`: Pipelines de ingesta inicial.
* `notebooks/03_Silver/`: Procesamiento, limpieza y normalización geoespacial (`st_transform`, `st_contains`).
* `notebooks/04_Gold/`: Población del Modelo Estrella.
* `docs/`: Documentación técnica exhaustiva (`Decisiones_tecnicas.md`) y diagramas.
* `viz/`: Capturas del Dashboard final.
