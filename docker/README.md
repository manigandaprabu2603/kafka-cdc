# PostgreSQL CDC with Debezium + Kafka + JDBC Sink

This repository demonstrates **Change Data Capture (CDC)** from a PostgreSQL source database to a PostgreSQL sink database using Debezium, Kafka, and the JDBC Sink Connector.

## 📊 Architecture

```
Postgres DB (source, wal_level=logical)
        ↓
Debezium PostgreSQL Source Connector
        ↓
Kafka Topic
        ↓
JDBC Sink Connector
        ↓
Postgres DB (sink)
```

## 📋 Prerequisites

- Docker & Docker Compose installed
- PowerShell (for Windows users) or Bash (Linux/Mac)

## 🚀 Quickstart

### Step 1: Start the environment

```bash
docker-compose up -d
```

This will spin up:
- PostgreSQL source database (port 5433)
- PostgreSQL sink database (port 5434)
- Apache Kafka (port 9092)
- Zookeeper (port 2181)
- Kafka Connect (port 8083)

### Step 2: Create source database schema and table

```bash
docker exec -it postgres-source psql -U postgres -d sourcedb
```

```sql
CREATE SCHEMA IF NOT EXISTS kafka;
CREATE TABLE kafka.users (
  id SERIAL PRIMARY KEY,
  name TEXT,
  email TEXT
);
\q
```

### Step 3: Create sink database schema and table

```bash
docker exec -it postgres-sink psql -U postgres -d sinkdb
```

```sql
CREATE SCHEMA IF NOT EXISTS kafka;
CREATE TABLE kafka.users (
  id SERIAL PRIMARY KEY,
  name TEXT,
  email TEXT
);
\q
```

### Step 4: Register source connector

```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:8083/connectors `
  -ContentType "application/json" `
  -InFile .\postgres-source.json
```
```bash
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  --data @postgres-source.json
```

### Step 5: Register sink connector

```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:8083/connectors `
  -ContentType "application/json" `
  -InFile .\postgres-sink.json
```
```bash
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  --data @postgres-sink.json
```

### Step 6: Check connector status

```powershell
Invoke-RestMethod http://localhost:8083/connectors/postgres-source-connector/status
Invoke-RestMethod http://localhost:8083/connectors/postgres-sink-connector/status
```
```bash
curl http://localhost:8083/connectors/postgres-source-connector/status
curl http://localhost:8083/connectors/postgres-sink-connector/status
```

Both should show **RUNNING** state.

### Step 7: Insert sample data in PostgreSQL source database

```bash
docker exec -it postgres-source psql -U postgres -d sourcedb -c "INSERT INTO kafka.users (name, email) VALUES ('firstuser', 'firstuser@test.com');"
docker exec -it postgres-source psql -U postgres -d sourcedb -c "INSERT INTO kafka.users (name, email) VALUES ('seconduser', 'seconduser@test.com');"
```

### Step 8: Validate replication in PostgreSQL sink database

```bash
docker exec -it postgres-sink psql -U postgres -d sinkdb -c "SELECT * FROM kafka.users;"
```

You should see both inserted records.

### Step 9: Check Kafka topics (optional)

```bash
docker exec -it kafka_kafka_1 bash -c "/kafka/bin/kafka-topics.sh --list --bootstrap-server kafka:9092"
```

### Step 10: Final Validation - DELETE Propagation

```bash
docker exec -it postgres-source psql -U postgres -d sourcedb -c "DELETE FROM kafka.users WHERE id = 2;"
```

Wait a few seconds, then check the sink:

```bash
docker exec -it postgres-sink psql -U postgres -d sinkdb -c "SELECT * FROM kafka.users;"
```

The record with `id = 2` should be deleted in the sink as well.

## ✅ End-to-End Validation Checklist

- ✅ PostgreSQL (source) is generating WAL changes
- ✅ Debezium Source Connector is capturing changes
- ✅ Apache Kafka is receiving events
- ✅ Debezium JDBC Sink Connector is consuming events
- ✅ PostgreSQL (sink) is applying INSERT, UPDATE, and DELETE correctly

## 📁 Project Structure

```
├── docker-compose.yml          # Docker services configuration
├── postgres-source.json        # Debezium PostgreSQL source connector config
├── postgres-sink.json          # JDBC sink connector config
└── README.md                   # This file
```

---

> **Note:** This setup is designed for learning and development. For production use, ensure proper security configurations, monitoring, and error handling.