# 🤖 ÔN TẬP NHANH MACHINE LEARNING CƠ BẢN
### Khái niệm · Thuật toán · Đánh giá · Deep Learning

---

## 1. MACHINE LEARNING LÀ GÌ?

Một nhánh của AI, cho phép máy tính **học từ dữ liệu** và đưa ra dự đoán mà không cần lập trình tường minh từng rule.

```
Dữ liệu + Thuật toán  →  Mô hình  →  Dự đoán
(Training)                              (Inference)
```

### 3 loại ML chính

| Loại | Dữ liệu | Ví dụ |
|---|---|---|
| **Supervised Learning** | Có nhãn (labeled) | Phân loại spam, dự đoán giá nhà |
| **Unsupervised Learning** | Không có nhãn | Phân cụm khách hàng, giảm chiều |
| **Reinforcement Learning** | Thưởng/phạt từ môi trường | Robot tự lái, chơi game |

---

## 2. SUPERVISED LEARNING

Dữ liệu gồm **Feature (X)** và **Label (Y)**. Mô hình học cách ánh xạ X → Y.

### 2 bài toán chính

| | Classification (Phân loại) | Regression (Hồi quy) |
|---|---|---|
| Output | Nhãn rời rạc | Giá trị liên tục |
| Ví dụ | Spam / Not Spam | Giá nhà = 2.5 tỷ |
| Metric | Accuracy, Precision, Recall, F1 | MAE, MSE, R² |

### Thuật toán phổ biến

**Linear Regression (Hồi quy tuyến tính)**
- Tìm đường thẳng tốt nhất: `Y = aX + b`
- Dùng cho: dự đoán giá, doanh thu, nhiệt độ
- Đơn giản, dễ hiểu, baseline tốt

**Logistic Regression (Hồi quy logistic)**
- Tên có "Regression" nhưng dùng cho **Classification**
- Output là xác suất (0 → 1), dùng sigmoid function
- Dùng cho: phân loại nhị phân (spam/not spam, bệnh/không bệnh)

**Decision Tree (Cây quyết định)**
- Chia dữ liệu bằng các câu hỏi If-Else liên tiếp
- Dễ hiểu, dễ giải thích (interpretable)
- Nhược điểm: dễ overfit nếu cây quá sâu

**Random Forest**
- **Ensemble** của nhiều Decision Tree
- Mỗi cây train trên tập con dữ liệu ngẫu nhiên (bagging)
- Kết quả = bỏ phiếu đa số (classification) hoặc trung bình (regression)
- Ít overfit hơn 1 cây đơn lẻ, chính xác hơn

**SVM (Support Vector Machine)**
- Tìm **siêu phẳng** (hyperplane) phân tách 2 lớp với **margin lớn nhất**
- Các điểm gần hyperplane nhất gọi là **support vectors**
- Dùng kernel trick để xử lý dữ liệu phi tuyến
- Tốt cho dữ liệu chiều cao, tập nhỏ-vừa

**KNN (K-Nearest Neighbors)**
- Phân loại dựa trên **K điểm gần nhất**
- Không cần training (lazy learner)
- Nhược điểm: chậm với dữ liệu lớn, nhạy với scale

**XGBoost / Gradient Boosting**
- **Ensemble** kiểu boosting: mỗi cây sau tập trung **sửa lỗi** cây trước
- XGBoost thêm regularization, xử lý missing values, song song hóa
- Thường **thắng competition** trên Kaggle
- Phù hợp dữ liệu bảng (tabular data)

---

## 3. UNSUPERVISED LEARNING

Dữ liệu **không có nhãn**. Mô hình tự tìm cấu trúc ẩn.

### Các bài toán chính

**Clustering (Phân cụm)**

| Thuật toán | Cách hoạt động |
|---|---|
| **K-Means** | Chia thành K cụm, mỗi điểm thuộc cụm có centroid gần nhất. Lặp cập nhật centroid đến hội tụ |
| **DBSCAN** | Tìm cụm dựa trên mật độ (density), tự phát hiện số cụm và outlier |
| **Hierarchical** | Gom nhóm từ dưới lên (agglomerative) hoặc chia từ trên xuống |

Ứng dụng: phân nhóm khách hàng, phát hiện bất thường, gom tài liệu

**Dimensionality Reduction (Giảm chiều)**

| Thuật toán | Mục đích |
|---|---|
| **PCA** | Giảm số feature, giữ lại phương sai lớn nhất |
| **t-SNE** | Trực quan hóa dữ liệu chiều cao xuống 2D/3D |

---

## 4. TIỀN XỬ LÝ DỮ LIỆU

