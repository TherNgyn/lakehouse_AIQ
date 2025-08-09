---
title : "Xây dựng Lớp Lưu trữ Dữ liệu"
date: 2025-08-10
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

## Lớp Lưu trữ Dữ liệu

Lớp lưu trữ dữ liệu là trung tâm của kiến trúc Lakehouse, nơi dữ liệu được tổ chức theo các tầng khác nhau từ dữ liệu thô đến dữ liệu đã được tinh chế. Trong workshop này, chúng ta sẽ sử dụng Amazon S3 làm nền tảng lưu trữ kết hợp với Delta Lake để triển khai các tầng dữ liệu.

![Lớp Lưu trữ Dữ liệu](/images/4-storage-layer/storage-overview.png)

### Mô hình Lưu trữ Đa tầng

Kiến trúc Lakehouse sử dụng mô hình lưu trữ đa tầng để tổ chức dữ liệu theo mức độ tinh chế:

1. **Lớp Bronze (Dữ liệu Thô)**:
   - Lưu trữ dữ liệu thô từ các nguồn
   - Ít hoặc không có chuyển đổi
   - Bảo toàn tính toàn vẹn của dữ liệu gốc

2. **Lớp Silver (Dữ liệu Đã Làm sạch)**:
   - Dữ liệu đã được làm sạch và chuẩn hóa
   - Áp dụng các quy tắc chất lượng dữ liệu
   - Loại bỏ các bản ghi trùng lặp và không hợp lệ

3. **Lớp Gold (Dữ liệu Đã Chuyển đổi)**:
   - Dữ liệu đã được tổng hợp và chuyển đổi
   - Phù hợp cho các trường hợp sử dụng cụ thể
   - Sẵn sàng cho phân tích và học máy

4. **Lớp Platinum (Dữ liệu Sẵn sàng cho Kinh doanh)**:
   - Dữ liệu phục vụ cho các báo cáo và bảng điều khiển
   - Tối ưu hóa cho truy vấn hiệu suất cao
   - Triển khai trong Amazon Redshift

### Delta Lake trên Amazon S3

Chúng ta sẽ sử dụng Delta Lake, một framework dữ liệu mã nguồn mở, để cung cấp các tính năng nâng cao cho dữ liệu được lưu trữ trong S3:

- **Giao dịch ACID**: Đảm bảo tính nhất quán của dữ liệu
- **Thực thi Lược đồ**: Tự động kiểm tra và áp dụng lược đồ
- **Du hành Thời gian**: Truy cập các phiên bản trước của dữ liệu
- **Lịch sử Kiểm tra**: Theo dõi thay đổi dữ liệu theo thời gian
- **Thao tác Cập nhật/Xóa/Hợp nhất**: Hỗ trợ các thao tác sửa đổi dữ liệu

### Nội dung

Trong phần này, chúng ta sẽ thiết lập lớp lưu trữ dữ liệu:

1. [Tạo và Cấu hình Bucket Amazon S3](4.1-s3-bucket/)
2. [Thiết lập Delta Lake](4.2-delta-lake/)
3. [Xây dựng Lớp Bronze](4.3-bronze-layer/)
4. [Xây dựng Lớp Silver](4.4-silver-layer/)
5. [Xây dựng Lớp Gold](4.5-gold-layer/)
6. [Cấu hình Amazon Redshift cho Lớp Platinum](4.6-platinum-layer/)
