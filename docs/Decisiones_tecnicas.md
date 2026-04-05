# Decisiones Técnicas

A continuación se detallan las principales decisiones de diseño, modelado y arquitectura tomadas durante el desarrollo del pipeline de datos.

## 1. Arquitectura Medallion
Se adoptó el paradigma de tres capas (Medallion Architecture) en Databricks para garantizar la trazabilidad y calidad de los datos:
* **Capa Bronze:** Ingesta cruda desde archivos CSV cargados en Databricks Volumes hacia tablas Delta.
* **Capa Silver:** Limpieza de datos. Se estandarizaron textos (minúsculas, eliminación de caracteres extraños originados en el scraping), manejo de nulos y deduplicación utilizando funciones de ventana (`ROW_NUMBER() OVER`).
* **Capa Gold:** Transformación en un Modelo Dimensional (Star Schema) y optimizado para consumo desde herramientas de BI.

## 2. Modelado de Datos (Capa Gold)
El objetivo de negocio es analítico, por lo que se diseñó un **Modelo Estrella** clásico que optimiza las consultas de agregación:
* **Generación de Claves Subrogadas:** Las dimensiones físicas (`dim_comunas`, `dim_barrios`, `dim_categoria`, `dim_entidad`) utilizan claves generadas automáticamente por el motor (`BIGINT GENERATED ALWAYS AS IDENTITY`) para maximizar el rendimiento de los cruces.
* **Métricas de la Tabla de Hechos:** La `fact_entidades` se diseñó con un nivel de granuralidad identidad y coordenadas(longitud y latitud). En lugar de contener descripciones, agrupa identificadores e introduce métricas de negocio (`es_generador_trafico` y `es_competencia`) que permiten calcular dinámicamente el Índice de Oportunidad.

## 3. Procesamiento Geoespacial
Uno de los mayores desafíos del proyecto fue estandarizar los distintos tipo de los datos espaciales y obtener el barrio y comuna de los que no poseían:
* **Estandarización de Sistemas de Coordenadas (CRS):** Los datasets oficiales de Hospitales y Escuelas traían geometrías en el formato oficial argentino *Gauss-Krüger BA (EPSG:9498)*. Para poder visualizarlos en plataformas estándar (y cruzarlos con los datos del scraper de kioscos), se utilizó la función `st_transform` para llevarlos a grados decimales *WGS84 (EPSG:4326)*.
* **Cruces Espaciales:** Los datos extraídos de Google Maps no contenían el barrio ni la comuna de los kioscos. Para solucionar esto en la capa Silver, se cruzaron los puntos de los kioscos contra los polígonos del dataset oficial de barrios utilizando la función espacial `st_contains`.

## 4. Estrategia de Scraping y Muestreo
Debido a las limitaciones típicas de las APIs de mapas, se optó por una estrategia de muestreo representativa. Se ejecutó el scraper enfocándose en 20 barrios específicos de CABA (aproximadamente el 40% de la ciudad) extrayendo una muestra de hasta 60 kioscos por zona. Esto permite tener un dataset lo suficientemente robusto para validar la efectividad de las métricas de densidad comercial sin requerir una ingesta masiva y costosa de toda la red comercial de la ciudad.

## 5. Diseño de KPIs y Lógica de Negocio
- **"Índice de Oportunidad"**: Este KPI resuelve matemáticamente la viabilidad de una ubicación. Cantidad de demanda(puntos de ínteres)/Cantidad de competencia.
- **Los 10 barrios con mayor nivel de oportunidad**(teniendo en cuenta el indice de oportunidad).
- **Los 10 barrios con mayor nivel de demanda.**
- **Los 10 barrios con mayoir nivel de saturación.**