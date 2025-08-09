---
title : "Xây dựng Phân tích Lớp Gold"
date : 2025-08-10
weight : 1
chapter : false
pre : " <b> 6.1 </b> "
---

## Xây dựng Phân tích Lớp Gold

Trong phần này, chúng ta sẽ tạo ra nhiều góc nhìn phân tích khác nhau về dữ liệu chất lượng không khí bằng cách chuyển đổi dữ liệu từ lớp Silver sang lớp Gold. Chúng ta sẽ tập trung vào việc tổng hợp theo thời gian và phân loại để cung cấp những hiểu biết có ý nghĩa về các mô hình chất lượng không khí.

### Phân tích Thời gian của Dữ liệu Chất lượng Không khí

Đầu tiên, hãy tạo các giá trị trung bình theo ngày, giờ và tháng của các chỉ số chất lượng không khí từ dữ liệu lớp Silver.

#### 1. Tính toán Giá trị Trung bình Theo Ngày

Tạo file có tên `daily_avg.py` với mã sau:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import avg, to_date, when, col

# Khởi tạo Spark Session với hỗ trợ Delta Lake
spark = SparkSession.builder \
    .appName("Daily AQI Average") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Đọc dữ liệu từ lớp Silver
df_silver = spark.read.format("delta").load("hdfs:///lakehouse/silver/air_quality_cleaned/")

# Tính toán giá trị trung bình theo ngày và gán mức AQI
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

# Ghi kết quả vào lớp Gold
df_gold.write.format("delta") \
    .mode("overwrite") \
    .save("hdfs:///lakehouse/gold/air_quality_daily_avg")
```

#### 2. Tính toán Giá trị Trung bình Theo Giờ

Tạo file có tên `hourly_avg.py` với mã sau:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import avg, hour, month

# Khởi tạo Spark Session với hỗ trợ Delta Lake
spark = SparkSession.builder \
    .appName("Hourly AQI Average") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Đọc dữ liệu từ lớp Silver
df_silver = spark.read.format("delta").load("hdfs:///lakehouse/silver/air_quality_cleaned/")

# Tính toán giá trị trung bình theo giờ
df_hourly_avg = df_silver.withColumn("hour", hour("date")) \
    .groupBy("hour") \
    .agg(
        avg("PM25").alias("avg_PM25"),
        avg("CO").alias("avg_CO"),
        avg("Temperature").alias("avg_temp")
    ) \
    .orderBy("hour")

# Ghi kết quả vào lớp Gold
df_hourly_avg.write.format("delta") \
    .mode("overwrite") \
    .save("hdfs:///lakehouse/gold/air_quality_hourly_avg")
```

#### 3. Tính toán Giá trị Trung bình Theo Tháng

Tạo file có tên `monthly_avg.py` với mã sau:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import avg, month

# Khởi tạo Spark Session với hỗ trợ Delta Lake
spark = SparkSession.builder \
    .appName("Monthly AQI Average") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Đọc dữ liệu từ lớp Silver
df_silver = spark.read.format("delta").load("hdfs:///lakehouse/silver/air_quality_cleaned/")

# Tính toán giá trị trung bình theo tháng
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

# Ghi kết quả vào lớp Gold
df_monthly_avg.write.format("delta") \
    .mode("overwrite") \
    .save("hdfs:///lakehouse/gold/air_quality_monthly_avg")
