---
title : "Trực quan hóa dữ liệu với QuickSight"
date : 2025-08-10
weight : 3
chapter : false
pre : " <b> 6.3 </b> "
---

## Trực quan hóa dữ liệu chất lượng không khí với Amazon QuickSight

Trong phần này, chúng ta sẽ tạo các bảng điều khiển tương tác để trực quan hóa dữ liệu chất lượng không khí từ các lớp Gold và Platinum. Chúng ta sẽ xây dựng các biểu đồ tương tự như dưới đây để giúp xác định các mẫu, mối tương quan và hiểu biết sâu sắc từ dữ liệu của chúng ta.

### Tổng quan thiết kế bảng điều khiển

Chiến lược trực quan hóa của chúng ta sẽ bao gồm ba bảng điều khiển chính:

1. **Bảng điều khiển tổng quan chất lượng không khí**
   - Thống kê tóm tắt về các chất ô nhiễm không khí
   - Phân bố các mức AQI
   - Xu hướng hàng tháng của tất cả các chỉ số môi trường

2. **Bảng điều khiển phân tích PM2.5**
   - Phân tích theo thời gian của PM2.5, độ ẩm và nhiệt độ
   - Phân bố mức AQI theo tháng
   - Chuỗi thời gian của các chỉ số chất lượng không khí chính

3. **Bảng điều khiển phân tích tương quan và xu hướng**
   - Nồng độ chất ô nhiễm trung bình hàng tháng
   - Tương quan giữa nhiệt độ và PM2.5
   - PM2.5 trung bình theo tháng và mức AQI

### Tạo bảng điều khiển QuickSight

#### Điều kiện tiên quyết

Trước khi bắt đầu, hãy đảm bảo bạn có:
- Quyền truy cập vào Amazon QuickSight với các quyền thích hợp
- Dữ liệu lớp Gold của bạn được xuất ra vị trí mà QuickSight có thể truy cập (bucket S3, Athena, v.v.)
- Hiểu biết cơ bản về giao diện QuickSight và các loại trực quan hóa

#### Bước 1: Thiết lập nguồn dữ liệu

1. Đăng nhập vào tài khoản AWS và điều hướng đến dịch vụ QuickSight
2. Nhấp vào **Datasets** (Tập dữ liệu) trong bảng điều hướng bên trái
3. Nhấp **New dataset** (Tập dữ liệu mới) và chọn nguồn dữ liệu của bạn:
   - Đối với truy cập S3 trực tiếp: Chọn **S3** và chỉ định đường dẫn đến các tập tin parquet của bạn
   - Đối với Athena: Chọn **Athena** và chọn cơ sở dữ liệu và bảng thích hợp
   - Đối với kết nối trực tiếp đến Delta Lake: Sử dụng trình kết nối Delta Lake (nếu có)

4. Cấu hình tập dữ liệu của bạn:
   ```
   - Tên nguồn dữ liệu: AirQualityAnalytics
   - Tùy chọn nhập: SPICE (cho phân tích nhanh hơn)
   - Bảng: Chọn các bảng gold_air_quality_daily_avg, gold_air_quality_hourly_avg, 
     và gold_air_quality_monthly_avg
   ```

5. Nhấp **Edit/Preview data** (Chỉnh sửa/Xem trước dữ liệu) để xác minh và chuẩn bị tập dữ liệu của bạn

6. Đặt kiểu dữ liệu thích hợp cho mỗi cột:
   - Cột ngày dưới dạng Date
   - Cột đo lường dưới dạng Decimal
   - AQI_Level dưới dạng String

7. Nhấp **Save & publish** (Lưu và xuất bản)

#### Bước 2: Tạo bảng điều khiển tổng quan chất lượng không khí

1. Từ Datasets, chọn tập dữ liệu AirQualityAnalytics của bạn và nhấp **Create analysis** (Tạo phân tích)

2. Thêm trực quan hóa **Table** (Bảng) cho thống kê tóm tắt:
   - Trường: Chọn tất cả các trường đo lường chất lượng không khí (CO, NO2, PM25, SO2, O3, Humidity)
   - Thêm trường tính toán cho count, max, mean, min và stddev
   - Cấu hình cài đặt bảng để hiển thị thống kê trong hàng
   - Sử dụng định dạng có điều kiện để làm nổi bật các giá trị trên ngưỡng

3. Thêm **Pie chart** (Biểu đồ tròn) cho phân bố mức AQI:
   - Trường: AQI_Level
   - Giá trị: Count (records)
   - Màu sắc: Cấu hình màu tùy chỉnh cho mỗi mức AQI (Good: xanh lam, Moderate: cam, v.v.)

4. Thêm **Line chart** (Biểu đồ đường) cho các chỉ số môi trường hàng tháng:
   - Trục X: month
   - Trục Y: Tổng giá trị trung bình cho mỗi chất ô nhiễm (CO, NO2, PM25, SO2, O3, Temperature)
   - Sử dụng các màu khác nhau cho mỗi đường để phân biệt giữa các chất ô nhiễm
   - Thêm tiêu đề: "Xu hướng chỉ số môi trường hàng tháng"

