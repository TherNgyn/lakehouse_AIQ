---
title : "Giới thiệu"
date: 2025-08-10
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

## Kiến trúc Lakehouse và chỉ số AIQ

**Kiến trúc Lakehouse** là một mô hình kiến trúc dữ liệu hiện đại kết hợp những ưu điểm của cả **Data Warehouse** và **Data Lake**. Mô hình này mang lại hiệu quả cao trong việc xử lý dữ liệu có cấu trúc và không có cấu trúc, đồng thời hỗ trợ các tính năng như:

- **Quản lý giao dịch ACID**: Đảm bảo tính toàn vẹn dữ liệu
- **Quản lý schema**: Hỗ trợ việc khai báo và thực thi schema
- **Hỗ trợ BI**: Khả năng tương tác trực tiếp với các công cụ Business Intelligence
- **Khả năng xử lý dữ liệu mở**: Có thể lưu trữ và xử lý nhiều loại dữ liệu khác nhau
- **Hỗ trợ các công cụ Data Science**: Tích hợp tốt với các framework machine learning và data science

### Chỉ số AIQ - AI Quality Index

**AIQ (AI Quality Index)** là một chỉ số đánh giá chất lượng và mức độ áp dụng AI trong các quy trình nghiệp vụ. Chỉ số này giúp các doanh nghiệp:

1. Đánh giá mức độ trưởng thành trong việc ứng dụng AI
2. Phân tích hiệu quả của các hệ thống AI đang sử dụng
3. So sánh và đối chiếu với các tiêu chuẩn ngành
4. Xác định các lĩnh vực cần cải thiện

### Tại sao cần Lakehouse cho đánh giá AIQ?

Đánh giá chỉ số AIQ đòi hỏi việc phân tích dữ liệu từ nhiều nguồn khác nhau, bao gồm:
- Dữ liệu cảm biến IoT (time-series)
- Dữ liệu giao dịch (structured)
- Dữ liệu hệ thống (logs)
- Dữ liệu phi cấu trúc (văn bản, hình ảnh)

Kiến trúc Lakehouse cung cấp nền tảng lý tưởng để:
- Thu thập dữ liệu từ nhiều nguồn khác nhau
- Lưu trữ dữ liệu thô một cách kinh tế
- Xử lý và chuyển đổi dữ liệu theo từng giai đoạn (Bronze → Silver → Gold → Platinum)
- Hỗ trợ phân tích dữ liệu chuyên sâu
- Cung cấp dữ liệu cho các công cụ trực quan hóa

### Tổng quan giải pháp

Workshop này sẽ hướng dẫn xây dựng một kiến trúc Lakehouse trên AWS với các thành phần sau:

![Kiến trúc Lakehouse](../images/lakehouse-architecture.png)

Trong workshop này, bạn sẽ:
1. Thiết lập một môi trường thu thập dữ liệu từ cảm biến IoT
2. Xây dựng quy trình thu thập và chuyển đổi dữ liệu
3. Thiết kế lớp lưu trữ dữ liệu theo mô hình Delta Lake
4. Xây dựng các job xử lý dữ liệu
5. Thiết lập môi trường phân tích dữ liệu
6. Xây dựng bảng điều khiển đánh giá chỉ số AIQ

Hãy bắt đầu với việc chuẩn bị môi trường phát triển!
