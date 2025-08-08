# Dự đoán giá bất động sản trên địa bàn Tp HCM
## Giới thiệu đề tài
Hiện nay, thị trường bất động sản tại TP. Hồ Chí Minh đang phát triển nhanh chóng nhưng cũng tiềm ẩn nhiều biến động. Việc không có một công cụ tham khảo nào có thể tham gia giúp đỡ người mua trong việc lựa chọn mặt hằng có giá trị phù hợp
hay giúp đỡ người bán trong việc định hình giá cả bất động sản nhanh chóng ngay từ ban đầu luôn là một vấn đề nan giải. Trên cơ sở đó, xây dựng một mô hình có thể tự động dự đoán giá nhà không chỉ phần nào tối ưu hai vấn đề trên
, mà còn có thể hỗ trợ các đơn vị quản lý đưa ra các chính sách phù hợp.

Dự án là một tập hợp xuyên suốt và liền mạch của nhiều quá trình, bao gồm:
- Thu thập dữ liệu.
- Tiền xử lý dữ liệu.
- Lựa chọn và xây dựng đặc trưng.
- Xây dựng mô hình.
- Đánh giá mô hình.

## Quá trình thực hiện
### Thu thập dữ liệu
**Tổng quan:** 
- Ở bước này, ta sẽ sử dụng Request để mở đường dẫn đến Web chứa Data cần thu thập, Beautiful Soup để truy xuất Html của trang Web, lưu vào các List 
được khởi tạo và sau khi hoàn thành, lưu lại dưới dạng một File CSV để sử dụng về sau.

Tạo các List để lưu các thông tin mà chúng ta có thể thu thập được sau khi xem xét trực quan một vài trang web,
bao gồm các thông tin:
- Diện tích căn nhà.
- Số nhà vệ sinh (WC).
- Số phòng ngủ (PN).
- Giá.
- Hướng nhà.
- Hướng ban công.
- Thông tin mô tả.
- Thời gian bài đăng:
  + Giờ (time).
  + Ngày/Tháng/Năm (Date).
- Loại nhà.
- Vị trí.

**Kết quả:**
- Thành công trong việc sử dụng được Beautiful Soup, Request và các hàm xung quanh để truy xuất thông tin từ trang Web.

**Khó khăn:**
- Có nhiều dữ liệu không có sẵn, nên có rất nhiều dữ liệu NULL.
- Cần nhiều thời gian (4 giờ) để hoàn thành.

#### Anyscale
**Tổng quan:** Ở giai đoạn này, ta sẽ thực hiện:
- Bằng cách xem xét các chuỗi trong phần Mô tả, ta nhận thấy ở đây sẽ có thông tin của các thuộc tính như: PN, WC, Diện tích, Giá
và một số Keyword mới mà ta có thể khai thác làm thuộc tính như: Tầng (Floor), Sổ sách/Sổ hồng/Pháp lý.
- Ta sẽ tìm cách để Fill các giá trị NULL trong các thuộc tính như: PN, WC, và tạo các thuộc tính mới bằng thông tin xử lý ở phần mô tả bằng Anyscale.

**Kết quả:**
- Thành công trong việc sử dụng Anyscale để trích xuất "Mô tả".

**Khó khăn:**
- Anyscale trả về nhiều giá trị không đúng như định dạng sẵn, làm mất nhiều thời gian để xử lý.
- Mỗi Token của Anyscale chỉ hoạt động trong vòng 1 giờ nên phải chạy thủ công nhiều lần để hoàn thành dữ liệu lớn.

### Tiền xử lý dữ liệu và Xây dựng đặc trưng
**Tổng quan:** Ở giai đoạn này, ta sẽ thực hiện:
- Nếu các giá trị từ Dataframe ban đầu là null, ta sẽ điền các giá trị từ bên phải (Dữ liệu lấy từ Mô tả) vừa xây dựng bên trên vào.
- Xây dựng và áp dụng các hàm chuyển đổi các Thuộc tính như: Area (Diện tích), PN (Số Phòng ngủ),... từ dạng object (string)
về dạng số.
- Xóa bỏ các Duplicates.
- Nhận thấy trực quan các giá trị Price có đơn vị chưa chính xác, ta sẽ Format các giá trị về bằng cách đưa các số lớn hơn > 1 tỷ / 1 tỷ, >1 triệu / 1 triệu,...
- Xử lý các Outliers (của Price, WC, PN, Area) bằng Percentile.
- Fill các giá trị null của các thuộc tính số bằng K-Neighbor Regressor.
- Xử lý các giá trị Categorical:
  + Chuyển dạng một vài giá trị của thuộc tính "SoSach" về dạng "Có", "Không", sau đó chuyển các giá trị chưa biết thành "Unknown".
  + Dùng Target Encoding cho các thuộc tính "Type" (Loại nhà), "district" (Vị trí), "SoSach" (Pháp lý).
