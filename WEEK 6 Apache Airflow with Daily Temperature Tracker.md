# Daily Temperature Tracker with Apache Airflow

## Overview

This project builds a scheduled data pipeline using **Apache Airflow** to:

- Fetch hourly temperature data for **Nairobi** from the **Open-Meteo API**
- Transform the data into a structured format using **Pandas**
- Load the data into a **PostgreSQL** database
- Learn orchestration essentials with Airflow

**API Used:** `https://api.open-meteo.com/v1/forecast?latitude=-1.2921&longitude=36.8219&hourly=temperature_2m`

## Objectives

- Integrate a real-time API with Airflow
- Implement DAGs and task orchestration
- Persist data using **CSV** and **PostgreSQL**
- Enhance the pipeline with:
  - Parameterization
  - Error handling
  - Scheduling logic

## Detailed Project Flow

### 1. Extract

Use Airflow's `PythonOperator` to fetch data:

- Endpoint: Open-Meteo API (hourly temperature)
- Location: Nairobi
- Handle potential API errors
- Capture response in JSON format

```python
response = requests.get(api_url)
if response.status_code == 200:
    data = response.json()
else:
    print("API not working")
```

### 2. Transform

- Parse JSON to extract timestamps and temperature
- Structure into a **Pandas DataFrame**
- Save as backup to CSV

```python
df = pd.DataFrame({
    "timestamp": data['hourly']['time'],
    "temperature": data['hourly']['temperature_2m']
})
df.to_csv("nairobi_weather.csv", index=False)
```

### 3. Load

- Load the CSV data into a **PostgreSQL** table: `nairobi_weather`
- Use `SQLAlchemy` for the connection

```python
engine = create_engine("postgresql://<username>:<password>@<host>:<port>/<database>?sslmode=require")
df.to_sql("nairobi_weather", engine, if_exists="append", index=False)
```

### 4. Schedule & Orchestrate

- Schedule DAG to run **hourly**
- Use `catchup=False` to avoid backfilling
- Task naming: `extract_temp`
- Logical task chaining is demonstrated below

## Code Implementation (DAG)

```python
import requests
import pandas as pd
from sqlalchemy import create_engine
from airflow import DAG
from datetime import datetime
from airflow.operators.python import PythonOperator

def nairobi_etl():
    url = "https://api.open-meteo.com/v1/forecast?latitude=-1.2921&longitude=36.8219&hourly=temperature_2m"
    response = requests.get(url)

    if response.status_code == 200:
        data = response.json()
    else:
        print("API not working")
        return

    weather_data = {
        "timestamp": data['hourly']['time'],
        "temperature": data['hourly']['temperature_2m']
    }

    df = pd.DataFrame(weather_data)
    df.to_csv("nairobi_weather.csv", index=False)

    engine = create_engine("postgresql://avnadmin:AVNS_G1ajzCj_WUpXrLzc-3t@pg-c63647-lagatkjosiah-692c.c.aivencloud.com:24862/defaultdb?sslmode=require")
    df.to_sql("nairobi_weather", engine, if_exists="append", index=False)

    return "Pipeline ran successfully"

default_args = {
    "owner": "Josiah",
    "depends_on_past": False
}

with DAG(
    dag_id="nairobi_weather",
    start_date=datetime(2025, 9, 4),
    schedule_interval="@hourly",
    catchup=False,
    default_args=default_args,
    description="Hourly temperature data pipeline for Nairobi"
) as dag:
    nairobi_etl_task = PythonOperator(
        task_id="extract_temp",
        python_callable=nairobi_etl
    )

nairobi_etl_task
```

## Notes

- **Security Tip:** Never expose real passwords or database URIs in public repos. Use environment variables or Airflow's **Connections UI**.
- **Improvements:** Add email alerts, retry logic, logging, and data validation for production-grade reliability.

## Conclusion

This project demonstrates how to create a **fully automated, hourly ETL pipeline** using Apache Airflow, fetching live data from a public API, transforming it, and persisting it in a local database — all within a reproducible and scalable orchestration framework.
