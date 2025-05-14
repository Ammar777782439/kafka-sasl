# Kafka in KRaft Mode

This project sets up Apache Kafka in KRaft mode (without ZooKeeper).

## Configuration

- **KRaft Mode**: Kafka runs in KRaft mode without ZooKeeper
- **Cluster ID**: `MkU3OEVBNTcwNTJENDM2Qk`
- **Listeners**:
  - PLAINTEXT: For regular communication (port 9092)
  - SASL_PLAINTEXT: For authenticated communication (port 9094)

## How to Start

Run the following command to start Kafka:

```bash
docker-compose up -d
```

## How to Use

### Creating a Topic

```bash
docker exec kafka kafka-topics.sh --bootstrap-server kafka:9092 \
  --create --topic test-topic --partitions 1 --replication-factor 1
```

### Producing Messages

```bash
docker exec -it kafka kafka-console-producer.sh --bootstrap-server kafka:9092 \
  --topic test-topic
```

### Consuming Messages

```bash
docker exec -it kafka kafka-console-consumer.sh --bootstrap-server kafka:9092 \
  --topic test-topic --from-beginning
```

## Docker Compose Configuration

The `docker-compose.yml` file sets up Kafka in KRaft mode with the following configuration:

```yaml
services:
  kafka:
    image: bitnami/kafka:3.5.1
    hostname: kafka
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      # KRaft mode settings
      - KAFKA_CFG_NODE_ID=1
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@kafka:9093
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,SASL_PLAINTEXT://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,SASL_PLAINTEXT://localhost:9094
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SASL_PLAINTEXT:SASL_PLAINTEXT
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_INTER_BROKER_LISTENER_NAME=PLAINTEXT
      
      # SASL configuration
      - KAFKA_CFG_SASL_ENABLED_MECHANISMS=PLAIN
      - KAFKA_CFG_SASL_MECHANISM_INTER_BROKER_PROTOCOL=PLAIN
      - KAFKA_CFG_LISTENER_NAME_SASL_PLAINTEXT_PLAIN_SASL_JAAS_CONFIG=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="admin-secret" user_admin="admin-secret" user_user1="user1-secret";
      - KAFKA_CFG_LISTENER_NAME_SASL_PLAINTEXT_SASL_ENABLED_MECHANISMS=PLAIN
      
      # KRaft cluster ID
      - KAFKA_KRAFT_CLUSTER_ID=MkU3OEVBNTcwNTJENDM2Qk
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
```

## Notes

- This setup includes both PLAINTEXT and SASL_PLAINTEXT listeners, but we're currently using the PLAINTEXT listener for simplicity.
- The SASL_PLAINTEXT listener is configured but requires additional troubleshooting to work correctly.
- For production environments, it's recommended to use secure authentication and encryption.
