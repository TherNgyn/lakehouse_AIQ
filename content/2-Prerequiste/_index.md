---
title : "Environment Setup"
date: 2025-08-10
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

{{% notice info %}}
To complete this workshop, you need an AWS account with access to EC2, VPC, IAM, S3, Lambda, IoT Core, MSK, AWS Glue, Redshift, Athena, and QuickSight services.
{{% /notice %}}

## Preparing the Environment for Lakehouse

To build a complete Lakehouse architecture on AWS, we need to set up several infrastructure components and services. This section will guide you step by step to create VPC, EC2, IAM Roles, and other necessary services.

### AWS Services We Will Use
1. **Amazon EC2**: For running virtual IoT devices and connecting to other services
2. **AWS IoT Core**: Service for managing IoT devices
3. **Amazon MSK**: AWS managed Apache Kafka service
4. **Amazon S3**: For storing data in Bronze, Silver, and Gold layers
5. **AWS Lambda**: For processing data from MSK and sending it to S3
6. **AWS Glue**: For transforming and processing data between layers
7. **Amazon Redshift**: For storing data in the Platinum layer for analysis
8. **Amazon Athena**: For querying data directly from S3
9. **Amazon QuickSight**: For data visualization and dashboard creation

### Content
  - [Creating VPC and EC2](2.1-createec2/)
  - [Creating IAM Role](2.2-createiamrole/)