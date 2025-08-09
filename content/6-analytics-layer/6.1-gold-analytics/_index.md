---
title : "Building Gold Layer Analytics"
date : 2025-08-10
weight : 1
chapter : false
pre : " <b> 6.1 </b> "
---

## Building Gold Layer Analytics

In this section, we'll create various analytical views of our air quality data by transforming the Silver layer data into the Gold layer. We'll focus on time-based aggregations and classification to provide meaningful insights into air quality patterns.

### Temporal Analysis of Air Quality Data

First, let's create daily, hourly, and monthly averages of our air quality metrics from the Silver layer data.

#### 1. Daily Average Calculation

Create a file named `daily_avg.py` with the following code:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import avg, to_date, when, col

# Initialize Spark Session with Delta Lake support
spark = SparkSession.builder \
    .appName("Daily AQI Average") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Read data from Silver layer
df_silver = spark.read.format("delta").load("hdfs:///lakehouse/silver/air_quality_cleaned/")

# Calculate daily averages and assign AQI levels
df_gold = df_silver.withColumn("date_only", to_date("date")) \
    .groupBy("date_only") \
    .agg(
        avg("PM25").alias("avg_PM25"),
        avg("Temperature").alias("avg_temp"),
        avg("Humidity").alias("avg_humidity")
    ) \
    .withColumn(
        "AQI_Level",
        when(col("avg_PM25") <= 12, "Good")
        .when((col("avg_PM25") > 12) & (col("avg_PM25") <= 35), "Average")
        .when((col("avg_PM25") > 35) & (col("avg_PM25") <= 55), "Moderate")
        .otherwise("Unhealthy")
    )

# Write results to Gold layer
df_gold.write.format("delta") \
    .mode("overwrite") \
    .save("hdfs:///lakehouse/gold/air_quality_daily_avg")
```

#### 2. Hourly Average Calculation

Create a file named `hourly_avg.py` with the following code:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import avg, hour, month

# Initialize Spark Session with Delta Lake support
spark = SparkSession.builder \
    .appName("Hourly AQI Average") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Read data from Silver layer
df_silver = spark.read.format("delta").load("hdfs:///lakehouse/silver/air_quality_cleaned/")

# Calculate hourly averages
df_hourly_avg = df_silver.withColumn("hour", hour("date")) \
    .groupBy("hour") \
    .agg(
        avg("PM25").alias("avg_PM25"),
        avg("CO").alias("avg_CO"),
        avg("Temperature").alias("avg_temp")
    ) \
    .orderBy("hour")

# Write results to Gold layer
df_hourly_avg.write.format("delta") \
    .mode("overwrite") \
    .save("hdfs:///lakehouse/gold/air_quality_hourly_avg")
```

#### 3. Monthly Average Calculation

Create a file named `monthly_avg.py` with the following code:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import avg, month

# Initialize Spark Session with Delta Lake support
spark = SparkSession.builder \
    .appName("Monthly AQI Average") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Read data from Silver layer
df_silver = spark.read.format("delta").load("hdfs:///lakehouse/silver/air_quality_cleaned/")

# Calculate monthly averages
df_monthly_avg = df_silver.withColumn("month", month("date")) \
    .groupBy("month") \
    .agg(
        avg("PM25").alias("avg_PM25"),
        avg("CO").alias("avg_CO"),
        avg("O3").alias("avg_O3"),
        avg("SO2").alias("avg_SO2"),
        avg("Temperature").alias("avg_temp"),
        avg("Humidity").alias("avg_humidity")
    )

# Write results to Gold layer
df_monthly_avg.write.format("delta") \
    .mode("overwrite") \
    .save("hdfs:///lakehouse/gold/air_quality_monthly_avg")
