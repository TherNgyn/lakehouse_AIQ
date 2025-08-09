---
title : "Setting Up API Gateway"
date : 2025-08-10
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---

## Setting Up API Gateway

In this section, we'll configure Amazon API Gateway to create RESTful APIs that can serve as another data ingestion point for our Lakehouse architecture. This provides a flexible way to receive data from various sources, such as web applications, mobile apps, or third-party systems.

### 1. Creating an API Gateway

Let's start by creating a REST API in API Gateway:

1. Navigate to the [Amazon API Gateway console](https://console.aws.amazon.com/apigateway/).
2. Click **Create API** and select **REST API** (not private).
3. Select **New API** and provide the following information:
   - **API name**: `aiq-data-api`
   - **Description**: API for AIQ data ingestion
   - **Endpoint Type**: Regional
4. Click **Create API**.

### 2. Creating a Resource and Method

Now, let's create a resource and a POST method to receive data:

1. In the left navigation pane, select **Resources**.
2. Click on **Actions** and select **Create Resource**.
3. Enter the following details:
   - **Resource Name**: `data`
   - **Resource Path**: `/data`
   - Select **Enable API Gateway CORS**
4. Click **Create Resource**.
5. With the `/data` resource selected, click **Actions** and select **Create Method**.
6. Select **POST** from the dropdown and click the checkmark.
7. Configure the POST method as follows:
   - **Integration type**: Lambda Function
   - **Use Lambda Proxy integration**: Check this box
   - **Lambda Region**: Select your region
   - **Lambda Function**: We'll create this in the next step
8. Don't click **Save** yet, as we need to create the Lambda function first.

### 3. Creating a Lambda Function for API Integration

We need to create a Lambda function that will process incoming API requests and send data to our MSK cluster:

1. Open a new browser tab and navigate to the [AWS Lambda console](https://console.aws.amazon.com/lambda/).
2. Click **Create function**.
3. Select **Author from scratch** and enter the following details:
   - **Function name**: `api-to-msk-function`
   - **Runtime**: Node.js 16.x
   - **Architecture**: x86_64
   - **Execution role**: Create a new role with basic Lambda permissions
4. Click **Create function**.
5. In the function code editor, replace the code with the following:

```javascript
const AWS = require('aws-sdk');
const { Kafka } = require('kafkajs');
const kafka = new Kafka({
  clientId: 'api-gateway-producer',
  brokers: process.env.BOOTSTRAP_SERVERS.split(','),
  ssl: true,
  sasl: {
    mechanism: 'scram-sha-512',
    username: process.env.KAFKA_USERNAME,
    password: process.env.KAFKA_PASSWORD
  }
});

const producer = kafka.producer();

exports.handler = async (event) => {
  try {
    // Parse the incoming data from API Gateway
    const body = JSON.parse(event.body);
    
    // Add a timestamp if not present
    if (!body.timestamp) {
      body.timestamp = new Date().toISOString();
    }
    
    // Add source information
    body.source = 'api_gateway';
    
    // Connect to Kafka
    await producer.connect();
    
    // Send data to Kafka topic
    await producer.send({
      topic: process.env.KAFKA_TOPIC,
      messages: [
        { value: JSON.stringify(body) }
      ]
    });
    
    // Disconnect from Kafka
    await producer.disconnect();
    
    // Return successful response
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      },
      body: JSON.stringify({ 
        message: 'Data successfully ingested',
        timestamp: body.timestamp
      })
    };
  } catch (error) {
    console.error('Error:', error);
    
    // Return error response
    return {
      statusCode: 500,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      },
      body: JSON.stringify({ 
        message: 'Error ingesting data',
        error: error.message
      })
    };
  }
};
```

6. Scroll down to the **Environment variables** section and add the following:
   - **BOOTSTRAP_SERVERS**: Your MSK bootstrap servers (SASL endpoint)
   - **KAFKA_USERNAME**: Username from your AWS Secrets Manager secret
   - **KAFKA_PASSWORD**: Password from your AWS Secrets Manager secret
   - **KAFKA_TOPIC**: `aiq-api-data`

7. In the **Configuration** tab:
   - Set **Memory** to 256 MB
   - Set **Timeout** to 30 seconds
   - In the **VPC** section, click **Edit** and select:
     - **VPC**: Lakehouse-VPC
     - **Subnets**: Select at least two private subnets
     - **Security groups**: Select MSK-SG

8. Click **Save**.

9. You'll also need to add the KafkaJS library as a dependency. We can do this using a Lambda layer:
   - In a terminal on your local machine, create a directory for the layer:
     ```bash
     mkdir -p kafka-layer/nodejs
     cd kafka-layer/nodejs
     npm init -y
     npm install kafkajs
     cd ..
     zip -r kafka-layer.zip nodejs
     ```
   - In the Lambda console, go to **Layers** in the left navigation pane
   - Click **Create layer**
   - Name it `kafkajs-layer`
   - Upload the zip file you created
   - Select the compatible runtimes (Node.js 16.x)
   - Click **Create**
   - Go back to your Lambda function
   - In the **Layers** section, click **Add a layer**
   - Select **Custom layers**
   - Choose the `kafkajs-layer` you just created
   - Click **Add**

### 4. Connecting API Gateway to Lambda

Now that we've created our Lambda function, let's connect it to API Gateway:

1. Return to the API Gateway console tab.
2. In the POST method setup, select your `api-to-msk-function` from the Lambda function dropdown.
3. Click **Save** and **OK** to grant API Gateway permission to invoke your Lambda function.

### 5. Creating a Kafka Topic for API Data

We need to create a new Kafka topic for data coming from the API:

1. Connect to your EC2 instance.
2. Using the Kafka client tools we set up in the previous section, create a new topic:

```bash
./kafka_2.13-2.8.1/bin/kafka-topics.sh --create \
--bootstrap-server $BOOTSTRAP_SERVERS \
--topic aiq-api-data \
--partitions 3 \
--replication-factor 3 \
--command-config client.properties
```

### 6. Deploying the API

Now, let's deploy our API to make it accessible:

1. In the API Gateway console, select **Actions** and click **Deploy API**.
2. For **Deployment stage**, select **New Stage**.
3. Enter `prod` for the **Stage name** and add a description.
4. Click **Deploy**.
5. Make note of the **Invoke URL** displayed at the top of the screen. This is the URL you'll use to send data to your API.

### 7. Testing the API

Let's test our API to ensure it's working correctly:

1. You can use a tool like cURL or Postman to send a POST request to your API.

Using cURL:
```bash
curl -X POST \
  <your-invoke-url>/data \
  -H 'Content-Type: application/json' \
  -d '{
  "device_id": "api-device-001",
  "temperature": 24.5,
  "humidity": 65.2,
  "pressure": 1012.3,
  "air_quality": 85.7,
  "data_type": "environmental"
}'
```

2. If successful, you should receive a response like:
```json
{
  "message": "Data successfully ingested",
  "timestamp": "2025-08-10T15:30:45.123Z"
}
```

3. To verify that the data is flowing to Kafka, run a consumer on your EC2 instance:
```bash
./kafka_2.13-2.8.1/bin/kafka-console-consumer.sh \
--bootstrap-server $BOOTSTRAP_SERVERS \
--topic aiq-api-data \
--from-beginning \
--command-config client.properties
```

4. You should see your JSON message appear in the consumer output.

### 8. Setting Up MSK Connect for API Data

Similar to what we did for IoT data, let's set up an MSK Connect connector to stream API data to S3:

1. Navigate to the Amazon MSK console and select **MSK Connect**.

2. Create a connector (if you haven't already created the S3 sink connector in the previous section, follow those steps first):
   - Click **Create connector**
   - Select the `s3-sink-connector` plugin
   - Configure connector settings:
     - **Name**: `api-to-s3-connector`
     - **Kafka cluster**: Select your MSK cluster
     - **Authentication**: Select IAM role-based authentication
     - **Workers**: Start with 1
     - **Connector configuration**: Enter the following JSON configuration:

```json
{
  "connector.class": "io.confluent.connect.s3.S3SinkConnector",
  "tasks.max": "1",
  "topics": "aiq-api-data",
  "s3.region": "<your-aws-region>",
  "s3.bucket.name": "<your-bronze-layer-bucket>",
  "s3.prefix": "api-data",
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

3. Replace `<your-aws-region>` and `<your-bronze-layer-bucket>` with your AWS region and S3 bucket name.

4. Configure the same service execution role you used for the previous connector.

5. Click **Create connector**.

### Summary

In this section, we've set up Amazon API Gateway as another data ingestion point for our Lakehouse architecture. We created a REST API with a Lambda function that sends incoming data to our MSK cluster. We also set up MSK Connect to stream this data to our S3-based Bronze layer.

The API Gateway provides a flexible way for various applications and systems to contribute data to our Lakehouse. In the next section, we'll explore how to use AWS DataSync to ingest data from existing data sources.
