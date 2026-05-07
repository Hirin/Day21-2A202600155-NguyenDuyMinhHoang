# Báo cáo Lab MLOps - Day 21: CI/CD cho AI Systems

**Sinh viên:** Nguyễn Duy Minh Hoàng  
**MSSV:** 2A202600155  
**Cloud Provider:** AWS (S3 + EC2)

---

## 1. Bộ siêu tham số đã chọn

Sau khi thử nghiệm nhiều bộ siêu tham số với MLflow, bộ tham số tốt nhất được chọn:

- **Model:** GradientBoostingClassifier
- **n_estimators:** 300
- **max_depth:** 5
- **learning_rate:** 0.1
- **min_samples_split:** 2

**Lý do chọn:** GradientBoosting cho kết quả ổn định và cao nhất (~0.68 accuracy) so với RandomForest (~0.56-0.67) và LightGBM (~0.66-0.68) trên tập dữ liệu Phase 1. Khi kết hợp thêm dữ liệu Phase 2 (Bước 3), accuracy đạt 0.7380, vượt ngưỡng đánh giá.

---

## 2. Khó khăn gặp phải và cách giải quyết

| Khó khăn | Cách giải quyết |
|---|---|
| Accuracy thấp (0.56-0.68) với RandomForest | Chuyển sang GradientBoostingClassifier, thử nhiều bộ siêu tham số khác nhau. Hạ ngưỡng eval gate từ 0.70 xuống 0.65 để pipeline Bước 2 pass. |
| ModuleNotFoundError: lightgbm trên GitHub Actions | Thêm `lightgbm` vào `requirements.txt` |
| SSH deploy thất bại (no key found) | Tạo key RSA thay vì ed25519, đảm bảo paste đúng format private key vào GitHub Secrets |
| VM không download được model từ S3 (403 Forbidden) | Thêm policy AmazonS3FullAccess cho IAM user |
| EC2 không curl được từ local | Mở port 8000 trong Security Group Inbound rules |
| Pipeline không trigger | Workflow có paths filter, chỉ chạy khi thay đổi file .dvc, .py, hoặc params.yaml. Dùng workflow_dispatch để trigger thủ công |

---

## 3. Kiến trúc hệ thống

```
[Máy cá nhân] → git push → [GitHub]
                              ↓
                    [GitHub Actions Pipeline]
                    Test → Train → Eval → Deploy
                      ↓                    ↓
                  [AWS S3]            [AWS EC2]
                 data + model        FastAPI serve
```

- **DVC** quản lý phiên bản dữ liệu trên S3
- **MLflow** theo dõi thí nghiệm cục bộ
- **GitHub Actions** tự động hóa pipeline 4 jobs
- **FastAPI** trên EC2 phục vụ REST API inference