```

### Exploratory Data Analysis (EDA)

Let's perform some exploratory data analysis to better understand our air quality data.

Create a file named `air_quality_eda.py` with the following code:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import avg, hour, month, when, col, to_date
from pyspark.sql.functions import count, isnan, stddev, min, max

# Initialize Spark Session with Delta Lake support
spark = SparkSession.builder \
    .appName("Air Quality EDA") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Read data from Silver layer
df_silver = spark.read.format("delta").load("/lakehouse/silver/air_quality_cleaned/")

print("Schema and Sample Data:")
df_silver.printSchema()
df_silver.show(5)

# 1. Basic descriptive statistics for numerical columns
print("\nBasic Statistics:")
df_silver.describe().show()

# 2. Check for null and NaN values in each column
print("\nNull and NaN Count:")
cols = []
for c, t in df_silver.dtypes:
    if t in ('double', 'float'):
        # Check null or isnan for numeric columns
        cols.append(count(when(col(c).isNull() | isnan(col(c)), c)).alias(c))
    else:
        # Only check null for other columns
        cols.append(count(when(col(c).isNull(), c)).alias(c))

df_silver.select(cols).show()

# 3. Count records by AQI_Level
print("\nAQI Level Distribution:")
df_silver.groupBy("AQI_Level").count().show()

# 4. Calculate daily PM2.5 statistics
print("\nDaily PM2.5 Statistics:")
df_silver = df_silver.withColumn("date_only", to_date(col("date")))

df_silver.groupBy("date_only") \
    .agg(
        avg("PM25").alias("avg_PM25"),
        min("PM25").alias("min_PM25"),
        max("PM25").alias("max_PM25")
    ).orderBy("date_only").show(10)

# 5. Count records by hour
print("\nHourly Distribution:")
df_silver.groupBy("hour").count().orderBy("hour").show(24)

# 6. Find date range of the dataset
print("\nData Date Range:")
df_silver.select(
    min("date").alias("start_date"),
    max("date").alias("end_date")
).show()
```

### Machine Learning: AQI Level Classification

Now, let's build a machine learning model to classify air quality levels based on sensor data.

Create a file named `platinum_classify_aqi.py` with the following code:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count
from pyspark.ml.feature import StringIndexer, VectorAssembler, StandardScaler
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

# Initialize Spark Session with Delta Lake support
spark = (
    SparkSession.builder
    .appName("AQI Classification Model")
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")
    .getOrCreate()
)

# Read data from Silver layer
df = spark.read.format("delta").load("/lakehouse/silver/air_quality_cleaned/")

# Cast data types appropriately
df = df.select(
    col("CO").cast("double"),
    col("NO2").cast("double"),
    col("PM25").cast("double"),
    col("Temperature").cast("double"),
    col("Humidity").cast("double"),
    col("AQI_Level"),
    col("date"),
    col("hour"),
    col("day"),
    col("month"),
    col("Station_No")
).dropna()

# Encode AQI_Level labels
indexer = StringIndexer(inputCol="AQI_Level", outputCol="label")
indexer_model = indexer.fit(df)
df = indexer_model.transform(df)

# Vectorize input features
feature_cols = ["CO", "NO2", "PM25", "Temperature", "Humidity"]
assembler = VectorAssembler(inputCols=feature_cols, outputCol="raw_features")
df_vector = assembler.transform(df)

# Standardize features
scaler = StandardScaler(inputCol="raw_features", outputCol="features", withMean=True, withStd=True)
scaler_model = scaler.fit(df_vector)
df_scaled = scaler_model.transform(df_vector)

# Split data into training and testing sets
train_data, test_data = df_scaled.randomSplit([0.8, 0.2], seed=123)

# Train Random Forest model
rf = RandomForestClassifier(labelCol="label", featuresCol="features", numTrees=50, seed=123)
rf_model = rf.fit(train_data)

# Evaluate the model
pred_rf = rf_model.transform(test_data)

evaluator = MulticlassClassificationEvaluator(labelCol="label", predictionCol="prediction")

