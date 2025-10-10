
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cjs2113bt4ioss1mqpwy.png)



## End-to-End YouTube Channel Analytics Pipeline
## Complete Implementation Guide

**Student Name:** Lagat Josiah 
**Date:** October 10, 2025  
**Channel Analyzed:** [Channel Name] (ID: UC_x5XG1OV2P6uZZ5FSM9Ttw)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Project Architecture](#project-architecture)
3. [Prerequisites & Environment Setup](#prerequisites--environment-setup)
4. [Phase 1: Data Ingestion](#phase-1-data-ingestion)
5. [Phase 2: Data Processing with PySpark](#phase-2-data-processing-with-pyspark)
6. [Phase 3: Workflow Orchestration with Airflow](#phase-3-workflow-orchestration-with-airflow)
7. [Phase 4: Visualization with Grafana](#phase-4-visualization-with-grafana)
8. [Phase 5: Containerization with Docker](#phase-5-containerization-with-docker)
9. [Results & Insights](#results--insights)
10. [Challenges & Solutions](#challenges--solutions)
11. [Conclusion & Future Work](#conclusion--future-work)

---

## Executive Summary

This project demonstrates the implementation of a complete data engineering pipeline for YouTube channel analytics. The solution ingests data from YouTube Data API v3, processes it using Apache Spark, orchestrates workflows with Apache Airflow, visualizes insights through Grafana dashboards, and packages everything using Docker containers.

**Key Technologies:**
- **Data Ingestion:** YouTube Data API v3, Python
- **Data Processing:** Apache Spark (PySpark)
- **Orchestration:** Apache Airflow 3.0.0
- **Visualization:** Grafana
- **Storage:** Aiven PostgreSQL (Cloud)
- **Containerization:** Docker & Docker Compose

---

## Project Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    YouTube Data API v3                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Data Ingestion Layer (Python)                   │
│  • Fetch channel statistics                                  │
│  • Fetch video metadata & metrics                            │
│  • Store raw data as JSON                                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│           Processing Layer (Apache Spark)                    │
│  • Load JSON data into DataFrames                            │
│  • Transform timestamps                                      │
│  • Calculate engagement metrics                              │
│  • Feature engineering                                       │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Storage Layer (Aiven PostgreSQL)                │
│  • channel_stats table                                       │
│  • video_stats table                                         │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│          Orchestration Layer (Apache Airflow)                │
│  DAG: Extract → Transform → Load → Validate                  │
│  Schedule: Daily execution                                   │
└─────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│           Visualization Layer (Grafana)                      │
│  • Top videos dashboard                                      │
│  • Engagement metrics                                        │
│  • Publishing patterns analysis                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Prerequisites & Environment Setup

### System Requirements

- **Operating System:** WSL2 (Ubuntu 20.04+) on Windows
- **RAM:** Minimum 4GB (8GB recommended)
- **Storage:** 10GB free space
- **Software:**
  - Docker Engine 20.10+
  - Docker Compose v2+
  - Python 3.11+
  - Java JDK 17 (for PySpark)

### Initial Setup Commands

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install docker.io -y
sudo usermod -aG docker $USER
newgrp docker

# Verify installations
docker --version
python3 --version
```

### Project Structure

```
youtube-analytics-pipeline/
├── dags/
│   └── youtube_analytics_dag.py
├── scripts/
│   ├── youtube_ingestion.py
│   └── spark_processing.py
├── data/
│   ├── raw/
│   └── processed/
├── config/
├── logs/
├── plugins/
├── docker-compose.yaml
├── Dockerfile
├── requirements.txt
└── .env
```

---

## Phase 1: Data Ingestion

### 1.1 YouTube API Setup

**Steps to obtain API credentials:**

1. Navigate to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project: "YouTube Analytics Pipeline"
3. Enable "YouTube Data API v3"
4. Create API credentials (API Key)
5. Note the API key securely

### 1.2 Channel Selection

**Selected Channel Details:**
- **Channel ID:** UC_x5XG1OV2P6uZZ5FSM9Ttw
- **Rationale:** [Explain why you chose this channel]

### 1.3 Data Ingestion Implementation

**File:** `scripts/youtube_ingestion.py`

**Key Features:**
- Fetches channel-level statistics (subscribers, total views, video count)
- Retrieves video-level data (title, publish date, views, likes, comments)
- Implements pagination for large datasets
- Saves raw data as JSON with timestamps

**Code Structure:**

```python
def fetch_channel_data(api_key, channel_id):
    """Fetch channel statistics from YouTube API"""
    # Implementation details in source file

def fetch_videos_data(api_key, channel_id, max_results=50):
    """Fetch video data with pagination"""
    # Implementation details in source file

def save_data(data, filename):
    """Save data to JSON file with timestamp"""
    # Implementation details in source file
```

**Sample Output:**
```
data/raw/
├── channel_data_20251010_120000.json
└── videos_data_20251010_120000.json
```

---

## Phase 2: Data Processing with PySpark

### 2.1 Spark Session Configuration

**File:** `scripts/spark_processing.py`

**Configuration:**
- Application Name: "YouTubeAnalytics"
- JDBC Driver: PostgreSQL JDBC 42.7.1
- SSL enabled for secure connections

### 2.2 Data Transformations

**Channel Data Processing:**
- Extract channel metadata
- Parse statistics (subscribers, views, video count)
- Add processing timestamp

**Video Data Processing:**
1. **Timestamp Conversion:**
   - Convert ISO 8601 timestamps to datetime objects
   - Extract hour and day of week for analysis

2. **Feature Engineering:**
   - Calculate engagement rate: `(likes + comments) / views * 100`
   - Handle missing values (default to 0)
   - Parse video duration

3. **Data Quality:**
   - Remove null values
   - Validate numeric fields
   - Ensure data consistency

### 2.3 Data Schema

**channel_stats table:**
```
├── channel_id (string)
├── channel_name (string)
├── subscribers (integer)
├── total_views (integer)
├── video_count (integer)
└── fetch_date (timestamp)
```

**video_stats table:**
```
├── video_id (string)
├── title (string)
├── published_at (string)
├── published_datetime (timestamp)
├── publish_hour (integer)
├── publish_day (integer)
├── views (integer)
├── likes (integer)
├── comments (integer)
├── duration (string)
└── engagement_rate (double)
```

### 2.4 Storage Strategy

**Primary Storage:** Aiven PostgreSQL (Cloud)
- Host: pg-c63647-lagatkjosiah-692c.c.aivencloud.com
- Port: 24862
- Database: defaultdb
- SSL Mode: Required

**Backup Storage:** Parquet files
- Location: `data/processed/`
- Format: Apache Parquet (columnar storage)

---

## Phase 3: Workflow Orchestration with Airflow

### 3.1 Airflow Architecture

**Executor:** LocalExecutor (simplified for development)

**Components:**
- API Server (port 8080)
- Scheduler
- DAG Processor
- Triggerer

### 3.2 DAG Implementation

**File:** `dags/youtube_analytics_dag.py`

**DAG Configuration:**
```python
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2025, 1, 1),
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}
```

**Schedule:** Daily execution (`@daily`)

### 3.3 Task Pipeline

```
┌─────────────────┐
│  Extract Task   │  ← Fetch data from YouTube API
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Transform Task  │  ← Process data with PySpark
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Validate Task   │  ← Verify data integrity
└─────────────────┘
```

**Task Definitions:**

1. **extract_youtube_data:** 
   - Calls YouTube API
   - Saves raw JSON
   - Returns timestamp

2. **transform_with_pyspark:**
   - Loads raw data
   - Applies transformations
   - Saves to PostgreSQL

3. **validate_data:**
   - Confirms successful execution
   - Logs completion status

### 3.4 Monitoring & Logging

**Access:** http://localhost:8080
- Username: airflow
- Password: airflow

**Features:**
- Visual DAG graph
- Task execution logs
- Retry mechanisms
- Execution history

---

## Phase 4: Visualization with Grafana

### 4.1 Data Source Configuration

**Connection Details:**
```
Name: Aiven YouTube Analytics
Type: PostgreSQL
Host: pg-c63647-lagatkjosiah-692c.c.aivencloud.com:24862
Database: defaultdb
User: avnadmin
SSL Mode: require
```

### 4.2 Dashboard 1: Top Performing Videos

**Query:**
```sql
SELECT 
  title,
  views,
  likes,
  comments,
  engagement_rate
FROM video_stats
ORDER BY views DESC
LIMIT 10
```

**Visualization:** Bar Chart
- X-axis: Video Title
- Y-axis: View Count
- Color: Engagement Rate (gradient)

**Insights:**
- Identifies highest performing content
- Correlates views with engagement
- Helps understand audience preferences

### 4.3 Dashboard 2: Engagement Rate Trends

**Query:**
```sql
SELECT 
  DATE(published_datetime) as date,
  AVG(engagement_rate) as avg_engagement,
  SUM(views) as total_views,
  COUNT(*) as video_count
FROM video_stats
GROUP BY DATE(published_datetime)
ORDER BY date DESC
LIMIT 30
```

**Visualization:** Time Series Graph
- X-axis: Date
- Y-axis: Average Engagement Rate
- Secondary Y-axis: Total Views

**Insights:**
- Tracks engagement over time
- Identifies trends and patterns
- Reveals optimal posting times

### 4.4 Dashboard 3: Publishing Time Analysis

**Query:**
```sql
SELECT 
  publish_day,
  publish_hour,
  COUNT(*) as video_count,
  AVG(views) as avg_views,
  AVG(engagement_rate) as avg_engagement
FROM video_stats
GROUP BY publish_day, publish_hour
ORDER BY publish_day, publish_hour
```

**Visualization:** Heatmap
- X-axis: Hour of Day (0-23)
- Y-axis: Day of Week (1-7)
- Color Intensity: Average Views or Engagement

**Insights:**
- Optimal publishing schedule
- Audience activity patterns
- Best times for maximum reach

### 4.5 Dashboard 4: Channel Growth Metrics

**Query:**
```sql
SELECT 
  fetch_date,
  subscribers,
  total_views,
  video_count
FROM channel_stats
ORDER BY fetch_date DESC
```

**Visualization:** Multi-line Graph

**Insights:**
- Subscriber growth trajectory
- View accumulation rate
- Content production velocity

---

## Phase 5: Containerization with Docker

### 5.1 Dockerfile

**Base Image:** apache/airflow:3.0.0

**Key Components:**
```dockerfile
# Install Java 17 for PySpark
RUN apt-get install -y openjdk-17-jdk-headless

# Install Python dependencies
RUN pip install --no-cache-dir -r /requirements.txt

# Download PostgreSQL JDBC driver
RUN curl -L https://jdbc.postgresql.org/download/postgresql-42.7.1.jar
```

### 5.2 Docker Compose Services

**Services Architecture:**

1. **postgres** - Airflow metadata database
2. **redis** - Message broker for Celery
3. **airflow-apiserver** - Web UI (port 8080)
4. **airflow-scheduler** - Task scheduler
5. **airflow-dag-processor** - DAG parser
6. **airflow-triggerer** - Event-driven tasks
7. **grafana** - Visualization (port 3000)

### 5.3 Volume Mounts

```yaml
volumes:
  - ./dags:/opt/airflow/dags
  - ./logs:/opt/airflow/logs
  - ./scripts:/opt/airflow/scripts
  - ./data:/opt/airflow/data
  - ./config:/opt/airflow/config
  - ./plugins:/opt/airflow/plugins
```

### 5.4 Environment Variables

**Configuration via .env file:**
```
YOUTUBE_API_KEY=<your_api_key>
YOUTUBE_CHANNEL_ID=UC_x5XG1OV2P6uZZ5FSM9Ttw
POSTGRES_USER=avnadmin
POSTGRES_PASSWORD=<password>
POSTGRES_HOST=pg-c63647-lagatkjosiah-692c.c.aivencloud.com
POSTGRES_PORT=24862
POSTGRES_DB=defaultdb
POSTGRES_SSL_MODE=require
```

### 5.5 Deployment Commands

```bash
# Build custom image
docker compose build

# Start all services
docker compose up -d

# Check service health
docker compose ps

# View logs
docker compose logs -f

# Stop services
docker compose down
```

---

## Results & Insights

### Key Findings

#### 1. Content Performance Analysis

**Top 3 Videos by Views:**
1. [Video Title] - [Views] views - [Engagement Rate]%
2. [Video Title] - [Views] views - [Engagement Rate]%
3. [Video Title] - [Views] views - [Engagement Rate]%

**Observations:**
- [Describe patterns in top-performing content]
- [Common characteristics of successful videos]

#### 2. Engagement Metrics

**Average Engagement Rate:** [X]%
- Likes per 1000 views: [X]
- Comments per 1000 views: [X]
- Engagement trend: [Increasing/Decreasing/Stable]

#### 3. Publishing Strategy Insights

**Optimal Publishing Times:**
- Best Day: [Day of week]
- Best Hour: [Hour range]
- Peak Engagement Window: [Time range]

**Content Frequency:**
- Average videos per week: [X]
- Most productive month: [Month]

#### 4. Channel Growth Trajectory

**Growth Metrics (Last 30 days):**
- Subscriber growth: [+X%]
- View growth: [+X%]
- Content output: [X videos]

---

## Challenges & Solutions

### Challenge 1: Docker Build Segmentation Fault

**Problem:** Docker Compose build command failed with segmentation fault

**Root Cause:** Conflict between BuildKit and legacy builder

**Solution:**
```bash
DOCKER_BUILDKIT=0 docker compose build
```

**Learning:** WSL2 environment requires specific Docker configurations

### Challenge 2: Celery Worker Not Starting

**Problem:** Tasks stuck in "queued" state, worker not processing

**Root Cause:** CeleryExecutor complexity in development environment

**Solution:** Switched to LocalExecutor
```yaml
AIRFLOW__CORE__EXECUTOR: LocalExecutor
```

**Learning:** Choose appropriate executor based on deployment scale

### Challenge 3: Environment Variables Not Loading

**Problem:** YouTube API credentials not accessible in containers

**Root Cause:** Variables not passed through docker-compose.yaml

**Solution:** Added explicit environment mapping
```yaml
environment:
  YOUTUBE_API_KEY: ${YOUTUBE_API_KEY}
  YOUTUBE_CHANNEL_ID: ${YOUTUBE_CHANNEL_ID}
```

**Learning:** Container isolation requires explicit configuration

### Challenge 4: PySpark JDBC Driver Missing

**Problem:** Spark couldn't connect to PostgreSQL

**Root Cause:** PostgreSQL JDBC driver not included in base image

**Solution:** Added driver download in Dockerfile
```dockerfile
RUN curl -L https://jdbc.postgresql.org/download/postgresql-42.7.1.jar
```

**Learning:** Custom images must include all dependencies

---

## Technical Specifications

### Performance Metrics

- **Data Ingestion Speed:** ~100 videos/minute
- **Spark Processing Time:** 15-30 seconds (100 videos)
- **DAG Execution Time:** 2-3 minutes (full pipeline)
- **Storage Efficiency:** Parquet reduces size by ~70%

### Scalability Considerations

- **Current Capacity:** 1000+ videos per run
- **API Quota:** 10,000 units/day (YouTube API)
- **Database Scaling:** Horizontal with Aiven PostgreSQL
- **Container Scaling:** Docker Swarm or Kubernetes ready

### Security Measures

- SSL/TLS encryption for all database connections
- API keys stored in environment variables (not in code)
- Aiven managed PostgreSQL with automatic backups
- Docker network isolation between services

---

## Conclusion & Future Work

### Project Achievements

✅ Successfully implemented end-to-end data pipeline  
✅ Automated daily data collection and processing  
✅ Created real-time analytics dashboards  
✅ Containerized entire solution for portability  
✅ Demonstrated proficiency in modern data engineering tools

### Learning Outcomes

1. **Data Engineering:** Designed and implemented ETL pipeline
2. **Cloud Integration:** Utilized managed database services (Aiven)
3. **Orchestration:** Mastered Apache Airflow workflow management
4. **Containerization:** Built production-ready Docker setup
5. **Visualization:** Created actionable business intelligence dashboards

### Future Enhancements

#### Short-term Improvements
1. **Sentiment Analysis:** Add NLP processing for video comments
2. **Competitor Analysis:** Compare multiple channels
3. **Alert System:** Email notifications for anomalies
4. **API Rate Limiting:** Intelligent quota management

#### Long-term Roadmap
1. **Machine Learning:** Predictive models for view forecasting
2. **Real-time Processing:** Apache Kafka for streaming data
3. **Advanced Analytics:** Audience demographic insights
4. **Mobile Dashboard:** Grafana mobile app integration
5. **Multi-cloud Deployment:** AWS/Azure/GCP compatibility

### Business Value

This pipeline provides:
- **Data-driven decision making** for content strategy
- **Automated reporting** saving hours of manual work
- **Trend identification** for competitive advantage
- **Scalable infrastructure** supporting business growth

---

## References & Resources

### Documentation
- [YouTube Data API v3 Documentation](https://developers.google.com/youtube/v3)
- [Apache Airflow Official Docs](https://airflow.apache.org/docs/)
- [PySpark Documentation](https://spark.apache.org/docs/latest/api/python/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Docker Compose Specification](https://docs.docker.com/compose/)

### Tools & Technologies
- Python 3.11
- Apache Airflow 3.0.0
- Apache Spark 3.5.0
- PostgreSQL 13
- Docker Engine 20.10+
- Grafana 10.x

### Source Code Repository
[Link to GitHub/GitLab repository with complete code]

---

## Appendix

### A. Complete File Listings

**requirements.txt:**
```
google-api-python-client==2.108.0
pyspark==3.5.0
psycopg2-binary==2.9.9
py4j==0.10.9.7
```

### B. Useful Commands Reference

```bash
# Start pipeline
docker compose up -d

# Stop pipeline
docker compose down

# View logs
docker compose logs -f [service_name]

# Rebuild after changes
docker compose build --no-cache

# Access Airflow CLI
docker compose exec airflow-scheduler airflow dags list

# Trigger DAG manually
docker compose exec airflow-scheduler airflow dags trigger youtube_analytics_pipeline
```

### C. Troubleshooting Guide

| Issue | Solution |
|-------|----------|
| DAG not appearing | Check `dags/` folder permissions |
| Task fails | Review logs in Airflow UI |
| Connection timeout | Verify network connectivity |
| Out of memory | Increase Docker memory allocation |

---

**Presentation Date:** 10/10/2025  
**Duration:** 15 minutes  
**Q&A:** 5 minutes

---

*This project demonstrates comprehensive data engineering skills including data ingestion, processing, orchestration, visualization, and containerization using industry-standard tools and best practices.*
