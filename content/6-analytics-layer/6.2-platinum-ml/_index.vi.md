---
title : "Tạo Lớp Học Máy Platinum"
date : 2025-08-10
weight : 2
chapter : false
pre : " <b> 6.2 </b> "
---

## Tạo Lớp Học Máy Platinum

Trong phần này, chúng ta sẽ xây dựng các mô hình học máy để phân tích dữ liệu chất lượng không khí của chúng ta. Chúng ta sẽ sử dụng dữ liệu lớp Silver để huấn luyện các mô hình phân loại có thể dự đoán mức AQI dựa trên các phép đo cảm biến, tạo ra các tài nguyên có giá trị cho lớp Platinum của chúng ta.

### Quy trình Phát triển Mô hình Học Máy

Pipeline học máy của chúng ta tuân theo các giai đoạn chính sau:

| Giai đoạn | Mục tiêu |
|-----------|----------|
| 1 | Đọc dữ liệu từ Delta Lake (lớp Silver) |
| 2 | Chuyển đổi kiểu dữ liệu phù hợp |
| 3 | Encode biến mục tiêu (AQI_Level) thành nhãn số |
| 4 | Chọn các đặc trưng phù hợp (CO, NO2, PM25, Nhiệt độ, Độ ẩm, v.v.) |
| 5 | Vector hóa và chuẩn hóa các đặc trưng |
| 6 | Chia dữ liệu thành tập huấn luyện/kiểm tra |
| 7 | Huấn luyện mô hình phân loại |
| 8 | Dự đoán và đánh giá hiệu suất |
| 9 | Lưu mô hình và kết quả dự đoán vào Delta Lake (lớp Platinum) |

### Tạo Mô hình Phân loại AQI

Hãy tạo một mô hình phân loại toàn diện dự đoán mức chất lượng không khí từ dữ liệu cảm biến của chúng ta. Tạo file có tên `platinum_classify_aqi_from_silver.py` với mã sau:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count
from pyspark.ml.feature import VectorAssembler, StandardScaler, StringIndexer
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

# 1. Khởi tạo SparkSession
spark = (
    SparkSession.builder
    .appName("AQI Classification Model")
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")
    .getOrCreate()
)

# 2. Đọc dữ liệu từ lớp Silver
df = spark.read.format("delta").load("/lakehouse/silver/air_quality_cleaned/")
df.show()

# 3. Ép kiểu dữ liệu phù hợp
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

# 4. Encode AQI_Level thành nhãn số
indexer = StringIndexer(inputCol="AQI_Level", outputCol="label")
indexer_model = indexer.fit(df)
df = indexer_model.transform(df)

# 5. Vector hóa các đặc trưng đầu vào
feature_cols = ["CO", "NO2", "PM25", "Temperature", "Humidity"]
assembler = VectorAssembler(inputCols=feature_cols, outputCol="raw_features")
df_vector = assembler.transform(df)

# 6. Chuẩn hóa đặc trưng
scaler = StandardScaler(inputCol="raw_features", outputCol="features", withMean=True, withStd=True)
scaler_model = scaler.fit(df_vector)
df_scaled = scaler_model.transform(df_vector)

# 7. Chia dữ liệu thành tập huấn luyện/kiểm tra
train_data, test_data = df_scaled.randomSplit([0.8, 0.2], seed=123)

# 8. Huấn luyện bộ phân loại Random Forest
rf = RandomForestClassifier(labelCol="label", featuresCol="features", numTrees=50, seed=123)
rf_model = rf.fit(train_data)

# 9. Thực hiện dự đoán và đánh giá
predictions = rf_model.transform(test_data)
evaluator = MulticlassClassificationEvaluator(labelCol="label", predictionCol="prediction")

# 10. Tính các chỉ số hiệu suất
accuracy = evaluator.setMetricName("accuracy").evaluate(predictions)
f1 = evaluator.setMetricName("f1").evaluate(predictions)
precision = evaluator.setMetricName("weightedPrecision").evaluate(predictions)
recall = evaluator.setMetricName("weightedRecall").evaluate(predictions)

print(f"\n\n\nHiệu suất Random Forest:")
print(f"Độ chính xác: {accuracy:.4f}")
print(f"Điểm F1: {f1:.4f}")
print(f"Precision: {precision:.4f}")
print(f"Recall: {recall:.4f}")

# 11. Lấy ánh xạ tên nhãn
label_names = indexer_model.labels

