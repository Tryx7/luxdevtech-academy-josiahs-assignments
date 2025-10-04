
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lqziofovtmzoa8ycw5l1.png)

## Complete Guide: Dockerizing Spark, Kafka, and Jupyter for YouTube Pipeline

## Table of Contents
1. [Project Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Project Structure Setup](#structure)
4. [Step 1: Environment Configuration](#step1)
5. [Step 2: Dockerizing Kafka](#step2)
6. [Step 3: Dockerizing Spark](#step3)
7. [Step 4: Dockerizing Jupyter](#step4)
8. [Step 5: Dockerizing Airflow](#step5)
9. [Step 6: Creating Docker Compose](#step6)
10. [Step 7: Building and Testing](#step7)
11. [Step 8: Complete Integration](#step8)

---

## Project Overview {#overview}

**Goal**: Build a containerized data pipeline that extracts YouTube data, processes it with Spark, streams through Kafka, and analyzes with Jupyter.

**Architecture**:
```
YouTube API → Airflow (Orchestration) → Kafka (Streaming) → Spark (Processing) → PostgreSQL (Storage) → Jupyter (Analysis)
```

**Technologies**:
- Docker & Docker Compose
- Apache Kafka (Message Streaming)
- Apache Spark (Data Processing)
- Jupyter Notebook (Analysis)
- Apache Airflow (Orchestration)
- PostgreSQL (Database)

---

## Prerequisites {#prerequisites}

### Required Software
```bash
# Check Docker installation
docker --version
# Expected: Docker version 20.x or higher

# Check Docker Compose
docker-compose --version
# Expected: docker-compose version 2.x or higher
```

### System Requirements
- **RAM**: Minimum 8GB (16GB recommended)
- **Disk Space**: 15GB free space
- **OS**: Linux, macOS, or Windows with WSL2

---

## Project Structure Setup {#structure}

### Create Project Directory

```bash
# Create main project directory
mkdir youtube-pipeline
cd youtube-pipeline

# Create all subdirectories
mkdir -p airflow/{dags,logs,plugins}
mkdir -p spark/scripts
mkdir -p jupyter/notebooks
mkdir -p youtube_extractor
mkdir -p certificates
mkdir -p data/postgres

# Create necessary files
touch docker-compose.yml
touch .env
touch airflow/Dockerfile
touch airflow/requirements.txt
touch airflow/dags/youtube_pipeline.py
touch spark/Dockerfile
touch spark/scripts/process_youtube_data.py
touch jupyter/Dockerfile
touch jupyter/notebooks/youtube_analysis.ipynb
touch youtube_extractor/Dockerfile
touch youtube_extractor/extractor.py
touch youtube_extractor/requirements.txt
```

**Final Structure:**
```
youtube-pipeline/
├── docker-compose.yml
├── .env
├── airflow/
│   ├── Dockerfile
│   ├── dags/
│   │   └── youtube_pipeline.py
│   ├── logs/
│   ├── plugins/
│   └── requirements.txt
├── spark/
│   ├── Dockerfile
│   └── scripts/
│       └── process_youtube_data.py
├── jupyter/
│   ├── Dockerfile
│   └── notebooks/
│       └── youtube_analysis.ipynb
├── youtube_extractor/
│   ├── Dockerfile
│   ├── extractor.py
│   └── requirements.txt
├── certificates/
│   └── ca.pem
└── data/
    └── postgres/
```

---

## Step 1: Environment Configuration {#step1}

### Create .env File

Create `.env` in the project root:

```bash
# YouTube API Configuration
YOUTUBE_API_KEY=your_youtube_api_key_here

# PostgreSQL Configuration
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres123
POSTGRES_DB=youtube_db
POSTGRES_PORT=5432

# Airflow Configuration
AIRFLOW_UID=50000
AIRFLOW__CORE__EXECUTOR=LocalExecutor
AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql+psycopg2://postgres:postgres123@postgres:5432/youtube_db
AIRFLOW__CORE__FERNET_KEY=your_fernet_key_here
AIRFLOW__CORE__LOAD_EXAMPLES=False

# Kafka Configuration
KAFKA_BROKER=kafka:9092
KAFKA_TOPIC_INPUT=youtube-data
KAFKA_TOPIC_OUTPUT=processed-data

# Spark Configuration
SPARK_MASTER_URL=spark://spark-master:7077
SPARK_WORKER_MEMORY=2G
SPARK_WORKER_CORES=2

# Jupyter Configuration
JUPYTER_TOKEN=spark123
```

### Generate Fernet Key for Airflow

```bash
python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
# Copy the output and replace 'your_fernet_key_here' in .env
```

---

## Step 2: Dockerizing Kafka {#step2}

### Understanding Kafka Setup

Kafka requires:
1. **Zookeeper**: For cluster coordination
2. **Kafka Broker**: For message handling

### Add Kafka Services (We'll build docker-compose.yml incrementally)

We'll create the complete docker-compose.yml in Step 6, but understand the Kafka configuration first:

**Kafka Configuration Explained:**
- **Port 9092**: Internal Docker communication
- **Port 9093**: External host access
- **Zookeeper Port 2181**: Coordination service

---

## Step 3: Dockerizing Spark {#step3}

### Create Spark Dockerfile

Create `spark/Dockerfile`:

```dockerfile
FROM bitnami/spark:3.5.0

# Switch to root user for installations
USER root

# Install Python dependencies
RUN pip install --no-cache-dir \
    kafka-python==2.0.2 \
    pyspark==3.5.0 \
    requests==2.31.0 \
    pandas==2.1.4 \
    psycopg2-binary==2.9.9

# Create directories
RUN mkdir -p /opt/spark-apps /opt/spark-data

# Set permissions
RUN chmod -R 777 /opt/spark-apps /opt/spark-data

# Switch back to spark user
USER 1001

# Set working directory
WORKDIR /opt/spark-apps
```

### Create Spark Processing Script

Create `spark/scripts/process_youtube_data.py`:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, window, count, avg, sum as spark_sum
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, TimestampType
import os

# Initialize Spark Session with Kafka support
def create_spark_session():
    """Create Spark session with Kafka integration"""
    spark = SparkSession.builder \
        .appName("YouTubeDataProcessor") \
        .master(os.getenv("SPARK_MASTER_URL", "spark://spark-master:7077")) \
        .config("spark.jars.packages", "org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0") \
        .config("spark.sql.streaming.checkpointLocation", "/opt/spark-data/checkpoint") \
        .getOrCreate()
    
    spark.sparkContext.setLogLevel("WARN")
    return spark

# Define schema for YouTube data
youtube_schema = StructType([
    StructField("channel_id", StringType(), True),
    StructField("channel_title", StringType(), True),
    StructField("subscriber_count", IntegerType(), True),
    StructField("total_views", IntegerType(), True),
    StructField("video_count", IntegerType(), True),
    StructField("timestamp", TimestampType(), True)
])

def process_youtube_stream():
    """Process streaming YouTube data from Kafka"""
    
    print("Starting Spark Streaming Application...")
    spark = create_spark_session()
    
    # Read from Kafka topic
    kafka_df = spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", os.getenv("KAFKA_BROKER", "kafka:9092")) \
        .option("subscribe", os.getenv("KAFKA_TOPIC_INPUT", "youtube-data")) \
        .option("startingOffsets", "latest") \
        .load()
    
    # Parse JSON data
    parsed_df = kafka_df.select(
        from_json(col("value").cast("string"), youtube_schema).alias("data")
    ).select("data.*")
    
    # Perform transformations
    processed_df = parsed_df \
        .withColumn("engagement_ratio", col("total_views") / col("subscriber_count")) \
        .withColumn("avg_views_per_video", col("total_views") / col("video_count"))
    
    # Aggregations with windowing (5-minute tumbling window)
    aggregated_df = processed_df \
        .withWatermark("timestamp", "10 minutes") \
        .groupBy(
            window(col("timestamp"), "5 minutes"),
            col("channel_title")
        ) \
        .agg(
            count("*").alias("record_count"),
            avg("subscriber_count").alias("avg_subscribers"),
            spark_sum("total_views").alias("total_views_sum")
        )
    
    # Write to console for monitoring
    console_query = processed_df \
        .writeStream \
        .outputMode("append") \
        .format("console") \
        .option("truncate", "false") \
        .start()
    
    # Write processed data back to Kafka
    kafka_output_query = processed_df \
        .selectExpr("channel_id as key", "to_json(struct(*)) as value") \
        .writeStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", os.getenv("KAFKA_BROKER", "kafka:9092")) \
        .option("topic", os.getenv("KAFKA_TOPIC_OUTPUT", "processed-data")) \
        .option("checkpointLocation", "/opt/spark-data/checkpoint/kafka-output") \
        .outputMode("append") \
        .start()
    
    # Write aggregated data to PostgreSQL
    def write_to_postgres(batch_df, batch_id):
        """Write batch data to PostgreSQL"""
        if not batch_df.isEmpty():
            batch_df.write \
                .format("jdbc") \
                .option("url", f"jdbc:postgresql://postgres:5432/{os.getenv('POSTGRES_DB')}") \
                .option("dbtable", "spark_aggregations") \
                .option("user", os.getenv("POSTGRES_USER")) \
                .option("password", os.getenv("POSTGRES_PASSWORD")) \
                .option("driver", "org.postgresql.Driver") \
                .mode("append") \
                .save()
    
    postgres_query = aggregated_df \
        .writeStream \
        .foreachBatch(write_to_postgres) \
        .outputMode("update") \
        .start()
    
    print("Spark Streaming application started successfully!")
    print(f"Consuming from topic: {os.getenv('KAFKA_TOPIC_INPUT', 'youtube-data')}")
    print(f"Publishing to topic: {os.getenv('KAFKA_TOPIC_OUTPUT', 'processed-data')}")
    
    # Await termination
    spark.streams.awaitAnyTermination()

if __name__ == "__main__":
    try:
        process_youtube_stream()
    except Exception as e:
        print(f"Error in Spark application: {e}")
        raise
```

---

## Step 4: Dockerizing Jupyter {#step4}

### Create Jupyter Dockerfile

Create `jupyter/Dockerfile`:

```dockerfile
FROM jupyter/pyspark-notebook:latest

# Switch to root for installations
USER root

# Install additional dependencies
RUN pip install --no-cache-dir \
    kafka-python==2.0.2 \
    psycopg2-binary==2.9.9 \
    matplotlib==3.8.2 \
    seaborn==0.13.0 \
    plotly==5.18.0 \
    pandas==2.1.4 \
    sqlalchemy==2.0.25

# Install system dependencies for PostgreSQL
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Create work directory
RUN mkdir -p /home/jovyan/work /home/jovyan/data

# Set permissions
RUN chown -R ${NB_UID}:${NB_GID} /home/jovyan/work /home/jovyan/data

# Switch back to jovyan user
USER ${NB_UID}

# Set working directory
WORKDIR /home/jovyan/work
```

### Create Sample Jupyter Notebook

Create `jupyter/notebooks/youtube_analysis.ipynb`:

```json
{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# YouTube Data Analysis\n",
    "## Connecting to PostgreSQL and Analyzing Channel Metrics"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "import pandas as pd\n",
    "import psycopg2\n",
    "import matplotlib.pyplot as plt\n",
    "import seaborn as sns\n",
    "from sqlalchemy import create_engine\n",
    "\n",
    "# Set style\n",
    "sns.set_style('whitegrid')\n",
    "plt.rcParams['figure.figsize'] = (12, 6)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Database connection\n",
    "engine = create_engine('postgresql://postgres:postgres123@postgres:5432/youtube_db')\n",
    "\n",
    "# Load channel statistics\n",
    "query = \"SELECT * FROM channel_stats ORDER BY subscriber_count DESC\"\n",
    "df = pd.read_sql(query, engine)\n",
    "\n",
    "print(f\"Loaded {len(df)} channels\")\n",
    "df.head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Visualize subscriber counts\n",
    "plt.figure(figsize=(12, 6))\n",
    "plt.bar(df['channel_title'], df['subscriber_count'])\n",
    "plt.xlabel('Channel')\n",
    "plt.ylabel('Subscribers')\n",
    "plt.title('YouTube Channels by Subscriber Count')\n",
    "plt.xticks(rotation=45, ha='right')\n",
    "plt.tight_layout()\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Calculate engagement metrics\n",
    "df['avg_views_per_video'] = df['total_views'] / df['video_count']\n",
    "df['engagement_ratio'] = df['total_views'] / df['subscriber_count']\n",
    "\n",
    "df[['channel_title', 'avg_views_per_video', 'engagement_ratio']].sort_values(\n",
    "    'engagement_ratio', ascending=False\n",
    ")"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "name": "python",
   "version": "3.11.0"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 4
}
```

---

## Step 5: Dockerizing Airflow {#step5}

### Create Airflow Dockerfile

Create `airflow/Dockerfile`:

```dockerfile
FROM apache/airflow:2.8.1-python3.11

# Switch to root for system packages
USER root

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

# Switch back to airflow user
USER airflow

# Copy requirements file
COPY requirements.txt /requirements.txt

# Install Python dependencies
RUN pip install --no-cache-dir -r /requirements.txt

# Create necessary directories
RUN mkdir -p /opt/airflow/dags /opt/airflow/logs /opt/airflow/plugins
```

### Create Airflow Requirements

Create `airflow/requirements.txt`:

```txt
apache-airflow-providers-postgres==5.10.0
apache-airflow-providers-apache-kafka==1.3.0
kafka-python==2.0.2
requests==2.31.0
psycopg2-binary==2.9.9
google-api-python-client==2.111.0
```

### Create YouTube Pipeline DAG

Create `airflow/dags/youtube_pipeline.py`:

```python
from datetime import datetime, timedelta
import json
import requests
from kafka import KafkaProducer
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.models import Variable

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 2,
    'retry_delay': timedelta(minutes=5)
}

# YouTube channels to monitor
CHANNEL_IDS = [
    "UC_x5XG1OV2P6uZZ5FSM9Ttw",  # Google for Developers
    "UCq-Fj5jknLsU6-M0R3PiHcA",  # YouTube Creators
    "UCBJycsmduvYEL83R_U4JriQ",  # MKBHD
]

def extract_youtube_data(**context):
    """Extract YouTube channel data via API"""
    import os
    
    api_key = os.getenv('YOUTUBE_API_KEY')
    if not api_key or api_key == 'your_youtube_api_key_here':
        raise ValueError("Please set YOUTUBE_API_KEY in .env file")
    
    print(f"Extracting data for {len(CHANNEL_IDS)} channels...")
    
    channel_data = []
    
    for channel_id in CHANNEL_IDS:
        try:
            url = "https://www.googleapis.com/youtube/v3/channels"
            params = {
                'part': 'snippet,statistics',
                'id': channel_id,
                'key': api_key
            }
            
            response = requests.get(url, params=params)
            data = response.json()
            
            if 'items' in data and data['items']:
                channel_info = data['items'][0]
                snippet = channel_info['snippet']
                statistics = channel_info['statistics']
                
                channel_record = {
                    'channel_id': channel_id,
                    'channel_title': snippet['title'],
                    'channel_description': snippet.get('description', '')[:500],
                    'channel_created_at': snippet['publishedAt'],
                    'total_views': int(statistics.get('viewCount', 0)),
                    'subscriber_count': int(statistics.get('subscriberCount', 0)),
                    'video_count': int(statistics.get('videoCount', 0)),
                    'timestamp': datetime.now().isoformat()
                }
                
                channel_data.append(channel_record)
                print(f"✓ Extracted: {snippet['title']}")
                
        except Exception as e:
            print(f"Error extracting {channel_id}: {e}")
    
    # Push to XCom for next task
    context['task_instance'].xcom_push(key='channel_data', value=channel_data)
    return f"Extracted {len(channel_data)} channels"

def publish_to_kafka(**context):
    """Publish extracted data to Kafka"""
    import os
    
    channel_data = context['task_instance'].xcom_pull(
        task_ids='extract_youtube_data',
        key='channel_data'
    )
    
    if not channel_data:
        print("No data to publish")
        return
    
    # Initialize Kafka producer
    producer = KafkaProducer(
        bootstrap_servers=os.getenv('KAFKA_BROKER', 'kafka:9092'),
        value_serializer=lambda v: json.dumps(v).encode('utf-8'),
        retries=3
    )
    
    topic = os.getenv('KAFKA_TOPIC_INPUT', 'youtube-data')
    
    for channel in channel_data:
        try:
            future = producer.send(topic, value=channel)
            future.get(timeout=10)
            print(f"✓ Published: {channel['channel_title']}")
        except Exception as e:
            print(f"Error publishing to Kafka: {e}")
    
    producer.flush()
    producer.close()
    
    print(f"✓ Published {len(channel_data)} records to Kafka topic '{topic}'")

def load_to_postgres(**context):
    """Load data into PostgreSQL"""
    channel_data = context['task_instance'].xcom_pull(
        task_ids='extract_youtube_data',
        key='channel_data'
    )
    
    if not channel_data:
        print("No data to load")
        return
    
    postgres_hook = PostgresHook(postgres_conn_id='postgres_default')
    
    insert_sql = """
        INSERT INTO channel_stats 
        (channel_id, channel_title, channel_description, channel_created_at, 
         total_views, subscriber_count, video_count, processed_at)
        VALUES (%s, %s, %s, %s, %s, %s, %s, CURRENT_TIMESTAMP)
        ON CONFLICT (channel_id) DO UPDATE SET
            channel_title = EXCLUDED.channel_title,
            total_views = EXCLUDED.total_views,
            subscriber_count = EXCLUDED.subscriber_count,
            video_count = EXCLUDED.video_count,
            processed_at = CURRENT_TIMESTAMP
    """
    
    for channel in channel_data:
        postgres_hook.run(insert_sql, parameters=(
            channel['channel_id'],
            channel['channel_title'],
            channel['channel_description'],
            channel['channel_created_at'],
            channel['total_views'],
            channel['subscriber_count'],
            channel['video_count']
        ))
    
    print(f"✓ Loaded {len(channel_data)} channels into PostgreSQL")

with DAG(
    'youtube_kafka_spark_pipeline',
    default_args=default_args,
    description='YouTube data pipeline with Kafka and Spark',
    schedule_interval=timedelta(hours=6),
    catchup=False,
    tags=['youtube', 'kafka', 'spark'],
) as dag:

    create_tables = PostgresOperator(
        task_id='create_tables',
        postgres_conn_id='postgres_default',
        sql='''
            CREATE TABLE IF NOT EXISTS channel_stats (
                channel_id VARCHAR(255) PRIMARY KEY,
                channel_title TEXT,
                channel_description TEXT,
                channel_created_at TIMESTAMP,
                total_views BIGINT,
                subscriber_count BIGINT,
                video_count BIGINT,
                processed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );

            CREATE TABLE IF NOT EXISTS spark_aggregations (
                window_start TIMESTAMP,
                window_end TIMESTAMP,
                channel_title TEXT,
                record_count INTEGER,
                avg_subscribers DOUBLE PRECISION,
                total_views_sum BIGINT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
        '''
    )

    extract_data = PythonOperator(
        task_id='extract_youtube_data',
        python_callable=extract_youtube_data,
        provide_context=True
    )

    publish_kafka = PythonOperator(
        task_id='publish_to_kafka',
        python_callable=publish_to_kafka,
        provide_context=True
    )

    load_postgres = PythonOperator(
        task_id='load_to_postgres',
        python_callable=load_to_postgres,
        provide_context=True
    )

    # Task dependencies
    create_tables >> extract_data >> [publish_kafka, load_postgres]
```

---

## Step 6: Creating Complete Docker Compose {#step6}

### Create docker-compose.yml

Create `docker-compose.yml` in the project root:

```yaml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:13
    container_name: postgres
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "5432:5432"
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
    networks:
      - youtube-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Zookeeper for Kafka
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"
    networks:
      - youtube-network
    healthcheck:
      test: ["CMD-SHELL", "echo ruok | nc localhost 2181"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Kafka Broker
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka
    depends_on:
      zookeeper:
        condition: service_healthy
    ports:
      - "9092:9092"
      - "9093:9093"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:9093
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
      KAFKA_NUM_PARTITIONS: 3
    networks:
      - youtube-network
    healthcheck:
      test: ["CMD-SHELL", "kafka-broker-api-versions --bootstrap-server localhost:9092"]
      interval: 10s
      timeout: 10s
      retries: 5

  # Spark Master
  spark-master:
    build:
      context: ./spark
      dockerfile: Dockerfile
    container_name: spark-master
    environment:
      - SPARK_MODE=master
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
      - SPARK_MASTER_WEBUI_PORT=8080
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
      - KAFKA_BROKER=${KAFKA_BROKER}
      - KAFKA_TOPIC_INPUT=${KAFKA_TOPIC_INPUT}
      - KAFKA_TOPIC_OUTPUT=${KAFKA_TOPIC_OUTPUT}
    ports:
      - "8080:8080"
      - "7077:7077"
    volumes:
      - ./spark/scripts:/opt/spark-apps
      - ./data:/opt/spark-data
    networks:
      - youtube-network
    depends_on:
      kafka:
        condition: service_healthy

  # Spark Worker 1
  spark-worker-1:
    build:
      context: ./spark
      dockerfile: Dockerfile
    container_name: spark-worker-1
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
      - SPARK_WORKER_MEMORY=${SPARK_WORKER_MEMORY}
      - SPARK_WORKER_CORES=${SPARK_WORKER_CORES}
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
      - SPARK_WORKER_WEBUI_PORT=8081
    ports:
      - "8081:8081"
    volumes:
      - ./spark/scripts:/opt/spark-apps
      - ./data:/opt/spark-data
    networks:
      - youtube-network
    depends_on:
      - spark-master

  # Spark Worker 2
  spark-worker-2:
    build:
      context: ./spark
      dockerfile: Dockerfile
    container_name: spark-worker-2
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
      - SPARK_WORKER_MEMORY=${SPARK_WORKER_MEMORY}
      - SPARK_WORKER_CORES=${SPARK_WORKER_CORES}
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
      - SPARK_WORKER_WEBUI_PORT=8082
    ports:
      - "8082:8082"
    volumes:
      - ./spark/scripts:/opt/spark-apps
      - ./data:/opt/spark-data
    networks:
      - youtube-network
    depends_on:
      - spark-master

  # Jupyter Notebook
  jupyter:
    build:
      context: ./jupyter
      dockerfile: Dockerfile
    container_name: jupyter
    environment:
      - JUPYTER_ENABLE_LAB=yes
      - JUPYTER_TOKEN=${JUPYTER_TOKEN}
      - SPARK_MASTER_URL=${SPARK_MASTER_URL}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    ports:
      - "8888:8888"
    volumes:
      - ./jupyter/notebooks:/home/jovyan/work
      - ./data:/home/jovyan/data
    networks:
      - youtube-network
    depends_on:
      - postgres
      - spark-master
    command: start-notebook.sh --NotebookApp.token='${JUPYTER_TOKEN}'

  # Apache Airflow
  airflow:
    build:
      context: ./airflow
      dockerfile: Dockerfile
    container_name: airflow
    environment:
      - AIRFLOW_UID=${AIRFLOW_UID}
      - AIRFLOW__CORE__EXECUTOR=${AIRFLOW__CORE__EXECUTOR}
      - AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=${AIRFLOW__CORE__SQL_ALCHEMY_CONN}
      - AIRFLOW__CORE__FERNET_KEY=${AIRFLOW__CORE__FERNET_KEY}
      - AIRFLOW__CORE__LOAD_EXAMPLES=${AIRFLOW__CORE__LOAD_EXAMPLES}
      - AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION=false
      - YOUTUBE_API_KEY=${YOUTUBE_API_KEY}
      - KAFKA_BROKER=${KAFKA_BROKER}
      - KAFKA_TOPIC_INPUT=${KAFKA_TOPIC_INPUT}
      - KAFKA_TOPIC_OUTPUT=${KAFKA_TOPIC_OUTPUT}
    ports:
      - "8085:8080"
    volumes:
      - ./airflow/dags:/opt/airflow/dags
      - ./airflow/logs:/opt/airflow/logs
      - ./airflow/plugins:/opt/airflow/plugins
    networks:
      - youtube-network
    depends_on:
      postgres:
        condition: service_healthy
      kafka:
        condition: service_healthy
    command: >
      bash -c "
        airflow db init &&
        airflow users create \
          --username admin \
          --password admin \
          --firstname Admin \
          --lastname User \
          --role Admin \
          --email admin@example.com || true &&
        airflow connections add 'postgres_default' \
          --conn-type 'postgres' \
          --conn-host 'postgres' \
          --conn-schema '${POSTGRES_DB}' \
          --conn-login '${POSTGRES_USER}' \
          --conn-password '${POSTGRES_PASSWORD}' \
          --conn-port '5432' || true &&
        airflow webserver & airflow scheduler
      "

networks:
  youtube-network:
    driver: bridge

volumes:
  postgres-data:
  zookeeper-data:
  kafka-data:
```

---

## Step 7: Building and Testing {#step7}

### Step 7.1: Build All Docker Images

```bash
# Navigate to project root
cd youtube-pipeline

# Build all services
docker-compose build

# This will build:
# - Spark Master and Workers (from spark/Dockerfile)
# - Jupyter Notebook (from jupyter/Dockerfile)
# - Airflow (from airflow/Dockerfile)
# - PostgreSQL, Kafka, Zookeeper use official images
```

**Expected Output:**
```
Building spark-master
Step 1/8 : FROM bitnami/spark:3.5.0
...
Successfully built abc123def456
Successfully tagged youtube-pipeline_spark-master:latest

Building jupyter
Step 1/7 : FROM jupyter/pyspark-notebook:latest
...
Successfully built 789ghi012jkl
Successfully tagged youtube-pipeline_jupyter:latest

Building airflow
Step 1/8 : FROM apache/airflow:2.8.1-python3.11
...
Successfully built 345mno678pqr
Successfully tagged youtube-pipeline_airflow:latest
```

### Step 7.2: Start Infrastructure Services

```bash
# Start PostgreSQL, Zookeeper, and Kafka first
docker-compose up -d postgres zookeeper kafka

# Wait for services to be healthy (about 30 seconds)
docker-compose ps

# Check health status
docker-compose ps | grep healthy
```

**Verify Services:**
```bash
# Check PostgreSQL
docker-compose exec postgres pg_isready -U postgres

# Check Kafka topics
docker-compose exec kafka kafka-topics --list --bootstrap-server localhost:9092
```

### Step 7.3: Create Kafka Topics

```bash
# Create input topic for YouTube data
docker-compose exec kafka kafka-topics --create \
  --bootstrap-server localhost:9092 \
  --topic youtube-data \
  --partitions 3 \
  --replication-factor 1 \
  --if-not-exists

# Create output topic for processed data
docker-compose exec kafka kafka-topics --create \
  --bootstrap-server localhost:9092 \
  --topic processed-data \
  --partitions 3 \
  --replication-factor 1 \
  --if-not-exists

# Verify topics created
docker-compose exec kafka kafka-topics --list --bootstrap-server localhost:9092
```

**Expected Output:**
```
Created topic youtube-data.
Created topic processed-data.

youtube-data
processed-data
```

### Step 7.4: Start Spark Cluster

```bash
# Start Spark Master and Workers
docker-compose up -d spark-master spark-worker-1 spark-worker-2

# Wait for Spark to initialize (about 20 seconds)
sleep 20

# Check Spark Master UI
echo "Spark Master UI: http://localhost:8080"

# Verify workers connected
docker-compose logs spark-master | grep "Registering worker"
```

**Access Spark UIs:**
- Spark Master: http://localhost:8080
- Worker 1: http://localhost:8081
- Worker 2: http://localhost:8082

### Step 7.5: Start Jupyter Notebook

```bash
# Start Jupyter
docker-compose up -d jupyter

# Get Jupyter URL
docker-compose logs jupyter | grep "http://127.0.0.1:8888"
```

**Access Jupyter:**
- URL: http://localhost:8888
- Token: `spark123` (from .env file)

### Step 7.6: Start Airflow

```bash
# Start Airflow (this takes 2-3 minutes to initialize)
docker-compose up -d airflow

# Watch initialization
docker-compose logs -f airflow

# Wait for "Airflow is ready" message
```

**Access Airflow:**
- URL: http://localhost:8085
- Username: `admin`
- Password: `admin`

### Step 7.7: Verify All Services Running

```bash
# Check all containers
docker-compose ps

# Should show all services as "Up" or "Up (healthy)"
```

**Expected Services:**
```
NAME            STATUS          PORTS
postgres        Up (healthy)    0.0.0.0:5432->5432/tcp
zookeeper       Up (healthy)    0.0.0.0:2181->2181/tcp
kafka           Up (healthy)    0.0.0.0:9092-9093->9092-9093/tcp
spark-master    Up              0.0.0.0:7077->7077/tcp, 0.0.0.0:8080->8080/tcp
spark-worker-1  Up              0.0.0.0:8081->8081/tcp
spark-worker-2  Up              0.0.0.0:8082->8082/tcp
jupyter         Up              0.0.0.0:8888->8888/tcp
airflow         Up              0.0.0.0:8085->8080/tcp
```

---

## Step 8: Complete Integration and Testing {#step8}

### Step 8.1: Submit Spark Streaming Job

```bash
# Submit the Spark streaming application to process Kafka data
docker-compose exec spark-master spark-submit \
  --master spark://spark-master:7077 \
  --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0,org.postgresql:postgresql:42.6.0 \
  --driver-memory 1g \
  --executor-memory 1g \
  /opt/spark-apps/process_youtube_data.py
```

**Expected Output:**
```
Starting Spark Streaming Application...
Spark Streaming application started successfully!
Consuming from topic: youtube-data
Publishing to topic: processed-data
```

**Keep this terminal open** - Spark will continuously process streaming data.

### Step 8.2: Trigger Airflow DAG

Open a new terminal:

```bash
# Access Airflow UI and enable the DAG
# Go to http://localhost:8085
# Find 'youtube_kafka_spark_pipeline' DAG
# Toggle it to 'On'
# Click 'Trigger DAG' button

# OR trigger from command line:
docker-compose exec airflow airflow dags trigger youtube_kafka_spark_pipeline

# Monitor DAG execution
docker-compose exec airflow airflow dags list-runs -d youtube_kafka_spark_pipeline
```

**In Airflow UI, you should see:**
1. `create_tables` - Creates database schema ✓
2. `extract_youtube_data` - Fetches YouTube API data ✓
3. `publish_to_kafka` - Sends data to Kafka topic ✓
4. `load_to_postgres` - Loads data to PostgreSQL ✓

### Step 8.3: Monitor Kafka Messages

Open another terminal:

```bash
# Monitor messages in youtube-data topic
docker-compose exec kafka kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic youtube-data \
  --from-beginning \
  --max-messages 5

# Monitor processed data output
docker-compose exec kafka kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic processed-data \
  --from-beginning \
  --max-messages 5
```

**Expected Output (youtube-data topic):**
```json
{
  "channel_id": "UC_x5XG1OV2P6uZZ5FSM9Ttw",
  "channel_title": "Google for Developers",
  "subscriber_count": 1200000,
  "total_views": 95000000,
  "video_count": 3500,
  "timestamp": "2024-01-15T10:30:00"
}
```

### Step 8.4: Verify Data in PostgreSQL

```bash
# Connect to PostgreSQL
docker-compose exec postgres psql -U postgres -d youtube_db

# Run queries
\dt  # List tables

# View channel statistics
SELECT channel_title, subscriber_count, total_views 
FROM channel_stats 
ORDER BY subscriber_count DESC;

# View Spark aggregations
SELECT * FROM spark_aggregations 
ORDER BY window_start DESC 
LIMIT 5;

# Exit
\q
```

**Expected Tables:**
- `channel_stats` - Raw YouTube data
- `spark_aggregations` - Processed windowed aggregations

### Step 8.5: Analyze Data in Jupyter

1. **Open Jupyter Notebook**: http://localhost:8888 (token: spark123)

2. **Open `youtube_analysis.ipynb`**

3. **Run all cells** to see:
   - Database connection
   - Data loading from PostgreSQL
   - Subscriber count visualizations
   - Engagement metrics calculations

4. **Create a new cell** to query Spark aggregations:

```python
# Query Spark aggregated data
query = """
SELECT 
    window_start,
    channel_title,
    record_count,
    avg_subscribers,
    total_views_sum
FROM spark_aggregations
ORDER BY window_start DESC
LIMIT 10
"""
spark_df = pd.read_sql(query, engine)
spark_df
```

### Step 8.6: Monitor Spark Processing

**Watch Spark Console Output:**
```bash
# In the terminal where Spark job is running, you'll see:
```

**Sample Output:**
```
+-------------+-------------------------+------------------+------------------+
|channel_id   |channel_title            |engagement_ratio  |avg_views_per_video|
+-------------+-------------------------+------------------+------------------+
|UC_x5...     |Google for Developers    |79.16             |27142.86          |
|UCq-Fj...    |YouTube Creators         |95.23             |31250.50          |
|UCBJy...     |MKBHD                    |88.45             |125000.75         |
+-------------+-------------------------+------------------+------------------+
```

### Step 8.7: Complete Pipeline Test

**Test the full data flow:**

```bash
# 1. Trigger Airflow DAG again
docker-compose exec airflow airflow dags trigger youtube_kafka_spark_pipeline

# 2. Watch Kafka messages arrive
docker-compose exec kafka kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic youtube-data \
  --from-beginning

# 3. Check Spark processing logs
docker-compose logs -f spark-master

# 4. Verify data updated in PostgreSQL
docker-compose exec postgres psql -U postgres -d youtube_db \
  -c "SELECT channel_title, processed_at FROM channel_stats ORDER BY processed_at DESC;"

# 5. Refresh Jupyter notebook and see updated data
```

---

## Step 9: Useful Commands and Maintenance

### Managing Services

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (clean slate)
docker-compose down -v

# Restart a specific service
docker-compose restart kafka

# View logs for specific service
docker-compose logs -f spark-master
docker-compose logs -f airflow
docker-compose logs -f kafka

# Check resource usage
docker stats
```

### Kafka Management

```bash
# List all topics
docker-compose exec kafka kafka-topics --list --bootstrap-server localhost:9092

# Describe a topic
docker-compose exec kafka kafka-topics --describe \
  --topic youtube-data \
  --bootstrap-server localhost:9092

# Delete a topic
docker-compose exec kafka kafka-topics --delete \
  --topic youtube-data \
  --bootstrap-server localhost:9092

# Check consumer group lag
docker-compose exec kafka kafka-consumer-groups \
  --bootstrap-server localhost:9092 \
  --describe \
  --group spark-streaming-group
```

### Spark Management

```bash
# Submit Spark job with different configurations
docker-compose exec spark-master spark-submit \
  --master spark://spark-master:7077 \
  --deploy-mode client \
  --executor-memory 2G \
  --total-executor-cores 4 \
  --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0,org.postgresql:postgresql:42.6.0 \
  /opt/spark-apps/process_youtube_data.py

# Check Spark application status
docker-compose exec spark-master spark-class org.apache.spark.deploy.Client \
  --master spark://spark-master:7077 \
  --status <driver-id>

# Kill a Spark application
docker-compose exec spark-master spark-class org.apache.spark.deploy.Client \
  --master spark://spark-master:7077 \
  --kill <driver-id>
```

### Database Management

```bash
# Backup PostgreSQL database
docker-compose exec postgres pg_dump -U postgres youtube_db > backup.sql

# Restore database
cat backup.sql | docker-compose exec -T postgres psql -U postgres -d youtube_db

# Reset database
docker-compose exec postgres psql -U postgres -d youtube_db \
  -c "DROP TABLE IF EXISTS channel_stats, spark_aggregations CASCADE;"
```

---

## Troubleshooting

### Issue 1: Kafka Cannot Connect

**Problem**: `Connection refused` errors when connecting to Kafka

**Solution**:
```bash
# Check if Kafka is healthy
docker-compose ps kafka

# Check Kafka logs
docker-compose logs kafka

# Ensure Zookeeper is running first
docker-compose up -d zookeeper
sleep 10
docker-compose up -d kafka
```

### Issue 2: Spark Job Fails

**Problem**: Spark streaming job fails with `ClassNotFoundException`

**Solution**:
```bash
# Make sure Kafka packages are included in spark-submit
docker-compose exec spark-master spark-submit \
  --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0,org.postgresql:postgresql:42.6.0 \
  /opt/spark-apps/process_youtube_data.py
```

### Issue 3: Airflow DAG Not Running

**Problem**: DAG doesn't trigger or shows errors

**Solution**:
```bash
# Check Airflow logs
docker-compose logs airflow | tail -100

# Test DAG syntax
docker-compose exec airflow python /opt/airflow/dags/youtube_pipeline.py

# Check database connection
docker-compose exec airflow airflow connections list

# Recreate connection
docker-compose exec airflow airflow connections delete postgres_default
docker-compose exec airflow airflow connections add 'postgres_default' \
  --conn-type 'postgres' \
  --conn-host 'postgres' \
  --conn-schema 'youtube_db' \
  --conn-login 'postgres' \
  --conn-password 'postgres123' \
  --conn-port '5432'
```

### Issue 4: Out of Memory

**Problem**: Containers crash with OOM errors

**Solution**:
```bash
# Reduce Spark worker memory in .env
SPARK_WORKER_MEMORY=1G

# Restart services
docker-compose restart spark-worker-1 spark-worker-2

# Check Docker resources
docker system df
docker system prune -a
```

### Issue 5: Port Already in Use

**Problem**: `Port is already allocated` error

**Solution**:
```bash
# Find process using the port (e.g., 8080)
lsof -i :8080

# Kill the process
kill -9 <PID>

# Or change port in docker-compose.yml
# Change "8080:8080" to "8090:8080"
```

### Issue 6: Jupyter Cannot Connect to PostgreSQL

**Problem**: `Connection refused` in Jupyter notebooks

**Solution**:
```python
# Use container name instead of localhost
engine = create_engine('postgresql://postgres:postgres123@postgres:5432/youtube_db')

# Verify network connectivity
!ping -c 3 postgres
```

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                          YouTube Data Pipeline                       │
└─────────────────────────────────────────────────────────────────────┘

   YouTube API
       │
       ▼
┌──────────────┐
│   Airflow    │──── Orchestrates Pipeline (Every 6 hours)
│  (Port 8085) │
└──────┬───────┘
       │
       ├──────────────────┬─────────────────────┐
       ▼                  ▼                     ▼
  Extract Data       Publish to          Load to
  from YouTube       Kafka Topic         PostgreSQL
       │                  │                     │
       ▼                  ▼                     ▼
┌──────────────┐    ┌──────────┐      ┌──────────────┐
│    Kafka     │◄───│  Spark   │─────►│  PostgreSQL  │
│  (Port 9092) │    │Processing│      │  (Port 5432) │
└──────────────┘    └──────────┘      └──────┬───────┘
   Topic: youtube-data                        │
   Topic: processed-data                      │
       │                                      │
       │                                      ▼
       │                              ┌──────────────┐
       └────────────────────────────►│   Jupyter    │
                                      │ (Port 8888)  │
                                      └──────────────┘
                                      Analysis & Viz
```

---

## Summary of Services and Ports

| Service | Port | Access URL | Purpose |
|---------|------|------------|---------|
| Airflow | 8085 | http://localhost:8085 | Workflow orchestration |
| Spark Master | 8080 | http://localhost:8080 | Spark cluster manager |
| Spark Worker 1 | 8081 | http://localhost:8081 | Processing node |
| Spark Worker 2 | 8082 | http://localhost:8082 | Processing node |
| Jupyter | 8888 | http://localhost:8888 | Data analysis |
| PostgreSQL | 5432 | localhost:5432 | Data storage |
| Kafka | 9092/9093 | kafka:9092 (internal) | Message streaming |
| Zookeeper | 2181 | zookeeper:2181 | Kafka coordination |

---

## Next Steps

1. **Add More Channels**: Edit `CHANNEL_IDS` in `youtube_pipeline.py`

2. **Custom Spark Processing**: Modify `process_youtube_data.py` to add:
   - Sentiment analysis
   - Trend detection
   - Anomaly detection

3. **Advanced Analytics**: Create more Jupyter notebooks for:
   - Time-series forecasting
   - Comparative analysis
   - Growth predictions

4. **Production Deployment**:
   - Add Kubernetes manifests
   - Implement monitoring (Prometheus/Grafana)
   - Add CI/CD pipeline
   - Secure with TLS/SSL

5. **Scaling**:
   - Add more Spark workers
   - Increase Kafka partitions
   - Implement caching layer (Redis)

---

## Conclusion

You now have a complete, Dockerized data pipeline with:

✅ Apache Kafka for real-time streaming

✅ Apache Spark for distributed processing 

✅ Jupyter for interactive analysis

✅ Airflow for workflow orchestration

✅ PostgreSQL for data persistence

**Congratulations!** You've successfully dockerized Spark, Kafka, and Jupyter for a production-grade data pipeline.
