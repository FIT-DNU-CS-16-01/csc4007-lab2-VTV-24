# Lab 2 Analysis Report (IMDB)

## 1. Data Audit

### 1.1. Tổng quan dữ liệu
- **Số lượng mẫu:** 50,000
- **Phân bố nhãn:**  
  - negative: 25,000  
  - positive: 25,000  
  → Dataset **cân bằng hoàn toàn (imbalance ratio = 1.0)**

- **Missing / empty text:** Không có
- **Duplicate:** ~1.6% (≈ 824–834 mẫu)

---

### 1.2. Phân bố độ dài văn bản
- **Độ dài ký tự:**
  - min: ~30
  - median: ~970
  - p95: ~3391
  - max: ~13,700

- **Độ dài từ:**
  - min: 4–6 từ
  - median: ~173 từ
  - p95: ~590 từ
  - max: ~2500 từ

---

### 1.3. Nhận xét quan trọng

#### (1) Dataset cân bằng
- **Bằng chứng:** số lượng positive = negative (25k / 25k)
- **Ý nghĩa:**  
  - accuracy phản ánh khá đúng chất lượng model  
  - macro-F1 và accuracy gần nhau  
- **Quyết định:** vẫn cần báo cáo macro-F1 để giữ chuẩn đánh giá

---

#### (2) Review có độ dài rất đa dạng
- **Bằng chứng:** từ 4 → 2500 từ, p95 ~590 từ
- **Ý nghĩa:**  
  - review dài chứa nhiều ý → dễ gây nhầm  
  - review ngắn thiếu ngữ cảnh → khó phân loại  
- **Quyết định:** giữ nguyên, nhưng cần phân tích lỗi ở phần error analysis

---

#### (3) Có duplicate (~1.6%)
- **Bằng chứng:** ~824–834 mẫu trùng
- **Ý nghĩa:**  
  - có thể gây bias nhẹ cho model  
- **Quyết định:** không loại bỏ vì tỷ lệ nhỏ, không ảnh hưởng lớn

---

## 2. Preprocessing Design

### 2.1. Các bước tiền xử lý đã sử dụng
- Chuyển toàn bộ text về **lowercase**
- Làm sạch cơ bản (loại bỏ nhiễu đơn giản)
- Thử nghiệm:
  - `--replace_number`: thay số bằng token chung
  - `--drop_punct`: loại bỏ dấu câu

---

### 2.2. Xử lý dấu câu
- **Thử bỏ dấu câu:** có
- **Nhận xét:**
  - dấu câu như `!!!`, `???` mang thông tin cảm xúc
- **Kết luận:**  
  → không nên loại bỏ hoàn toàn vì có thể mất tín hiệu sentiment

---

### 2.3. Xử lý số
- **Có thử `replace_number`**
- **Ưu điểm:**
  - giảm nhiễu từ các số cụ thể
- **Nhược điểm:**
  - mất thông tin như `10/10`, `1/10`
- **Kết luận:**  
  → cần cân nhắc, không phải lúc nào cũng tốt

---

### 2.4. Những bước KHÔNG làm
- Không cleaning quá mạnh:
  - không xóa toàn bộ dấu câu
  - không loại bỏ các pattern cảm xúc

👉 Mục tiêu: **giữ lại tín hiệu sentiment**

---

## 3. Experiment Comparison

### 3.1. Bảng kết quả

| Run | Text version | Vectorizer | Model | ngram | Macro-F1 | Accuracy | Ghi chú |
|---|---|---|---|---|---:|---:|---|
| 1 | cleaned | BoW | Logistic Regression | unigram | 0.8956 | 0.8956 | baseline |
| 2 | cleaned | TF-IDF | Logistic Regression | unigram | 0.9064 | 0.9064 | cải thiện rõ |
| 3 | cleaned | TF-IDF | LinearSVM | unigram | **0.9100** | **0.9100** | tốt nhất |

---

### 3.2. So sánh BoW và TF-IDF

- BoW: 0.8956  
- TF-IDF: 0.9064  

→ **TF-IDF tốt hơn ~1%**

**Giải thích:**
- BoW:
  - chỉ đếm số lần xuất hiện từ
  - bị ảnh hưởng bởi từ phổ biến
