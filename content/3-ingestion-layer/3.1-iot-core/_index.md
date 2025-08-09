---
title : "Configuring AWS IoT Core"
date : 2025-08-10
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

## Configuring AWS IoT Core

AWS IoT Core is a managed service that helps connect and manage billions of IoT devices. In this section, we will set up AWS IoT Core to collect data from sensors, then route this data to Amazon MSK via AWS IoT Rules.

### 1. Create a Thing in AWS IoT Core

First, we'll create a "thing" that represents an IoT device in AWS IoT Core:

1. Log in to the [AWS Management Console](https://console.aws.amazon.com/) and access the [AWS IoT Core service](https://console.aws.amazon.com/iot/).
2. In the left menu, select **Manage** and then select **Things**.
3. Click **Create things**.
4. Choose **Create a single thing** and click **Next**.
5. Enter information for the thing:
   - **Thing name**: aiq-sensor
   - Keep the rest of the options at their default values
6. Click **Next**.

### 2. Create and Download Certificates

For an IoT device to connect securely to AWS IoT Core, we need to create certificates:

1. On the **Add a certificate for your thing** screen, select **Auto-generate a new certificate**.
2. Click **Next**.
3. On the next page, download all files:
   - Certificate for this thing
   - Public key
   - Private key
   - Root CA
4. Click **Activate** to activate the certificate.
5. Click **Attach a policy**.

### 3. Create and Attach Policy

Next, we'll create a policy to grant permissions to the device:

1. Click **Create policy**.
2. Enter the policy information:
   - **Name**: aiq-sensor-policy
   - **Action**: iot:Connect, iot:Publish, iot:Subscribe, iot:Receive
   - **Resource ARN**: *
   - **Effect**: Allow
3. Click **Create**.
4. Go back to the policy attachment page, select the policy you just created and click **Register Thing**.

### 4. Create IAM Role for AWS IoT Rule

We need to create an IAM Role so that the AWS IoT Rule can send data to Amazon MSK:

1. Access the [AWS IAM Console](https://console.aws.amazon.com/iam/).
2. Select **Roles** and click **Create role**.
3. Choose **IoT** as the service and click **Next**.
4. Find and select the following policies:
   - AmazonMSKFullAccess
   - AWSIoTFullAccess
   - AWSIoTRuleActions
5. Click **Next**.
6. Name the role: **IoTtoMSKRole** and click **Create role**.

### 5. Create AWS IoT Rule

Now we will create an IoT Rule to route data from IoT Core to Amazon MSK:

1. Return to the [AWS IoT Core Console](https://console.aws.amazon.com/iot/).
2. In the left menu, select **Message routing** and then select **Rules**.
3. Click **Create rule**.
4. Enter the rule information:
   - **Name**: aiq_sensor_to_msk
   - **Description**: Route sensor data to MSK
5. In the **Rule query statement** section, enter the following SQL statement:
   ```sql
   SELECT *, timestamp() as timestamp FROM 'aiq/sensors/#'
   ```
6. Click **Add action**.
7. Select **Send a message to an Apache Kafka broker** and click **Configure action**.
8. Enter the connection information to MSK:
   - **Destination**: Select VPC destination
   - **VPC**: Select Lakehouse-VPC
   - **Subnet**: Select Private Subnet 1
   - **Security group**: Select MSK-SG
   - **Apache Kafka topic**: aiq-sensor-data
   - **Bootstrap servers**: Enter the bootstrap server address of the MSK cluster (format: broker1:port,broker2:port,broker3:port)
   - **Security protocol**: SASL_SSL
   - **SASL mechanism**: SCRAM-SHA-512
   - **Username**: ${get_secret('<secret_name>', 'SecretString', 'username', '<role_arn>')}
   - **Password**: ${get_secret('<secret_name>', 'SecretString', 'password', '<role_arn>')}
   - Replace `<secret_name>` with the name of your MSK secret in AWS Secrets Manager
   - Replace `<role_arn>` with the ARN of the IoTtoMSKRole
9. Select **IoTtoMSKRole** in the **Role** section.
10. Click **Add action** and then click **Create rule**.

### 6. Create Virtual IoT Device Code

Now, we will create a virtual device running on EC2 to simulate an IoT sensor:

1. Connect to the EC2 instance created in the previous section.
2. Create a directory for the project:

```bash
mkdir -p ~/aiq-sensor && cd ~/aiq-sensor
```

3. Upload the certificate files downloaded in step 2.

4. Create a file `sensor.js` with the following content:

```javascript
const awsIot = require('aws-iot-device-sdk');
const fs = require('fs');
const path = require('path');

// IoT connection configuration
const device = awsIot.device({
  keyPath: path.join(__dirname, 'private.key'),
  certPath: path.join(__dirname, 'certificate.pem'),
  caPath: path.join(__dirname, 'rootCA.pem'),
  clientId: 'aiq-sensor-' + Math.floor(Math.random() * 100000),
  host: '<IOT_ENDPOINT>' // Replace with your endpoint
});

// Connect to IoT Core
device.on('connect', function() {
  console.log('Connected to AWS IoT');
  setInterval(publishData, 5000);
});

// Function to generate simulated data
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

// Send data to AWS IoT Core
function publishData() {
  const data = generateSensorData();
  console.log('Publishing data:', data);
  device.publish('aiq/sensors/env', JSON.stringify(data));
}

// Error handling
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