### Quy trình chuẩn

```
Dữ liệu thô
    ↓
1. Xử lý missing values (điền trung bình, median, mode hoặc xóa)
    ↓
2. Xử lý outliers (IQR, Z-score)
    ↓
3. Encoding biến phân loại
    ↓
4. Feature Scaling
    ↓
5. Feature Selection / Engineering
    ↓
6. Chia Train / Validation / Test
    ↓
Dữ liệu sẵn sàng training
```

### Encoding biến phân loại

| Phương pháp | Khi nào dùng | Ví dụ |
|---|---|---|
| **Label Encoding** | Biến có thứ tự (ordinal) | Thấp=1, Trung bình=2, Cao=3 |
| **One-Hot Encoding** | Biến không thứ tự (nominal) | Đỏ→[1,0,0], Xanh→[0,1,0] |

### Feature Scaling

| Phương pháp | Công thức | Khi nào dùng |
|---|---|---|
| **Normalization** (Min-Max) | (x - min) / (max - min) → [0, 1] | KNN, Neural Network |
| **Standardization** (Z-score) | (x - mean) / std → mean=0, std=1 | SVM, Logistic Regression, PCA |

Tại sao cần scaling? Các feature có thang đo khác nhau (tuổi: 0-100, thu nhập: 0-1 tỷ) → thuật toán dựa trên khoảng cách bị feature lớn chi phối.

### Chia dữ liệu

```
Toàn bộ dữ liệu
├── Train (70-80%) → Huấn luyện mô hình
├── Validation (10-15%) → Điều chỉnh hyperparameter
└── Test (10-15%) → Đánh giá cuối cùng (KHÔNG được dùng để tune)
```

---

## 5. ĐÁNH GIÁ MÔ HÌNH

### Classification Metrics

**Confusion Matrix:**
```
                  Dự đoán
                Positive  Negative
Thực tế  Pos │   TP    │   FN    │
         Neg │   FP    │   TN    │
```

| Metric | Công thức | Ý nghĩa |
|---|---|---|
| **Accuracy** | (TP+TN) / Tổng | Tỷ lệ đúng tổng thể |
| **Precision** | TP / (TP+FP) | Trong số dự đoán Positive, bao nhiêu đúng? |
| **Recall** | TP / (TP+FN) | Trong số thực sự Positive, tìm được bao nhiêu? |
| **F1-Score** | 2 × (P×R)/(P+R) | Trung bình điều hòa Precision & Recall |
| **AUC-ROC** | Diện tích dưới ROC | Khả năng phân biệt giữa 2 lớp ở mọi threshold |

**Khi nào dùng metric nào?**
- Dữ liệu cân bằng → **Accuracy** OK
- Dữ liệu mất cân bằng → dùng **Precision, Recall, F1**
- Cần đánh giá tổng thể → **AUC-ROC**
- Phát hiện bệnh (bỏ sót = nguy hiểm) → ưu tiên **Recall**
- Lọc spam (báo nhầm = phiền) → ưu tiên **Precision**

### Regression Metrics

| Metric | Ý nghĩa |
|---|---|
| **MAE** (Mean Absolute Error) | Trung bình sai số tuyệt đối |
| **MSE** (Mean Squared Error) | Trung bình bình phương sai số (phạt lỗi lớn nhiều hơn) |
| **RMSE** | Căn bậc 2 của MSE (cùng đơn vị với Y) |
| **R²** | Tỷ lệ phương sai được giải thích (0→1, càng cao càng tốt) |

### Cross-Validation

**K-Fold Cross Validation:**
```
Fold 1: [TEST] [Train] [Train] [Train] [Train]
Fold 2: [Train] [TEST] [Train] [Train] [Train]
Fold 3: [Train] [Train] [TEST] [Train] [Train]
Fold 4: [Train] [Train] [Train] [TEST] [Train]
Fold 5: [Train] [Train] [Train] [Train] [TEST]

→ Kết quả = Trung bình 5 lần → đánh giá ổn định hơn
```

---

## 6. OVERFITTING vs UNDERFITTING

```
Underfitting          Vừa phải (Good Fit)       Overfitting
Mô hình quá          Tổng quát tốt             Mô hình quá
đơn giản              ✅                        phức tạp

Train: thấp          Train: cao                 Train: rất cao
Test: thấp           Test: cao                  Test: thấp
→ High Bias          → Balanced                 → High Variance
```

### Cách chống Overfitting

