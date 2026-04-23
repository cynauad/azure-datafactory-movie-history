# Azure Data Factory — Movie History Pipeline

Proyecto práctico desarrollado como parte del curso **Master en Azure Databricks & Spark para Data Engineers [A-Z]** en Udemy.

Implementa un pipeline de datos end-to-end sobre Azure para procesar información histórica de películas, utilizando Azure Data Factory como orquestador y Azure Databricks para la transformación de datos.

---

## Arquitectura

El proyecto sigue una **arquitectura medallón** con dos fases principales:

```
Fuente de datos
      │
      ▼
┌─────────────────────────────────────┐
│  pl_ingest_movie_history_data       │  ← Fase 1: Ingesta (Bronze)
│  12 notebooks en paralelo           │
└─────────────────────────────────────┘
      │  (solo si la ingesta es exitosa)
      ▼
┌─────────────────────────────────────┐
│  pl_transformation_movie_history_   │  ← Fase 2: Transformación
│  data — 4 notebooks en paralelo     │
└─────────────────────────────────────┘
      │
      ▼
   ADLS Gen2 (Bronze Layer)
```

El pipeline maestro `pl_proccess_movie_history` orquesta ambas fases de forma secuencial.

---

## Stack tecnológico

| Servicio | Rol |
|---|---|
| **Azure Data Factory** | Orquestación de pipelines |
| **Azure Databricks** | Procesamiento y transformación con PySpark |
| **Azure Data Lake Storage Gen2** | Almacenamiento (capa Bronze) |
| **Power BI** *(integración futura)* | Visualización |

---

## Estructura del repositorio

```
azure-datafactory-movie-history/
├── factory/                          # Configuración de la instancia ADF
├── linkedService/                    # Conexiones a servicios externos
│   ├── LS_Databricks.json            # Conexión a Azure Databricks
│   └── ls_movie_history_adls.json    # Conexión a ADLS Gen2
├── dataset/                          # Definición de datasets
│   └── dataset_movie_history_bronze.json
├── pipeline/                         # Definición de pipelines
│   ├── pl_proccess_movie_history.json           # Pipeline maestro
│   ├── pl_ingest_movie_history_data.json        # Ingesta (12 fuentes)
│   └── pl_transformation_movie_history_data.json # Transformaciones
├── trigger/                          # Triggers de ejecución
│   └── tg_process_movie_history.json
└── publish_config.json               # Configuración de publicación (rama adf_publish)
```

---

## Repositorios relacionados

Los notebooks de Databricks que este pipeline orquesta se encuentran en un repositorio separado:

**[Databricks-movie-history](https://github.com/cauad85/Databricks-movie-history)** — contiene toda la lógica de procesamiento PySpark organizada en dos carpetas:

| Carpeta | Descripción |
|---|---|
| `ingestion/` | Notebooks ejecutados por `pl_ingest_movie_history_data` (12 notebooks, uno por entidad) |
| `transformation/` | Notebooks ejecutados por `pl_transformation_movie_history_data` (4 notebooks de resultados analíticos) |

---

## Pipelines

### `pl_proccess_movie_history` — Pipeline maestro

Orquesta el flujo completo. Acepta dos parámetros:

| Parámetro | Descripción |
|---|---|
| `p_environment` | Entorno de ejecución (default: `Production`) |
| `p_file_date` | Fecha de los archivos a procesar |

Ejecuta ingesta → transformación en secuencia, pasando `p_file_date` a cada hijo.

---

### `pl_ingest_movie_history_data` — Ingesta

Verifica que la carpeta de datos exista en ADLS Gen2 antes de procesar. Si no existe, envía un email de error vía Databricks. Si existe, lanza **12 notebooks en paralelo**:

- movie · language · genre · country · person
- movie_genre · movie_cast · language_role
- production_company · movie_company · movie_languages · production_country

---

### `pl_transformation_movie_history_data` — Transformación

Mismo patrón de verificación de carpeta. Si los datos están disponibles, ejecuta **4 notebooks en paralelo**:

- Movie + Género + Idioma
- País + Empresa productora
- Agrupado por Movie-Género
- Agrupado por Movie-País

---

## Trigger

`tg_process_movie_history` es un **ScheduleTrigger** de recurrencia mensual (configurado con frecuencia de 12 meses). Estado actual: `Stopped`.

---

## Conceptos del curso aplicados

- Procesamiento de datos **batch incremental** con parámetro de fecha
- Pipelines **parametrizados** y reutilizables
- **Control de flujo** con actividades `GetMetadata` e `IfCondition`
- Ejecución paralela de notebooks Databricks desde ADF
- **Manejo de errores** con notificación por email
- Integración **ADF + ADLS Gen2 + Databricks** sobre Azure
- Publicación de ARM templates mediante rama `adf_publish`

---

## Requisitos previos

- Suscripción activa en **Microsoft Azure**
- Instancia de **Azure Data Factory** (`dbricks-course-with-adf`)
- Workspace de **Azure Databricks** con notebooks desplegados en `/Users/auad03@gmail.com/movie-history/`
- Cuenta de almacenamiento **ADLS Gen2** (`moviehistory22`) con contenedor `bronze`

---

## Curso de referencia

**Master en Azure Databricks & Spark para Data Engineers [A-Z]**
Plataforma: [Udemy](https://www.udemy.com)

Temas cubiertos relevantes a este proyecto:
- Orquestación de pipelines con Azure Data Factory
- Arquitectura Data Lake y Lakehouse con Delta Lake
- Construcción de pipelines ETL con Apache Spark SQL y PySpark
- Clusters, Notebooks y Jobs en Azure Databricks
- Unity Catalog y gestión de seguridad