# Calculate metrics
accuracy = evaluator.setMetricName("accuracy").evaluate(pred_rf)
f1 = evaluator.setMetricName("f1").evaluate(pred_rf)
precision = evaluator.setMetricName("weightedPrecision").evaluate(pred_rf)
recall = evaluator.setMetricName("weightedRecall").evaluate(pred_rf)

print("\nRandom Forest Performance:")
print(f"Accuracy: {accuracy:.4f}")
print(f"F1 Score: {f1:.4f}")
print(f"Precision: {precision:.4f}")
print(f"Recall: {recall:.4f}")

# Get label names for confusion matrix
label_names = indexer_model.labels

# Create and display confusion matrix
confusion_df = pred_rf.groupBy("label", "prediction") \
    .agg(count("*").alias("count")) \
    .orderBy("label", "prediction")

confusion_data = confusion_df.collect()

print("\nConfusion Matrix - Random Forest:")
print(f"{'Label':<5} {'Label Name':<30} {'Predicted':<10} {'Prediction Name':<30} {'Count':<6}")
print("-" * 90)
for row in confusion_data:
    label_idx = int(row['label'])
    pred_idx = int(row['prediction'])
    label_name = label_names[label_idx]
    pred_name = label_names[pred_idx]
    count_val = row['count']
    print(f"{label_idx:<5} {label_name:<30} {pred_idx:<10} {pred_name:<30} {count_val:<6}")

# Display feature importances
print("\nFeature Importances - Random Forest:")
for feature, importance in zip(feature_cols, rf_model.featureImportances):
    print(f"{feature}: {importance:.4f}")

# Save the Random Forest model
rf_model.save("/lakehouse/platinum/air_quality_classified/aqi_rf_model_from_silver")

# Save the prediction results to Delta Lake
pred_rf.select("CO", "NO2", "PM25", "Temperature", "Humidity", "AQI_Level", "label", "prediction") \
    .write.format("delta") \
    .mode("overwrite") \
    .save("/lakehouse/platinum/air_quality_classified/air_quality_predictions_from_silver")

# Stop the Spark session
spark.stop()
```

### Exporting Data for Visualization

Finally, let's export our gold layer data for visualization in tools like Power BI:

```bash
# Create a local directory for export
mkdir -p ~/air_quality_export

# Copy parquet files from HDFS to local directory
hdfs dfs -get /lakehouse/gold/air_quality_daily_avg/part-00000-*.parquet ~/air_quality_export/daily.parquet
hdfs dfs -get /lakehouse/gold/air_quality_hourly_avg/part-00000-*.parquet ~/air_quality_export/hourly.parquet
hdfs dfs -get /lakehouse/gold/air_quality_monthly_avg/part-00000-*.parquet ~/air_quality_export/monthly.parquet

# Copy files to your local machine (run from local terminal)
scp username@remote-server:~/air_quality_export/*.parquet .
```

### Analyzing Model Results

After running the classification model, you can analyze the results with:

```python
from pyspark.sql import SparkSession

# Initialize Spark Session
spark = SparkSession.builder \
    .appName("Delta Lake App") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Read prediction results
pred_df = spark.read.format("delta").load("/lakehouse/platinum/air_quality_classified/air_quality_predictions_from_silver")

# Show predictions vs actual labels
pred_df.select("AQI_Level", "label", "prediction").show(20, truncate=False)
```

### Key Insights from AQI Analysis

Based on our data analysis:

1. **AQI Distribution**: 
   - Average quality (~58% of data points)
   - Good quality (~21%)
   - Moderate and Unhealthy combined (~16%)

2. **Feature Importance**:
   - PM2.5 is the most important feature for predicting AQI level
   - Temperature and humidity also play significant roles

3. **Temporal Patterns**:
   - Air quality varies throughout the day, with certain hours showing consistently worse readings
   - Monthly patterns show seasonal variations in air quality

These insights form the foundation for our visualization dashboards and AIQ evaluation in the next sections.
