---
title : "Setting Up Amazon MSK"
date : 2025-08-10
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

## Setting Up Amazon MSK

In this section, we'll set up Amazon Managed Streaming for Apache Kafka (MSK) to process streaming data from AWS IoT Core. MSK will serve as our streaming platform for real-time data processing in the Lakehouse architecture.

### 1. Creating an MSK Cluster

First, let's create an Amazon MSK cluster:

1. Navigate to the [Amazon MSK console](https://console.aws.amazon.com/msk/).
2. Click **Create cluster**.
3. Choose **Custom create** and enter the following details:
   - **Cluster name**: `lakehouse-msk-cluster`
   - **Kafka version**: Select the latest stable version (e.g., 2.8.1)
4. In the **Networking** section:
   - **Virtual Private Cloud (VPC)**: Select `Lakehouse-VPC`
   - **Subnets**: Select at least two private subnets
   - **Security Groups**: Create a new security group `MSK-SG` or select an existing one
5. For **Encryption**:
   - Enable encryption in transit
   - Enable encryption at rest using a KMS key
6. For **Authentication**:
   - Enable SASL/SCRAM authentication
   - Create a new secret in AWS Secrets Manager with a username and password
7. For **Monitoring**:
   - Enable monitoring with Prometheus
   - Enable open monitoring with Prometheus
8. For **Cluster configuration**:
   - Instance type: `kafka.m5.large` (for development) or choose based on your needs
   - Number of brokers: 3 (minimum for production)
   - Storage: Start with 100 GiB per broker and enable auto-scaling
9. Click **Create cluster**.

The cluster will take about 15-20 minutes to be created. Once it's active, proceed to the next step.

### 2. Configure Security Settings for IAM Access

To enable both SASL/SCRAM authentication for IoT Rule and IAM authentication for MSK Connect:

1. In the MSK console, select your cluster.
2. Click on **Actions** and select **Edit security settings**.
3. In addition to SASL/SCRAM, also select **IAM role-based authentication**.
4. Click **Save changes**.

This update will take approximately 10-15 minutes to complete.

### 3. Create a Kafka Topic

Now we'll create a Kafka topic to receive data from AWS IoT Core:

1. Connect to your EC2 instance configured in the previous section.
2. Install Java and the Kafka client tools if not already installed:

```bash
sudo yum install -y java-11-amazon-corretto-headless
mkdir ~/kafka-client && cd ~/kafka-client
wget https://archive.apache.org/dist/kafka/2.8.1/kafka_2.13-2.8.1.tgz
tar -xzf kafka_2.13-2.8.1.tgz
```

3. Configure the Kafka client for SASL/SCRAM authentication:

```bash
# Create the truststore
cp /usr/lib/jvm/java-11-amazon-corretto.x86_64/lib/security/cacerts kafka.truststore.jks

# Create the JAAS configuration
cat > kafka_client_jaas.conf << EOF
KafkaClient {
  org.apache.kafka.common.security.scram.ScramLoginModule required
  username="<your-msk-username>"
  password="<your-msk-password>";
};
EOF

# Create the client properties file
cat > client.properties << EOF
security.protocol=SASL_SSL
sasl.mechanism=SCRAM-SHA-512
ssl.truststore.location=$(pwd)/kafka.truststore.jks
EOF
```

4. Replace `<your-msk-username>` and `<your-msk-password>` with the credentials you stored in AWS Secrets Manager.

5. Set the required environment variables:

```bash
export BOOTSTRAP_SERVERS=$(aws kafka get-bootstrap-brokers --cluster-arn <your-cluster-arn> --query 'BootstrapBrokerStringSaslScram' --output text)
export KAFKA_OPTS="-Djava.security.auth.login.config=$(pwd)/kafka_client_jaas.conf"
```

6. Replace `<your-cluster-arn>` with your MSK cluster's ARN.

7. Create the Kafka topic:

```bash
./kafka_2.13-2.8.1/bin/kafka-topics.sh --create \
--bootstrap-server $BOOTSTRAP_SERVERS \
--topic aiq-sensor-data \
--partitions 3 \
--replication-factor 3 \
--command-config client.properties
```

8. Verify the topic creation:

```bash
./kafka_2.13-2.8.1/bin/kafka-topics.sh --list \
--bootstrap-server $BOOTSTRAP_SERVERS \
--command-config client.properties
```

### 4. Testing the Kafka Integration

Let's test that data from AWS IoT Core is flowing correctly to your MSK cluster:

1. Ensure your IoT device simulator (created in the previous section) is running and sending data.

2. On your EC2 instance, set up a Kafka consumer to read messages from the topic:

```bash
./kafka_2.13-2.8.1/bin/kafka-console-consumer.sh \
--bootstrap-server $BOOTSTRAP_SERVERS \
--topic aiq-sensor-data \
--from-beginning \
--command-config client.properties
```

3. You should start seeing JSON messages with sensor data appearing in the console.

### 5. Setting Up MSK Connect for Amazon S3 (Bronze Layer)

Now we'll set up MSK Connect to stream data from our Kafka topic to Amazon S3, which will serve as our Bronze layer in the Lakehouse architecture:

1. Navigate to the Amazon MSK console and select **MSK Connect** from the left sidebar.

2. Create a custom plugin:
   - Click **Create custom plugin**
   - Upload the Amazon S3 sink connector JAR file (confluent-kafka-connect-s3)
   - Name your plugin `s3-sink-connector`
   - Click **Create custom plugin**

3. Create a connector:
   - Click **Create connector**
   - Select the `s3-sink-connector` plugin
   - Configure connector settings:
     - **Name**: `msk-to-s3-connector`
     - **Kafka cluster**: Select your MSK cluster
     - **Authentication**: Select IAM role-based authentication
     - **Workers**: Start with 1 (adjust based on volume)
     - **Connector configuration**: Enter the following JSON configuration:

```json
{
  "connector.class": "io.confluent.connect.s3.S3SinkConnector",
  "tasks.max": "1",
  "topics": "aiq-sensor-data",
  "s3.region": "<your-aws-region>",
  "s3.bucket.name": "<your-bronze-layer-bucket>",
  "s3.part.size": "5242880",
  "flush.size": "1000",
  "storage.class": "io.confluent.connect.s3.storage.S3Storage",
  "format.class": "io.confluent.connect.s3.format.json.JsonFormat",
  "partitioner.class": "io.confluent.connect.storage.partitioner.TimeBasedPartitioner",
  "path.format": "'year'=YYYY/'month'=MM/'day'=dd/'hour'=HH",
  "locale": "en-US",
  "timezone": "UTC",
  "partition.duration.ms": "3600000",
  "rotate.schedule.interval.ms": "60000"
}
```

4. Replace `<your-aws-region>` and `<your-bronze-layer-bucket>` with your AWS region and S3 bucket name.

5. Configure service execution role:
   - Create a new service execution role or select an existing one
   - Ensure it has the necessary permissions to write to S3
   - Click **Create connector**

6. The connector will start running shortly. You can monitor its status on the MSK Connect dashboard.

### 6. Testing the S3 Integration

Let's verify that data is flowing correctly from MSK to S3:

1. Ensure your IoT device simulator is running and sending data.

2. Wait a few minutes for the data to propagate through the pipeline.

3. Navigate to the [Amazon S3 console](https://console.aws.amazon.com/s3/) and browse to your bronze layer bucket.

4. You should see folders organized by year, month, day, and hour, containing JSON files with sensor data.

### Summary

In this section, we've set up Amazon MSK as the streaming platform for our Lakehouse architecture. We've configured the MSK cluster with the necessary security settings, created a Kafka topic to receive data from AWS IoT Core, and set up MSK Connect to stream data to Amazon S3 as our Bronze layer.

In the next section, we'll set up additional ingestion methods using Amazon API Gateway and AWS DataSync to complete our data ingestion layer.
