# Báo cáo benchmark LightGBM trên AWS

1. Tôi dùng **AWS EC2**, region **us-east-1**, zone **us-east-1a**, instance type **t3.small** cho máy chạy LightGBM (CPU, kiến trúc x86_64). Thông tin instance được đối chiếu từ state Terraform với IP private `10.0.10.108`, trùng với máy trong ảnh terminal. Source commit của repo hiện tại là `55539f67d7c78b43afe334a2ec3271c4bfdbbe2d`. Có thay đổi chưa commit trong Terraform và thư mục `submission/`, nên SHA này là commit nền, chưa đại diện đầy đủ cho mã đã chạy benchmark.

2. Dataset `creditcard.csv` có **284.807 dòng**, trong đó **492 dòng gian lận**. Tôi chia dữ liệu theo tỷ lệ **60% train / 20% validation / 20% test**, tương ứng **170.883 / 56.962 / 56.962 dòng**. Tôi dùng `train_test_split` hai lần với `stratify=y` (lần chia tiếp theo dùng `stratify=y_trainval`) để giữ phân bố nhãn, seed **16**. Model là `LGBMClassifier`, dùng 2 luồng CPU, tối đa 300 iteration và early stopping sau 20 iteration không cải thiện AUC trên tập validation.

3. Load dữ liệu mất **2,2704 giây**; training mất **3,4967 giây**; best iteration là **68**. Thời gian load chỉ đo thao tác đọc CSV; thời gian training đo lời gọi `model.fit`, bao gồm đánh giá validation và early stopping.

4. Trên tập test, model đạt **AUC-ROC 0,976848**, **Accuracy 99,9508%**, **F1-Score 0,847826**, **Precision 90,6977%**, **Recall 79,5918%**. Ngưỡng phân loại là **0,5**. Do dữ liệu mất cân bằng mạnh, tôi xem Precision, Recall và F1 cùng với Accuracy để đánh giá khả năng phát hiện gian lận.

5. Latency dự đoán 1 dòng là **1,1847 ms**; throughput batch 1.000 dòng là **328.514,46 dòng/giây**, tương ứng **3,0440 ms/batch**. Tôi đo bằng `time.perf_counter()` quanh `predict_proba` với đầu vào pandas, warm-up trước khi đo và không tính warm-up vào kết quả. Tôi lấy **trung vị của 50 lần** cho 1 dòng và **trung vị của 10 lần** cho batch 1.000 dòng; throughput = 1.000 / thời gian batch tính bằng giây. Kết quả được ghi lúc **00:13:58 ngày 04/10/2026 (UTC+7)**, tương ứng `2026-10-03T17:13:58.509825+00:00` trong JSON.

6. Trong ảnh `top`, đồng hồ hiển thị **17:14:00** (ảnh không ghi ngày hoặc múi giờ). CPU có **30,5% user**, **0,3% system**, **69,2% idle**; RAM tổng **1.910,5 MiB**, dùng **224,8 MiB**, available **1.488,0 MiB**. Ảnh `free -h` ghi RAM tổng **1,9 GiB**, dùng **223 MiB**, available **1,5 GiB**, không có swap. Network trên interface `ens5` ghi nhận RX **27.959.038 byte / 191.492 packet**, TX **797.674 byte / 8.482 packet**, không có lỗi hoặc packet bị drop. Đây là bộ đếm tích lũy, chưa phải tốc độ mạng. Các ảnh là snapshot, chưa chứng minh mức sử dụng cao nhất trong lúc training.

   Ảnh đính kèm: [CPU và RAM qua top](submission/resource_top.png), [RAM qua free -h](submission/resource_free.png), [Network qua ip -s link](submission/resource_ip-s.png).

7. Tại thời điểm chụp [ảnh billing](submission/billing.png) (**00:12 ngày 04/10/2026, UTC+7**, theo thời điểm tôi ghi nhận), AWS Billing hiển thị **“Data unavailable”** ở phần chi phí và chưa có cost breakdown. Tài khoản đang dùng free plan, credit còn **100,00 USD**, thời hạn còn **182 ngày**. Tôi chưa có số liệu chi phí thực tế cho lần benchmark và không coi “Data unavailable” là chi phí 0 USD. **Chưa lập ước tính chi phí riêng**.

8. Tôi đã lưu kết quả trong [benchmark_result.json](submission/benchmark_result.json) và script trong [benchmark.py](submission/benchmark.py) tại thư mục nộp bài; **chưa có bằng chứng thời điểm tải kết quả về**. Sau khi hoàn thành, tôi đã chạy `terraform destroy` để dọn dẹp hạ tầng. [Ảnh kết quả dọn dẹp](submission/terraform_destroy.png) hiển thị **“Destroy complete! Resources: 27 destroyed.”**, xác nhận Terraform đã hoàn tất xóa 27 tài nguyên trong lần chạy này. Thời điểm hoàn tất dọn dẹp được tôi ghi nhận là **00:35:45 ngày 04/10/2026 (UTC+7)**; timestamp này được bổ sung khi viết báo cáo, không hiển thị trong ảnh.

Số liệu benchmark trong báo cáo lấy từ JSON đã lưu. [Ảnh terminal](submission/benchmark_terminal.png) ghi một lần chạy lúc `2026-10-03T17:11:28.911434+00:00`, có thời gian load/training và inference khác JSON; ảnh này chỉ minh họa việc chạy script, không phải bằng chứng số liệu của cùng lần chạy được báo cáo.
