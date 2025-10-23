Of course. This documentation page is a comprehensive tutorial for getting started with Debezium, specifically using the MySQL connector. Below is a proper, structured guide that distills the essential steps and concepts from the provided documentation.

### A Practical Guide to Getting Started with Debezium and MySQL

This guide will walk you through running a local Debezium environment using Docker to capture real-time changes from a MySQL database.

---

### **Prerequisites**

*   **Docker:** Ensure Docker is installed and running on your machine.
*   **Basic Understanding:** Familiarity with databases and the command line is helpful.

---

### **Step 1: Understanding the Architecture**

Before we start, it's crucial to understand the components we'll be running:

1.  **Apache Kafka & Zookeeper:** The streaming platform Debezium is built on. Debezium uses Kafka to store change events.
2.  **MySQL Database:** The source database we want to monitor. It comes pre-loaded with a sample `inventory` database.
3.  **Kafka Connect Service:** The runtime that hosts and executes connectors. The Debezium MySQL connector is a plugin for this service.
4.  **Debezium MySQL Connector:** The component that connects to MySQL, reads its binary log (binlog), and streams changes to Kafka topics.

---

### **Step 2: Starting the Services**

Open separate terminal windows for each of the following commands to see the logs.

#### **1. Start Apache Kafka**

This command starts a Kafka broker in a container named `kafka`.

```bash
docker run -it --rm --name kafka -p 9092:9092 \
  --hostname kafka \
  -e CLUSTER_ID=my-cluster \
  -e NODE_ID=1 \
  -e NODE_ROLE=combined \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS=1@kafka:9093 \
  -e KAFKA_LISTENERS=PLAINTEXT://kafka:9092,CONTROLLER://kafka:9093 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092 \
  quay.io/debezium/kafka:3.3
```

#### **2. Start MySQL with Sample Database**

This starts a MySQL server with a pre-configured `inventory` database and user.

```bash
docker run -it --rm --name mysql -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=debezium \
  -e MYSQL_USER=mysqluser \
  -e MYSQL_PASSWORD=mysqlpw \
  quay.io/debezium/example-mysql:3.3
```

#### **3. Start the Kafka Connect Service with Debezium**

This starts the Kafka Connect service and includes the Debezium connector libraries.

```bash
docker run -it --rm --name connect -p 8083:8083 \
  -e GROUP_ID=1 \
  -e CONFIG_STORAGE_TOPIC=my_connect_configs \
  -e OFFSET_STORAGE_TOPIC=my_connect_offsets \
  -e STATUS_STORAGE_TOPIC=my_connect_statuses \
  --link kafka:kafka \
  --link mysql:mysql \
  quay.io/debezium/connect:3.3
```

---

### **Step 3: Verify the Services**

In a new terminal, test if the Kafka Connect REST API is accessible. This API is used to manage connectors.

```bash
curl -H "Accept:application/json" localhost:8083/
```
You should see a response with the Kafka Connect version.

Check for any existing connectors (the list should be empty for now):
```bash
curl -H "Accept:application/json" localhost:8083/connectors/
```

---

### **Step 4: Deploy the Debezium MySQL Connector**

Now, we tell Kafka Connect to start the Debezium connector and begin monitoring the MySQL database.

Create a file named `register-mysql.json` with the following configuration:

```json
{
  "name": "inventory-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "mysql",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "dbz",
    "database.server.id": "184054",
    "topic.prefix": "dbserver1",
    "database.include.list": "inventory",
    "schema.history.internal.kafka.bootstrap.servers": "kafka:9092",
    "schema.history.internal.kafka.topic": "schemahistory.inventory"
  }
}
```

**Key Configuration Explained:**
*   `name`: A unique name for your connector.
*   `connector.class`: Specifies the MySQL connector.
*   `database.hostname`, `user`, `password`: Connection details for MySQL.
*   `topic.prefix`: Prefix for all Kafka topics created by this connector.
*   `database.include.list`: The specific database(s) to monitor.
*   `schema.history.internal...`: Tells the connector to store the database schema history in a Kafka topic.

Use `curl` to send this configuration to the Kafka Connect API:

```bash
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d @register-mysql.json
```

**Verify the connector is running:**
```bash
curl -H "Accept:application/json" localhost:8083/connectors/inventory-connector
```

---

### **Step 5: Observe Debezium in Action**

#### **1. Watch the Change Events**

Start a Kafka console consumer to watch the topic for the `customers` table.

```bash
docker run -it --rm --name watcher --link kafka:kafka quay.io/debezium/kafka:3.3 watch-topic -a -k dbserver1.inventory.customers
```
You will immediately see an initial snapshot of the existing data in the `customers` table, represented as **Create (`c`)** events.

#### **2. Make a Database Change**

Connect to the MySQL database using the command line:
```bash
docker run -it --rm --name mysqlterm --link mysql mysql:8.2 sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -umysqluser -p"mysqlpw"'
```

In the MySQL client, switch to the `inventory` database and update a customer:
```sql
USE inventory;
UPDATE customers SET first_name='Anne Marie' WHERE id=1004;
```

#### **3. See the Change Event**

Switch back to the terminal running the `watcher`. You will see a new **Update (`u`)** event. The JSON payload will show:
*   `"op": "u"`
*   `"before"`: The state of the row before the update.
*   `"after"`: The new state of the row after the update.

#### **4. Test a Delete**

In the MySQL client, delete a record:
```sql
DELETE FROM customers WHERE id=1004;
```
In the `watcher` terminal, you will see a **Delete (`d`)** event, followed by a **tombstone event** (a message with a key and a `null` value), which signals Kafka to eventually clean up old messages with the same key.

---

### **Step 6: Testing Resilience (Optional)**

1.  Stop the Kafka Connect container: `docker stop connect`.
2.  Make more changes in the MySQL database (e.g., INSERT new records).
3.  Restart the Kafka Connect container using the command from **Step 2.3**.
4.  Observe that upon restart, the connector reads the binlog from where it left off and immediately generates the change events for the changes you made while it was down. This demonstrates Debezium's durability and fault tolerance.

---

### **Step 7: Clean Up**

Stop all running Docker containers. Since we used the `--rm` flag, they will be automatically removed when stopped.

```bash
docker stop mysqlterm watcher connect mysql kafka
```

---

### **Next Steps & Deeper Dive**

*   **Explore Other Connectors:** Try the same tutorial with PostgreSQL, MongoDB, or SQL Server. The Debezium examples repository provides Docker Compose files for each.
*   **Avro Serialization:** Configure the connector to use Avro and the Schema Registry for a more efficient binary format and better schema management.
*   **Single Message Transforms (SMTs):** Use SMTs to route events to different topics, filter out fields, or mask sensitive data on the fly.
*   **Outbox Pattern:** Implement the outbox pattern for reliable microservice communication using Debezium's `OutboxEventRouter` transformation.

This guide provides a solid foundation for understanding the core value proposition of Debezium: turning your existing database into a stream of immutable change events.