5. Định dạng và sắp xếp các trực quan hóa trên bảng điều khiển của bạn:
   - Thay đổi kích thước các phần tử để hiển thị phù hợp
   - Thêm tiêu đề mô tả và chú thích
   - Sử dụng các điều khiển bộ lọc để cho phép lọc tương tác theo phạm vi ngày

#### Bước 3: Tạo bảng điều khiển phân tích PM2.5

1. Tạo phân tích mới từ tập dữ liệu của bạn

2. Thêm **Line chart** (Biểu đồ đường) cho PM2.5, độ ẩm và nhiệt độ theo thời gian:
   - Trục X: date
   - Trục Y: avg_PM25, avg_humidity, avg_temp
   - Sử dụng các màu khác nhau cho mỗi đường
   - Thêm tiêu đề: "Các chỉ số chất lượng không khí theo thời gian"

3. Thêm **Stacked bar chart** (Biểu đồ cột chồng) cho phân bố AQI hàng tháng:
   - Trục X: month
   - Trục Y: Count of records
   - Nhóm/Màu sắc: AQI_Level
   - Hiển thị dưới dạng biểu đồ cột chồng 100%
   - Tiêu đề: "Phân bố mức AQI theo tháng"

4. Thêm trực quan **KPI** để hiển thị so sánh kỳ hiện tại với kỳ trước:
   - Giá trị chính: avg_PM25 kỳ hiện tại
   - Giá trị so sánh: avg_PM25 kỳ trước
   - Định dạng để hiển thị phần trăm thay đổi
   - Thêm định dạng có điều kiện (xanh lá cho cải thiện, đỏ cho suy giảm)

5. Định dạng và sắp xếp các trực quan hóa:
   - Thêm tham số bảng điều khiển để chọn phạm vi ngày
   - Cấu hình đồng bộ hóa giữa các trực quan hóa
   - Thêm hiểu biết sâu sắc mô tả dưới dạng hộp văn bản

#### Bước 4: Tạo bảng điều khiển phân tích tương quan và xu hướng

1. Tạo phân tích mới từ tập dữ liệu của bạn

2. Thêm **Clustered bar chart** (Biểu đồ cột nhóm) cho các chất ô nhiễm trung bình hàng tháng:
   - Trục X: month
   - Trục Y: Sum of avg_PM25, Sum of avg_SO2, Sum of avg_O3
   - Nhóm các cột theo loại chất ô nhiễm
   - Sắp xếp theo tháng tăng dần
   - Tiêu đề: "Nồng độ chất ô nhiễm trung bình theo tháng"

3. Thêm **Scatter plot** (Biểu đồ phân tán) cho tương quan nhiệt độ và PM2.5:
   - Trục X: avg_temp
   - Trục Y: avg_PM25
   - Kích thước: số lượng bản ghi (tùy chọn)
   - Thêm đường xu hướng
   - Tiêu đề: "Mối quan hệ giữa nhiệt độ và PM2.5"

4. Thêm **Line chart** (Biểu đồ đường) cho PM2.5 theo tháng và mức AQI:
   - Trục X: month
   - Trục Y: avg_PM25
   - Màu sắc/Nhóm: AQI_Level
   - Sử dụng màu sắc phù hợp cho các mức AQI
   - Tiêu đề: "PM2.5 trung bình theo tháng và mức AQI"

5. Định dạng và sắp xếp các trực quan hóa:
   - Thêm hiểu biết sâu sắc dưới dạng hộp văn bản
   - Cấu hình tính tương tác của bảng điều khiển
   - Thêm bộ lọc cho mức AQI và phạm vi ngày

#### Bước 5: Nâng cao bảng điều khiển của bạn

1. **Thêm tính tương tác**:
   - Cấu hình lọc chéo giữa các trực quan hóa
   - Thêm điều khiển hành động để mở rộng dữ liệu
   - Tạo trường tính toán cho các chỉ số nâng cao

2. **Cải thiện định dạng**:
   - Sử dụng bảng màu nhất quán phù hợp với các mức AQI
   - Thêm nhãn trục và định dạng thích hợp
   - Bao gồm văn bản giải thích cho các trực quan hóa phức tạp

3. **Tạo điều khiển bảng điều khiển**:
   - Thêm bộ chọn phạm vi ngày
   - Tạo bộ lọc thả xuống cho các trạm, mức AQI
   - Bao gồm điều khiển tham số để điều chỉnh ngưỡng

4. **Thêm hiểu biết sâu sắc và tường thuật**:
   - Sử dụng hộp văn bản để làm nổi bật các phát hiện chính
   - Thêm cảnh báo có điều kiện cho vi phạm ngưỡng
   - Bao gồm đường tham chiếu cho các tiêu chuẩn quy định

### Phân tích bảng điều khiển của bạn

Khi bảng điều khiển của bạn hoàn thành, bạn có thể thực hiện phân tích chi tiết:

#### Phân tích bảng điều khiển tổng quan chất lượng không khí

