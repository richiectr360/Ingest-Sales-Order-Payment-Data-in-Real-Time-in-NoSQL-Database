# Data Engineering Project: Ingest Sales Order & Payment Data in Real Time in NoSQL Database

This project demonstrates how to integrate Apache Cassandra with Google Pub/Sub for processing e-commerce order and payment data. It involves consuming mock order and payment data from Pub/Sub topics, ingesting them into a Cassandra database, and handling failures by sending problematic data to a Dead Letter Queue (DLQ).

## Table of Contents

- [Project Overview](#project-overview)
- [Problem Statement](#problem-statement)
- [Tech Stack](#tech-stack)
- [Setup Instructions](#setup-instructions)
  - [Prerequisites](#prerequisites)
  - [Docker Setup for Cassandra](#docker-setup-for-cassandra)
  - [Google Cloud Pub/Sub Setup](#google-cloud-pubsub-setup)
  - [Cassandra Table Creation](#cassandra-table-creation)
- [Running the Application](#running-the-application)
  - [Order and Payment Data Producers](#order-and-payment-data-producers)
  - [Data Consumers](#data-consumers)
- [Python Dependencies](#python-dependencies)
- [Initial Setup and Requirements for Pub/Sub](#initial-setup-and-requirements-for-pub-sub)
- [Conclusion](#conclusion)

## Project Overview

This project provides a scalable solution to process and store high-throughput e-commerce data. It simulates the ingestion of orders and payments data from Google Cloud Pub/Sub into Apache Cassandra, a highly scalable NoSQL database. It ensures that data consistency is maintained and provides fault-tolerance through a Dead Letter Queue (DLQ) for error handling.

## Problem Statement

E-commerce platforms handle massive amounts of data, such as orders and payments, in real-time. Ensuring the reliability and scalability of data pipelines that capture, process, and store this information is critical for effective business operations. Common challenges include:

1. **Data Ingestion**: Managing high-throughput, real-time ingestion of structured data.
2. **Data Consistency**: Maintaining data consistency across systems when data arrives asynchronously (e.g., an order before its payment).
3. **Error Handling**: Robust error handling mechanisms to deal with invalid or missing data.
4. **Scalability**: Supporting high scalability for real-time data ingestion and processing.

This project addresses these challenges by using Google Cloud Pub/Sub for reliable message delivery and Apache Cassandra for scalable, high-availability data storage. It also handles errors by pushing problematic data to a Dead Letter Queue (DLQ), ensuring no data is lost.

## Tech Stack

The project uses the following technologies:

- **Google Cloud Pub/Sub**: A message-oriented middleware service for asynchronous communication between microservices and real-time data pipelines.
- **Apache Cassandra**: A distributed NoSQL database designed for handling large amounts of data across many commodity servers without any single point of failure.
- **Docker**: For containerizing and running Cassandra.
- **Python**: For the data producer and consumer scripts.
  - **google-cloud-pubsub**: Pub/Sub client library to interact with Google Cloud Pub/Sub.
  - **cassandra-driver**: Python driver for connecting to and working with Apache Cassandra.

## Setup Instructions

### Prerequisites

- **Google Cloud SDK**: Install the Google Cloud SDK to access `gcloud` CLI commands for Pub/Sub management.
- **Docker**: Docker is required to run the Cassandra database using the provided Docker Compose configuration.

### Docker Setup for Cassandra

1. Clone the repository and navigate to the project directory.
   
2. Use the provided `docker-compose-cassandra.yml` file to start the Cassandra service:

    ```bash
    docker compose -f docker-compose-cassandra.yml up -d
    ```

3. After starting the Cassandra container, you can access the Cassandra shell:

    ```bash
    docker exec -it cassandra-container cqlsh
    ```

### Google Cloud Pub/Sub Setup

1. **Create Pub/Sub Topics**: In your Google Cloud project, create three Pub/Sub topics:
   - `orders_data`
   - `payments_data`
   - `dlq_payments_data`
   
2. **Authenticate GCP Account**: Authenticate the Google Cloud CLI using the following command:

    ```bash
    gcloud auth application-default login
    ```

3. **Create IAM Service Account**: In the Google Cloud Console, create a new service account and assign the following roles:
   - Pub/Sub Publisher
   - Pub/Sub Subscriber

4. **Download Service Account Key**: Download the service account key as a JSON file and replace the contents in the default auth config file:

    ```bash
    /Users/your_username/.config/gcloud/application_default_credentials.json
    ```

### Cassandra Table Creation

Create the keyspace and table in Cassandra using the following CQL commands:

```cql
CREATE KEYSPACE IF NOT EXISTS ecom_store WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 1 };

CREATE TABLE ecom_store.orders_payments_facts (
    order_id BIGINT PRIMARY KEY,
    customer_id BIGINT,
    item TEXT,
    quantity BIGINT,
    price DOUBLE,
    shipping_address TEXT,
    order_status TEXT,
    creation_date TEXT,
    payment_id BIGINT,
    payment_method TEXT,
    card_last_four TEXT,
    payment_status TEXT,
    payment_datetime TEXT
);
```


## Running the Application

### Order and Payment Data Producers

The order_data_producer.py and payments_data_producer.py scripts simulate the production of order and payment data. These scripts publish mock data to the corresponding Pub/Sub topics.

- Run the order data producer:

```bash
python3 order_data_producer.py
```

```bash
python3 payments_data_producer.py
```

### Data Consumers

The order_data_consumer.py and ingest_in_fact_table.py scripts consume messages from the Pub/Sub topics and ingest the data into Cassandra.

- Run the order data consumer:

```bash
python3 order_data_consumer.py
```

- Run the payment data ingestion script:

```bash
python3 ingest_in_fact_table.py
```

## Python Dependencies

### Order and Payment Data Producers
The order_data_producer.py and payments_data_producer.py scripts simulate the production of order and payment data. These scripts publish mock data to the corresponding Pub/Sub topics.

- Run the order data producer:

```bash
python3 order_data_producer.py
```

- Run the payment data producer:

```bash
python3 payments_data_producer.py
```

### Data Consumers

The order_data_consumer.py and ingest_in_fact_table.py scripts consume messages from the Pub/Sub topics and ingest the data into Cassandra.

- Run the order data consumer:

```bash
python3 order_data_consumer.py
```

- Run the payment data ingestion script:

```bash
python3 ingest_in_fact_table.py
```

## Python Dependencies

Install the necessary Python dependencies using the following commands:

```bash
pip3 install google-cloud-pubsub
pip3 install cassandra-driver
```

## Initial Setup and Requirements for Pub/Sub


1. Google Cloud SDK: Make sure Google Cloud SDK is installed to access gcloud CLI commands.
2. Pub/Sub Topics: Create Pub/Sub topics named as orders_data, payments_data, and dlq_payments_data with default subscribers.
3. Authentication: Before publishing/consuming data in Pub/Sub, authenticate your Google Cloud account:

```bash
gcloud auth application-default login
```

4. Service Account: Create a service account with Pub/Sub producer and subscriber roles in Google Cloud IAM & Admin.
5. Service Account Keys: Generate keys for the service account. Replace the content in:

```bash
/Users/your_username/.config/gcloud/application_default_credentials.json
```

## Docker Command to Start Cassandra

To start the Cassandra service:

```bash
docker compose -f docker-compose-cassandra.yml up -d
```

Once the Cassandra container is running:

1. Open the container terminal.
2. Run cqlsh to open the Cassandra shell.

## Cassandra Table Creation

To create the keyspace and table in Cassandra, use the following commands in the cqlsh shell:

```cql
CREATE KEYSPACE IF NOT EXISTS ecom_store WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 1 };

CREATE TABLE ecom_store.orders_payments_facts (
    order_id BIGINT PRIMARY KEY,
    customer_id BIGINT,
    item TEXT,
    quantity BIGINT,
    price DOUBLE,
    shipping_address TEXT,
    order_status TEXT,
    creation_date TEXT,
    payment_id BIGINT,
    payment_method TEXT,
    card_last_four TEXT,
    payment_status TEXT,
    payment_datetime TEXT
);
```

## Conclusion

This project demonstrates a simple data pipeline integrating Google Pub/Sub and Apache Cassandra. By following the setup instructions, you can simulate an e-commerce data pipeline that processes and stores order and payment data while handling failures via a Dead Letter Queue (DLQ). This setup provides a scalable, reliable solution for real-time data ingestion and error handling in distributed systems.
