# YouTube Analytics Pipeline Using Delta Lake and PySpark.

Complete data pipeline for extracting, processing, and analyzing data from multiple YouTube channels using Delta Lake and PySpark.

## 🎯 Overview

This pipeline extracts data from YouTube API, processes it with PySpark, stores it in Delta Lake with medallion architecture, and generates comprehensive analytics.

## 📊 Data Flow

```
YouTube API → Extract → Process (PySpark) → Delta Lake (Bronze/Silver/Gold) → Analytics
```

### Pipeline Stages

1. **Extract Channels** - Channel statistics and metadata
2. **Extract Videos** - Video details, views, likes, comments count
3. **Extract Comments** - User comments and engagement metrics
4. **Process Data** - Clean, transform, and enrich with PySpark
5. **Write to Delta Lake** - Store in medallion architecture
6. **Generate Analytics** - Create insights and reports
7. **Quality Checks** - Validate data quality

## 🔑 YouTube API Setup

### Get Your API Key

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project or select existing
3. Enable YouTube Data API v3
4. Go to Credentials → Create Credentials → API Key
5. Copy your API key

### Add to Environment

```bash
# Edit .env file
nano .env

# Add your API key
YOUTUBE_API_KEY=your_actual_api_key_here
```

### API Quota Limits

- **Free tier**: 10,000 units/day
- **Channel read**: 1 unit
- **Video list**: 1 unit per 50 videos
- **Comments**: 1 unit per request
- Pipeline uses ~200-500 units per run (depending on configuration)

## 📺 Configured Channels

The pipeline tracks these channels by default:

- **Google Developers** - Technology and development
- **Fireship** - Quick coding tutorials
- **freeCodeCamp** - Programming education
- **The Net Ninja** - Web development
- **Web Dev Simplified** - Modern web development

### Add More Channels

Edit `scripts/extract_youtube_channels.py`:

```python
YOUTUBE_CHANNELS = [
    {
        'channel_id': 'UCxxxxxxxxxxxxxx',  # Get from channel URL
        'channel_name': 'Channel Name'
    },
    # Add more...
]
```

To find channel ID:
1. Go to channel page
2. View page source
3. Search for `"channelId":"`
4. Or use: `https://www.youtube.com/channel/CHANNEL_ID`

## 🚀 Running the Pipeline

### Option 1: Via Airflow UI

1. Navigate to http://localhost:8080
2. Login (airflow/airflow)
3. Find `youtube_analytics_pipeline` DAG
4. Toggle to enable
5. Click "Trigger DAG"

### Option 2: Via Command Line

```bash
# Trigger the complete pipeline
docker compose exec airflow-webserver airflow dags trigger youtube_analytics_pipeline

# Check status
docker compose exec airflow-webserver airflow dags list-runs -d youtube_analytics_pipeline

# View task logs
docker compose exec airflow-webserver airflow tasks logs youtube_analytics_pipeline extract_youtube_channels latest
```

### Option 3: Run Scripts Individually

```bash
# 1. Extract channels
docker compose exec spark-master python /opt/spark-scripts/extract_youtube_channels.py

# 2. Extract videos
docker compose exec spark-master python /opt/spark-scripts/extract_youtube_videos.py

# 3. Extract comments
docker compose exec spark-master python /opt/spark-scripts/extract_youtube_comments.py

# 4. Process data
docker compose exec spark-master spark-submit \
  --master spark://spark-master:7077 \
  --packages io.delta:delta-spark_2.12:3.0.0 \
  --conf "spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension" \
  --conf "spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog" \
  /opt/spark-scripts/process_youtube_data.py

# 5. Write to Delta Lake
docker compose exec spark-master spark-submit \
  --master spark://spark-master:7077 \
  --packages io.delta:delta-spark_2.12:3.0.0 \
  --conf "spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension" \
  --conf "spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog" \
  /opt/spark-scripts/write_youtube_delta.py

# 6. Generate analytics
docker compose exec spark-master spark-submit \
  --master spark://spark-master:7077 \
  --packages io.delta:delta-spark_2.12:3.0.0 \
  --conf "spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension" \
  --conf "spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog" \
  /opt/spark-scripts/youtube_analytics.py

# 7. Run quality checks
docker compose exec spark-master python /opt/spark-scripts/youtube_quality_checks.py
```

## 📁 Data Structure

### Delta Lake Tables

