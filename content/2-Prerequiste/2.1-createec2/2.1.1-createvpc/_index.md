---
title : "Creating VPC"
date: 2025-08-10
weight : 1
chapter : false
pre : " <b> 2.1.1 </b> "
---

## Creating VPC for Lakehouse Architecture

In this section, we will create a new VPC to deploy the Lakehouse architecture. This VPC will provide a separate and secure network environment for the different components of the system.

### Step 1: Create a VPC

1. Log in to the [AWS Management Console](https://console.aws.amazon.com/) and access the [VPC service](https://console.aws.amazon.com/vpc/home).
   + Select **Your VPC** from the left menu.
   + Click the **Create VPC** button.

![Tạo VPC](/images/2.prerequisite/001-createvpc.png)

2. Tại trang **Create VPC**:
   + Trong trường **Name tag**, nhập **Lakehouse-VPC**.
   + Trong trường **IPv4 CIDR**, nhập: **10.0.0.0/16**.
   + Nhấn **Create VPC**.

### Bước 2: Tạo Internet Gateway

Để VPC có thể kết nối với Internet, chúng ta cần tạo và gắn một Internet Gateway:

1. Chọn **Internet Gateways** từ menu bên trái và nhấn **Create internet gateway**.
2. Nhập tên cho Internet Gateway: **Lakehouse-IGW**.
3. Nhấn **Create internet gateway**.

![Tạo Internet Gateway](/images/2.prerequisite/002-createvpc.png)

4. Sau khi tạo xong, chọn Internet Gateway vừa tạo, nhấn nút **Actions** và chọn **Attach to VPC**.
5. Chọn VPC vừa tạo (Lakehouse-VPC) và nhấn **Attach internet gateway**.

### Bước 3: Tạo NAT Gateway

NAT Gateway cho phép các instance trong private subnet kết nối với internet hoặc các dịch vụ khác bên ngoài VPC:

1. Chọn **NAT Gateways** từ menu bên trái và nhấn **Create NAT gateway**.
2. Cấu hình NAT Gateway:
   - **Name**: Lakehouse-NAT
   - **Subnet**: Sau này sẽ chọn một public subnet (sẽ tạo trong bước tiếp theo)
   - **Connectivity type**: Public
   - **Elastic IP allocation ID**: Nhấn **Allocate Elastic IP** để tạo địa chỉ IP mới

### Kiến trúc VPC cho Lakehouse

Trong kiến trúc Lakehouse của chúng ta, VPC đóng vai trò quan trọng trong việc cung cấp một môi trường mạng an toàn và hiệu quả:

1. **Public Subnet**: Chứa các thành phần như:
   - API Gateway để tiếp nhận dữ liệu từ bên ngoài
   - IoT Core để kết nối với các thiết bị IoT

2. **Private Subnet**: Chứa các thành phần xử lý và lưu trữ dữ liệu như:
   - Amazon MSK để làm message broker
   - AWS Glue để xử lý ETL
   - Amazon DocumentDB để lưu trữ dữ liệu có cấu trúc
   - Các endpoint để truy cập đến S3 và các dịch vụ khác

3. **Security Group**: Kiểm soát lưu lượng mạng giữa các thành phần trong VPC

![Kiến trúc VPC](/images/2.prerequisite/vpc-architecture.png)

Trong phần tiếp theo, chúng ta sẽ tạo các public subnet và private subnet cho VPC này.

![VPC](/images/2.prerequisite/002-createvpc.png)