# 12. Tạo và hiển thị ma trận nhầm lẫn
confusion_df = predictions.groupBy("label", "prediction") \
    .agg(count("*").alias("count")) \
    .orderBy("label", "prediction")

confusion_data = confusion_df.collect()

print("\n\n\nMa trận nhầm lẫn - Random Forest:")
print(f"{'Nhãn':<5} {'Tên Nhãn':<30} {'Dự đoán':<10} {'Tên Dự đoán':<30} {'Số lượng':<6}")
print("-" * 90)
for row in confusion_data:
    label_idx = int(row['label'])
    pred_idx = int(row['prediction'])
    label_name = label_names[label_idx]
    pred_name = label_names[pred_idx]
    count_val = row['count']
    print(f"{label_idx:<5} {label_name:<30} {pred_idx:<10} {pred_name:<30} {count_val:<6}")

# 13. Hiển thị độ quan trọng của đặc trưng
print("\n\n\nĐộ quan trọng của đặc trưng - Random Forest:")
for feature, importance in zip(feature_cols, rf_model.featureImportances):
    print(f"{feature}: {importance:.4f}")

# 14. Lưu mô hình vào lớp Platinum
rf_model.save("/lakehouse/platinum/air_quality_classified/aqi_rf_model_from_silver")

# 15. Lưu kết quả dự đoán vào Delta Lake
predictions.select("CO", "NO2", "PM25", "Temperature", "Humidity", "AQI_Level", "label", "prediction") \
    .write.format("delta") \
    .mode("overwrite") \
    .save("/lakehouse/platinum/air_quality_classified/air_quality_predictions_from_silver")

# Dừng phiên Spark
spark.stop()
```

### Chạy Job Huấn luyện Mô hình

Để huấn luyện mô hình, thực thi script với Spark submit, đảm bảo gói Delta Lake được bao gồm:

```bash
spark-submit --packages io.delta:delta-spark_2.12:3.0.0 platinum_classify_aqi_from_silver.py
```

### Phân tích Kết quả Mô hình

Sau khi chạy mô hình, bạn có thể phân tích kết quả để hiểu hiệu suất của nó:

#### 1. Đọc Kết quả Dự đoán

```python
from pyspark.sql import SparkSession

# Khởi tạo SparkSession
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

#### 2. Diễn giải Kết quả

Khi phân tích kết quả dự đoán, hãy chú ý đến:

| AQI_Level | label | prediction | Diễn giải |
|-----------|-------|------------|-----------|
| Average   | 0     | 0          | Dự đoán đúng |
| Unhealthy | 2     | 3          | Không đúng: mô hình nhầm lẫn với "Moderate" |
| Moderate  | 3     | 3          | Dự đoán đúng |
| Average   | 0     | 3          | Không đúng: mô hình nhầm lẫn "Average" với "Moderate" |

#### 3. Phân tích Mô hình Nâng cao

Để phân tích chi tiết hơn, hãy xem xét:

- **Ma trận Nhầm lẫn**: Trực quan hóa độ chính xác của dự đoán trên tất cả các mức AQI
- **Độ Quan trọng của Đặc trưng**: Hiểu các phép đo nào ảnh hưởng nhiều nhất đến dự đoán chất lượng không khí
- **Precision/Recall theo Lớp**: Kiểm tra hiệu suất mô hình cho các mức chất lượng không khí cụ thể

### Các Kết quả Đầu ra

Quá trình huấn luyện mô hình tạo ra hai kết quả chính trong lớp Platinum:

| Loại | Đường dẫn |
|------|-----------|
| Mô hình Đã huấn luyện | /lakehouse/platinum/air_quality_classified/aqi_rf_model_from_silver |
| Dự đoán & Nhãn thực tế | /lakehouse/platinum/air_quality_classified/air_quality_predictions_from_silver |

### Tích hợp với Phân tích và Trực quan hóa

Kết quả từ các mô hình học máy của chúng ta có thể được:

1. Xuất dưới dạng file Parquet để trực quan hóa trong các công cụ như Power BI
2. Sử dụng cho phân tích nâng cao hơn trong lớp Platinum
3. Tận dụng cho các chỉ số đánh giá AIQ
4. Tích hợp vào các dịch vụ dự đoán thời gian thực

Trong phần tiếp theo, chúng ta sẽ khám phá cách đánh giá hiệu suất mô hình của chúng ta và tích hợp nó với framework AIQ của chúng ta.
