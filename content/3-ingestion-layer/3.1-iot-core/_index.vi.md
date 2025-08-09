---
title : "Cấu hình AWS IoT Core"
date : 2025-08-10
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

## Cấu hình AWS IoT Core

AWS IoT Core là một dịch vụ được quản lý giúp kết nối và quản lý hàng tỷ thiết bị IoT. Trong phần này, chúng ta sẽ thiết lập AWS IoT Core để thu thập dữ liệu từ các cảm biến, sau đó định tuyến dữ liệu này đến Amazon MSK thông qua AWS IoT Rules.

### 1. Tạo một Thing trong AWS IoT Core

Đầu tiên, chúng ta sẽ tạo một "thing" đại diện cho thiết bị IoT trong AWS IoT Core:

1. Đăng nhập vào [AWS Management Console](https://console.aws.amazon.com/) và truy cập dịch vụ [AWS IoT Core](https://console.aws.amazon.com/iot/).
2. Trong menu bên trái, chọn **Manage** và sau đó chọn **Things**.
3. Nhấn **Create things**.
4. Chọn **Create a single thing** và nhấn **Next**.
5. Nhập thông tin cho thing:
   - **Thing name**: aiq-sensor
   - Giữ nguyên các tùy chọn còn lại ở giá trị mặc định
6. Nhấn **Next**.

### 2. Tạo và Tải xuống Chứng chỉ

Để thiết bị IoT có thể kết nối an toàn với AWS IoT Core, chúng ta cần tạo các chứng chỉ:

1. Ở màn hình **Add a certificate for your thing**, chọn **Auto-generate a new certificate**.
2. Nhấn **Next**.
3. Ở trang tiếp theo, tải xuống tất cả các file:
   - Certificate for this thing
   - Public key
   - Private key
   - Root CA
4. Nhấn **Activate** để kích hoạt chứng chỉ.
5. Nhấn **Attach a policy**.

### 3. Tạo và Đính kèm Policy

Tiếp theo, chúng ta sẽ tạo một policy để cấp quyền cho thiết bị:

1. Nhấn **Create policy**.
2. Nhập thông tin policy:
   - **Name**: aiq-sensor-policy
   - **Action**: iot:Connect, iot:Publish, iot:Subscribe, iot:Receive
   - **Resource ARN**: *
   - **Effect**: Allow
3. Nhấn **Create**.
4. Quay lại trang đính kèm policy, chọn policy vừa tạo và nhấn **Register Thing**.

### 4. Tạo IAM Role cho AWS IoT Rule

Chúng ta cần tạo IAM Role để AWS IoT Rule có thể gửi dữ liệu đến Amazon MSK:

1. Truy cập [AWS IAM Console](https://console.aws.amazon.com/iam/).
2. Chọn **Roles** và nhấn **Create role**.
3. Chọn **IoT** làm service và nhấn **Next**.
4. Tìm và chọn các policy sau:
   - AmazonMSKFullAccess
   - AWSIoTFullAccess
   - AWSIoTRuleActions
5. Nhấn **Next**.
6. Đặt tên role: **IoTtoMSKRole** và nhấn **Create role**.

### 5. Tạo AWS IoT Rule

Bây giờ chúng ta sẽ tạo một IoT Rule để định tuyến dữ liệu từ IoT Core đến Amazon MSK:

1. Quay lại [AWS IoT Core Console](https://console.aws.amazon.com/iot/).
2. Trong menu bên trái, chọn **Message routing** và sau đó chọn **Rules**.
3. Nhấn **Create rule**.
4. Nhập thông tin rule:
   - **Name**: aiq_sensor_to_msk
   - **Description**: Route sensor data to MSK
5. Trong phần **Rule query statement**, nhập SQL statement sau:
   ```sql
   SELECT *, timestamp() as timestamp FROM 'aiq/sensors/#'
   ```
6. Nhấn **Add action**.
7. Chọn **Send a message to an Apache Kafka broker** và nhấn **Configure action**.
8. Nhập thông tin về kết nối đến MSK:
   - **Destination**: Chọn VPC destination
   - **VPC**: Chọn Lakehouse-VPC
   - **Subnet**: Chọn Private Subnet 1
   - **Security group**: Chọn MSK-SG
   - **Apache Kafka topic**: aiq-sensor-data
   - **Bootstrap servers**: Nhập địa chỉ bootstrap server của cụm MSK (format: broker1:port,broker2:port,broker3:port)
   - **Security protocol**: SASL_SSL
   - **SASL mechanism**: SCRAM-SHA-512
   - **Username**: ${get_secret('<secret_name>', 'SecretString', 'username', '<role_arn>')}
   - **Password**: ${get_secret('<secret_name>', 'SecretString', 'password', '<role_arn>')}
   - Thay thế `<secret_name>` bằng tên của secret MSK trong AWS Secrets Manager
   - Thay thế `<role_arn>` bằng ARN của role IoTtoMSKRole
9. Chọn **IoTtoMSKRole** trong phần **Role**.
10. Nhấn **Add action** và sau đó nhấn **Create rule**.

### 6. Tạo mã thiết bị ảo IoT

Bây giờ, chúng ta sẽ tạo một thiết bị ảo chạy trên EC2 để mô phỏng cảm biến IoT:

1. Kết nối đến EC2 instance đã tạo trong phần trước.
2. Tạo thư mục cho dự án:

```bash
mkdir -p ~/aiq-sensor && cd ~/aiq-sensor
```

3. Tải lên các file chứng chỉ đã tải xuống trong bước 2.

4. Tạo file `sensor.js` với nội dung sau:

```javascript
const awsIot = require('aws-iot-device-sdk');
const fs = require('fs');
const path = require('path');

// Cấu hình kết nối IoT
const device = awsIot.device({
  keyPath: path.join(__dirname, 'private.key'),
  certPath: path.join(__dirname, 'certificate.pem'),
  caPath: path.join(__dirname, 'rootCA.pem'),
  clientId: 'aiq-sensor-' + Math.floor(Math.random() * 100000),
  host: '<IOT_ENDPOINT>' // Thay thế bằng endpoint của bạn
});

// Kết nối đến IoT Core
device.on('connect', function() {
  console.log('Connected to AWS IoT');
  setInterval(publishData, 5000);
});

// Hàm tạo dữ liệu mô phỏng
function generateSensorData() {
  const data = {
    device_id: 'sensor-001',
    timestamp: new Date().toISOString(),
    temperature: 20 + Math.random() * 10,
    humidity: 40 + Math.random() * 20,
    pressure: 1000 + Math.random() * 10,
    air_quality: 50 + Math.random() * 100,
    noise_level: 30 + Math.random() * 20,
    light_level: 100 + Math.random() * 900,
    battery_level: 70 + Math.random() * 30,
    sensor_status: Math.random() > 0.1 ? 'active' : 'warning'
  };
  return data;
}

// Gửi dữ liệu đến AWS IoT Core
function publishData() {
  const data = generateSensorData();
  console.log('Publishing data:', data);
  device.publish('aiq/sensors/env', JSON.stringify(data));
}

// Xử lý lỗi
device.on('error', function(error) {
  console.log('Error:', error);
});
```

5. Thay thế `<IOT_ENDPOINT>` bằng endpoint của AWS IoT Core của bạn. Bạn có thể tìm endpoint này trong AWS IoT Core Console, phần **Settings**.

6. Cài đặt các dependency:

```bash
npm init -y
npm install aws-iot-device-sdk
```

7. Chạy chương trình:

```bash
node sensor.js
```

Thiết bị ảo sẽ bắt đầu gửi dữ liệu về nhiệt độ, độ ẩm, và các thông số môi trường khác đến AWS IoT Core mỗi 5 giây.

### 7. Kiểm tra dữ liệu

Để kiểm tra xem dữ liệu đã được gửi thành công hay chưa:

1. Trong AWS IoT Core Console, chọn **Test** từ menu bên trái.
2. Trong phần **MQTT client**, đăng ký (subscribe) đến topic `aiq/sensors/#`.
3. Bạn sẽ thấy dữ liệu từ thiết bị ảo được hiển thị.

### Tóm tắt

Chúng ta đã hoàn thành việc cấu hình AWS IoT Core để thu thập dữ liệu từ thiết bị cảm biến IoT. Dữ liệu này được định tuyến đến Amazon MSK thông qua AWS IoT Rule. Trong phần tiếp theo, chúng ta sẽ thiết lập Amazon MSK để xử lý dữ liệu này.
