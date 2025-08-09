---
title : "Thiết lập API Gateway"
date : 2025-08-10
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---

## Thiết lập API Gateway

Trong phần này, chúng ta sẽ cấu hình Amazon API Gateway để tạo các API RESTful có thể đóng vai trò là một điểm tiếp nhận dữ liệu khác cho kiến trúc Lakehouse của chúng ta. Điều này cung cấp một cách linh hoạt để nhận dữ liệu từ nhiều nguồn khác nhau, như ứng dụng web, ứng dụng di động hoặc hệ thống của bên thứ ba.

### 1. Tạo API Gateway

Hãy bắt đầu bằng việc tạo một REST API trong API Gateway:

1. Điều hướng đến [Amazon API Gateway console](https://console.aws.amazon.com/apigateway/).
2. Nhấn **Create API** và chọn **REST API** (không phải private).
3. Chọn **New API** và cung cấp thông tin sau:
   - **API name**: `aiq-data-api`
   - **Description**: API for AIQ data ingestion
   - **Endpoint Type**: Regional
4. Nhấn **Create API**.

### 2. Tạo Resource và Method

Bây giờ, hãy tạo một resource và một phương thức POST để nhận dữ liệu:

1. Trong thanh điều hướng bên trái, chọn **Resources**.
2. Nhấn vào **Actions** và chọn **Create Resource**.
3. Nhập các chi tiết sau:
   - **Resource Name**: `data`
   - **Resource Path**: `/data`
   - Chọn **Enable API Gateway CORS**
4. Nhấn **Create Resource**.
5. Với resource `/data` được chọn, nhấn **Actions** và chọn **Create Method**.
6. Chọn **POST** từ danh sách và nhấn dấu tích.
7. Cấu hình phương thức POST như sau:
   - **Integration type**: Lambda Function
   - **Use Lambda Proxy integration**: Đánh dấu ô này
   - **Lambda Region**: Chọn region của bạn
   - **Lambda Function**: Chúng ta sẽ tạo cái này trong bước tiếp theo
8. Đừng nhấn **Save** vội, vì chúng ta cần tạo hàm Lambda trước.

### 3. Tạo Hàm Lambda cho Tích hợp API

Chúng ta cần tạo một hàm Lambda sẽ xử lý các yêu cầu API đến và gửi dữ liệu đến cụm MSK của chúng ta:

1. Mở một tab trình duyệt mới và điều hướng đến [AWS Lambda console](https://console.aws.amazon.com/lambda/).
2. Nhấn **Create function**.
3. Chọn **Author from scratch** và nhập các chi tiết sau:
   - **Function name**: `api-to-msk-function`
   - **Runtime**: Node.js 16.x
   - **Architecture**: x86_64
   - **Execution role**: Tạo một vai trò mới với các quyền Lambda cơ bản
4. Nhấn **Create function**.
5. Trong trình soạn thảo mã hàm, thay thế mã với mã sau:

```javascript
const AWS = require('aws-sdk');
const { Kafka } = require('kafkajs');
const kafka = new Kafka({
  clientId: 'api-gateway-producer',
  brokers: process.env.BOOTSTRAP_SERVERS.split(','),
  ssl: true,
  sasl: {
    mechanism: 'scram-sha-512',
    username: process.env.KAFKA_USERNAME,
    password: process.env.KAFKA_PASSWORD
  }
});

const producer = kafka.producer();

exports.handler = async (event) => {
  try {
    // Parse dữ liệu đến từ API Gateway
    const body = JSON.parse(event.body);
    
    // Thêm timestamp nếu chưa có
    if (!body.timestamp) {
      body.timestamp = new Date().toISOString();
    }
    
    // Thêm thông tin nguồn
    body.source = 'api_gateway';
    
    // Kết nối đến Kafka
    await producer.connect();
    
    // Gửi dữ liệu đến topic Kafka
    await producer.send({
      topic: process.env.KAFKA_TOPIC,
      messages: [
        { value: JSON.stringify(body) }
      ]
    });
    
    // Ngắt kết nối từ Kafka
    await producer.disconnect();
    
    // Trả về phản hồi thành công
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      },
      body: JSON.stringify({ 
        message: 'Dữ liệu đã được tiếp nhận thành công',
        timestamp: body.timestamp
      })
    };
  } catch (error) {
    console.error('Lỗi:', error);
    
    // Trả về phản hồi lỗi
    return {
      statusCode: 500,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      },
      body: JSON.stringify({ 
        message: 'Lỗi khi tiếp nhận dữ liệu',
        error: error.message
      })
    };
  }
};
```

6. Cuộn xuống phần **Environment variables** và thêm các biến sau:
   - **BOOTSTRAP_SERVERS**: Bootstrap servers MSK của bạn (SASL endpoint)
   - **KAFKA_USERNAME**: Tên người dùng từ secret trong AWS Secrets Manager của bạn
   - **KAFKA_PASSWORD**: Mật khẩu từ secret trong AWS Secrets Manager của bạn
   - **KAFKA_TOPIC**: `aiq-api-data`

7. Trong tab **Configuration**:
   - Đặt **Memory** là 256 MB
   - Đặt **Timeout** là 30 giây
   - Trong phần **VPC**, nhấn **Edit** và chọn:
     - **VPC**: Lakehouse-VPC
     - **Subnets**: Chọn ít nhất hai private subnet
     - **Security groups**: Chọn MSK-SG

8. Nhấn **Save**.

9. Bạn cũng cần thêm thư viện KafkaJS như một phụ thuộc. Chúng ta có thể làm điều này bằng cách sử dụng một layer Lambda:
   - Trong terminal trên máy cục bộ của bạn, tạo một thư mục cho layer:
     ```bash
     mkdir -p kafka-layer/nodejs
     cd kafka-layer/nodejs
     npm init -y
     npm install kafkajs
     cd ..
     zip -r kafka-layer.zip nodejs
     ```
   - Trong bảng điều khiển Lambda, đi đến **Layers** trong thanh điều hướng bên trái
   - Nhấn **Create layer**
   - Đặt tên là `kafkajs-layer`
   - Tải lên file zip bạn đã tạo
   - Chọn các runtime tương thích (Node.js 16.x)
   - Nhấn **Create**
   - Quay lại hàm Lambda của bạn
   - Trong phần **Layers**, nhấn **Add a layer**
   - Chọn **Custom layers**
   - Chọn `kafkajs-layer` bạn vừa tạo
   - Nhấn **Add**

### 4. Kết nối API Gateway với Lambda

Bây giờ chúng ta đã tạo hàm Lambda, hãy kết nối nó với API Gateway:

1. Quay lại tab bảng điều khiển API Gateway.
2. Trong thiết lập phương thức POST, chọn `api-to-msk-function` từ danh sách thả xuống Lambda function.
3. Nhấn **Save** và **OK** để cấp quyền cho API Gateway gọi hàm Lambda của bạn.

### 5. Tạo Kafka Topic cho Dữ liệu API

Chúng ta cần tạo một Kafka topic mới cho dữ liệu đến từ API:

1. Kết nối đến EC2 instance của bạn.
2. Sử dụng các công cụ client Kafka chúng ta đã thiết lập trong phần trước, tạo một topic mới:

```bash
./kafka_2.13-2.8.1/bin/kafka-topics.sh --create \
--bootstrap-server $BOOTSTRAP_SERVERS \
--topic aiq-api-data \
--partitions 3 \
--replication-factor 3 \
--command-config client.properties
```

### 6. Triển khai API

Bây giờ, hãy triển khai API của chúng ta để làm cho nó có thể truy cập:

1. Trong bảng điều khiển API Gateway, chọn **Actions** và nhấn **Deploy API**.
2. Đối với **Deployment stage**, chọn **New Stage**.
3. Nhập `prod` cho **Stage name** và thêm mô tả.
4. Nhấn **Deploy**.
5. Ghi lại **Invoke URL** hiển thị ở đầu màn hình. Đây là URL bạn sẽ sử dụng để gửi dữ liệu đến API của mình.

### 7. Kiểm tra API

Hãy kiểm tra API của chúng ta để đảm bảo nó hoạt động đúng:

1. Bạn có thể sử dụng một công cụ như cURL hoặc Postman để gửi một yêu cầu POST đến API của bạn.

Sử dụng cURL:
```bash
curl -X POST \
  <invoke-url-của-bạn>/data \
  -H 'Content-Type: application/json' \
  -d '{
  "device_id": "api-device-001",
  "temperature": 24.5,
  "humidity": 65.2,
  "pressure": 1012.3,
  "air_quality": 85.7,
  "data_type": "environmental"
}'
```

2. Nếu thành công, bạn sẽ nhận được phản hồi như:
```json
{
  "message": "Dữ liệu đã được tiếp nhận thành công",
  "timestamp": "2025-08-10T15:30:45.123Z"
}
```

3. Để xác minh rằng dữ liệu đang chảy đến Kafka, chạy một consumer trên EC2 instance của bạn:
```bash
./kafka_2.13-2.8.1/bin/kafka-console-consumer.sh \
--bootstrap-server $BOOTSTRAP_SERVERS \
--topic aiq-api-data \
--from-beginning \
--command-config client.properties
```

4. Bạn sẽ thấy thông điệp JSON của mình xuất hiện trong đầu ra consumer.

### 8. Thiết lập MSK Connect cho Dữ liệu API

Tương tự như những gì chúng ta đã làm cho dữ liệu IoT, hãy thiết lập một connector MSK Connect để truyền dữ liệu API đến S3:

1. Điều hướng đến bảng điều khiển Amazon MSK và chọn **MSK Connect**.

2. Tạo một connector (nếu bạn chưa tạo S3 sink connector trong phần trước, hãy làm theo các bước đó trước):
   - Nhấn **Create connector**
   - Chọn plugin `s3-sink-connector`
   - Cấu hình cài đặt connector:
     - **Name**: `api-to-s3-connector`
     - **Kafka cluster**: Chọn cụm MSK của bạn
     - **Authentication**: Chọn xác thực dựa trên vai trò IAM
     - **Workers**: Bắt đầu với 1
     - **Connector configuration**: Nhập cấu hình JSON sau:

```json
{
  "connector.class": "io.confluent.connect.s3.S3SinkConnector",
  "tasks.max": "1",
  "topics": "aiq-api-data",
  "s3.region": "<vùng-aws-của-bạn>",
  "s3.bucket.name": "<tên-bucket-lớp-bronze>",
  "s3.prefix": "api-data",
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

3. Thay thế `<vùng-aws-của-bạn>` và `<tên-bucket-lớp-bronze>` bằng vùng AWS và tên bucket S3 của bạn.

4. Cấu hình cùng vai trò thực thi dịch vụ mà bạn đã sử dụng cho connector trước đó.

5. Nhấn **Create connector**.

### Tóm tắt

Trong phần này, chúng ta đã thiết lập Amazon API Gateway như một điểm tiếp nhận dữ liệu khác cho kiến trúc Lakehouse của chúng ta. Chúng ta đã tạo một REST API với một hàm Lambda gửi dữ liệu đến đến cụm MSK. Chúng ta cũng đã thiết lập MSK Connect để truyền dữ liệu này đến lớp Bronze dựa trên S3 của chúng ta.

API Gateway cung cấp một cách linh hoạt cho các ứng dụng và hệ thống khác nhau đóng góp dữ liệu vào Lakehouse của chúng ta. Trong phần tiếp theo, chúng ta sẽ khám phá cách sử dụng AWS DataSync để tiếp nhận dữ liệu từ các nguồn dữ liệu hiện có.
