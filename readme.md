# Spark ETL on GCP Dataproc (NYC Taxi Data)

End-to-end Spark ETL pipeline that processes NYC taxi parquet data and writes curated outputs to Google Cloud Storage and BigQuery.

## Architecture

![Project Overview](/docs/images/etl_spark.png)

## Table of Contents
- [Problem Statement](#problem-statement)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Setup](#setup)
- [How to Run](#how-to-run)
  - [SparkSQL to GCS](#sparksql-to-gcs)
  - [SparkSQL to BigQuery](#sparksql-to-bigquery)
- [Results and Validation](#results-and-validation)
- [Data Source](#data-source)
- [Key Takeaways](#key-takeaways)
- [Future Improvements](#future-improvements)

## Problem Statement

Build a reproducible Spark pipeline on GCP Dataproc to transform raw NYC taxi datasets into analytics-ready outputs in:
- Google Cloud Storage (Parquet)
- BigQuery (table for analysis)

## Tech Stack

- **Processing:** PySpark, SparkSQL
- **Cloud:** GCP Dataproc, GCS, BigQuery
- **Language:** Python 3.11
- **Runtime:** Spark 3.5, Java 11
- **Data Format:** Parquet

## Project Structure

```text
code/
  spark_gcs/
    use_sparkSQL.py
  spark_bigquery/
    use_sparkSQL_write_to_bigquery.py
docs/
  images/
notebooks/
result/
README.md
```

## Setup

### Compute Environment
- VM: e2-standard-4, Ubuntu 20.04, 30GB disk
- Python 3.11.5, Spark 3.5.0, Scala 2.13.8, Java 11

### Dataproc Cluster
- Image: 2.1.35-debian11
- Master: e2-standard-2
- Workers: 2 x n2-standard-4

## How to Run

### SparkSQL to GCS

```bash
gcloud dataproc jobs submit pyspark \
  --cluster=de-spark-cluster \
  --region=asia-south1 \
  gs://de-spark-storage/sparkSQL_job/script/use_sparkSQL.py \
  -- \
    --input_green=gs://de-spark-storage/sparkSQL_job/input/green_taxi/green_tripdata_2023-01.parquet \
    --input_yellow=gs://de-spark-storage/sparkSQL_job/input/yellow_taxi/yellow_tripdata_2023-01.parquet \
    --output=gs://de-spark-storage/sparkSQL_job/output
```

### SparkSQL to BigQuery

```bash
gcloud dataproc jobs submit pyspark \
  --cluster=de-spark-cluster \
  --region=asia-south1 \
  --jars=gs://spark-lib/bigquery/spark-bigquery-latest_2.12.jar \
  gs://de-spark-storage/sparkSQL_bigquery_job/script/use_sparkSQL_write_to_bigquery.py \
  -- \
    --input_green=gs://de-spark-storage/sparkSQL_bigquery_job/input/green_taxi/green_tripdata_2023-01.parquet \
    --input_yellow=gs://de-spark-storage/sparkSQL_bigquery_job/input/yellow_taxi/yellow_tripdata_2023-01.parquet \
    --output=trips_data.revenue_calculation
```

## Results and Validation

### Dataproc Jobs
![Dataproc Jobs](/docs/images/dataproc_jobs.png)

- BigQuery write job: `272de51c6a434039a90e616a621489e3`
- GCS write job: `4211a3c6b2264e11bec90beed3eb317a`

### GCS Pipeline Output
![GCS Job Config](/docs/images/spark_gcs_job_config.png)
![GCS Job Status](/docs/images/spark_gcs_job_status.png)
![GCS Job Output](/docs/images/spark_gcs_job_output.png)
![GCS Output Files](/docs/images/spark_gcs_output_files.png)

### BigQuery Pipeline Output
![BQ Job Config](/docs/images/spark_bq_job_config.png)
![BQ Job Status](/docs/images/spark_bq_job_status.png)
![BQ Job Output](/docs/images/spark_bq_job_output.png)
![BQ Table Info](/docs/images/spark_bq_output_table_info.png)
![BQ Table Schema](/docs/images/spark_bq_output_table_schema.png)
![BQ Table Preview](/docs/images/spark_bq_output_table_preview.png)

## Data Source

NYC Taxi & Limousine Commission website - https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

## Key Takeaways

- Running Spark jobs at scale with Dataproc
- Managing cloud storage paths and pipeline organization
- Writing Spark output directly into BigQuery using connector jars

## Future Improvements

- Parameterize month/year and input paths
- Add partitioning strategy and data quality checks
- Add workflow orchestration (for example, Airflow/Composer)
- Add CI checks and unit/integration tests
