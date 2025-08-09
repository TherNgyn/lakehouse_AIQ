---
title : "Building Storage Layer"
date: 2025-08-10
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

## Data Storage Layer

The data storage layer is the core of the Lakehouse architecture, where data is organized in different tiers from raw to refined. In this workshop, we will use Amazon S3 as the storage foundation combined with Delta Lake to implement the data tiers.

![Data Storage Layer](/images/4-storage-layer/storage-overview.png)

### Multi-layer Storage Model

The Lakehouse architecture uses a multi-layer storage model to organize data based on refinement level:

1. **Bronze Layer (Raw Data)**:
   - Stores raw data from sources
   - Little to no transformation
   - Preserves the integrity of original data

2. **Silver Layer (Cleansed Data)**:
   - Data that has been cleaned and normalized
   - Data quality rules applied
   - Duplicate and invalid records removed

3. **Gold Layer (Transformed Data)**:
   - Data that has been aggregated and transformed
   - Fitted for specific use cases
   - Ready for analytics and machine learning

4. **Platinum Layer (Business Ready Data)**:
   - Data serving for reports and dashboards
   - Optimized for high-performance queries
   - Deployed in Amazon Redshift

### Delta Lake on Amazon S3

We will use Delta Lake, an open-source data framework, to provide advanced features for data stored in S3:

- **ACID Transactions**: Ensures data consistency
- **Schema Enforcement**: Automatically checks and enforces schema
- **Time Travel**: Access previous versions of data
- **Audit History**: Track data changes over time
- **Update/Delete/Merge Operations**: Support data modification operations

### Content

In this section, we will set up the data storage layer:

1. [Creating and Configuring Amazon S3 Buckets](4.1-s3-bucket/)
2. [Setting Up Delta Lake](4.2-delta-lake/)
3. [Building the Bronze Layer](4.3-bronze-layer/)
4. [Building the Silver Layer](4.4-silver-layer/)
5. [Building the Gold Layer](4.5-gold-layer/)
6. [Configuring Amazon Redshift for Platinum Layer](4.6-platinum-layer/)
