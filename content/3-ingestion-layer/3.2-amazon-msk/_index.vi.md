---
title : "Thiết lập Amazon MSK"
date : 2025-08-10
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

## Thiết lập Amazon MSK

Trong phần này, chúng ta sẽ thiết lập Amazon Managed Streaming for Apache Kafka (MSK) để xử lý dữ liệu truyền phát từ AWS IoT Core. MSK sẽ đóng vai trò là nền tảng streaming cho việc xử lý dữ liệu theo thời gian thực trong kiến trúc Lakehouse.

### 1. Tạo cụm MSK

Đầu tiên, hãy tạo một cụm Amazon MSK:

1. Điều hướng đến [Amazon MSK console](https://console.aws.amazon.com/msk/).
2. Nhấn **Create cluster**.
3. Chọn **Custom create** và nhập các thông tin sau:
   - **Cluster name**: `lakehouse-msk-cluster`
   - **Kafka version**: Chọn phiên bản ổn định mới nhất (ví dụ: 2.8.1)
4. Trong phần **Networking**:
   - **Virtual Private Cloud (VPC)**: Chọn `Lakehouse-VPC`
   - **Subnets**: Chọn ít nhất hai private subnet
   - **Security Groups**: Tạo security group mới `MSK-SG` hoặc chọn một cái hiện có
5. Đối với **Encryption**:
   - Bật mã hóa trong quá trình truyền (encryption in transit)
   - Bật mã hóa lúc nghỉ (encryption at rest) sử dụng khóa KMS
6. Đối với **Authentication**:
   - Bật xác thực SASL/SCRAM
   - Tạo một secret mới trong AWS Secrets Manager với tên người dùng và mật khẩu
7. Đối với **Monitoring**:
   - Bật giám sát với Prometheus
   - Bật giám sát mở với Prometheus
8. Đối với **Cluster configuration**:
   - Loại instance: `kafka.m5.large` (cho môi trường phát triển) hoặc chọn dựa trên nhu cầu của bạn
   - Số lượng broker: 3 (tối thiểu cho môi trường sản xuất)
   - Bộ nhớ: Bắt đầu với 100 GiB cho mỗi broker và bật tự động mở rộng
9. Nhấn **Create cluster**.

Việc tạo cụm sẽ mất khoảng 15-20 phút. Sau khi nó hoạt động, tiếp tục bước tiếp theo.

### 2. Cấu hình Cài đặt Bảo mật cho Truy cập IAM

Để bật cả xác thực SASL/SCRAM cho IoT Rule và xác thực IAM cho MSK Connect:

1. Trong bảng điều khiển MSK, chọn cụm của bạn.
2. Nhấn vào **Actions** và chọn **Edit security settings**.
3. Ngoài SASL/SCRAM, cũng chọn **IAM role-based authentication**.
4. Nhấn **Save changes**.

Cập nhật này sẽ mất khoảng 10-15 phút để hoàn thành.

### 3. Tạo Kafka Topic

Bây giờ chúng ta sẽ tạo một Kafka topic để nhận dữ liệu từ AWS IoT Core:

1. Kết nối đến EC2 instance đã được cấu hình trong phần trước.
2. Cài đặt Java và công cụ client Kafka nếu chưa được cài đặt:

```bash
sudo yum install -y java-11-amazon-corretto-headless
mkdir ~/kafka-client && cd ~/kafka-client
wget https://archive.apache.org/dist/kafka/2.8.1/kafka_2.13-2.8.1.tgz
tar -xzf kafka_2.13-2.8.1.tgz
```

3. Cấu hình client Kafka cho xác thực SASL/SCRAM:

```bash
# Tạo truststore
cp /usr/lib/jvm/java-11-amazon-corretto.x86_64/lib/security/cacerts kafka.truststore.jks

# Tạo cấu hình JAAS
cat > kafka_client_jaas.conf << EOF
KafkaClient {
  org.apache.kafka.common.security.scram.ScramLoginModule required
  username="<tên-người-dùng-msk>"
  password="<mật-khẩu-msk>";
};
EOF

# Tạo file thuộc tính client
cat > client.properties << EOF
security.protocol=SASL_SSL
sasl.mechanism=SCRAM-SHA-512
ssl.truststore.location=$(pwd)/kafka.truststore.jks
EOF
```

4. Thay thế `<tên-người-dùng-msk>` và `<mật-khẩu-msk>` bằng thông tin đăng nhập bạn đã lưu trong AWS Secrets Manager.

5. Đặt các biến môi trường cần thiết:

```bash
export BOOTSTRAP_SERVERS=$(aws kafka get-bootstrap-brokers --cluster-arn <arn-cụm-của-bạn> --query 'BootstrapBrokerStringSaslScram' --output text)
export KAFKA_OPTS="-Djava.security.auth.login.config=$(pwd)/kafka_client_jaas.conf"
```

6. Thay thế `<arn-cụm-của-bạn>` bằng ARN của cụm MSK của bạn.

7. Tạo Kafka topic:

```bash
./kafka_2.13-2.8.1/bin/kafka-topics.sh --create \
--bootstrap-server $BOOTSTRAP_SERVERS \
--topic aiq-sensor-data \
--partitions 3 \
--replication-factor 3 \
--command-config client.properties
```

8. Xác minh việc tạo topic:

```bash
./kafka_2.13-2.8.1/bin/kafka-topics.sh --list \
--bootstrap-server $BOOTSTRAP_SERVERS \
--command-config client.properties
```

### 4. Kiểm tra Tích hợp Kafka

Hãy kiểm tra xem dữ liệu từ AWS IoT Core có đang chảy chính xác đến cụm MSK của bạn không:

1. Đảm bảo rằng thiết bị mô phỏng IoT của bạn (đã tạo trong phần trước) đang chạy và gửi dữ liệu.

2. Trên EC2 instance của bạn, thiết lập một Kafka consumer để đọc tin nhắn từ topic:

```bash
./kafka_2.13-2.8.1/bin/kafka-console-consumer.sh \
--bootstrap-server $BOOTSTRAP_SERVERS \
--topic aiq-sensor-data \
--from-beginning \
--command-config client.properties
```

3. Bạn sẽ thấy các thông điệp JSON với dữ liệu cảm biến xuất hiện trong bảng điều khiển.

### 5. Thiết lập MSK Connect cho Amazon S3 (Lớp Bronze)

Bây giờ chúng ta sẽ thiết lập MSK Connect để truyền dữ liệu từ topic Kafka đến Amazon S3, nơi sẽ đóng vai trò là lớp Bronze trong kiến trúc Lakehouse của chúng ta:

1. Điều hướng đến bảng điều khiển Amazon MSK và chọn **MSK Connect** từ thanh bên trái.

2. Tạo plugin tùy chỉnh:
   - Nhấn **Create custom plugin**
   - Tải lên file JAR của Amazon S3 sink connector (confluent-kafka-connect-s3)
   - Đặt tên cho plugin của bạn `s3-sink-connector`
   - Nhấn **Create custom plugin**

3. Tạo connector:
   - Nhấn **Create connector**
   - Chọn plugin `s3-sink-connector`
   - Cấu hình cài đặt connector:
     - **Name**: `msk-to-s3-connector`
     - **Kafka cluster**: Chọn cụm MSK của bạn
     - **Authentication**: Chọn xác thực dựa trên vai trò IAM
     - **Workers**: Bắt đầu với 1 (điều chỉnh dựa trên khối lượng)
     - **Connector configuration**: Nhập cấu hình JSON sau:

```json
{
  "connector.class": "io.confluent.connect.s3.S3SinkConnector",
  "tasks.max": "1",
  "topics": "aiq-sensor-data",
  "s3.region": "<vùng-aws-của-bạn>",
  "s3.bucket.name": "<tên-bucket-lớp-bronze>",
  "s3.part.size": "5242880",
  "flush.size": "1000",
  "storage.class": "io.confluent.connect.s3.storage.S3Storage",
  "format.class": "io.confluent.connect.s3.format.json.JsonFormat",
  "partitioner.class": "io.confluent.connect.storage.partitioner.TimeBasedPartitioner",
  "path.format": "'year'=YYYY/'month'=MM/'day'=dd/'hour'=HH",
  "locale": "en-US",
  "timezone": "UTC",
  "partition.duration.ms": "3600000",
  "rotate.schedule.interval.ms": "60000"
}
```

4. Thay thế `<vùng-aws-của-bạn>` và `<tên-bucket-lớp-bronze>` bằng vùng AWS và tên bucket S3 của bạn.

5. Cấu hình vai trò thực thi dịch vụ:
   - Tạo một vai trò thực thi dịch vụ mới hoặc chọn một vai trò hiện có
   - Đảm bảo nó có các quyền cần thiết để ghi vào S3
   - Nhấn **Create connector**

6. Connector sẽ bắt đầu chạy trong thời gian ngắn. Bạn có thể theo dõi trạng thái của nó trên bảng điều khiển MSK Connect.

### 6. Kiểm tra Tích hợp S3

Hãy xác minh rằng dữ liệu đang chảy chính xác từ MSK đến S3:

1. Đảm bảo thiết bị mô phỏng IoT của bạn đang chạy và gửi dữ liệu.

2. Đợi vài phút để dữ liệu lan truyền qua pipeline.

3. Điều hướng đến [Amazon S3 console](https://console.aws.amazon.com/s3/) và duyệt đến bucket lớp bronze của bạn.

4. Bạn sẽ thấy các thư mục được tổ chức theo năm, tháng, ngày và giờ, chứa các file JSON với dữ liệu cảm biến.

### Tóm tắt

Trong phần này, chúng ta đã thiết lập Amazon MSK làm nền tảng streaming cho kiến trúc Lakehouse của chúng ta. Chúng ta đã cấu hình cụm MSK với các cài đặt bảo mật cần thiết, tạo một Kafka topic để nhận dữ liệu từ AWS IoT Core và thiết lập MSK Connect để truyền dữ liệu đến Amazon S3 như lớp Bronze của chúng ta.

Trong phần tiếp theo, chúng ta sẽ thiết lập các phương pháp nhập dữ liệu bổ sung sử dụng Amazon API Gateway và AWS DataSync để hoàn thành lớp nhập dữ liệu của chúng ta.
