# Apache Kafka: Core Concepts and Applications - Course Summary

**Student Name:** Josiah Lagat  
**Course:** Data Engineering: Real-Time Data Processing with Kafka
**Date:** September 21, 2025

## 1. Introduction

Before taking Real-Time Data Processing with Kafka, I knew Apache Kafka was a "streaming platform," but I didn't fully grasp its significance. This course has shown me that it's far more than a messaging queue; it's a foundational piece of infrastructure for building real-time, responsive applications. This summary will cover its core architecture, its role in data engineering, and the practical realities of using it in production, based on our course materials and readings.

## 2. Core Architecture: The Distributed Log

The most important concept I learned is that Kafka's power comes from its simplicity. At its heart, it is a **distributed, append-only log**. This model is incredibly effective for writing and reading sequences of events in order.

### Key Components

* **Topics and Partitions:** A `topic` is a category or feed name (e.g., `user-logins`). Each topic is split into **partitions**, which enable parallelism. A crucial design takeaway is that the number of partitions is a critical trade-off: more partitions allow more consumers but add operational overhead.
* **Producers and Consumers:** **Producers** publish messages to a topic. **Consumers** read from them. The most powerful aspect of this is the decoupling; producers and consumers operate independently of each other.

### Partitioning Strategy

The **partition key** determines which partition a message is written to. This is vital for maintaining order. For example, using a `user_id` as the key ensures all events for that user are processed in sequence.

We practiced this by building a producer that ingested live data from the **OpenWeather API**. This was a great lab because it showed how to connect a real-world data source to Kafka. We built a service that periodically fetched weather data for a list of cities and streamed it into a topic, using the city name as the key.

```python
from kafka import KafkaProducer
import requests, json, time

producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

API_KEY = "your_api_key"
cities = ["London", "New York", "Tokyo"]

while True:
    for city in cities:
        url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}"
        response = requests.get(url).json()
        weather_data = {
            "city": city,
            "temp": response['main']['temp'],
            "conditions": response['weather'][0]['description']
        }
        producer.send('weather-updates', key=city.encode('utf-8'), value=weather_data)
    time.sleep(300) # Fetch data every 5 minutes
```

*Code Example 1: A producer fetching data from the OpenWeather API and streaming it to Kafka.*

## 3. The Kafka Ecosystem: Managed Services and KRaft

**Cluster & Brokers:** A Kafka cluster consists of multiple brokers (servers), which manage the partitions and provide fault tolerance.

**KRaft:** A major recent development is the move from Apache ZooKeeper for metadata management to KRaft (Kafka Raft metadata mode). This simplifies deployment.

**Managed Services:** We also discussed the operational complexity of self-hosting a cluster. This is where managed platforms like Confluent Cloud become relevant. They handle the underlying infrastructure, scaling, and maintenance, allowing developers to focus just on building their streaming applications instead of managing servers. For a startup or a team without dedicated SREs( Site Reliability Engineering ), this seems like a very practical option.

## 4. Data Engineering Applications

This is where Kafka becomes truly exciting, enabling new architectural patterns.

### Change Data Capture (CDC)

CDC captures every change in a database (inserts, updates, deletes) and streams them to Kafka. Tools like Debezium make this possible, allowing other systems (data warehouses, caches) to stay in sync in real-time, moving beyond hourly batch updates.

### Event-Driven Architecture

Kafka acts as the backbone for microservices to communicate asynchronously via events. This decouples services, making the overall system more resilient and scalable.

A basic consumer for our weather data topic would look like this:

```python
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'weather-updates',
    bootstrap_servers='localhost:9092',
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

for message in consumer:
    data = message.value
    print(f"Weather in {data['city']}: {data['temp']}K, {data['conditions']}")
```

*Code Example 2: A consumer reading the weather updates from Kafka.*

## 5. Production Considerations

Running Kafka in production involves crucial configuration and monitoring.

### Performance Tuning

Defaults are not enough. Key producer settings include:

* `batch.size` & `linger.ms`: For sending larger, more efficient batches.
* `compression.type` (e.g., snappy): To save network bandwidth.
* `acks`: A trade-off between speed and durability (0 for speed, all for strong consistency).

### Monitoring

The most critical metric is consumer lag. If it grows, it means consumers can't keep up with producers, indicating a serious bottleneck. Tools like Prometheus are essential.

## 6. Industry Use Cases

Kafka isn't just theoretical; it solves real-world scaling problems:

* **Netflix:** Processes trillions of events daily for recommendations and monitoring.
* **LinkedIn** (its creator): Powers the entire newsfeed and activity tracking system.
* **Uber:** The core of its real-time ride-matching and dynamic pricing systems.

## 7. Conclusion & Key Takeaway

This course shifted my perspective from batch processing to event-driven thinking. The labs, like connecting the OpenWeather API, made the theory tangible. I also see the value in managed services like Confluent for reducing operational overhead. The core lesson was understanding the power of an event as the source of truth, which is foundational for building scalable and resilient modern applications.
