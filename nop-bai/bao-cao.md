# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Đỗ Thái Sơn |
| MSSV | 2A202603021 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/tsun165/K4-L3-DAY21-DoThaiSon-2A202603021-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 50 | 0.05 | 3 | 0.6256 | 0.8540 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 3 | 200 | 0.1 | 4 | 0.7182 | 0.8760 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=4`.

**Lý do:** Bộ tham số ở lần chạy 3 mang lại điểm F1 của lớp dương cao nhất trên tập holdout (0.7182), vượt qua ngưỡng Quality Gate của hệ thống (0.65). Trong bài toán Gradient Boosting, việc tăng số lượng cây (n_estimators) kết hợp với độ sâu phù hợp (max_depth=4) giúp mô hình nắm bắt tốt hơn các tương tác phi tuyến tính giữa các đặc trưng mà không bị overfitting. Đáng chú ý, lần chạy 2 đạt accuracy cao nhất (0.8780) nhưng F1 lại thấp hơn lần 3 (0.7109 so với 0.7182), cho thấy accuracy không phản ánh trọn vẹn năng lực nhận diện lớp thiểu số.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult có phân bố lớp mất cân bằng đáng kể: chỉ khoảng 24.8% số mẫu thuộc lớp thu nhập cao (>50K USD/năm). Nếu một mô hình đơn giản luôn dự đoán nhãn là "thu nhập thấp" cho mọi cá nhân, mô hình đó vẫn đạt accuracy lên tới 75.2%, nhưng hoàn toàn vô dụng trên thực tế vì không phát hiện được bất kỳ người có thu nhập cao nào (Recall = 0, F1 = 0). Do đó, chỉ số Accuracy gây hiểu nhầm nghiêm trọng trong bối cảnh dữ liệu lệch lớp. 

Ngược lại, F1-score của lớp dương là trung bình điều hòa giữa Precision và Recall, đo lường chính xác khả năng tìm kiếm và độ chuẩn xác đối với nhóm đối tượng quan trọng (thu nhập cao). Lab không sử dụng tham số `average="weighted"` hay `average="macro"` vì các cách tính này sẽ bị lớp đa số chi phối, làm mất đi ý nghĩa giám sát nghiêm ngặt của Quality Gate.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Lỗi AccessDenied khi tạo S3 bucket qua AWS CLI | User IAM bị ràng buộc bởi Permissions Boundary mặc định | Truy cập IAM Console để gỡ Permissions Boundary và gán lại quyền AmazonS3FullAccess |
| Chứng chỉ SSL hết hạn khi tải dữ liệu từ UCI repository | Server của UCI archive có SSL certificate đã hết hạn | Bổ sung ngữ cảnh SSL unverified trong prepare_data.py để bỏ qua kiểm tra chứng chỉ |
| Quản lý xác thực Cloud trong GitHub Actions runner | Runner môi trường ảo cần quyền truy cập S3 và SSH VM | Sử dụng GitHub Secrets để lưu trữ credentials và tự động xuất biến môi trường trong pipeline |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7182 | 0.8760 |
| Bước 3 (thêm `train_batch2`) | 0.7240 | 0.8780 |

**Nhận xét:** Khi bổ sung thêm 22,361 mẫu dữ liệu mới từ `train_batch2` (tổng cộng 44,722 mẫu huấn luyện), cả F1-score và Accuracy đều tăng trưởng tích cực (F1 tăng từ 0.7182 lên 0.7240). Quy trình Continuous Training được kích hoạt hoàn toàn tự động qua commit DVC, chứng minh pipeline CI/CD vận hành ổn định và mô hình cải thiện hiệu năng khi có thêm dữ liệu huấn luyện.
