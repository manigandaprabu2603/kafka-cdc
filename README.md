# Kafka CDC

This repository demonstrates **Change Data Capture (CDC)** patterns using Apache Kafka.

It includes both local and cloud-based implementations for streaming database changes in real time.

---

## 📦 Implementations

### 1️⃣ docker/
Local development setup using:

- PostgreSQL (source & sink)
- Debezium PostgreSQL Connector
- Apache Kafka
- Kafka Connect
- JDBC Sink Connector
- Docker Compose

👉 See `docker/README.md` for setup instructions.

---

### 2️⃣ aws/ (Coming Soon)
Cloud implementation using:

- Amazon RDS (PostgreSQL)
- Amazon MSK (Managed Kafka)
- Kafka Connect (MSK Connect or self-managed)
- Infrastructure as Code (Terraform planned)

---

## 🎯 Goal of This Repository

- Demonstrate end-to-end CDC pipelines
- Showcase real-time data replication
- Provide production-ready architectural patterns
- Serve as a learning & portfolio project

---

Each folder contains its own setup instructions and configuration details.