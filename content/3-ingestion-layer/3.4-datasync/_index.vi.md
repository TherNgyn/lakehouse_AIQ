---
title : "Thiết lập AWS DataSync"
date : 2025-08-10
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---

## Thiết lập AWS DataSync

Trong phần này, chúng ta sẽ cấu hình AWS DataSync để chuyển dữ liệu hiện có từ hệ thống lưu trữ tại chỗ hoặc các hệ thống lưu trữ AWS khác vào kiến trúc Lakehouse của chúng ta. DataSync cung cấp một cách liền mạch để di chuyển dữ liệu lịch sử hoặc đồng bộ hóa dữ liệu thường xuyên từ nhiều nguồn khác nhau vào lớp Bronze dựa trên S3 của chúng ta.

### 1. Tìm hiểu về AWS DataSync

AWS DataSync là dịch vụ truyền dữ liệu giúp đơn giản hóa, tự động hóa và tăng tốc việc di chuyển dữ liệu giữa các hệ thống lưu trữ tại chỗ và dịch vụ lưu trữ AWS, hoặc giữa các dịch vụ lưu trữ AWS với nhau. Đối với kiến trúc Lakehouse của chúng ta, DataSync giúp chúng ta:

- Di chuyển dữ liệu lịch sử vào Lakehouse
- Thiết lập truyền dữ liệu định kỳ từ các hệ thống cũ
- Tiếp nhận dữ liệu từ máy chủ tệp, hệ thống NAS hoặc các nền tảng lưu trữ đối tượng khác
- Đảm bảo tính nhất quán của dữ liệu trên các vị trí lưu trữ khác nhau

### 2. Tạo DataSync Agent (cho Nguồn Tại Chỗ)

Nếu bạn cần chuyển dữ liệu từ hệ thống tại chỗ, bạn sẽ cần triển khai một DataSync agent:

