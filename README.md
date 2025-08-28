# Netflix MovieLens ELT Pipeline with dbt (🧡) & Snowflake (❄️)

---

##  Overview  
This project demonstrates a full **end-to-end ELT (Extract, Load, Transform)** pipeline using **dbt (🧡)** as the transformation engine and **Snowflake (❄️)** as the cloud data warehouse—leveraging **Amazon S3** for raw data ingestion. The goal is to apply advanced dbt concepts and produce analytics-ready data models with modular, maintainable architecture.

---

## 1. Architecture & Data Flow

The pipeline follows the ELT paradigm:

### Extract → Load
- **Extract**: MovieLens dataset (simulating Netflix) was collected in CSV format.
- **Load**: CSV files were uploaded to an **S3 bucket**. Using a Snowflake **external stage (`Netflix_stage`)**, data was loaded directly into Snowflake's `raw` schema via the `COPY INTO` command.

### Transform (via dbt)
- dbt connects to Snowflake and manages transformations across multiple layers:
  - **Staging Models** (`src_movies`, `src_ratings`, etc.): Clean and normalize raw data (e.g., renaming columns, type casting).
  - **Dimensional Models** (`dim_movies`, `dim_users`, `dim_genome_tags`): Business entities modeled as tables with aggregations and joins.
  - **Fact Models** (`fct_genome_scores`, etc.): Events and metrics modeled as fact tables.
  - **Ephemeral Models** (e.g., `dim_movies_with_tags`): Used for joins/logic during compilation (not persisted).
  - **Seeds**: Reference static data like `seed_movie_release_dates.csv` loaded via `dbt seed`.
  - **Sources**: Standardized raw source metadata defined in `sources.yml` for consistency and freshness checks.
  - **Snapshots**: SCD Type 2 snapshots (e.g., `snap_tags.sql`) tracking historical changes with time-based versioning (`dbt_valid_from`, `dbt_valid_to`) using `dbt_utils.generate_surrogate_key`.
  - **Tests**:
    - *Generic* (schema-level): `unique`, `not_null`, `relationships`, `accepted_values`.
    - *Singular* (SQL-based): `relevance_score_test.sql` ensuring business logic integrity.
  - **Macros**: Reusable templated constructs like `no_nulls_in_columns.sql` plus use of `ref()`, `source()`, `config()` and `dbt_utils`.
  - **Documentation & Lineage**: Fully described models and columns in `schema.yml`, auto-generated docs accessed via `dbt docs generate && dbt docs serve`. Interactive lineage graph visualizes model dependencies.

---

## 2. Snowflake Setup & Data Loading

- Created a Snowflake account with a `movielens` database, separated into `raw` and `dev` schemas for clear environments.
- Configured an S3 bucket (`Netflix_movie_data`) and stage (`Netflix_stage`) for seamless ingestion.
- Populated `raw_*` tables in Snowflake using `COPY INTO` from the external stage.

---

## 3. dbt Project Details

- Initialized a local **dbt project** (`Netflix`), configuring connection credentials (account, warehouse, role, dev schema).
- Used `dbt_project.yml` to define default materializations and map folder structure (`staging/`, `dim/`, `fct/`) to model behaviors.
- Models implemented across staging, dimensional, fact, ephemeral, seed, source, snapshot, test, macro, and documentation layers.

---

## 4. dbt Materializations Used

- **View**: For staging models—fast to build and always reflects source.
- **Table**: For dimension and fact models—persisted for performance.
- **Incremental**: For models like `fct_ratings`—updates only new/changed data.
- **Ephemeral**: For models like `dim_movies_with_tags`—compiled into queries only.

---

## 5. Additional Components

- **Seeds**: Static CSV-based tables (e.g., release dates) loaded via `dbt seed`.
- **Sources**: Documented raw source tables in `sources.yml` for traceability.
- **Snapshots**: Time-versioned history stored in `snap_tags.sql`.
- **Tests & Validation**: Using both generic schema checks and custom logic tests.
- **Macros & Packages**: Leveraged built-in and custom logic for code reuse (`dbt_utils.generate_surrogate_key`).
- **Documentation & Lineage**: Generated browsable docs and lineage visualizations via `dbt docs`.

---

## 6. Analysis & Insights

- Created analysis files like `movie_analysis.sql` for ad-hoc query generation (e.g., average ratings per movie). These are compiled with `dbt compile` and executed directly in Snowflake.

---

## 7. Skills Demonstrated

- **Advanced SQL & Dimensional Modeling**: Crafting staging, dimension, fact transformations.
- **dbt Mastery**: Modeling, materializations, seeds, sources, snapshots, macros, packages, tests, documentation, and lineage.
- **Cloud Warehousing**: Snowflake schema management and ingestion via S3.
- **ELT Pipeline Design**: Structured engineering with separation of raw vs. transformed environments.
- **Data Quality & Governance**: Implementing tests, versioned snapshots, and documentation for transparency.
- **Modular & Maintainable Code**: Organized project structure with reuse in mind.
- **Git-Ready Architecture**: Thoughtful project structure aligns with version control best practices (consistent naming, modular folder layout) :contentReference[oaicite:1]{index=1}.

---

## 8. Getting Started

```bash
git clone <YOUR_REPO_URL>
cd your_project_folder

# Install dependencies
pip install dbt-snowflake

# Configure your ~/.dbt/profiles.yml with Snowflake credentials

# Run models
dbt run

# Run tests
dbt test

# Generate docs
dbt docs generate
dbt docs serve
```
--
## License
This project is published under the MIT License — feel free to fork, explore, and enhance.
