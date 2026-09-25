# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Nguyễn Lê Thế Anh

Công cụ gán nhãn đã dùng: CVAT

## 1. Dữ liệu và cách chia tập

Tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng đệm ở giữa, thay vì chia ngẫu nhiên. Nếu chia ngẫu nhiên, cùng một xe có thể vừa được AI học vừa được dùng để chấm. Điều này làm điểm số trên tập kiểm thử bị lệch cao hơn thực tế, vì AI đã "nhìn thấy" xe đó trong quá trình học.

## 2. Mô hình khởi đầu lạnh (cold start)

Dòng vòng 0 từ `rounds_table.md`:

Mô hình khởi đầu lạnh không khớp nhãn tham chiếu ở các loại xe nhỏ và xe ở xa. Độ phủ (recall) theo kích thước xe cho thấy xe nhỏ chỉ được tìm thấy khoảng 0.182, trong khi xe vừa và lớn lần lượt là 0.547 và 0.561. Điều này nghĩa là xe ở xa bị bỏ sót nhiều hơn xe ở gần.

Ví dụ, trong ảnh `compare_round0.jpg`, một xe nhỏ ở góc dưới bên phải bị bỏ sót. Tuy nhiên, nhãn tham chiếu cũng do máy tạo và chưa được người kiểm tra, nên có thể nhãn tham chiếu sai chứ không phải mô hình.

## 3. Chiến lược chọn mẫu

Công thức `score = W_U·U + W_A·A + W_D·D` được dùng để tính điểm cho mỗi ảnh. Trong đó:
- `U` là độ bất định của mô hình.
- `A` là số khung AI còn lưỡng lự.
- `D` là khoảng cách thời gian giữa ảnh này và ảnh khác.

`MIN_GAP_S` đảm bảo hai ảnh trong cùng lô phải cách nhau ít nhất 2 giây, vì camera đứng yên, ảnh sát nhau gần như giống hệt.

Ví dụ, trong `SELECTION.md`, tôi đã chọn frame_0182.jpg, frame_0099.jpg và frame_0107.jpg vì chúng có điểm cao và AI khoanh nhiều xe nhưng còn nhiều khung chưa chắc. Tuy nhiên, frame_0372.jpg (hạng 6, điểm 0.910) không được chọn vì nó gần giống với frame_0369.jpg.

Điểm cao chỉ nghĩa là AI đang phân vân, không chứng minh sửa ảnh đó sẽ làm AI giỏi hơn.


## 4. Các vòng học chủ động (active learning)

Dòng vòng 1 từ `rounds_table.md`:

Trong vòng 1, tôi đã sửa nhãn như sau:
- Giữ nguyên: 8 khung.
- Chỉnh sửa: 5 khung.
- Xóa: 3 khung.
- Thêm mới: 4 khung.

So với vòng 0, AP50 tăng từ 0.771 lên 0.812. Xe nhỏ cải thiện recall từ 0.182 lên 0.250, xe vừa từ 0.547 lên 0.600, nhưng xe lớn giảm nhẹ từ 0.561 xuống 0.590.

Ví dụ, trong `compare_round1.jpg`, một xe nhỏ ở góc dưới bên phải đã được phát hiện sau khi tôi thêm khung trong vòng 1. Điều này cho thấy việc sửa nhãn đã giúp mô hình nhận diện tốt hơn.

## 5. Kết luận và giới hạn

Kết quả vòng 1 so với cold start cho thấy mô hình đã cải thiện, đặc biệt ở việc phát hiện xe nhỏ. Tôi quyết định tiếp tục thêm một vòng nữa để cải thiện recall cho xe lớn và xe ở xa.

Hai trường hợp còn yếu:
1. Xe ở xa chỉ còn hai chấm đèn, khó phát hiện.
2. Xe bị cắt mép ảnh, chỉ còn một phần thân xe.

Tuy nhiên, việc sửa thêm sẽ tốn thời gian, và cần tránh chọn hai ảnh sát nhau vì chúng gần như cùng một cảnh. Ngoài ra, tập kiểm thử chỉ có 20 ảnh, xe quá nhỏ không tính, và nhãn tham chiếu chưa được người kiểm tra. Những giới hạn này có thể ảnh hưởng đến kết luận.

Nếu AP50 giảm, tôi sẽ kiểm tra lại các khung đã sửa trước khi cho AI học thêm.


