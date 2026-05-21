# The-4nalySIS
# Dự Báo Nhu Cầu Chuỗi Cung Ứng Tối Giản (Non-Reindexed Demand Forecasting)

## Mô tả
Dự án tập trung vào việc xây dựng hệ thống dự báo sản lượng bán ra (Quantity) của các mã sản phẩm (SKU) theo ngày nhằm tối ưu hóa công tác quản lý tồn kho và kế hoạch cung ứng. 

Khác với các phương pháp tiếp cận chuỗi thời gian truyền thống thường yêu cầu tái cấu trúc toàn bộ lưới ngày (Reindex) gây phình to bộ nhớ và làm chậm CPU, giải pháp này áp dụng tư duy *Ước lượng phân phối nhu cầu dựa trên bối cảnh tĩnh (Context-Based Demand Estimation)*:
* *Không Reindex / Không tạo dòng trống:* Giữ nguyên cấu trúc dữ liệu thực tế (Sparse Matrix), giúp tối ưu hóa 100% tài nguyên RAM và tốc độ huấn luyện.
* *Xử lý Zero-Inflation & Outliers:* Áp dụng phép biến đổi logarit $y_{new} = \ln(y + 1)$ kết hợp hàm mất mát chống nhiễu ngoại lai *Huber Loss* nhằm xử lý triệt để tình trạng phân phối dữ liệu bị lệch phải nghiêm trọng (Highly Skewed).
* *Đặc trưng cốt lõi:* Khai thác sâu các biến chu kỳ thời gian (Tháng, Ngày trong tháng, Thứ), hệ thống ngày lễ Dương lịch & Âm lịch Việt Nam, cùng đặc trưng định giá cốt lõi như Biên lợi nhuận gộp (Profit_Margin).

## Cách chạy

### 1. Yêu cầu môi trường
Cài đặt các thư viện cần thiết trước khi thực thi mã nguồn:
```bash
pip install lightgbm numpy pandas scikit-learn matplotlib seaborn lunarcalendar
2. Cấu trúc luồng xử lý (Pipeline)
Mã nguồn được tổ chức nhất quán theo các bước sau trong Notebook:

Bước 1: Làm sạch dữ liệu, xử lý định dạng số lẻ và gộp nhóm (groupby) theo Ngày và Mã sản phẩm sử dụng giá trị median cho đơn giá.

Bước 2: Giới hạn ngoại lai (Clip Outliers) lượng bán ở phân vị 99.9% để loại bỏ nhiễu đột biến.

Bước 3: Trích xuất đặc trưng bối cảnh thời gian thực, tích hợp thư viện LunarDate để tự động gán nhãn các ngày lễ lớn tại Việt Nam (Tết Nguyên Đán, Giỗ tổ Hùng Vương, rằm tháng bảy,...).

Bước 4: Ép kiểu dữ liệu (downcast) thông minh về float32 và int8/int16 giúp giải phóng bộ nhớ RAM trước khi đưa vào mô hình.

Bước 5: Phân chia dữ liệu theo Mask thời gian nghiêm ngặt (Tập Test chiếm 28 ngày cuối cùng, tập Validation chiếm 28 ngày kế trước) để chống rò rỉ dữ liệu tương lai.

Bước 6: Huấn luyện mô hình LGBMRegressor trên không gian Logarit của Target, thiết lập early_stopping để bắt điểm tối ưu.

Bước 7: Dự báo tương lai trên ma trận bối cảnh mở rộng và xuất file kết quả nộp bài (submission_Binh_Yeu.csv).