- Trực quan hóa các thuộc tính để xem xét phân phối của thuộc tính.
- Vẽ Heatmap để xem xét Correlation giữa các thuộc tính, sau đó loại bỏ các thuộc tính không cần thiết.

**Ma trận tương quan cho thấy:**
- Biến phụ thuộc Price phụ thuộc tương đối cao các biến "Area", "WC", "PN".
- Các thuộc tính "Floor", "HouseType", "Location" cũng ảnh hưởng tương đối đến y.
- Các thuộc tính "Month", "Year", "Day", "Legal" hầu như không ảnh hưởng đến y.

**Kết quả:**
- Thành công trong việc làm sạch và trả về một Dataframe gần như đầy đủ và hợp lý để chuẩn bị cho Modeling.
  
**Khó khăn:**
- Không biết nên sử dụng phương pháp nào để xử lý các Outlier hiệu quả.
- Không biết nên sử dụng phương pháp nào để xử lý các Categorical Attribute hiệu quả hơn.

### Xây dựng mô hình và đánh giá
**Tổng quan:** Ở giai đoạn này, ta sẽ:
- Chia dữ liệu thành tập X (Gồm các thuộc tính hay biến giải thích: Diện tích, Số phòng ngủ,...) và y (Gồm biến phụ thuộc: Price).
- Tiến hành chạy một số mô hình lên dữ liệu bao gồm:
  + Linear Regression.
  + Ridge Regression.
  + Random Forest Regressor.
  + XGBoost Regressor.
- Dùng các chỉ số đánh giá các mô hình:
  + R2 Score.
  + Mean Square Error.
- So sánh các chỉ số của các mô hình.

**So sánh R2 Score giữa các mô hình:**
- R2 Score tăng dần theo độ phức tạp của mô hình (Linear < Ridge < RandomForest <= XGBoost).
- R2 Score của mô hình Linear Regression tương đối thấp (~ 0.4).
- Ridge Regression cho ra R2 Score ~ 0.5.
- Mô hình RandomForest và XGBoost hoạt động khá tốt trên Dataset khi cho R2 Score ~ 0.6.

**So sánh MSE giữa các mô hình:**
- MSE giảm dần theo độ phức tạp của mô hình (Linear > Ridge > RandomForest >= XGBoost).
- MSE của mô hình Linear Regression tương đối cao (> 5).
- Mô hình RandomForest và XGBoost hoạt động khá tốt trên Dataset khi cho MSE ~ 3,5.

**Kết quả:**
- Các mô hình hoạt động được trên Dataset.
- Các thông số đo được có kết quả tốt dần theo độ phức tạp của mô hình, cụ thể:
  + MSE giảm dần khi sử dụng mô hình càng phức tạp.
  + R2 Score tăng dần khi sử dụng mô hình càng cao.

- Giải thích:
  + Linear Regression (R2 Score: 0.4, MSE: 5.7): Mô hình LR hoạt động không tốt đối với các thuộc tính đầu vào phi tuyến tính và dễ bị ảnh hưởng bởi các Outliers và dữ liệu phức tạp.
  + Ridge Regression (R2 Score: 0.5, MSE: 4.79): Mô hình RR tương tự như LR khi không cho độ chính xác cao với các thuộc tính phi tuyến nhưng nhờ có thêm các Regularization giúp giảm thiểu Overfitting.
  + Random Forest Regression (R2 Score: 0.61, MSE: 3.72): Mô hình RFR cho hiệu quả khá tốt khi có thể hoạt động tốt với các dữ liệu phi tuyến, sử dụng nhiều Decision Tree để đưa ra quyết định nên giảm thiểu Over và Underfitting, cải thiện độ chính xác và ổn định.
  + XGBoost Regressor (R2 Score: 0.62, MSE: 3.65): Mô hình XGB cho hiệu quả tốt nhất khi vừa hoạt động tốt với dữ liệu phi tuyến, vừa kết hợp của các cây quyết định và các Regularization để giảm thiểu Overfitting.

**Khó khăn:**
- Chưa đủ kiến thức để có thể sử dụng các mô hình phức tạp hơn.
- Chưa biết nhiều cách để có thể nâng hiệu suất của mô hình.