1. Điều hướng đến [AWS DataSync console](https://console.aws.amazon.com/datasync/).
2. Nhấn **Create agent**.
3. Chọn loại triển khai phù hợp với môi trường của bạn:
   - **Amazon EC2**: Nếu bạn muốn triển khai agent trong AWS
   - **VMware**: Cho môi trường VMware
   - **KVM**: Cho hypervisor KVM
   - **Hyper-V**: Cho Microsoft Hyper-V
   - **Hardware appliance**: Cho AWS Snowcone
4. Làm theo hướng dẫn để triển khai agent trong môi trường của bạn.
5. Sau khi triển khai, agent sẽ tạo ra một khóa kích hoạt. Sao chép khóa này.
6. Trong bảng điều khiển DataSync, nhập khóa kích hoạt và cung cấp tên cho agent của bạn.
7. Nhấn **Create agent**.

> Lưu ý: Nếu bạn chỉ chuyển dữ liệu giữa các dịch vụ AWS, bạn không cần tạo DataSync agent.

### 3. Tạo DataSync Task

Bây giờ, hãy tạo một DataSync task để chuyển dữ liệu:

1. Trong bảng điều khiển DataSync, chọn **Tasks** từ thanh điều hướng bên trái.
2. Nhấn **Create task**.

#### 3.1. Cấu hình Vị trí Nguồn

3. Trên trang "Configure source location":
   - Nếu chuyển từ hệ thống tại chỗ:
     - Chọn **Create a new location**
     - Loại vị trí: Chọn loại thích hợp (NFS, SMB, HDFS, v.v.)
     - Agent: Chọn agent bạn đã tạo
     - Nhập thông tin máy chủ và đường dẫn cho dữ liệu nguồn của bạn
   - Nếu chuyển từ dịch vụ AWS khác:
     - Chọn **Create a new location**
     - Loại vị trí: Chọn dịch vụ AWS thích hợp (S3, EFS, FSx, v.v.)
     - Cấu hình các cài đặt cụ thể cho dịch vụ đó

4. Nhấn **Next**.

#### 3.2. Cấu hình Vị trí Đích

5. Trên trang "Configure destination location":
   - Chọn **Create a new location**
   - Loại vị trí: **Amazon S3 bucket**
   - S3 bucket: Chọn bucket lớp Bronze của bạn
   - S3 storage class: **Standard**
   - Folder: Chỉ định một tiền tố như `historical-data/`
   - IAM role: Tạo một vai trò mới hoặc chọn một vai trò hiện có với các quyền cần thiết

6. Nhấn **Next**.

#### 3.3. Cấu hình Cài đặt Task

7. Trên trang "Configure settings":
   - Tên task: `historical-data-sync`
   - Xác minh dữ liệu: Chọn một tùy chọn dựa trên nhu cầu của bạn (ChecksumOnly, TaskLevel, hoặc None)
   - Tùy chọn chuyển dữ liệu: Cấu hình dựa trên nhu cầu của bạn
   - Ghi nhật ký task: Bật ghi nhật ký CloudWatch
   - Lịch trình: Chọn chạy một lần hoặc theo lịch trình

8. Nhấn **Next**.

9. Xem lại cấu hình của bạn và nhấn **Create task**.

### 4. Chạy và Giám sát DataSync Task

Sau khi bạn đã tạo task:

1. Chọn task từ danh sách tasks.
2. Nhấn **Start**.
3. Xem lại cấu hình task một lần nữa và nhấn **Start**.
4. Bạn có thể theo dõi tiến trình của task trên trang chi tiết task.
5. Sau khi task hoàn thành, bạn có thể kiểm tra nhật ký CloudWatch để biết thông tin chi tiết về việc chuyển dữ liệu.

### 5. Thiết lập Chuyển Dữ liệu Định kỳ

Đối với dữ liệu cần được đồng bộ hóa thường xuyên:

1. Chỉnh sửa task hiện có của bạn hoặc tạo một task mới.
2. Trong phần "Configure settings", thiết lập lịch trình:
   - Chọn **Scheduled**
   - Chọn tần suất (hàng giờ, hàng ngày, hàng tuần, v.v.)
   - Đặt các ngày và giờ cụ thể
   - Cấu hình ngày bắt đầu

3. Lưu các thay đổi của bạn.

### 6. Tích hợp Dữ liệu Đã Chuyển với Glue Catalog

Sau khi dữ liệu của bạn được chuyển đến S3, bạn sẽ cần tích hợp nó với AWS Glue Catalog để làm cho nó có thể truy cập bởi các công cụ xử lý dữ liệu của bạn:

1. Điều hướng đến [AWS Glue console](https://console.aws.amazon.com/glue/).
2. Chọn **Crawlers** từ thanh điều hướng bên trái.
3. Nhấn **Add crawler**.
4. Làm theo hướng dẫn để cấu hình crawler của bạn:
   - Tên: `historical-data-crawler`
   - Chọn loại kho dữ liệu: **S3**
   - Đường dẫn bao gồm: Chỉ định đường dẫn S3 nơi task DataSync của bạn chuyển dữ liệu
   - IAM role: Tạo một vai trò mới hoặc sử dụng một vai trò hiện có với quyền Glue
   - Lịch trình: Cấu hình tần suất crawler nên chạy
   - Cơ sở dữ liệu đầu ra: Chọn hoặc tạo một cơ sở dữ liệu trong Glue Catalog
   - Cấu hình các tùy chọn khác khi cần

5. Chạy crawler để khám phá lược đồ của dữ liệu đã chuyển.
6. Sau khi crawler hoàn thành, các bảng sẽ có sẵn trong Glue Catalog, cho phép bạn truy vấn dữ liệu bằng Athena, Redshift Spectrum hoặc các dịch vụ khác.

### 7. Ví dụ: Chuyển Dữ liệu Thời tiết Lịch sử

Hãy đi qua một ví dụ về việc chuyển dữ liệu thời tiết lịch sử từ một bucket S3 hiện có vào Lakehouse của chúng ta:

1. Tạo một DataSync task:
   - Nguồn: Bucket S3 chứa dữ liệu thời tiết lịch sử
   - Đích: Bucket S3 lớp Bronze của bạn với tiền tố `bronze/historical/weather/`
   - Lịch trình: Chuyển một lần cho ví dụ này

2. Chạy task để chuyển dữ liệu.

3. Tạo một Glue crawler để phân loại dữ liệu:
   - Kho dữ liệu: Đường dẫn S3 đến dữ liệu đã chuyển
   - Cơ sở dữ liệu: `bronze_db`
   - Tiền tố bảng: `historical_weather_`

4. Chạy crawler để tạo bảng trong Glue Catalog.

5. Bây giờ bạn có thể truy vấn dữ liệu này bằng Athena:

```sql
SELECT * 
FROM bronze_db.historical_weather_data
WHERE year = 2024
LIMIT 10;
```

### Tóm tắt

Trong phần này, chúng ta đã thiết lập AWS DataSync để chuyển dữ liệu hiện có vào kiến trúc Lakehouse của chúng ta. DataSync cung cấp một cách đáng tin cậy và hiệu quả để di chuyển dữ liệu lịch sử và thiết lập chuyển dữ liệu định kỳ từ nhiều nguồn khác nhau.

Với DataSync, chúng ta có thể đảm bảo Lakehouse của chúng ta chứa tất cả dữ liệu liên quan, cho dù nó đến từ luồng trực tiếp (thông qua IoT Core và API Gateway) hay từ các hệ thống hiện có. Trong phần tiếp theo, chúng ta sẽ khám phá cách thiết lập Lớp Lưu trữ của Lakehouse bằng Amazon S3 và Delta Lake.