#### Bronze Layer (Raw Data)
- `bronze_youtube_channels` - Channel metadata
- `bronze_youtube_videos` - Video details (partitioned by year/month)
- `bronze_youtube_comments` - User comments

#### Silver Layer (Cleaned Data)
- `silver_youtube_channels` - Validated channels
- `silver_youtube_videos` - Enriched video data
- `silver_youtube_comments` - Processed comments

#### Gold Layer (Analytics)
- `gold_youtube_channel_performance` - Channel metrics
- `gold_youtube_duration_performance` - Video length analysis
- `gold_youtube_monthly_trends` - Publishing patterns
- `gold_youtube_comment_activity` - Engagement metrics

## 📈 Analytics Generated

### 1. Top Performing Channels
- Subscriber count
- Total views
- Average engagement rate
- Content velocity

### 2. Video Duration Analysis
- Optimal video length
- Performance by duration category
- Engagement patterns

### 3. Content Trends
- Publishing frequency
- Monthly performance
- Growth patterns

### 4. Engagement Insights
- Like-to-view ratio
- Comment activity
- Viral content identification

### 5. Comment Analysis
- Sentiment patterns
- Question detection
- User engagement levels

## 📊 Viewing Results

### Option 1: Check Reports

```bash
# View analytics report
cat data/reports/youtube_analytics_report.json | jq .

# View quality report
cat data/reports/youtube_quality_report.json | jq .
```

### Option 2: Query Delta Lake (Jupyter)

Access Jupyter at http://localhost:8888

```python
from pyspark.sql import SparkSession
from delta import *

spark = SparkSession.builder \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Read channel performance
df = spark.read.format("delta").load("/opt/delta-lake/gold_youtube_channel_performance")
df.show()

# Top videos by engagement
videos = spark.read.format("delta").load("/opt/delta-lake/silver_youtube_videos")
videos.orderBy("engagement_rate", ascending=False).select("title", "view_count", "engagement_rate").show(10)
```

### Option 3: SQL Queries

```python
# Create temp views
spark.read.format("delta").load("/opt/delta-lake/silver_youtube_videos").createOrReplaceTempView("videos")

# Query with SQL
spark.sql("""
    SELECT 
        channel_title,
        COUNT(*) as video_count,
        AVG(view_count) as avg_views,
        AVG(engagement_rate) as avg_engagement
    FROM videos
    GROUP BY channel_title
    ORDER BY avg_views DESC
""").show()
```

## 🔍 Monitoring

### Check Pipeline Status

```bash
# View Airflow DAG runs
docker compose logs airflow-scheduler | grep youtube_analytics

# Check Spark job status
docker compose logs spark-master | tail -50

# View extraction logs
docker compose exec spark-master ls -lh /opt/spark-data/raw/youtube/
```

### Data Quality Metrics

The quality checks validate:

- ✅ No null values in critical fields
- ✅ No duplicate records
- ✅ Valid data ranges (no negative counts)
- ✅ Referential integrity
- ✅ Data freshness

## ⚙️ Configuration

### Adjust API Limits

Edit script files to control API usage:

```python
# scripts/extract_youtube_videos.py
MAX_VIDEOS_PER_CHANNEL = 50  # Reduce to save quota

# scripts/extract_youtube_comments.py
MAX_COMMENTS_PER_VIDEO = 100  # Reduce to save quota
```

### Schedule Changes

Edit `dags/youtube_analytics_pipeline.py`:

```python
dag = DAG(
    'youtube_analytics_pipeline',
    schedule_interval='@daily',  # Change to @weekly, @hourly, etc.
    ...
)
```

## 🐛 Troubleshooting

### API Key Issues

```bash
# Test API key
docker compose exec spark-master python3 << 'EOF'
import os
import requests
from dotenv import load_dotenv

load_dotenv()
api_key = os.getenv('YOUTUBE_API_KEY')

if not api_key:
    print("ERROR: YOUTUBE_API_KEY not set!")
else:
    # Test API call
    response = requests.get(
        'https://www.googleapis.com/youtube/v3/channels',
        params={'part': 'snippet', 'id': 'UC_x5XG1OV2P6uZZ5FSM9Ttw', 'key': api_key}
    )
    if response.status_code == 200:
        print("✓ API key is valid!")
    else:
        print(f"✗ API error: {response.status_code}")
        print(response.json())
EOF
```

**Common API Errors:**
- `403` - Invalid API key or quota exceeded
- `400` - Malformed request
- `404` - Channel/video not found

### No Data Extracted

