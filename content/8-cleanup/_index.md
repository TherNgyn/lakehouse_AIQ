---
title : "Dọn dẹp tài nguyên"
date: 2025-08-10
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

## Dọn dẹp tài nguyên

Sau khi hoàn thành workshop, bạn nên dọn dẹp các tài nguyên AWS đã tạo để tránh phát sinh chi phí không mong muốn. Phần này sẽ hướng dẫn bạn cách xóa tất cả các tài nguyên đã tạo trong workshop này.

### 1. Xóa Amazon QuickSight Dashboard và Dataset

1. Truy cập [Amazon QuickSight Console](https://quicksight.aws.amazon.com/).
2. Xóa tất cả dashboard và dataset đã tạo:
   - Chọn **Dashboards** từ menu bên trái, chọn dashboard AIQ, nhấn **Delete**.
   - Chọn **Datasets**, chọn các dataset đã tạo, nhấn **Delete**.

### 2. Xóa Amazon SageMaker Notebook và Endpoint

1. Truy cập [Amazon SageMaker Console](https://console.aws.amazon.com/sagemaker/).
2. Xóa các notebook instance:
   - Chọn **Notebook instances** từ menu bên trái.
   - Chọn notebook instance đã tạo và nhấn **Actions** > **Stop**.
   - Sau khi dừng, chọn **Actions** > **Delete**.

3. Xóa các endpoint:
   - Chọn **Endpoints** từ menu bên trái.
   - Chọn endpoint đã tạo và nhấn **Actions** > **Delete**.

### 3. Xóa AWS Glue Jobs, Crawlers, và Database

1. Truy cập [AWS Glue Console](https://console.aws.amazon.com/glue/).
2. Xóa các Glue job:
   - Chọn **Jobs** từ menu bên trái.
   - Chọn các job đã tạo và nhấn **Action** > **Delete**.

3. Xóa các Crawler:
   - Chọn **Crawlers** từ menu bên trái.
   - Chọn các crawler đã tạo và nhấn **Delete**.

4. Xóa các table và database:
   - Chọn **Databases** từ menu bên trái.
   - Chọn database đã tạo, nhấn **Actions** > **Delete database**.
   - Đảm bảo chọn tùy chọn **Delete all tables in the database**.

### 4. Xóa Amazon Redshift Cluster

1. Truy cập [Amazon Redshift Console](https://console.aws.amazon.com/redshift/).
2. Chọn cluster đã tạo.
3. Nhấn **Actions** > **Delete**.
4. Tắt tùy chọn **Create final snapshot** và nhập tên cluster để xác nhận.
5. Nhấn **Delete cluster**.

### 5. Xóa Amazon MSK Cluster

1. Truy cập [Amazon MSK Console](https://console.aws.amazon.com/msk/).
2. Chọn cluster đã tạo.
3. Nhấn **Actions** > **Delete cluster**.
4. Nhập tên cluster để xác nhận và nhấn **Delete**.

### 6. Xóa AWS IoT Core Resources

1. Truy cập [AWS IoT Core Console](https://console.aws.amazon.com/iot/).
2. Xóa rule:
   - Chọn **Message routing** > **Rules**.
   - Chọn rule đã tạo và nhấn **Delete**.

3. Xóa thing:
   - Chọn **Manage** > **Things**.
   - Chọn thing đã tạo và nhấn **Delete**.

4. Xóa certificate:
   - Chọn **Manage** > **Certificates**.
   - Chọn certificate đã tạo, nhấn **Actions** > **Delete**.

5. Xóa policy:
   - Chọn **Manage** > **Policies**.
   - Chọn policy đã tạo và nhấn **Delete**.

### 7. Xóa S3 Bucket

1. Truy cập [Amazon S3 Console](https://console.aws.amazon.com/s3/).
2. Chọn bucket đã tạo trong workshop.
3. Nhấn **Empty** để xóa tất cả đối tượng trong bucket.
4. Sau khi bucket trống, nhấn **Delete**.
5. Nhập tên bucket để xác nhận và nhấn **Delete bucket**.

### 8. Xóa EC2 Instance

1. Truy cập [Amazon EC2 Console](https://console.aws.amazon.com/ec2/).
2. Chọn instance đã tạo.
3. Nhấn **Actions** > **Instance State** > **Terminate**.
4. Xác nhận bằng cách nhấn **Terminate**.

### 9. Xóa IAM Roles và Policies

1. Truy cập [AWS IAM Console](https://console.aws.amazon.com/iam/).
2. Xóa roles:
   - Chọn **Roles** từ menu bên trái.
   - Tìm và chọn các role đã tạo (IoTCoreRole, IoTtoMSKRole, v.v.).
   - Nhấn **Delete** và xác nhận.

3. Xóa policies:
   - Chọn **Policies** từ menu bên trái.
   - Tìm và chọn các policy tùy chỉnh đã tạo.
   - Nhấn **Actions** > **Delete**.
   - Nhập tên policy để xác nhận và nhấn **Delete**.

### 10. Xóa VPC và các thành phần liên quan

1. Truy cập [Amazon VPC Console](https://console.aws.amazon.com/vpc/).
2. Xóa NAT Gateway:
   - Chọn **NAT Gateways** từ menu bên trái.
   - Chọn NAT Gateway đã tạo và nhấn **Actions** > **Delete NAT gateway**.
   - Xác nhận bằng cách nhấn **Delete**.

3. Xóa VPC Endpoint:
   - Chọn **Endpoints** từ menu bên trái.
   - Chọn endpoint đã tạo và nhấn **Actions** > **Delete VPC endpoints**.
   - Xác nhận bằng cách nhấn **Delete**.

4. Xóa Internet Gateway:
   - Chọn **Internet Gateways** từ menu bên trái.
   - Chọn IGW đã tạo và nhấn **Actions** > **Detach from VPC**.
   - Sau khi đã tách, nhấn **Actions** > **Delete internet gateway**.
   - Xác nhận bằng cách nhấn **Delete**.

5. Xóa Subnet:
   - Chọn **Subnets** từ menu bên trái.
   - Chọn các subnet đã tạo và nhấn **Actions** > **Delete subnet**.
   - Xác nhận bằng cách nhấn **Delete**.

6. Xóa Route Table:
   - Chọn **Route Tables** từ menu bên trái.
   - Chọn route table tùy chỉnh đã tạo và nhấn **Actions** > **Delete route table**.
   - Xác nhận bằng cách nhấn **Delete**.

7. Xóa Security Group:
   - Chọn **Security Groups** từ menu bên trái.
   - Chọn security group đã tạo (không phải default) và nhấn **Actions** > **Delete security group**.
   - Xác nhận bằng cách nhấn **Delete**.

8. Xóa VPC:
   - Chọn **Your VPCs** từ menu bên trái.
   - Chọn VPC đã tạo và nhấn **Actions** > **Delete VPC**.
   - Xác nhận bằng cách nhấn **Delete**.

### Xác nhận dọn dẹp

Sau khi hoàn thành các bước trên, kiểm tra lại các dịch vụ để đảm bảo rằng tất cả tài nguyên đã được xóa thành công. Điều này sẽ giúp tránh phát sinh chi phí không mong muốn trong tương lai.

Chúc mừng bạn đã hoàn thành workshop xây dựng kiến trúc Lakehouse cho bài toán đánh giá chỉ số AIQ! Hy vọng rằng workshop này đã cung cấp cho bạn những kiến thức và kỹ năng cần thiết để xây dựng các giải pháp dữ liệu hiện đại trên AWS.
