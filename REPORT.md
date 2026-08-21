# BÁO CÁO THỰC HÀNH MLOPS LAB - CI/CD CHO AI SYSTEMS
**Khóa học:** AIInAction - VinUni (K3 - Day 21)  
**Học viên:** Nguyễn Anh Quân  
**Mã số:** 2A202601251  
**GitHub Repository:** [https://github.com/QuanAnh280605/K3-Track2-Day21-2A202601251-NguyenAnhQuan](https://github.com/QuanAnh280605/K3-Track2-Day21-2A202601251-NguyenAnhQuan)

---

## 1. Kết Quả Thực Nghiệm & Lựa Chọn Siêu Tham Số (Bước 1)

Trong quá trình thực nghiệm cục bộ với mô hình `RandomForestClassifier`, 3 bộ siêu tham số đã được thử nghiệm và theo dõi tự động trên MLflow:

| Lần chạy | `n_estimators` | `max_depth` | `min_samples_split` | Accuracy | F1 Score (weighted) |
|---|---|---|---|---|---|
| **Run 1** | 100 | 5 | 2 | 0.5640 | 0.5534 |
| **Run 2** | 50 | 3 | 2 | 0.5580 | 0.5185 |
| **Run 3** | 200 | 12 | 2 | **0.6640** | **0.6620** |

- **Bộ siêu tham số được chọn:** `n_estimators: 200`, `max_depth: 12`, `min_samples_split: 2`.
- **Lý do lựa chọn:** Với bài toán phân loại chất lượng rượu vang từ 12 đặc trưng hóa học phức tạp, cây quyết định có độ sâu nông (`max_depth = 3, 5`) bị hiện tượng underfitting. Khi tăng `max_depth` lên 12 và số lượng cây `n_estimators` lên 200, mô hình học được các tương quan phi tuyến tốt hơn đáng kể, nâng accuracy từ 0.5640 lên **0.6640** và F1-score lên **0.6620**.

---

## 2. Bảng So Sánh Hiệu Suất Mô Hình (Bước 2 vs Bước 3)

| Chỉ số | Bước 2 (Tập dữ liệu ban đầu: 2.998 mẫu) | Bước 3 (Bổ sung dữ liệu mới: 5.996 mẫu) | Nhận xét |
|---|---|---|---|
| **Kích thước tập Train** | 2.998 mẫu (`train_phase1.csv`) | 5.996 mẫu (`train_phase1` + `train_phase2`) | Dữ liệu tăng gấp đôi |
| **Accuracy** | **0.6640** | **0.7580** | Tăng +9.4% |
| **F1 Score** | **0.6620** | **0.7540** | Tăng +9.2% |
| **Eval Gate (>= 0.70)** | ❌ BỊ CHẶN (Hủy deploy tự động) | ✅ PASSED (Vượt ngưỡng 0.70) | Cơ chế kiểm soát hoạt động chính xác |
| **Trạng thái Deploy** | Bị dừng tại job Eval | Triển khai tự động lên Cloud VM | VM phục vụ model mới nhất |

---

## 3. Khó Khăn Gặp Phải & Giải Pháp Xử Lý

1. **Khó khăn về SSH Key & Metadata trên Google Compute Engine:**
   - *Vấn đề:* Đăng nhập SSH với user `admin` mặc định của Windows bị từ chối do Ubuntu trên GCP hạn chế tên người dùng hệ thống này.
   - *Giải pháp:* Tạo tài khoản riêng `mlops` và gán SSH key công khai qua instance metadata (`ssh-keys="mlops:..."`).
2. **Xung đột đường dẫn cục bộ Windows khi chạy CI/CD trên Linux Runner:**
   - *Vấn đề:* Thư mục `mlruns/` chứa đường dẫn `file:///D:/...` vô tình bị commit lên git, khiến pytest trên runner Ubuntu báo lỗi `PermissionError: [Errno 13] Permission denied: '/D:'`.
   - *Giải pháp:* Bổ sung `mlruns/` vào `.gitignore`, xóa sạch tracking khỏi git và cấu hình URI mặc định `sqlite:///mlflow.db`.
3. **Cấu hình Systemd Service trên Cloud VM:**
   - *Vấn đề:* Ký tự escape biến môi trường khiến systemd báo lỗi cú pháp service.
   - *Giải pháp:* Chuẩn hóa nội dung file `mlops-serve.service`, thực hiện `daemon-reload` và kiểm thử tự động khởi động server REST API với FastAPI.