```

### Phân tích Dữ liệu Khám phá (EDA)

Bây giờ, chúng ta sẽ thực hiện một số phân tích dữ liệu khám phá để hiểu rõ hơn về dữ liệu chất lượng không khí của chúng ta.

Tạo file có tên `air_quality_eda.py` với mã sau:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import avg, hour, month, when, col, to_date
from pyspark.sql.functions import count, isnan, stddev, min, max

# Khởi tạo Spark Session với hỗ trợ Delta Lake
spark = SparkSession.builder \
    .appName("Air Quality EDA") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Đọc dữ liệu từ lớp Silver
df_silver = spark.read.format("delta").load("/lakehouse/silver/air_quality_cleaned/")

print("Cấu trúc và Mẫu Dữ liệu:")
df_silver.printSchema()
df_silver.show(5)

# 1. Thống kê mô tả cơ bản cho các cột số
print("\nThống kê Cơ bản:")
df_silver.describe().show()

# 2. Kiểm tra giá trị null và NaN trong mỗi cột
print("\nĐếm Null và NaN:")
cols = []
for c, t in df_silver.dtypes:
    if t in ('double', 'float'):
        # Kiểm tra null hoặc isnan cho cột số
        cols.append(count(when(col(c).isNull() | isnan(col(c)), c)).alias(c))
    else:
        # Chỉ kiểm tra null cho cột khác
        cols.append(count(when(col(c).isNull(), c)).alias(c))

df_silver.select(cols).show()

# 3. Đếm bản ghi theo AQI_Level
print("\nPhân bố mức AQI:")
df_silver.groupBy("AQI_Level").count().show()

# 4. Tính toán thống kê PM2.5 hàng ngày
print("\nThống kê PM2.5 Hàng ngày:")
df_silver = df_silver.withColumn("date_only", to_date(col("date")))

df_silver.groupBy("date_only") \
    .agg(
        avg("PM25").alias("avg_PM25"),
        min("PM25").alias("min_PM25"),
        max("PM25").alias("max_PM25")
    ).orderBy("date_only").show(10)

# 5. Đếm bản ghi theo giờ
print("\nPhân bố Theo Giờ:")
df_silver.groupBy("hour").count().orderBy("hour").show(24)

# 6. Tìm phạm vi ngày của bộ dữ liệu
print("\nKhoảng thời gian Dữ liệu:")
df_silver.select(
    min("date").alias("start_date"),
    max("date").alias("end_date")
).show()
```

### Học Máy: Phân loại mức AQI

Bây giờ, chúng ta sẽ xây dựng một mô hình học máy để phân loại các mức chất lượng không khí dựa trên dữ liệu cảm biến.

Tạo file có tên `platinum_classify_aqi.py` với mã sau:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count
from pyspark.ml.feature import StringIndexer, VectorAssembler, StandardScaler
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

# Khởi tạo Spark Session với hỗ trợ Delta Lake
spark = (
    SparkSession.builder
    .appName("AQI Classification Model")
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()
)

# Đọc dữ liệu từ lớp Silver
df = spark.read.format("delta").load("/lakehouse/silver/air_quality_cleaned/")

# Chuyển đổi kiểu dữ liệu một cách phù hợp
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

# Encode nhãn AQI_Level
indexer = StringIndexer(inputCol="AQI_Level", outputCol="label")
indexer_model = indexer.fit(df)
df = indexer_model.transform(df)

# Vector hóa các đặc trưng đầu vào
feature_cols = ["CO", "NO2", "PM25", "Temperature", "Humidity"]
assembler = VectorAssembler(inputCols=feature_cols, outputCol="raw_features")
df_vector = assembler.transform(df)

# Chuẩn hóa đặc trưng
scaler = StandardScaler(inputCol="raw_features", outputCol="features", withMean=True, withStd=True)
scaler_model = scaler.fit(df_vector)
df_scaled = scaler_model.transform(df_vector)

# Chia dữ liệu thành tập huấn luyện và tập kiểm tra
train_data, test_data = df_scaled.randomSplit([0.8, 0.2], seed=123)

# Huấn luyện mô hình Random Forest
rf = RandomForestClassifier(labelCol="label", featuresCol="features", numTrees=50, seed=123)
rf_model = rf.fit(train_data)

# Đánh giá mô hình
pred_rf = rf_model.transform(test_data)

evaluator = MulticlassClassificationEvaluator(labelCol="label", predictionCol="prediction")

# Tính toán các chỉ số
accuracy = evaluator.setMetricName("accuracy").evaluate(pred_rf)
f1 = evaluator.setMetricName("f1").evaluate(pred_rf)
precision = evaluator.setMetricName("weightedPrecision").evaluate(pred_rf)
recall = evaluator.setMetricName("weightedRecall").evaluate(pred_rf)

print("\nHiệu suất Random Forest:")
print(f"Độ chính xác: {accuracy:.4f}")
print(f"Điểm F1: {f1:.4f}")
print(f"Precision: {precision:.4f}")
print(f"Recall: {recall:.4f}")

# Lấy tên nhãn cho ma trận nhầm lẫn
label_names = indexer_model.labels