| Phương pháp | Cách hoạt động |
|---|---|
| **Regularization** | Thêm penalty vào hàm loss để giới hạn trọng số |
| → L1 (Lasso) | Thêm tổng giá trị tuyệt đối trọng số. Có thể đưa trọng số về 0 → feature selection |
| → L2 (Ridge) | Thêm tổng bình phương trọng số. Co nhỏ trọng số nhưng không về 0 |
| **Dropout** | Tắt ngẫu nhiên % neuron mỗi lần train (Neural Network) |
| **Early Stopping** | Dừng train khi validation error bắt đầu tăng |
| **Thêm dữ liệu** | Nhiều dữ liệu hơn → mô hình tổng quát hơn |
| **Data Augmentation** | Tạo thêm dữ liệu bằng biến đổi (xoay ảnh, lật ảnh...) |
| **Cross-Validation** | Đánh giá ổn định hơn, tránh may mắn trên 1 tập test |
| **Giảm độ phức tạp** | Ít feature hơn, cây nông hơn, ít layer hơn |

---

## 7. DEEP LEARNING CƠ BẢN

### Neural Network

```
Input Layer     Hidden Layers     Output Layer
   (X)          (học features)       (Y)

  O ─────┐
  O ──────→  O ──→ O ──→  O  →  Kết quả
  O ──────→  O ──→ O ──→  O
  O ─────┘

Mỗi kết nối có trọng số (weight)
Mỗi neuron: z = Σ(wi × xi) + b → activation(z) → output
```

### Activation Functions

| Function | Output | Dùng cho |
|---|---|---|
| **Sigmoid** | (0, 1) | Output layer phân loại nhị phân |
| **Tanh** | (-1, 1) | Hidden layers (ít dùng nay) |
| **ReLU** | max(0, x) | Hidden layers (phổ biến nhất) |
| **Softmax** | Xác suất các lớp (tổng = 1) | Output layer phân loại đa lớp |

Tại sao cần activation? Nếu không có, mạng chỉ là tổ hợp tuyến tính → không học được quan hệ phức tạp.

### Gradient Descent

Thuật toán tối ưu để **tìm trọng số tốt nhất**:
1. Tính loss (sai số giữa dự đoán và thực tế)
2. Tính gradient (đạo hàm loss theo trọng số)
3. Cập nhật trọng số theo hướng giảm loss
4. Lặp lại cho đến khi hội tụ

```
w_mới = w_cũ - learning_rate × gradient
```

**Learning Rate:**
- Quá lớn → dao động, không hội tụ
- Quá nhỏ → hội tụ rất chậm
- Thường bắt đầu: 0.001 hoặc 0.01

### Các biến thể Gradient Descent

| Loại | Cách hoạt động |
|---|---|
| **Batch GD** | Dùng toàn bộ dữ liệu mỗi bước → chính xác nhưng chậm |
| **Stochastic GD (SGD)** | Dùng 1 mẫu mỗi bước → nhanh nhưng dao động |
| **Mini-batch GD** | Dùng batch nhỏ (32, 64, 128) → cân bằng, phổ biến nhất |

### Các vấn đề trong Deep Learning

**Vanishing Gradient**
- Gradient quá nhỏ qua nhiều lớp → lớp đầu không học được
- Xảy ra với Sigmoid/Tanh
- Giải pháp: ReLU, Batch Normalization, Skip Connection (ResNet)

**Batch Normalization**
- Chuẩn hóa output mỗi lớp về mean=0, std=1
- Giúp training ổn định, nhanh hơn, dùng learning rate lớn hơn

### Kiến trúc phổ biến

| Kiến trúc | Dùng cho | Đặc điểm |
|---|---|---|
| **CNN** (Convolutional) | Ảnh, video | Convolution layers trích xuất features từ ảnh |
| **RNN** (Recurrent) | Chuỗi, văn bản, time series | Có "bộ nhớ" xử lý tuần tự |
| **LSTM** | Chuỗi dài | Cải tiến RNN, nhớ được long-term |
| **Transformer** | NLP, mọi thứ | Self-Attention, xử lý song song, nền tảng GPT/BERT |

**Transformer vs RNN:**
- RNN: xử lý tuần tự (từng từ) → chậm, khó nhớ xa
- Transformer: Self-Attention nhìn toàn bộ chuỗi cùng lúc → song song, nhớ xa tốt hơn

---

## 8. KHÁI NIỆM QUAN TRỌNG KHÁC

### Bias vs Variance

| | High Bias | High Variance |
|---|---|---|
| Hiện tượng | Underfitting | Overfitting |
| Train error | Cao | Thấp |
| Test error | Cao | Cao |
| Nguyên nhân | Mô hình quá đơn giản | Mô hình quá phức tạp |
| Giải pháp | Thêm features, dùng mô hình phức tạp hơn | Regularization, thêm data, giảm features |

