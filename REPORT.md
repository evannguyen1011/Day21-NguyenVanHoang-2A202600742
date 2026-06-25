# Báo Cáo Lab Day 21 - CI/CD cho AI Systems
**Sinh viên:** Nguyễn Văn Hoàng — 2A202600742

---

## Bước 1 — Thực nghiệm cục bộ (Hoàn thành)

### Kết quả 4 thí nghiệm MLflow

| Lần | n_estimators | max_depth | min_samples_split | Accuracy | F1 |
|-----|-------------|-----------|-------------------|----------|----|
| 1 | 100 | 5 | 2 | 0.5640 | 0.5534 |
| 2 | 50 | 3 | 2 | 0.5580 | 0.5185 |
| 3 | 200 | 10 | 5 | 0.6440 | 0.6417 |
| **4** | **200** | **null** | **5** | **0.6600** | **0.6586** |

### Bộ siêu tham số tốt nhất được chọn
```yaml
n_estimators: 200
max_depth: null
min_samples_split: 5
```

**Lý do chọn:** Lần 4 cho accuracy cao nhất (0.66) và F1 cao nhất (0.6586). `max_depth=null` (không giới hạn độ sâu cây) giúp mô hình học được các pattern phức tạp hơn trong dữ liệu, kết hợp với `n_estimators=200` (nhiều cây hơn) cho độ ổn định cao hơn. `min_samples_split=5` giảm overfitting so với giá trị mặc định 2.

---

## Bước 2 — CI/CD Pipeline (Code hoàn chỉnh, cloud bị gián đoạn)

### Code đã hoàn thành
- `src/train.py` — Script huấn luyện đầy đủ, tích hợp MLflow
- `src/serve.py` — FastAPI server với endpoint `/health` và `/predict`
- `tests/test_train.py` — 3 unit tests, tất cả PASS khi chạy cục bộ
- `.github/workflows/mlops.yml` — Pipeline 4 jobs: Test → Train → Eval (gate 0.70) → Deploy

### Sự cố AWS Account

Trong quá trình thực hiện lab, AWS Access Key ID bị lộ do vô tình chia sẻ qua giao diện chat (không phải terminal). AWS phát hiện key công khai và tự động:
1. Deactivate Access Key
2. Suspend toàn bộ tài khoản AWS

**Bằng chứng:** Ảnh chụp màn hình đính kèm — AWS Console hiển thị "Authentication failed because your account has been suspended."

### Những gì đã hoàn thành trước khi account bị suspend
- ✅ Tạo S3 bucket `mlops-lab-hoang-day21` (region us-east-1)
- ✅ `dvc push` thành công — 3 files dữ liệu đã lên S3
- ✅ Tạo EC2 instance `i-0c35f149f994691ad` tại IP `54.242.92.173` (t2.micro, Ubuntu 22.04)
- ✅ Tạo security group mở port 22 và 8000
- ✅ Tạo SSH key pair `mlops-deploy`
- ❌ Không thể SSH vào VM để cài FastAPI do account bị suspend ngay sau đó
- ❌ Không thể thêm GitHub Secrets (cần credentials hoạt động)
- ❌ Pipeline GitHub Actions chưa chạy được

---

## Bước 3 — Huấn luyện liên tục (Code sẵn sàng)

Script `add_new_data.py` đã được kiểm tra. Workflow `.github/workflows/mlops.yml` được cấu hình trigger trên `data/**.dvc` — sẵn sàng chạy tự động khi có credentials hoạt động.

---

## Khó khăn gặp phải

1. **IAM user thiếu quyền:** User `ai-lab-user` ban đầu chỉ có `AmazonVPCFullAccess` và `IAMFullAccess`, không có S3 hay EC2. Đã tự gán thêm quyền qua CLI nhờ IAMFullAccess.
2. **AWS key bị lộ và account bị suspend:** Do sơ suất chia sẻ credentials qua giao diện chat thay vì terminal. AWS phát hiện và suspend account trong vòng vài phút.
3. **Hậu quả:** Không thể hoàn thiện phần deploy lên VM và chạy GitHub Actions pipeline.

---

## Kết luận

Bước 1 hoàn thành đầy đủ với 4 thí nghiệm MLflow. Toàn bộ code Bước 2 và 3 đã được viết và commit lên GitHub, unit tests pass. Phần triển khai cloud (EC2 + GitHub Actions) không thể hoàn thành do sự cố tài khoản AWS ngoài ý muốn trong quá trình làm lab.

Repo GitHub: https://github.com/evannguyen1011/Day21-NguyenVanHoang-2A202600742