# Tạo và hiển thị ma trận nhầm lẫn
confusion_df = pred_rf.groupBy("label", "prediction") \
    .agg(count("*").alias("count")) \
    .orderBy("label", "prediction")

confusion_data = confusion_df.collect()

print("\nMa trận nhầm lẫn - Random Forest:")
print(f"{'Nhãn':<5} {'Tên Nhãn':<30} {'Dự đoán':<10} {'Tên Dự đoán':<30} {'Số lượng':<6}")
print("-" * 90)
for row in confusion_data:
    label_idx = int(row['label'])
    pred_idx = int(row['prediction'])
    label_name = label_names[label_idx]
    pred_name = label_names[pred_idx]
    count_val = row['count']
    print(f"{label_idx:<5} {label_name:<30} {pred_idx:<10} {pred_name:<30} {count_val:<6}")

# Hiển thị độ quan trọng của các đặc trưng
print("\nĐộ quan trọng của đặc trưng - Random Forest:")
for feature, importance in zip(feature_cols, rf_model.featureImportances):
    print(f"{feature}: {importance:.4f}")

# Lưu mô hình Random Forest
rf_model.save("/lakehouse/platinum/air_quality_classified/aqi_rf_model_from_silver")

# Lưu kết quả dự đoán vào Delta Lake
pred_rf.select("CO", "NO2", "PM25", "Temperature", "Humidity", "AQI_Level", "label", "prediction") \
    .write.format("delta") \
    .mode("overwrite") \
    .save("/lakehouse/platinum/air_quality_classified/air_quality_predictions_from_silver")

# Dừng phiên Spark
spark.stop()
```

### Xuất Dữ liệu để Trực quan hóa

Cuối cùng, hãy xuất dữ liệu lớp gold của chúng ta để trực quan hóa trong các công cụ như Power BI:

```bash
# Tạo thư mục cục bộ để xuất
mkdir -p ~/air_quality_export

# Sao chép các file parquet từ HDFS về thư mục cục bộ
hdfs dfs -get /lakehouse/gold/air_quality_daily_avg/part-00000-*.parquet ~/air_quality_export/daily.parquet
hdfs dfs -get /lakehouse/gold/air_quality_hourly_avg/part-00000-*.parquet ~/air_quality_export/hourly.parquet
hdfs dfs -get /lakehouse/gold/air_quality_monthly_avg/part-00000-*.parquet ~/air_quality_export/monthly.parquet

# Sao chép các file về máy cục bộ của bạn (chạy từ terminal cục bộ)
scp username@remote-server:~/air_quality_export/*.parquet .
```

### Phân tích Kết quả Mô hình

Sau khi chạy mô hình phân loại, bạn có thể phân tích kết quả với:

```python
from pyspark.sql import SparkSession

# Khởi tạo Spark Session
spark = SparkSession.builder \
    .appName("Delta Lake App") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Đọc kết quả dự đoán
pred_df = spark.read.format("delta").load("/lakehouse/platinum/air_quality_classified/air_quality_predictions_from_silver")

# Hiển thị dự đoán so với nhãn thực tế
pred_df.select("AQI_Level", "label", "prediction").show(20, truncate=False)
```

### Những Hiểu biết Chính từ Phân tích AQI

Dựa trên phân tích dữ liệu của chúng ta:

1. **Phân bố AQI**: 
   - Chất lượng trung bình (~58% điểm dữ liệu)
   - Chất lượng tốt (~21%)
   - Mức trung bình và không lành mạnh kết hợp (~16%)

2. **Độ Quan trọng của Đặc trưng**:
   - PM2.5 là đặc trưng quan trọng nhất để dự đoán mức AQI
   - Nhiệt độ và độ ẩm cũng đóng vai trò quan trọng

3. **Mô hình Thời gian**:
   - Chất lượng không khí thay đổi trong suốt ngày, với một số giờ nhất định cho thấy kết quả đọc tệ hơn
   - Các mô hình hàng tháng cho thấy sự thay đổi theo mùa trong chất lượng không khí

Những hiểu biết này tạo nền tảng cho bảng điều khiển trực quan hóa và đánh giá AIQ của chúng ta trong các phần tiếp theo.