```bash
# Check if files were created
ls -lh data/raw/youtube/

# View extraction logs
docker compose logs spark-master | grep "EXTRACTING"

# Run without API key (uses sample data)
unset YOUTUBE_API_KEY
docker compose exec spark-master python /opt/spark-scripts/extract_youtube_channels.py
```

### Spark Job Failures

```bash
# Check Spark master status
docker compose ps spark-master

# View Spark logs
docker compose logs spark-master

# Restart Spark
docker compose restart spark-master spark-worker

# Check memory allocation
docker stats spark-master spark-worker
```

### Delta Lake Errors

```bash
# Check Delta Lake directory
ls -lh delta-lake/

# Verify Delta tables
docker compose exec spark-master python3 << 'EOF'
from delta.tables import DeltaTable
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

try:
    delta_table = DeltaTable.forPath(spark, "/opt/delta-lake/bronze_youtube_channels")
    print("✓ Delta table exists")
    delta_table.history(1).show()
except Exception as e:
    print(f"✗ Error: {e}")
EOF
```

## 📚 Example Use Cases

### 1. Content Strategy Analysis

```python
# Find optimal publishing time
videos_df = spark.read.format("delta").load("/opt/delta-lake/silver_youtube_videos")

videos_df.groupBy("published_month") \
    .agg({"view_count": "avg", "engagement_rate": "avg"}) \
    .orderBy("published_month") \
    .show()
```

### 2. Competitor Analysis

```python
# Compare channel performance
channels_df = spark.read.format("delta").load("/opt/delta-lake/silver_youtube_channels")

channels_df.select(
    "channel_name",
    "subscriber_count",
    "view_count",
    "avg_views_per_video"
).orderBy("subscriber_count", ascending=False).show()
```

### 3. Engagement Pattern Discovery

```python
# Find high-engagement videos
videos_df.filter("view_count > 10000") \
    .orderBy("engagement_rate", ascending=False) \
    .select("title", "channel_title", "view_count", "like_count", "engagement_rate") \
    .show(20, truncate=50)
```

### 4. Content Gap Analysis

```python
# Find popular topics with fewer videos
from pyspark.sql.functions import explode, col

# Analyze tags
videos_with_tags = videos_df.filter(col("tags").isNotNull())
tags_exploded = videos_with_tags.select(explode("tags").alias("tag"), "view_count")

tag_performance = tags_exploded.groupBy("tag") \
    .agg(
        count("*").alias("video_count"),
        avg("view_count").alias("avg_views")
    ) \
    .filter("video_count < 5 AND avg_views > 50000") \
    .orderBy("avg_views", ascending=False)

tag_performance.show(20, truncate=False)
```

## 🔄 Incremental Updates

For production use, implement incremental updates:

```python
# In extract_youtube_videos.py
# Add logic to only fetch videos after last extraction

from datetime import datetime, timedelta

# Get last extraction date
last_extraction = get_last_extraction_date()

# Filter videos published after last extraction
params = {
    'publishedAfter': last_extraction.isoformat() + 'Z'
}
```

## 📊 Dashboard Integration

### Export to BI Tools

```python
# Export to CSV for Tableau/PowerBI
spark.read.format("delta") \
    .load("/opt/delta-lake/gold_youtube_channel_performance") \
    .coalesce(1) \
    .write.mode("overwrite") \
    .option("header", "true") \
    .csv("/opt/spark-data/exports/channel_performance")
```

### Create Visualizations

```python
import matplotlib.pyplot as plt
import pandas as pd

# Read data
df = spark.read.format("delta").load("/opt/delta-lake/gold_youtube_monthly_trends").toPandas()

# Plot monthly trends
plt.figure(figsize=(12, 6))
plt.plot(df['published_month'], df['total_views'], marker='o')
plt.title('Monthly View Trends')
plt.xlabel('Month')
plt.ylabel('Total Views')
plt.savefig('/opt/notebooks/monthly_trends.png')
```

## 🔐 Best Practices

### 1. Protect Your API Key
- ✅ Never commit .env to git
- ✅ Use environment variables
- ✅ Rotate keys periodically
- ✅ Monitor API usage in Google Cloud Console

### 2. Optimize API Usage
- ✅ Cache results when possible
- ✅ Batch requests efficiently
- ✅ Use incremental updates
- ✅ Monitor quota consumption

### 3. Data Quality
- ✅ Run quality checks regularly
- ✅ Handle API errors gracefully
- ✅ Validate data before processing
- ✅ Keep audit logs

