---
title : "Phân tích và Trực quan hóa"
date: 2025-08-10
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

## Lớp Phân tích Dữ liệu

Lớp phân tích dữ liệu cung cấp các công cụ và dịch vụ để truy vấn, phân tích và trực quan hóa dữ liệu từ kiến trúc Lakehouse. Trong workshop này, chúng ta sẽ sử dụng Amazon Athena để truy vấn dữ liệu trực tiếp từ S3, Amazon QuickSight để tạo dashboard, và Amazon SageMaker để xây dựng các mô hình học máy.

![Lớp Phân tích Dữ liệu](/images/6-analytics-layer/analytics-overview.png)

### Phân tích SQL với Amazon Athena

Amazon Athena là một dịch vụ truy vấn tương tác giúp phân tích dữ liệu trong S3 bằng SQL tiêu chuẩn. Với Athena, chúng ta có thể:

- Truy vấn dữ liệu trực tiếp từ S3 mà không cần phải tải hoặc chuyển đổi dữ liệu
- Sử dụng SQL tiêu chuẩn để phân tích dữ liệu
- Tích hợp với AWS Glue Data Catalog để quản lý metadata
- Pay-per-query, chỉ trả tiền cho dữ liệu được quét

### Trực quan hóa với Amazon QuickSight

Amazon QuickSight là một dịch vụ phân tích kinh doanh có khả năng mở rộng, serverless, được nhúng, và được ML hỗ trợ. Với QuickSight, chúng ta có thể:

- Tạo và xuất bản các dashboard tương tác
- Kết nối với nhiều nguồn dữ liệu bao gồm Athena, Redshift, và S3
- Chia sẻ insights với người dùng và nhúng dashboard vào ứng dụng
- Sử dụng ML để tạo ra các insights tự động và dự báo

### Machine Learning với Amazon SageMaker

Amazon SageMaker là một dịch vụ ML đầy đủ, giúp các nhà khoa học dữ liệu và nhà phát triển xây dựng, huấn luyện và triển khai mô hình ML nhanh chóng. Với SageMaker, chúng ta có thể:

- Chuẩn bị dữ liệu và tạo tính năng
- Xây dựng và huấn luyện mô hình ML
- Triển khai và quản lý mô hình trong môi trường production
- Tự động hóa và quản lý toàn bộ vòng đời ML

### Nội dung

Trong phần này, chúng ta sẽ khám phá các khả năng phân tích của kiến trúc Lakehouse:

1. [Xây dựng Phân tích Lớp Gold](6.1-gold-analytics/)
2. [Tạo Lớp Học Máy Platinum](6.2-platinum-ml/)
3. [Trực quan hóa dữ liệu với QuickSight](6.3-visualization/)
