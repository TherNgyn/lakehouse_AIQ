---
title : "Xử lý và chuyển đổi dữ liệu"
date: 2025-08-10
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

## Lớp xử lý dữ liệu - Data Processing Layer

Lớp xử lý dữ liệu đóng vai trò quan trọng trong việc biến đổi dữ liệu thô thành thông tin có giá trị. Trong workshop này, chúng ta sẽ sử dụng AWS Glue để xây dựng các quy trình ETL (Extract, Transform, Load) để xử lý dữ liệu giữa các tầng trong kiến trúc Lakehouse.

![Data Processing Layer](/images/5-processing-layer/processing-overview.png)

### Quy trình ETL với AWS Glue

AWS Glue là một dịch vụ ETL được quản lý hoàn toàn, giúp chúng ta dễ dàng chuẩn bị và chuyển đổi dữ liệu. Chúng ta sẽ sử dụng AWS Glue để:

1. **Làm sạch dữ liệu (Bronze → Silver)**: 
   - Loại bỏ dữ liệu trùng lặp
   - Xử lý giá trị null và ngoại lệ
   - Chuẩn hóa định dạng dữ liệu

2. **Tổng hợp dữ liệu (Silver → Gold)**:
   - Tính toán các chỉ số tổng hợp
   - Kết hợp dữ liệu từ nhiều nguồn
   - Áp dụng các quy tắc nghiệp vụ

3. **Tối ưu hóa dữ liệu (Gold → Platinum)**:
   - Tạo các bảng sao cho truy vấn hiệu quả
   - Phân vùng dữ liệu để tối ưu hiệu suất
   - Chuẩn bị dữ liệu cho báo cáo và dashboard

### Glue Data Catalog

AWS Glue Data Catalog hoạt động như một kho metadata trung tâm, giúp chúng ta quản lý và khám phá dữ liệu. Catalog này sẽ:

- Lưu trữ định nghĩa schema của tất cả các dataset
- Theo dõi vị trí vật lý của dữ liệu
- Tự động phát hiện schema thông qua Glue Crawler
- Cung cấp khả năng tìm kiếm và phát hiện dữ liệu

### Nội dung

Trong phần này, chúng ta sẽ thiết lập các thành phần xử lý dữ liệu:

1. [Cấu hình AWS Glue Data Catalog](5.1-glue-catalog/)
2. [Xây dựng Glue Crawler](5.2-glue-crawler/)
3. [Tạo Glue Job làm sạch dữ liệu](5.3-etl-bronze-silver/)
4. [Tạo Glue Job tổng hợp dữ liệu](5.4-etl-silver-gold/)
5. [Thiết lập Airflow để điều phối luồng dữ liệu](5.5-airflow-orchestration/)