- TF-IDF:
  - giảm trọng số từ phổ biến
  - nhấn mạnh từ mang tính phân biệt

👉 Phù hợp hơn với sentiment classification

---

### 3.3. So sánh Logistic Regression và LinearSVM

- LogReg: 0.9064  
- LinearSVM: 0.9100  

→ **SVM tốt hơn ~0.4%**

**Giải thích:**
- dữ liệu TF-IDF là **sparse**
- LinearSVM:
  - hoạt động tốt với dữ liệu sparse
  - tìm boundary tốt hơn

---

### 3.4. Nhận xét theo từng lớp

#### TF-IDF + LogReg
- negative recall thấp hơn (~0.894)
- positive recall cao hơn (~0.918)

→ hơi bias về positive

#### TF-IDF + SVM
- negative recall: ~0.900
- positive recall: ~0.920

→ cân bằng hơn

---

### 3.5. Kết luận cấu hình

👉 **TF-IDF + LinearSVM là tốt nhất**

Lý do:
- accuracy cao nhất
- macro-F1 cao nhất
- cân bằng giữa hai lớp

---

## 4. Error Analysis (>= 10 lỗi)

### 4.1. Tổng quan

- Nhiều lỗi có **confidence > 0.95**
→ mô hình rất tự tin nhưng sai

👉 cho thấy:
> model học pattern bề mặt, chưa hiểu ngữ nghĩa

---

### 4.2. Nhóm lỗi chính

#### (1) Phủ định (Negation)
- ví dụ: *not good, hardly, despite*
- mô hình không hiểu quan hệ phủ định

---

#### (2) Mixed sentiment
- review vừa khen vừa chê
- mô hình bị nhiễu bởi từ khóa mạnh

---

#### (3) Từ khóa mạnh nhưng ngữ cảnh đảo nghĩa
- từ như *great, wonderful* xuất hiện trong câu tiêu cực

---

#### (4) Sarcasm / mỉa mai
- nghĩa thực ≠ nghĩa từ

---

### 4.3. Bảng lỗi

| ID | True | Pred | Nhóm lỗi | Giải thích |
|---|---|---|---|---|
| 31245 | negative | positive | từ khóa mạnh | chứa từ tích cực |
| 37061 | negative | positive | phủ định | có negation |
| 8347 | negative | positive | phủ định | “despite” |
| 17596 | positive | negative | phủ định | “hardly” |
| 34543 | negative | positive | mixed | nhiều ý |
| 8659 | negative | positive | sarcasm | nghĩa ngược |
| 29148 | negative | positive | mixed | sentiment trộn |
| 16142 | positive | negative | sarcasm | mỉa mai |
| 22685 | positive | negative | mixed | nhiều chiều |
| 14750 | positive | negative | phủ định | đảo nghĩa |

---

## 5. Reflection

### 5.1. Pipeline tốt nhất
→ **TF-IDF + LinearSVM**

- phù hợp dữ liệu sparse
- hiệu suất cao nhất

---

### 5.2. Accuracy vs Macro-F1
- gần như bằng nhau (~0.91)
- do dataset cân bằng

---

### 5.3. Nếu dữ liệu lệch lớp
→ **Macro-F1 quan trọng hơn**

- vì đánh giá công bằng từng lớp
- không bị bias bởi lớp lớn

---

### 5.4. Hướng cải tiến (Lab 3)

- sử dụng mô hình deep learning:
  - LSTM
  - BERT

👉 để:
- hiểu ngữ cảnh tốt hơn
- xử lý phủ định và sarcasm

---

## 6. Conclusion

Bài lab đã xây dựng thành công pipeline NLP cổ điển gồm:
- tiền xử lý
- vector hóa (BoW, TF-IDF)
- mô hình baseline (LogReg, SVM)

Kết quả cho thấy:
- TF-IDF tốt hơn BoW
- LinearSVM tốt hơn Logistic Regression
- lỗi chủ yếu đến từ hạn chế trong việc hiểu ngữ nghĩa

👉 Đây là nền tảng quan trọng trước khi chuyển sang các mô hình học sâu.