---
title : "Introduction"
date :  "2025-08-10" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

## Lakehouse Architecture and AIQ Index

**Lakehouse Architecture** is a modern data architecture model that combines the advantages of both **Data Warehouse** and **Data Lake**. This model provides high efficiency in processing structured and unstructured data, while supporting features such as:

- **ACID Transaction Management**: Ensures data integrity
- **Schema Management**: Supports schema declaration and enforcement
- **BI Support**: Direct interaction capability with Business Intelligence tools
- **Open Data Processing**: Ability to store and process various types of data
- **Data Science Tools Support**: Good integration with machine learning and data science frameworks
  
### AIQ - AI Quality Index

**AIQ (AI Quality Index)** is a quality assessment index for AI applications in business processes. This index helps businesses:

1. Evaluate the maturity level in AI application
2. Analyze the effectiveness of AI systems in use
3. Compare and benchmark against industry standards
4. Identify areas for improvement

### Why Lakehouse for AIQ Evaluation?

Evaluating the AIQ index requires analyzing data from various sources, including:
- IoT sensor data (time-series)
- Transaction data (structured)
- System data (logs)
- Unstructured data (text, images)

The Lakehouse architecture provides an ideal foundation to:
- Collect data from various sources
- Store raw data economically
- Process and transform data through stages (Bronze → Silver → Gold → Platinum)
- Support in-depth data analysis
- Provide data for visualization tools

### Solution Overview

This workshop will guide you in building a Lakehouse architecture on AWS with the following components:

![Lakehouse Architecture](../images/lakehouse-architecture.png)

In this workshop, you will:
1. Set up an environment for collecting data from IoT sensors
2. Build data collection and transformation pipelines
3. Design data storage layers using the Delta Lake model
4. Create data processing jobs
5. Configure an analytics environment
6. Build dashboards for AIQ index evaluation

Let's start with preparing the development environment!