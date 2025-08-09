---
title : "Creating the Platinum Machine Learning Layer"
date : 2025-08-10
weight : 2
chapter : false
pre : " <b> 6.2 </b> "
---

## Creating the Platinum Machine Learning Layer

In this section, we'll build machine learning models to analyze our air quality data. We'll use the Silver layer data to train classification models that can predict AQI levels based on sensor measurements, creating valuable assets for our Platinum layer.

### Machine Learning Model Development Process

Our machine learning pipeline follows these key stages:

| Stage | Objective |
|-------|-----------|
| 1 | Load data from Delta Lake (Silver layer) |
| 2 | Convert data types appropriately |
| 3 | Encode the target variable (AQI_Level) into numerical labels |
| 4 | Select appropriate features (CO, NO2, PM25, Temperature, Humidity, etc.) |
| 5 | Vectorize and normalize features |
| 6 | Split data into training/testing sets |
| 7 | Train classification models |
| 8 | Predict and evaluate performance |
| 9 | Save models and prediction results to Delta Lake (Platinum layer) |

### Creating an AQI Classification Model

Let's create a comprehensive classification model that predicts air quality levels from our sensor data. Create a file named `platinum_classify_aqi_from_silver.py` with the following code:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count
from pyspark.ml.feature import VectorAssembler, StandardScaler, StringIndexer
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

# 1. Initialize SparkSession
spark = (
    SparkSession.builder
    .appName("AQI Classification Model")
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")
    .getOrCreate()
)

# 2. Read data from Silver layer
df = spark.read.format("delta").load("/lakehouse/silver/air_quality_cleaned/")
df.show()

# 3. Cast data types appropriately
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

# 4. Encode AQI_Level into numerical labels
indexer = StringIndexer(inputCol="AQI_Level", outputCol="label")
indexer_model = indexer.fit(df)
df = indexer_model.transform(df)

# 5. Vectorize input features
feature_cols = ["CO", "NO2", "PM25", "Temperature", "Humidity"]
assembler = VectorAssembler(inputCols=feature_cols, outputCol="raw_features")
df_vector = assembler.transform(df)

# 6. Normalize features
scaler = StandardScaler(inputCol="raw_features", outputCol="features", withMean=True, withStd=True)
scaler_model = scaler.fit(df_vector)
df_scaled = scaler_model.transform(df_vector)

# 7. Split data into training/testing sets
train_data, test_data = df_scaled.randomSplit([0.8, 0.2], seed=123)

# 8. Train Random Forest classifier
rf = RandomForestClassifier(labelCol="label", featuresCol="features", numTrees=50, seed=123)
rf_model = rf.fit(train_data)

# 9. Make predictions and evaluate
predictions = rf_model.transform(test_data)
evaluator = MulticlassClassificationEvaluator(labelCol="label", predictionCol="prediction")

# 10. Calculate performance metrics
accuracy = evaluator.setMetricName("accuracy").evaluate(predictions)
f1 = evaluator.setMetricName("f1").evaluate(predictions)
precision = evaluator.setMetricName("weightedPrecision").evaluate(predictions)
recall = evaluator.setMetricName("weightedRecall").evaluate(predictions)

print(f"\n\n\nRandom Forest Performance:")
print(f"Accuracy: {accuracy:.4f}")
print(f"F1 Score: {f1:.4f}")
print(f"Precision: {precision:.4f}")
print(f"Recall: {recall:.4f}")

# 11. Get label name mapping
label_names = indexer_model.labels

# 12. Create and display confusion matrix
confusion_df = predictions.groupBy("label", "prediction") \
    .agg(count("*").alias("count")) \
    .orderBy("label", "prediction")

confusion_data = confusion_df.collect()

print("\n\n\nConfusion Matrix - Random Forest:")
print(f"{'Label':<5} {'Label Name':<30} {'Predicted':<10} {'Prediction Name':<30} {'Count':<6}")
print("-" * 90)
for row in confusion_data:
    label_idx = int(row['label'])
    pred_idx = int(row['prediction'])
    label_name = label_names[label_idx]
    pred_name = label_names[pred_idx]
    count_val = row['count']
    print(f"{label_idx:<5} {label_name:<30} {pred_idx:<10} {pred_name:<30} {count_val:<6}")

# 13. Display feature importances
print("\n\n\nFeature Importances - Random Forest:")
for feature, importance in zip(feature_cols, rf_model.featureImportances):
    print(f"{feature}: {importance:.4f}")

# 14. Save the model to Platinum layer
rf_model.save("/lakehouse/platinum/air_quality_classified/aqi_rf_model_from_silver")

# 15. Save prediction results to Delta Lake
predictions.select("CO", "NO2", "PM25", "Temperature", "Humidity", "AQI_Level", "label", "prediction") \
    .write.format("delta") \
    .mode("overwrite") \
    .save("/lakehouse/platinum/air_quality_classified/air_quality_predictions_from_silver")

# Stop the Spark session
spark.stop()
```

### Running the Model Training Job

To train the model, execute the script with Spark submit, ensuring the Delta Lake package is included:

```bash
spark-submit --packages io.delta:delta-spark_2.12:3.0.0 platinum_classify_aqi_from_silver.py
```

### Analyzing Model Results

After running the model, you can analyze the results to understand its performance:

#### 1. Reading Prediction Results

```python
from pyspark.sql import SparkSession

# Initialize SparkSession
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

#### 2. Interpreting Results

When analyzing the prediction results, pay attention to:

| AQI_Level | label | prediction | Interpretation |
|-----------|-------|------------|----------------|
| Average   | 0     | 0          | Correct prediction |
| Unhealthy | 2     | 3          | Incorrect: model confused with "Moderate" |
| Moderate  | 3     | 3          | Correct prediction |
| Average   | 0     | 3          | Incorrect: model confused "Average" with "Moderate" |

#### 3. Advanced Model Analysis

For more detailed analysis, consider:

- **Confusion Matrix**: Visualize prediction accuracy across all AQI levels
- **Feature Importances**: Understand which measurements most influence air quality predictions
- **Precision/Recall by Class**: Examine model performance for specific air quality levels

### Output Artifacts

The model training process produces two key artifacts in the Platinum layer:

| Type | Path |
|------|------|
| Trained Model | /lakehouse/platinum/air_quality_classified/aqi_rf_model_from_silver |
| Predictions & Actual Labels | /lakehouse/platinum/air_quality_classified/air_quality_predictions_from_silver |

### Integration with Analytics and Visualization

The results from our machine learning models can be:

1. Exported as Parquet files for visualization in tools like Power BI
2. Used for further analytics in the Platinum layer
3. Leveraged for AIQ evaluation metrics
4. Integrated into real-time prediction services

In the next section, we'll explore how to evaluate our model's performance and integrate it with our AIQ framework.
