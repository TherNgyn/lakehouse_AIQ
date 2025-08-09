---
title : "Analytics and Visualization"
date: 2025-08-10
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

## Data Analytics Layer

The data analytics layer provides tools and services for querying, analyzing, and visualizing data from the Lakehouse architecture. In this workshop, we will use Amazon Athena to query data directly from S3, Amazon QuickSight to create dashboards, and Amazon SageMaker to build machine learning models.

![Data Analytics Layer](/images/6-analytics-layer/analytics-overview.png)

### SQL Analytics with Amazon Athena

Amazon Athena is an interactive query service that makes it easy to analyze data in S3 using standard SQL. With Athena, we can:

- Query data directly from S3 without loading or transforming data
- Use standard SQL to analyze data
- Integrate with the AWS Glue Data Catalog for metadata management
- Pay-per-query, only paying for data scanned

### Visualization with Amazon QuickSight

Amazon QuickSight is a scalable, serverless, embeddable, and ML-powered business analytics service. With QuickSight, we can:

- Create and publish interactive dashboards
- Connect to multiple data sources including Athena, Redshift, and S3
- Share insights with users and embed dashboards into applications
- Use ML to generate automatic insights and forecasts

### Machine Learning with Amazon SageMaker

Amazon SageMaker is a fully managed ML service that enables data scientists and developers to build, train, and deploy ML models quickly. With SageMaker, we can:

- Prepare data and create features
- Build and train ML models
- Deploy and manage models in production environments
- Automate and manage the entire ML lifecycle

### Content

In this section, we will explore the analytics capabilities of our Lakehouse architecture:

1. [Building Gold Layer Analytics](6.1-gold-analytics/)
2. [Creating the Platinum Machine Learning Layer](6.2-platinum-ml/)
3. [Data Visualization with QuickSight](6.3-visualization/)
