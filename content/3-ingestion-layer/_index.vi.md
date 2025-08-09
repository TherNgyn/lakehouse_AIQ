---
title : "Xây dựng Lớp Thu thập Dữ liệu"
date: 2025-08-10
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

## Lớp Thu thập Dữ liệu

Lớp thu thập dữ liệu là thành phần đầu tiên và quan trọng trong kiến trúc Lakehouse. Lớp này chịu trách nhiệm thu thập dữ liệu từ nhiều nguồn khác nhau, đảm bảo dữ liệu được truyền đáng tin cậy vào hệ thống.

Trong workshop này, chúng ta sẽ thiết lập các kênh thu thập dữ liệu sau:

1. **AWS IoT Core**: Thu thập dữ liệu từ các thiết bị cảm biến IoT
2. **API Gateway**: Thu thập dữ liệu từ các API bên ngoài
3. **DataSync**: Nhập dữ liệu từ các file CSV

![Lớp Thu thập Dữ liệu](/images/3-ingestion-layer/ingestion-overview.png)

### Mô hình Xử lý Dữ liệu

Chúng ta sẽ sử dụng mô hình kết hợp của xử lý dữ liệu theo thời gian thực và theo lô:

1. **Xử lý Thời gian thực**: 
   - Dữ liệu từ IoT Core và API Gateway sẽ được gửi đến Amazon MSK
   - AWS Lambda sẽ xử lý dữ liệu từ MSK và lưu trữ nó trong Lớp Bronze S3

2. **Xử lý Theo lô**:
   - Dữ liệu từ DataSync sẽ được đồng bộ trực tiếp vào Lớp Bronze S3
   - AWS Glue sẽ thực hiện các công việc ETL định kỳ để xử lý dữ liệu

### Nội dung

Trong phần này, chúng ta sẽ thiết lập các thành phần của lớp thu thập dữ liệu:

1. [Cấu hình AWS IoT Core](3.1-iot-core/)
2. [Thiết lập Amazon MSK](3.2-amazon-msk/)
3. [Xây dựng API Gateway](3.3-api-gateway/)
4. [Cấu hình AWS DataSync](3.4-datasync/)