### Ensemble Methods

| Phương pháp | Cách hoạt động | Ví dụ |
|---|---|---|
| **Bagging** | Train song song trên tập con ngẫu nhiên → vote/average | Random Forest |
| **Boosting** | Train tuần tự, mỗi mô hình sửa lỗi mô hình trước | XGBoost, AdaBoost |
| **Stacking** | Kết hợp output nhiều mô hình làm input cho mô hình meta | Stacked models |

Bagging **giảm variance** (chống overfit). Boosting **giảm bias** (tăng accuracy).

### Data Leakage

Thông tin từ tập test/tương lai **rò rỉ** vào quá trình training → kết quả đánh giá ảo cao, thực tế thất bại.

Ví dụ sai: normalize toàn bộ dữ liệu **trước** khi chia train/test → test set đã ảnh hưởng đến mean/std.

Cách tránh: luôn chia train/test **trước**, rồi mới fit scaler/encoder trên train, transform cả train và test.

### Curse of Dimensionality

Khi số chiều (features) tăng → dữ liệu trở nên **thưa thớt**, khoảng cách mất ý nghĩa, cần dữ liệu tăng theo cấp số nhân.

Giải pháp: PCA, Feature Selection, Regularization.

### Concept Drift

Phân phối dữ liệu **thay đổi theo thời gian** → mô hình cũ không còn chính xác.

Ví dụ: hành vi mua hàng thay đổi sau COVID → mô hình gợi ý sản phẩm sai.

Giải pháp: monitoring metrics, phát hiện drift, retrain định kỳ.

---

## 9. BẢNG TRA NHANH THUẬT TOÁN

| Thuật toán | Loại | Bài toán | Ưu điểm | Nhược điểm |
|---|---|---|---|---|
| Linear Regression | Supervised | Regression | Đơn giản, nhanh | Chỉ tuyến tính |
| Logistic Regression | Supervised | Classification | Nhanh, output xác suất | Chỉ tuyến tính |
| Decision Tree | Supervised | Cả hai | Dễ hiểu, interpretable | Dễ overfit |
| Random Forest | Supervised | Cả hai | Chính xác, ít overfit | Chậm, khó giải thích |
| XGBoost | Supervised | Cả hai | Rất chính xác, nhanh | Cần tune nhiều |
| SVM | Supervised | Classification | Tốt chiều cao | Chậm dữ liệu lớn |
| KNN | Supervised | Cả hai | Đơn giản | Chậm, nhạy scale |
| K-Means | Unsupervised | Clustering | Nhanh, đơn giản | Cần chọn K trước |
| PCA | Unsupervised | Giảm chiều | Giảm noise, tăng tốc | Mất interpretability |
| Neural Network | Supervised | Cả hai | Học quan hệ phức tạp | Cần nhiều data, chậm |

---

## 10. CÂU HỎI PHỎNG VẤN HAY GẶP

**Q: Overfitting là gì? Cách xử lý?**
→ Mô hình học quá tốt trên train nhưng kém trên dữ liệu mới. Xử lý: regularization (L1/L2), dropout, early stopping, thêm dữ liệu, cross-validation.

**Q: Precision vs Recall — khi nào ưu tiên cái nào?**
→ Precision: khi false positive đắt (lọc spam → báo nhầm email quan trọng). Recall: khi false negative nguy hiểm (phát hiện ung thư → bỏ sót bệnh nhân).

**Q: Random Forest vs XGBoost?**
→ RF: bagging, train song song, giảm variance, ít tune. XGBoost: boosting, train tuần tự sửa lỗi, giảm bias, thường chính xác hơn nhưng cần tune nhiều hơn.

**Q: Tại sao cần feature scaling?**
→ Thuật toán dựa trên khoảng cách (KNN, SVM, Neural Network) bị feature giá trị lớn chi phối. Scaling đưa về cùng thang đo.

**Q: Supervised vs Unsupervised?**
→ Supervised: có nhãn, học dự đoán (phân loại, hồi quy). Unsupervised: không nhãn, tự tìm cấu trúc (phân cụm, giảm chiều).

**Q: Bias-Variance Tradeoff?**
→ Mô hình đơn giản: high bias, low variance (underfit). Phức tạp: low bias, high variance (overfit). Mục tiêu: tìm điểm cân bằng cho tổng lỗi thấp nhất.

---

> 💡 **Mẹo ôn thi**: Với vị trí DA, tập trung nhớ **khái niệm, khi nào dùng thuật toán nào, cách đánh giá mô hình**. Không cần nhớ công thức toán hay code chi tiết — đó là việc tra docs khi cần.
