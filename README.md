# 📄 Resume–Job Description Matching với DistilBERT

Hệ thống tự động ghép nối **CV ứng viên** với **mô tả công việc (JD)** dựa trên độ tương đồng ngữ nghĩa, sử dụng mô hình **DistilBERT** và **Cosine Similarity**.

---

## 🗂️ Cấu trúc dự án

```
├── CODE.ipynb                        # Notebook chính: EDA, tiền xử lý, embedding, matching
├── pdf_extracted_skills_education.csv  # Dữ liệu CV trích xuất từ PDF (Skills, Education, Category)
├── training_data.csv                 # Dữ liệu mô tả công việc (JD) từ các công ty
└── BÁO_CÁO.docx                     # Báo cáo chi tiết của dự án
```

---

## 🎯 Mục tiêu

Tự động xếp hạng và tìm ra **Top-N ứng viên phù hợp nhất** cho mỗi vị trí tuyển dụng, dựa trên nội dung CV (kỹ năng + học vấn) và mô tả công việc.

---

## 📦 Dữ liệu

| File | Mô tả | Số lượng |
|------|-------|----------|
| `pdf_extracted_skills_education.csv` | CV trích xuất từ PDF, gồm các cột: `Skills`, `Education`, `ID`, `Category` | 2,484 hồ sơ |
| `training_data.csv` | Mô tả công việc từ các công ty lớn (Google, Apple, Netflix,...), gồm: `company_name`, `job_description`, `position_title`, `model_response` | 853 JD |

**Các danh mục CV (Category):** Accountant, Engineer, Designer, và nhiều ngành nghề khác.

---

## ⚙️ Pipeline xử lý

```
PDF CVs  ──►  Trích xuất Skills & Education  ──►  Làm sạch văn bản
                                                        │
Job Descriptions  ──────────────────────────────────────┘
                                                        │
                                              DistilBERT Embeddings
                                                        │
                                            Cosine Similarity Matrix
                                                        │
                                            Top-N Candidates per JD
```

### 1. Tiền xử lý văn bản (`text_cleaning`)
- Chuyển về chữ thường
- Mở rộng từ viết tắt (`don't` → `do not`)
- Xóa URL, email, số điện thoại
- Xóa dấu câu và ký tự đặc biệt

### 2. Phân tích khám phá dữ liệu (EDA)
- Phân bố số lượng CV theo từng danh mục nghề nghiệp
- Phân tích độ dài văn bản (theo percentile: 5%, 50%, 80%, 90%, 95%) của cả CV và JD
- So sánh độ dài từ giữa CV và JD

### 3. Tạo Embeddings
- Mô hình: `distilbert-base-uncased` (HuggingFace Transformers)
- Tokenize văn bản với `padding=True`, `truncation=True`
- Lấy trung bình `last_hidden_state` theo chiều token → vector 768 chiều

### 4. Tính độ tương đồng & Ranking
- Tính **Cosine Similarity** giữa tất cả cặp (JD, CV)
- Xếp hạng ứng viên theo điểm giảm dần
- Trả về **Top-5 ứng viên** cho mỗi JD

---

## 🚀 Hướng dẫn chạy

### 1. Cài đặt thư viện

```bash
pip install numpy pandas contractions tqdm seaborn matplotlib transformers torch scikit-learn
```

### 2. Chuẩn bị dữ liệu

Đặt các file sau vào cùng thư mục với `CODE.ipynb`:
- `pdf_extracted_skills_education.csv`
- `training_data.csv`

### 3. Chạy Notebook

```bash
jupyter notebook CODE.ipynb
```

Chạy tuần tự các cell từ đầu đến cuối.

---

## 📊 Kết quả mẫu

```
Top candidates for JD 1 - Position: Sales Specialist
  Candidate 1 - Similarity Score: 0.9231 - SALES/10554236.pdf
  Candidate 2 - Similarity Score: 0.9187 - BUSINESS-DEVELOPMENT/10674770.pdf
  Candidate 3 - Similarity Score: 0.9102 - SALES/11163645.pdf
  ...
```

---

## 🛠️ Công nghệ sử dụng

| Thư viện | Mục đích |
|----------|----------|
| `pandas`, `numpy` | Xử lý và phân tích dữ liệu |
| `transformers` | Tải và sử dụng mô hình DistilBERT |
| `torch` | Backend tính toán tensor |
| `scikit-learn` | Tính Cosine Similarity |
| `seaborn`, `matplotlib` | Trực quan hóa dữ liệu |
| `contractions`, `re` | Làm sạch văn bản |

---

## 📝 Ghi chú

- Số lượng JD (853) ít hơn CV (2,469) nên kết quả so sánh EDA có thể bị thiên lệch — chỉ mang tính chất minh họa.
- Mô hình DistilBERT được dùng ở chế độ **inference** (không fine-tune), phù hợp với bài toán matching zero-shot.
- Có thể mở rộng bằng cách tăng `num_top_candidates` hoặc thay bằng mô hình mạnh hơn (`bert-base`, `sentence-transformers`).
