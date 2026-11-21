# DA Spark Pipeline

An end-to-end batch data pipeline for e-commerce data. Apache Airflow orchestrates PySpark jobs that transform nine raw CSV datasets into cleaned Parquet layers and load analytics-ready tables into PostgreSQL.

## Architecture

The pipeline follows a Medallion-style flow:

1. **Bronze:** Read source CSV files and store them as Parquet.
2. **Silver:** Clean and transform the data, enrich customer and seller records with geolocation, and route invalid order items to a dead-letter queue (DLQ).
3. **Data quality:** Validate customer records and order statuses with Great Expectations, and fail the run if the DLQ rate reaches 10%.
4. **Gold:** Upsert dimensions and facts into a PostgreSQL star schema.

![Pipeline architecture](./img/pipeline_architecture.png)

![Data lineage](./img/data_lineage.png)

## Input data

The raw CSV datasets are not included in this repository. Obtain a copy from a source you are permitted to use, then place the nine expected CSV files in `data/raw/` before running the pipeline. Generated Bronze, Silver, and DLQ data is also kept out of version control.

## Key features

- Daily workflow orchestration with Apache Airflow.
- Distributed data processing with PySpark.
- Parquet storage for Bronze and Silver data.
- Data quality checks and invalid-record handling through a DLQ.
- PostgreSQL star schema with JDBC-based upserts.
- Docker Compose environment with Airflow, Spark, and PostgreSQL services.

## Technology stack

| Component | Technology |
| --- | --- |
| Orchestration | Apache Airflow |
| Processing | Apache Spark / PySpark |
| Data quality | Great Expectations |
| Data lake storage | Parquet |
| Data mart | PostgreSQL |
| Local environment | Docker Compose |

## Run locally

**Prerequisites:** Docker Desktop (or Docker Engine) with the Docker Compose plugin, and enough resources to run the Airflow and Spark containers.

1. If you do not already have a local `.env` file, copy `.env.example` to `.env`. Set the database and Airflow admin passwords and the admin email in `.env`. This file is ignored by Git; do not commit it.

2. Obtain the input datasets and place the CSV files in `data/raw/`.

3. Initialize the Airflow database and local admin account:

   ```bash
   docker compose up airflow-init
   ```

4. Start the services:

   ```bash
   docker compose up -d
   ```

5. Open the Airflow UI at [http://localhost:8080](http://localhost:8080). Sign in with the local credentials configured in `.env`, then unpause and trigger the `ecommerce_etl_spark` DAG.

6. Monitor the DAG run in Airflow. The tasks run in order: extract to Bronze, transform to Silver, validate data quality, load to Gold, and report execution status.

To stop the local environment:

```bash
docker compose down
```

## Project structure

```text
.
├── dags/        # Airflow DAG
├── data/
│   ├── raw/     # Source CSV datasets
│   └── dlq/     # Rejected records
├── img/         # Architecture and lineage diagrams
├── plugins/     # Airflow plugins
├── sql/         # PostgreSQL schema initialization
├── src/         # PySpark ETL jobs
├── docker-compose.yml
├── Dockerfile
└── requirements.txt
```