### 4. Performance
- ✅ Partition large tables by date
- ✅ Optimize Delta tables regularly
- ✅ Use appropriate Spark memory settings
- ✅ Monitor job execution times

## 📈 Scaling Considerations

### Handling More Channels

```python
# Process channels in batches
BATCH_SIZE = 10

for i in range(0, len(YOUTUBE_CHANNELS), BATCH_SIZE):
    batch = YOUTUBE_CHANNELS[i:i+BATCH_SIZE]
    process_batch(batch)
    time.sleep(1)  # Rate limiting
```

### Distributed Processing

```python
# Use Spark's parallelism
channel_ids_rdd = spark.sparkContext.parallelize(channel_ids, numSlices=4)
results = channel_ids_rdd.map(extract_channel_data).collect()
```

### Data Retention

```python
# Clean old data (run monthly)
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "/opt/delta-lake/bronze_youtube_videos")

# Keep only last 12 months
delta_table.delete("published_year < year(current_date()) - 1")

# Vacuum old versions
delta_table.vacuum(168)  # 7 days retention
```

## 🎓 Learning Resources

### YouTube API Documentation
- [YouTube Data API v3](https://developers.google.com/youtube/v3)
- [API Quotas](https://developers.google.com/youtube/v3/getting-started#quota)
- [Code Samples](https://developers.google.com/youtube/v3/code_samples)

### Delta Lake Resources
- [Delta Lake Documentation](https://docs.delta.io/)
- [Best Practices](https://docs.delta.io/latest/best-practices.html)
- [Performance Tuning](https://docs.delta.io/latest/optimizations-oss.html)

### PySpark Resources
- [PySpark Documentation](https://spark.apache.org/docs/latest/api/python/)
- [PySpark SQL Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)

## 🤝 Contributing

Want to enhance the pipeline? Consider adding:

- Sentiment analysis on comments
- Thumbnail analysis
- Video category classification
- Predictive analytics (view forecasting)
- Real-time streaming updates
- Additional data sources (Twitter, Instagram)

## 📝 Notes

### Sample Data Mode

If you don't have a YouTube API key, the pipeline will automatically generate sample data for testing. This allows you to:

- Test the complete pipeline flow
- Learn Delta Lake operations
- Understand PySpark transformations
- Validate analytics logic

### Production Deployment

For production use:

- Set up proper error handling and retries
- Implement monitoring and alerting
- Use secrets management (AWS Secrets Manager, Vault)
- Set up automated backups
- Configure proper logging
- Implement data governance policies

## 🆘 Support

### Check Logs

```bash
# Airflow logs
docker compose logs -f airflow-scheduler

# Spark logs
docker compose logs -f spark-master

# All logs
docker compose logs -f
```

### Common Issues

**Issue: API Quota Exceeded**
- Solution: Wait 24 hours or reduce `MAX_VIDEOS_PER_CHANNEL`

**Issue: Out of Memory**
```bash
# Increase Spark memory in docker-compose.yaml
environment:
  - SPARK_WORKER_MEMORY=4G
  - SPARK_DRIVER_MEMORY=4G
```

**Issue: Slow Extraction**
- Solution: Process fewer videos or add more workers

## 📊 Sample Output

```
YOUTUBE ANALYTICS GENERATION: STARTING
============================================================

TOP PERFORMING CHANNELS
------------------------------------------------------------
Top 10 Channels by Total Views:
+-------------------+------------+-------------+------------+----------------+
|channel_title      |video_count |total_views  |total_likes |avg_engagement  |
+-------------------+------------+-------------+------------+----------------+
|freeCodeCamp       |500         |150000000    |3000000     |2.5             |
|Fireship           |300         |120000000    |2800000     |3.1             |
|Google Developers  |450         |100000000    |2000000     |2.2             |
+-------------------+------------+-------------+------------+----------------+

VIDEO DURATION ANALYSIS
------------------------------------------------------------
Performance by Duration Category:
+--------------------+------------+----------+----------+---------------+
|duration_category   |video_count |avg_views |avg_likes |avg_engagement |
+--------------------+------------+----------+----------+---------------+
|Medium (3-10min)    |850         |45000     |1200      |3.2            |
|Long (10-30min)     |420         |52000     |1500      |3.0            |
|Short (<3min)       |230         |35000     |950       |3.5            |
+--------------------+------------+----------+----------+---------------+

✓ Report saved to: /opt/spark-data/reports/youtube_analytics_report.json
```

---

**Happy Analyzing! 🎉**
