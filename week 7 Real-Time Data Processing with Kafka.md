# Apache Kafka: Core Concepts and Applications - Course Summary

**Student Name:** Lagat Josiah
**Course:**  Data Engineering : Real-Time Data Processing with Kafka
**Date:** September 21, 2025

## 1. Introduction

Before taking CS 598, I knew Apache Kafka was a "streaming platform," but I didn't fully grasp its significance. This course has shown me that it's far more than a messaging queue; it's a foundational piece of infrastructure for building real-time, responsive applications. This summary will cover its core architecture, its role in data engineering, and the practical realities of using it in production, based on our course materials and readings.

## 2. Core Architecture: The Distributed Log

The most important concept I learned is that Kafka's power comes from its simplicity. At its heart, it is a **distributed, append-only log**. This model is incredibly effective for writing and reading sequences of events in order.

### Key Components

*   **Topics and Partitions:** A `topic` is a category or feed name (e.g., `user-logins`). Each topic is split into **partitions**, which enable parallelism. A crucial design takeaway is that the number of partitions is a critical trade-off: more partitions allow more consumers but add operational overhead.
*   **Producers and Consumers:** **Producers** publish messages to a topic. **Consumers** read from them. The most powerful aspect of this is the decoupling; producers and consumers operate independently of each other.

### Partitioning Strategy

The **partition key** determines which partition a message is written to. This is vital for maintaining order. For example, using a `user_id` as the key ensures all events for that user are processed in sequence.

We practiced this with a simple producer:

```python
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8') # Serialize to JSON
)

user_event = {"user_id": 123, "action": "purchase", "item_id": "sku_456"}
producer.send('user-activity-topic', key=str(user_event['user_id']).encode('utf-8'), value=user_event)