1. **Chỉ số môi trường cao nhất**:
   - CO cho thấy giá trị trung bình cao nhất (993.92)
   - SO₂ theo sau với 224.61, sau đó là O₃ (94.23), NO₂ (96.44)
   - CO cũng cho thấy biến động cao (độ lệch chuẩn là 560.39)

2. **Xu hướng hàng tháng**:
   - CO đạt đỉnh vào tháng 4 và tháng 11, giảm đáng kể vào tháng 7-8
   - PM2.5 và SO2 giảm trong mùa mưa (giữa năm)
   - Chất lượng không khí được cải thiện trong những tháng mưa do các hạt được rửa trôi

3. **Phân bố AQI**:
   - Chất lượng không khí có xu hướng xấu hơn trong những tháng khô
   - Chất lượng không khí tốt hơn xuất hiện trong mùa mưa

#### Hiểu biết từ bảng điều khiển phân tích PM2.5

1. **Mẫu theo thời gian**:
   - PM2.5 dao động đáng kể, với đỉnh điểm vào đầu và cuối năm 2021
   - Trong mùa mưa (tháng 6-9), PM2.5 giảm xuống 10-20 µg/m³
   - Mối quan hệ ngược với độ ẩm: khi độ ẩm tăng, PM2.5 giảm

2. **Ảnh hưởng của nhiệt độ**:
   - PM2.5 có xu hướng tăng trong thời kỳ nhiệt độ thấp hơn
   - Điều này phù hợp với sự tích tụ hạt trong không khí lạnh, khô

3. **Phân bố AQI theo tháng**:
   - Các tháng mùa hè (tháng 6-9) cho thấy tỷ lệ phần trăm AQI "Good" cao nhất
   - Tháng 1 và tháng 12 cho thấy nhiều mức AQI "Moderate" hơn
   - Mẫu theo mùa rõ ràng trong chất lượng không khí

#### Phát hiện từ bảng điều khiển tương quan và xu hướng

1. **Phân bố chất ô nhiễm**:
   - SO₂ cao nhất theo khối lượng (phạm vi 200-280)
   - PM2.5 tăng vào tháng 1, 2, 4, 5, 10, 11, 12
   - Giảm đáng kể trong tháng mưa 6-9

2. **Mối quan hệ nhiệt độ-PM2.5**:
   - Mối quan hệ ngược: khi nhiệt độ tăng, PM2.5 giảm
   - Nồng độ PM2.5 cao nhất xảy ra ở nhiệt độ thấp hơn (26.5-27.2°C)
   - Ở nhiệt độ trên 28°C, PM2.5 giảm xuống còn 15-20 µg/m³

3. **Tương quan PM2.5 và AQI**:
   - Phân tầng rõ ràng giữa các mức AQI và giá trị PM2.5
   - Tháng 1, tháng 3, tháng 12 cho thấy mức PM2.5 "Moderate" cao nhất (~40 µg/m³)
   - Các tháng mùa hè cho thấy PM2.5 thấp hơn, tương ứng với AQI "Good" hoặc "Average"

### Xuất bản và chia sẻ bảng điều khiển

Khi bảng điều khiển của bạn hoàn thiện:

1. Nhấp **Publish dashboard** (Xuất bản bảng điều khiển) để lưu công việc của bạn
2. Thiết lập làm mới theo lịch để giữ dữ liệu hiện tại
3. Chia sẻ với các bên liên quan bằng cách:
   - Sử dụng chia sẻ bảng điều khiển trực tiếp
   - Nhúng vào các cổng thông tin nội bộ
   - Lên lịch báo cáo email
   - Xuất dưới dạng PDF để sử dụng ngoại tuyến

### Các thực hành tốt nhất cho bảng điều khiển chất lượng không khí QuickSight

1. **Sử dụng trực quan hóa thích hợp**:
   - Biểu đồ đường cho xu hướng theo thời gian
   - Biểu đồ cột cho so sánh
   - Biểu đồ phân tán cho tương quan
   - Bảng cho thống kê chi tiết

2. **Áp dụng mã màu nhất quán**:
   - Xanh lá/xanh dương cho chất lượng không khí tốt
   - Vàng/cam cho điều kiện trung bình
   - Đỏ cho điều kiện không lành mạnh

3. **Cung cấp bối cảnh**:
   - Thêm đường tham chiếu cho các tiêu chuẩn chất lượng không khí
   - Bao gồm văn bản giải thích để diễn giải các chỉ số
   - Sử dụng chú thích để làm nổi bật các sự kiện quan trọng

4. **Kích hoạt khám phá tự phục vụ**:
   - Thêm bộ lọc và điều khiển tương tác
   - Bao gồm khả năng mở rộng
   - Cung cấp điều khiển tham số để phân tích kịch bản

Bằng cách làm theo các bước này, bạn sẽ tạo các bảng điều khiển chất lượng không khí toàn diện cung cấp những hiểu biết có thể thực hiện về mẫu chất lượng không khí, mối tương quan giữa các yếu tố môi trường và biến đổi theo mùa trong mức độ ô nhiễm.
