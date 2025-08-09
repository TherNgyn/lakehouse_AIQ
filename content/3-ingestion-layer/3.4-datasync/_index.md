---
title : "Setting Up AWS DataSync"
date : 2025-08-10
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---

## Setting Up AWS DataSync

In this section, we'll configure AWS DataSync to transfer existing data from on-premises or other AWS storage systems to our Lakehouse architecture. DataSync provides a seamless way to migrate historical data or regularly synchronize data from various sources into our S3-based Bronze layer.

### 1. Understanding AWS DataSync

AWS DataSync is a data transfer service that simplifies, automates, and accelerates moving data between on-premises storage systems and AWS storage services, or between AWS storage services. For our Lakehouse architecture, DataSync helps us:

- Migrate historical data into our Lakehouse
- Set up recurring transfers from legacy systems
- Ingest data from file servers, NAS systems, or other object storage platforms
- Ensure data consistency across different storage locations

### 2. Creating a DataSync Agent (for On-Premises Sources)

If you need to transfer data from an on-premises system, you'll need to deploy a DataSync agent:

1. Navigate to the [AWS DataSync console](https://console.aws.amazon.com/datasync/).
2. Click **Create agent**.
3. Choose the deployment type that fits your environment:
   - **Amazon EC2**: If you want to deploy the agent in AWS
   - **VMware**: For VMware environments
   - **KVM**: For KVM hypervisors
   - **Hyper-V**: For Microsoft Hyper-V environments
   - **Hardware appliance**: For AWS Snowcone
4. Follow the instructions to deploy the agent in your environment.
5. Once deployed, the agent will generate an activation key. Copy this key.
6. In the DataSync console, enter the activation key and provide a name for your agent.
7. Click **Create agent**.

> Note: If you're only transferring data between AWS services, you don't need to create a DataSync agent.

### 3. Creating a DataSync Task

Now, let's create a DataSync task to transfer data:

1. In the DataSync console, select **Tasks** from the left navigation pane.
2. Click **Create task**.

#### 3.1. Configure Source Location

3. On the "Configure source location" page:
   - If transferring from on-premises:
     - Choose **Create a new location**
     - Location type: Select the appropriate type (NFS, SMB, HDFS, etc.)
     - Agent: Select the agent you created
     - Enter the server and path information for your source data
   - If transferring from another AWS service:
     - Choose **Create a new location**
     - Location type: Select the appropriate AWS service (S3, EFS, FSx, etc.)
     - Configure the specific settings for that service

4. Click **Next**.

#### 3.2. Configure Destination Location

5. On the "Configure destination location" page:
   - Choose **Create a new location**
   - Location type: **Amazon S3 bucket**
   - S3 bucket: Select your Bronze layer bucket
   - S3 storage class: **Standard**
   - Folder: Specify a prefix like `historical-data/`
   - IAM role: Create a new role or choose an existing one with the necessary permissions

6. Click **Next**.

#### 3.3. Configure Task Settings

7. On the "Configure settings" page:
   - Task name: `historical-data-sync`
   - Verify data: Choose an option based on your needs (ChecksumOnly, TaskLevel, or None)
   - Data transfer options: Configure based on your needs
   - Task logging: Enable CloudWatch logging
   - Schedule: Choose whether to run once or on a schedule

8. Click **Next**.

9. Review your configuration and click **Create task**.

### 4. Running and Monitoring Your DataSync Task

Once you've created your task:

1. Select the task from the tasks list.
2. Click **Start**.
3. Review the task configuration one more time and click **Start**.
4. You can monitor the progress of your task on the task details page.
5. After the task completes, you can check CloudWatch logs for detailed information about the transfer.

### 5. Setting Up Recurring Transfers

For data that needs to be synchronized regularly:

1. Edit your existing task or create a new one.
2. In the "Configure settings" section, set the schedule:
   - Choose **Scheduled**
   - Select the frequency (hourly, daily, weekly, etc.)
   - Set the specific days and times
   - Configure the start date

3. Save your changes.

### 6. Integrating Transferred Data with Glue Catalog

Once your data is transferred to S3, you'll need to integrate it with AWS Glue Catalog to make it accessible to your data processing tools:

1. Navigate to the [AWS Glue console](https://console.aws.amazon.com/glue/).
2. Select **Crawlers** from the left navigation pane.
3. Click **Add crawler**.
4. Follow the wizard to configure your crawler:
   - Name: `historical-data-crawler`
   - Choose the data store type: **S3**
   - Include path: Specify the S3 path where your DataSync task transferred the data
   - IAM role: Create a new role or use an existing one with Glue permissions
   - Schedule: Configure how often the crawler should run
   - Output database: Select or create a database in the Glue Catalog
   - Configure other options as needed

5. Run the crawler to discover the schema of your transferred data.
6. Once the crawler completes, the tables will be available in the Glue Catalog, allowing you to query the data using Athena, Redshift Spectrum, or other services.

### 7. Example: Transferring Historical Weather Data

Let's walk through an example of transferring historical weather data from an existing S3 bucket to our Lakehouse:

1. Create a DataSync task:
   - Source: S3 bucket containing historical weather data
   - Destination: Your Bronze layer S3 bucket with the prefix `bronze/historical/weather/`
   - Schedule: One-time transfer for this example

2. Run the task to transfer the data.

3. Create a Glue crawler to catalog the data:
   - Data store: S3 path to the transferred data
   - Database: `bronze_db`
   - Table prefix: `historical_weather_`

4. Run the crawler to create tables in the Glue Catalog.

5. You can now query this data using Athena:

```sql
SELECT * 
FROM bronze_db.historical_weather_data
WHERE year = 2024
LIMIT 10;
```

### Summary

In this section, we've set up AWS DataSync to transfer existing data into our Lakehouse architecture. DataSync provides a reliable and efficient way to migrate historical data and set up recurring transfers from various sources.

With DataSync, we can ensure our Lakehouse contains all relevant data, whether it's from live streams (via IoT Core and API Gateway) or from existing systems. In the next section, we'll explore how to set up the Storage Layer of our Lakehouse using Amazon S3 and Delta Lake.
