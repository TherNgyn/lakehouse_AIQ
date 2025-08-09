---
title : "Building Data Ingestion Layer"
date: 2025-08-10
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

## Data Ingestion Layer

The data ingestion layer is the first and crucial component in the Lakehouse architecture. This layer is responsible for collecting data from various sources, ensuring data is reliably transmitted into the system.

In this workshop, we will set up the following data collection channels:

1. **AWS IoT Core**: Collect data from IoT sensor devices
2. **API Gateway**: Collect data from external APIs
3. **DataSync**: Import data from CSV files

![Data Ingestion Layer](/images/3-ingestion-layer/ingestion-overview.png)

### Data Processing Model

We will use a combined model of real-time and batch data processing:

1. **Real-time Processing**: 
   - Data from IoT Core and API Gateway will be sent to Amazon MSK
   - AWS Lambda will process data from MSK and store it in the S3 Bronze Layer

2. **Batch Processing**:
   - Data from DataSync will be synchronized directly into the S3 Bronze Layer
   - AWS Glue will perform periodic ETL jobs to process data

### Content

In this section, we will set up the components of the data ingestion layer:

1. [Configuring AWS IoT Core](3.1-iot-core/)
2. [Setting Up Amazon MSK](3.2-amazon-msk/)
3. [Building API Gateway](3.3-api-gateway/)
4. [Configuring AWS DataSync](3.4-datasync/)
