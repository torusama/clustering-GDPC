# Đồ án Tiền xử lý dữ liệu (Data Preprocessing)

Đồ án cá nhân nghiên cứu, triển khai và đánh giá các kỹ thuật tiền xử lý trên hai loại dữ liệu: **chuỗi thời gian tài chính** và **ảnh X-quang ngực**. Nội dung bám sát đề bài `preprocessing.md` của môn Khai thác dữ liệu (Data Mining), bao gồm cả các yêu cầu nâng cao.

## 1. Thông tin đồ án

| Thông tin | Nội dung |
| --- | --- |
| Sinh viên | [Họ và tên] |
| MSSV | [Mã số sinh viên] |
| Môn học | Khai thác dữ liệu (Data Mining) |
| Ngôn ngữ triển khai | Python |
| Phần (a) | Dữ liệu chuỗi thời gian tài chính |
| Phần (b) | Dữ liệu ảnh số – X-quang ngực |
| Báo cáo | Jupyter Book trên GitHub Pages |
| Link Jupyter Book | Chưa xuất bản – cập nhật sau khi triển khai |
| Trạng thái | Kế hoạch triển khai; chưa có kết quả thực nghiệm |

## 2. Mục tiêu và phạm vi

Đồ án tập trung vào phân tích thống kê dữ liệu trước khi xử lý (EDA), xây dựng pipeline tiền xử lý có thể tái sử dụng, so sánh các kỹ thuật thông qua kiểm định thống kê và chỉ số định lượng, đồng thời đánh giá ảnh hưởng của tiền xử lý bằng các mô hình học máy đơn giản.

- Phần chuỗi thời gian: phân tích dữ liệu chứng khoán, phát hiện dị thường và dự báo giá đóng cửa phiên tiếp theo.
- Phần ảnh: phân tích đặc điểm ảnh X-quang và đánh giá tác động của tiền xử lý đến khả năng phân loại nhãn ảnh.
- Thực hiện toàn bộ nội dung bắt buộc và các mục nâng cao tương ứng với hai loại dữ liệu đã chọn.
- Không triển khai ứng dụng giao dịch, hệ thống chẩn đoán y khoa, giao diện web riêng hoặc mô hình Deep Learning. Jupyter Book là sản phẩm báo cáo của đồ án.

---

## 3. Phần (a) – Dữ liệu chuỗi thời gian tài chính (30%)

### 3.1. Lựa chọn tập dữ liệu

**Dataset:** New York Stock Exchange (NYSE).

**Nguồn:** [New York Stock Exchange – Kaggle](https://www.kaggle.com/datasets/dgawlik/nyse)

Đề bài yêu cầu dữ liệu có tần suất theo ngày hoặc giờ, kéo dài ít nhất hai năm. Dataset NYSE cung cấp dữ liệu giao dịch hằng ngày, phần lớn trong giai đoạn 2010–2016, phù hợp với hướng thị trường chứng khoán được gợi ý trong đề.

Chỉ sử dụng `prices-split-adjusted.csv`, chứa giá đã điều chỉnh theo các đợt chia tách cổ phiếu. Không sử dụng dữ liệu tài chính doanh nghiệp hoặc toàn bộ danh sách cổ phiếu trong bộ dữ liệu.

| Mã | Công ty | Mục đích sử dụng |
| --- | --- | --- |
| V | Visa | Chuỗi chính cho EDA, tiền xử lý và dự báo |
| MA | Mastercard | Phân tích Granger Causality |
| AXP | American Express | Phân tích Granger Causality |
| JPM | JPMorgan Chase | Phân tích Granger Causality |

**Các trường dữ liệu sử dụng:** `date` (ngày giao dịch), `symbol` (mã cổ phiếu), `close` (giá đóng cửa). Các cột khác không thuộc phạm vi phân tích chính.

**Phạm vi:** dự kiến sử dụng dữ liệu giai đoạn 2010–2016 của bốn mã trên. Sau khi tải, kiểm tra ngày bắt đầu, ngày kết thúc, số phiên, giá trị thiếu và khoảng thời gian chung; xác nhận mỗi chuỗi sử dụng đáp ứng yêu cầu về độ dài. Chuỗi được sắp xếp theo phiên giao dịch; cuối tuần và ngày thị trường đóng cửa không được xem là dữ liệu thiếu.

**Bài toán:** dự báo giá đóng cửa của Visa tại phiên giao dịch tiếp theo bằng Random Forest Regressor. Kết quả chỉ phục vụ đánh giá tiền xử lý trên dữ liệu lịch sử, không phục vụ quyết định đầu tư.

### 3.2. Phân tích khám phá chuỗi thời gian (EDA)

Thực hiện trước khi tiền xử lý để hiểu cấu trúc và đặc điểm thống kê của chuỗi giá Visa.

| Yêu cầu của đề | Phương án triển khai | Kết quả cần có |
| --- | --- | --- |
| Time Plot | Vẽ giá đóng cửa theo thời gian; nhận diện xu hướng (trend), mùa vụ (seasonality) và nhiễu (noise). Không mặc định chuỗi giá phải có mùa vụ. | Biểu đồ và nhận xét đặc điểm chuỗi. |
| ACF và PACF | Tính tự tương quan và tự tương quan riêng phần. Chọn lag tối đa theo gợi ý `floor(min(10 × log10(n), n/5))`, với `n` là số phiên hợp lệ. | Hai biểu đồ, khoảng tin cậy và nhận xét các lag đáng chú ý. |
| Rolling Statistics | Tính rolling mean và rolling standard deviation với cửa sổ 5, 21, 63 phiên, xấp xỉ một tuần, một tháng và ba tháng giao dịch. | Biểu đồ thống kê trượt, nhận xét sơ bộ về tính dừng. |

### 3.3. Các kỹ thuật tiền xử lý nâng cao

#### 3.3.1. Xử lý khoảng trống và so sánh nội suy

**Yêu cầu:** Tạo 5% giá trị thiếu nhân tạo theo cơ chế MCAR; đánh giá khả năng khôi phục của từng phương pháp.

**Triển khai:** chọn ngẫu nhiên 5% điểm giá đã quan sát với seed cố định. Lưu giá thật và vị trí bị che để các phương pháp sử dụng cùng một tập kiểm tra. So sánh bốn nhóm phương pháp với năm cấu hình:

1. Forward Fill: sử dụng giá trị quan sát gần nhất ở quá khứ.
2. Linear Interpolation: nội suy tuyến tính.
3. Polynomial Interpolation bậc 2 và bậc 3: thử cả hai bậc đa thức.
4. Cubic Spline Interpolation: nội suy spline bậc ba.

**Đánh giá:** MAE và RMSE chỉ trên các điểm bị che; Ljung-Box test để kiểm tra tự tương quan của phần dư, ghi rõ cách xử lý các vị trí không liên tiếp và độ trễ kiểm định. Báo cáo số điểm khôi phục thành công và cách xử lý khoảng trống ở biên để so sánh công bằng.

**Kết quả:** bảng MAE, RMSE, thống kê kiểm định và p-value; biểu đồ đối chiếu dữ liệu gốc với dữ liệu khôi phục; giải thích lựa chọn phương pháp phù hợp. Thí nghiệm nội suy hai phía dùng cho phân tích hồi cứu, không được dùng dữ liệu tương lai để tạo đặc trưng dự báo.

#### 3.3.2. Kiểm định và xử lý tính dừng (Stationarity)

**Yêu cầu:** Kiểm định chuỗi gốc và thực hiện biến đổi khi chuỗi không dừng.

**Triển khai:** chạy ADF (Augmented Dickey-Fuller), KPSS và Phillips-Perron (PP). Khi cần, thử sai phân bậc 1, sai phân bậc 2, logarithm và Box-Cox; kiểm định lại sau từng phép biến đổi. Các phép log và Box-Cox được áp dụng trên dữ liệu dương phù hợp; không mặc định mọi biến đổi đều tạo ra chuỗi dừng.

**Đánh giá:** sử dụng mức ý nghĩa dự kiến 0,05; diễn giải đúng giả thuyết của từng kiểm định (KPSS có giả thuyết gốc về tính dừng, khác với ADF và PP). Nếu kết quả mâu thuẫn, giải thích thay vì kết luận chỉ dựa trên một p-value.

**Kết quả:** bảng thống kê kiểm định và p-value của chuỗi gốc và từng chuỗi biến đổi, biểu đồ trước/sau, kết luận phép biến đổi phù hợp. Vẽ lại ACF/PACF trên chuỗi thích hợp để hỗ trợ lựa chọn lag.

#### 3.3.3. Phân rã chuỗi thời gian (Decomposition)

**Yêu cầu:** Áp dụng Additive, Multiplicative và STL; đánh giá tỉ lệ phương sai phần dư.

**Triển khai:** sử dụng `statsmodels` để phân rã chuỗi giá thành xu hướng, mùa vụ và phần dư. Xác định chu kỳ mùa vụ có cơ sở từ EDA và lịch giao dịch, thử nghiệm theo cùng chu kỳ khi so sánh. Không khẳng định thị trường có mùa vụ nếu kết quả không hỗ trợ.

| Phương pháp | Nội dung |
| --- | --- |
| Additive | Biểu diễn chuỗi bằng tổng xu hướng, mùa vụ và phần dư. |
| Multiplicative | Biểu diễn chuỗi dương bằng tích các thành phần. |
| STL | Tách xu hướng và mùa vụ bằng LOESS. |

**Đánh giá:** Residual Variance Ratio (tỉ lệ phương sai phần dư so với chuỗi gốc); quy đổi sai số tái tạo về cùng thang giá khi cần so sánh các mô hình cộng và nhân.

**Kết quả:** biểu đồ các thành phần, bảng so sánh phương pháp và chuỗi đã loại mùa vụ/xu hướng để phục vụ phát hiện dị thường, trích xuất đặc trưng. Phần phân rã toàn chuỗi chỉ dùng cho phân tích hồi cứu; đặc trưng dự báo phải được tính từ lịch sử có sẵn tại thời điểm dự báo.

#### 3.3.4. Trích xuất đặc trưng thời gian (Feature Extraction)

**Yêu cầu:** Mã hóa chu kỳ thời gian, bổ sung ngày lễ và sự kiện đặc biệt, tính thống kê trượt ở nhiều cửa sổ.

**Triển khai:**

| Nhóm đặc trưng | Nội dung |
| --- | --- |
| Cyclical Features | Mã hóa ngày trong tuần và tháng trong năm bằng sin/cos theo chu kỳ tương ứng. |
| Holiday Features | Sử dụng `holidays` cho ngày lễ tại Mỹ; đối chiếu lịch giao dịch và tạo cờ phiên trước/sau ngày nghỉ. |
| Special Events | Tạo cờ sự kiện lịch đã biết trước, chẳng hạn phiên cuối tháng/quý; ghi rõ quy tắc xác định. |
| Rolling Features | Mean, standard deviation, min, max, skewness với cửa sổ 5, 21, 63 phiên. |

**Kết quả:** bảng dữ liệu chứa các đặc trưng mới, giải thích ý nghĩa của từng nhóm và xử lý các giá trị thiếu phát sinh ở đầu chuỗi.

#### 3.3.5. Phát hiện dị thường (Anomaly Detection)

**Yêu cầu:** Cài đặt, so sánh ba phương pháp và đánh dấu các điểm bất thường trên biểu đồ thời gian.

| Phương pháp | Cách triển khai |
| --- | --- |
| Z-score | Tính độ lệch chuẩn hóa trên chuỗi đã loại mùa vụ; đánh dấu điểm vượt ngưỡng. |
| Isolation Forest | Phát hiện các cửa sổ có đặc trưng bất thường thông qua mô hình Isolation Forest. |
| STL Residual Thresholding | Phát hiện điểm có phần dư STL vượt ngưỡng. Chọn phương pháp này thay cho Autoencoder 1D để giới hạn tài nguyên. |

**Kết quả:** biểu đồ đánh dấu các phiên bất thường, số lượng và tỉ lệ điểm được phát hiện, so sánh mức độ trùng khớp giữa ba phương pháp và thảo luận nguyên nhân có thể. Không tự động loại bỏ mọi biến động bất thường vì chúng có thể là biến động thực của thị trường. Không báo cáo độ chính xác phát hiện dị thường nếu không có nhãn đối chứng.

#### 3.3.6. Xây dựng ma trận đặc trưng và dự báo

**Yêu cầu:** Xây dựng feature matrix đầy đủ; huấn luyện Random Forest Regressor dự báo giá trị tại `t+1`; đánh giá MAE và RMSE trên tập test.

**Triển khai:**

1. Sử dụng PACF trên dữ liệu huấn luyện để xác định số lag `p`, tạo các đặc trưng từ `t-1` đến `t-p` và giá hiện tại nếu đã biết tại thời điểm dự báo.
2. Kết hợp lag, rolling statistics, calendar features và đặc trưng phân rã được tính từ dữ liệu có sẵn đến thời điểm `t`.
3. Tạo biến mục tiêu là giá đóng cửa ở phiên kế tiếp `t+1`.
4. Chia train/test theo thứ tự thời gian (dự kiến 80/20), không xáo trộn và không đưa nhãn tương lai vào dữ liệu huấn luyện.
5. Huấn luyện `RandomForestRegressor` của scikit-learn; chỉ dùng tập train để xác định các tham số học được và lựa chọn đặc trưng.
6. Dự báo tập test, tính MAE và RMSE theo đơn vị giá gốc.

**Kết quả:** feature matrix, cấu hình mô hình, bảng MAE/RMSE, biểu đồ giá thực tế và dự báo; nhận xét chất lượng pipeline. Có thể dùng dự báo ngây thơ lấy giá phiên hiện tại làm mốc đối chiếu; đây là bổ sung nhỏ, không phải yêu cầu riêng của đề.

#### 3.3.7. [Nâng cao] Granger Causality

**Yêu cầu:** Kiểm định quan hệ Granger trên dữ liệu đa biến, lập ma trận nhân quả và đồ thị có hướng.

**Triển khai:** đồng bộ chuỗi giá đóng cửa của V, MA, AXP, JPM theo ngày giao dịch chung; kiểm định và xử lý tính dừng cho từng chuỗi (ưu tiên log-return nếu phù hợp). Chọn số lag có cơ sở, kiểm định từng cặp theo cả hai chiều với `statsmodels`. Có 12 quan hệ có hướng giữa bốn mã. Ghi rõ mức ý nghĩa và cách xử lý kiểm định nhiều lần (dự kiến Benjamini–Hochberg).

**Kết quả:** ma trận p-value với hàng là biến cung cấp thông tin và cột là biến được dự báo; đồ thị có hướng từ các quan hệ có ý nghĩa thống kê; thảo luận kết quả. Quan hệ Granger chỉ phản ánh thông tin dự báo trong mô hình kiểm định, không chứng minh quan hệ nhân quả thực tế.

---

## 4. Phần (b) – Dữ liệu ảnh số: X-quang ngực (30%)

### 4.1. Lựa chọn tập dữ liệu

**Dataset:** NIH Chest X-rays (ChestX-ray14).

**Nguồn:** [NIH Chest X-rays – Kaggle](https://www.kaggle.com/datasets/nih-chest-xrays/data)

Bộ dữ liệu gốc gồm khoảng 112.120 ảnh X-quang ngực của 30.805 bệnh nhân, với 14 nhãn bất thường và nhãn `No Finding`. Ảnh được cung cấp dưới dạng PNG, kích thước phổ biến 1024 × 1024. Đây là bộ dữ liệu thuộc nhóm ảnh X-quang được đề bài gợi ý, đáp ứng điều kiện ít nhất 5.000 ảnh và 5 lớp ở quy mô bộ gốc.

**Phạm vi triển khai:** dự kiến chọn khoảng 7.000 ảnh từ sáu lớp dưới đây, chỉ giữ ảnh có duy nhất một nhãn thuộc danh sách. Loại bỏ các ảnh đồng thời mang nhiều nhãn, không gộp chúng thành một lớp tùy ý.

| Nhãn gốc | Mô tả |
| --- | --- |
| No Finding | Không có bất thường được gắn nhãn |
| Atelectasis | Xẹp phổi |
| Cardiomegaly | Tim to |
| Effusion | Tràn dịch màng phổi |
| Infiltration | Hình ảnh thâm nhiễm |
| Nodule | Nốt phổi |

**Thông tin cần dùng từ metadata:** `Image Index`, `Finding Labels`, `Patient ID`; đối chiếu tên file metadata và đường dẫn ảnh của phiên bản Kaggle khi tải.

**Chuẩn bị dữ liệu:**

- Đọc metadata trước, lọc nhãn và kiểm kê ảnh hợp lệ. Tạo danh sách ảnh cần lấy với seed cố định.
- Lấy mẫu theo tỉ lệ lớp trong tập đã lọc, không cân bằng lớp ngay từ đầu. Điều chỉnh cỡ mẫu nếu cần để sau khi xử lý trùng lặp vẫn có ít nhất 5.000 ảnh và 5 lớp; mục tiêu giữ đủ sáu lớp.
- Nếu Kaggle cung cấp dữ liệu theo các gói lớn, tải hoặc xử lý từng gói, chỉ giữ ảnh thuộc danh sách cần dùng. Giới hạn 7.000 ảnh là phạm vi phân tích, không bảo đảm dung lượng tải từ nguồn cũng giảm tương ứng.
- Chia train/test theo nhóm `Patient ID` (dự kiến 80/20), tránh ảnh của cùng bệnh nhân hoặc ảnh trùng xuất hiện ở hai tập. Lưu danh sách ảnh và phân chia để tái lập thí nghiệm.

**Bài toán:** phân loại sáu nhãn ảnh X-quang bằng Logistic Regression nhằm đánh giá tác động của tiền xử lý. Nhãn của dataset không phải chẩn đoán được xác minh độc lập; kết quả không sử dụng cho chẩn đoán lâm sàng.

> **Lưu ý về dữ liệu mức xám:** X-quang không có thông tin màu thực. Việc biểu diễn ảnh qua RGB, HSV và LAB không tạo thêm thông tin màu, và hai cách Z-score có thể tương đương trên ảnh một kênh hoặc RGB lặp kênh. Vẫn thực hiện và phân tích các thí nghiệm theo đề, đồng thời xác nhận với giảng viên cách đánh giá mục không gian màu trên ảnh mức xám.

### 4.2. Phân tích thống kê tập dữ liệu (EDA)

Thực hiện trên toàn bộ tập ảnh được chọn trước các biến đổi tiền xử lý chính.

| Yêu cầu của đề | Phương án triển khai | Kết quả cần có |
| --- | --- | --- |
| Phân phối pixel | Tích lũy histogram mức xám trên toàn tập và vẽ KDE; nếu biểu diễn RGB, báo cáo từng kênh và giải thích chúng trùng nhau. Có thể tính KDE từ histogram hoặc mẫu pixel đại diện, ghi rõ phương pháp. | Histogram, KDE và phân tích phân phối. |
| Class Imbalance | Thống kê số lượng, tỉ lệ mỗi lớp và tỉ số giữa lớp lớn nhất với lớp nhỏ nhất; kiểm tra ngưỡng gấp 3 lần. | Bảng, biểu đồ phân phối lớp và nhận xét. |
| Ảnh trùng/gần trùng | Tính pHash trên toàn tập, so sánh khoảng cách Hamming, chọn ngưỡng và kiểm tra các trường hợp gần trùng trước khi loại bỏ. | Số ảnh trùng, tỉ lệ trùng, cách xử lý và số ảnh còn lại. |
| Độ sáng và tương phản | Tính mean intensity và standard deviation của từng ảnh, tổng hợp theo lớp. | Boxplot độ sáng và độ tương phản theo lớp. |

### 4.3. Các kỹ thuật tiền xử lý và phân tích tác động

#### 4.3.1. Thay đổi kích thước và chất lượng ảnh (Resize)

**Yêu cầu:** thử ít nhất ba kích thước và đánh giá mức mất thông tin bằng SSIM, PSNR.

**Triển khai:** resize ảnh về 32 × 32, 64 × 64 và 128 × 128 với cùng quy tắc nội suy. Để tính SSIM/PSNR, đưa ảnh đã resize trở lại kích thước ảnh gốc và so sánh với ảnh tham chiếu từ Kaggle trên cùng thang cường độ.

**Kết quả:** bảng SSIM, PSNR theo kích thước; đường cong SSIM theo kích thước; ảnh minh họa và kết luận lựa chọn kích thước phù hợp dựa trên chất lượng ảnh, hiệu năng phân loại và tài nguyên.

#### 4.3.2. Chuyển đổi không gian màu (Color Space)

**Yêu cầu:** so sánh ít nhất ba không gian màu bằng PCA với 50 thành phần.

| Không gian màu | Phương án |
| --- | --- |
| Grayscale | Sử dụng một kênh mức xám gốc. |
| RGB | Biểu diễn mức xám bằng ba kênh giống nhau. |
| HSV | Chuyển từ RGB sang Hue, Saturation, Value. |
| LAB | Chuyển từ RGB sang Lightness, a, b. |

**Đánh giá:** giữ cùng danh sách ảnh và kích thước; quy định nhất quán thang giá trị của các kênh, tính explained variance tích lũy của 50 thành phần PCA và so sánh hiệu quả phân loại. Xử lý các kênh có phương sai bằng 0 để tránh lỗi số học.

**Kết quả:** bảng explained variance, hình ảnh các kênh và nhận xét về độ dư thừa của biểu diễn màu. Tỉ lệ phương sai giải thích không tự chứng minh thông tin phân loại tốt hơn; cần đối chiếu với Ablation Study.

#### 4.3.3. Chuẩn hóa ảnh (Normalization)

**Yêu cầu:** so sánh bốn phương pháp chuẩn hóa và đánh giá thay đổi phân phối bằng KS test.

1. Min-Max đưa pixel về khoảng [0, 1].
2. Min-Max đưa pixel về khoảng [-1, 1].
3. Global Z-score: chuẩn hóa bằng trung bình và độ lệch chuẩn toàn tập train.
4. Per-channel Z-score: chuẩn hóa theo trung bình và độ lệch chuẩn từng kênh của tập train.

**Đánh giá:** tính KS statistic và p-value trên mẫu pixel theo quy tắc cố định; trực quan hóa phân phối trước/sau. Giải thích hạn chế của p-value do pixel có tương quan không gian và hai tập trước/sau có liên hệ, không coi p-value nhỏ là bằng chứng chất lượng ảnh tăng. Các kênh hằng số được xử lý an toàn; không tính thống kê chuẩn hóa bằng tập test.

**Kết quả:** bảng KS statistic, p-value, biểu đồ phân phối và nhận xét; so sánh kết quả phân loại. Giữ kết quả trung thực nếu các phương pháp tương đương đối với ảnh mức xám.

#### 4.3.4. Tăng cường dữ liệu (Data Augmentation)

**Yêu cầu:** xây dựng pipeline ít nhất năm phép biến đổi và so sánh phân phối đặc trưng bằng t-SNE.

| Phép biến đổi | Cách thực hiện |
| --- | --- |
| Rotation | Xoay nhẹ ảnh, giới hạn góc để giữ cấu trúc giải phẫu. |
| Random Crop | Cắt nhẹ và đưa về kích thước chung; tránh mất vùng ngực quan trọng. |
| Gaussian Noise | Bổ sung nhiễu Gaussian cường độ thấp. |
| Brightness Adjustment | Điều chỉnh độ sáng trong khoảng nhỏ. |
| Contrast Adjustment | Điều chỉnh độ tương phản trong khoảng nhỏ. |

Không lật dọc/ngang trong phạm vi thí nghiệm này để tránh thay đổi định hướng giải phẫu. Tạo tối đa một phiên bản tăng cường cho mỗi ảnh train trong thí nghiệm chính, giữ seed và cấu hình cố định. Không augmentation tập test.

**Kết quả:** ví dụ trước/sau biến đổi, trực quan hóa t-SNE của ảnh gốc và ảnh augmentation trong cùng một phép chiếu trên đặc trưng nhất quán; so sánh kết quả phân loại trước/sau và nhận xét sự thay đổi phân phối.

#### 4.3.5. Ablation Study – Đánh giá tác động của các kỹ thuật

**Yêu cầu:** dùng k-NN hoặc Logistic Regression để so sánh kết quả phân loại trước và sau khi áp dụng từng kỹ thuật.

**Mô hình lựa chọn:** `LogisticRegression` của scikit-learn. Mô hình cơ sở (baseline) vẫn cần đọc ảnh, chuyển về cùng kích thước, biểu diễn mức xám, flatten thành vector số và chia train/test. Dự kiến dùng ảnh mức xám 64 × 64, không áp dụng chuẩn hóa bổ sung hoặc augmentation trong baseline.

| Nhóm thí nghiệm | Các cấu hình so sánh |
| --- | --- |
| Baseline | Tiền xử lý tối thiểu, dùng làm mốc. |
| Resize | 32 × 32, 64 × 64, 128 × 128. |
| Color Space | Grayscale, RGB, HSV, LAB. |
| Normalization | Không chuẩn hóa bổ sung và bốn cách chuẩn hóa. |
| Augmentation | Không tăng cường và có tăng cường. |

Giữ cùng danh sách train/test theo bệnh nhân, cùng cách đánh giá, cùng thiết lập mô hình phù hợp; chỉ thay đổi kỹ thuật đang khảo sát. Giữ lại và báo cáo cảnh báo hội tụ nếu có, không diễn giải kết quả mô hình chưa hội tụ như một so sánh đáng tin cậy. Có thể sử dụng thêm cấu hình tổng hợp sau khi đã phân tích từng kỹ thuật, nhưng không thay thế các đối chứng riêng.

**Kết quả:** Accuracy, F1-macro (chỉ số lựa chọn của đồ án để phản ánh mất cân bằng lớp), bảng và biểu đồ so sánh; nhận xét kỹ thuật nào cải thiện, không cải thiện hoặc làm giảm hiệu năng. Không yêu cầu đạt một mức accuracy cố định; mục tiêu là đánh giá tác động của tiền xử lý.

#### 4.3.6. [Nâng cao] Phân tích PCA trên không gian đặc trưng ảnh

**Yêu cầu:** chiếu toàn bộ tập dữ liệu lên PCA, xác định số thành phần giải thích 90%, 95%, 99% phương sai và trực quan hóa PCA/t-SNE.

**Triển khai:** sử dụng ảnh mức xám đã resize (dự kiến 64 × 64) và flatten; thực hiện PCA trên toàn bộ tập ảnh đã scope cho phân tích mô tả. Sử dụng đủ số thành phần để xác định được cả ba ngưỡng phương sai, không giới hạn ở 50 thành phần của thí nghiệm không gian màu. Nếu dùng PCA làm đầu vào mô hình, fit riêng trên tập train để tránh rò rỉ.

**Kết quả:** Scree Plot, biểu đồ phương sai tích lũy, số thành phần tương ứng với ba ngưỡng; PCA 2D và t-SNE 2D tô màu theo nhãn lớp; thảo luận khả năng phân tách. Nếu lấy mẫu cho t-SNE vì tài nguyên, ghi rõ số mẫu và cách lấy mẫu; phần PCA vẫn thực hiện trên toàn tập đã chọn.

#### 4.3.7. [Nâng cao] Phát hiện cạnh và phân tích đặc trưng cục bộ

**Yêu cầu:** sử dụng Sobel, Prewitt, Canny, mỗi phương pháp ít nhất hai bộ siêu tham số; tính Edge Density theo lớp và kiểm định ANOVA một chiều.

| Phương pháp | Hai cấu hình dự kiến |
| --- | --- |
| Sobel | Hai ngưỡng trên độ lớn gradient đã chuẩn hóa. |
| Prewitt | Hai ngưỡng trên độ lớn gradient đã chuẩn hóa. |
| Canny | Hai cặp ngưỡng thấp/cao khác nhau. |

Chốt các giá trị ngưỡng sau khi khảo sát dữ liệu train và lưu cấu hình để tái lập. Dùng cùng kích thước ảnh để so sánh; nhị phân hóa bản đồ cạnh rồi tính `Edge Density = số pixel cạnh / tổng số pixel`.

**Đánh giá:** thống kê Edge Density theo lớp, ANOVA một chiều cho từng cấu hình; báo cáo thống kê F và p-value, thảo luận các giả định của kiểm định (đặc biệt khi nhiều ảnh cùng bệnh nhân). Nếu kiểm tra thêm hiệu năng phân loại với đặc trưng cạnh/PCA, giữ nguyên cách chia tập và đối chứng tương ứng.

**Kết quả:** ảnh minh họa phát hiện cạnh, bảng và biểu đồ Edge Density, kết luận về sự khác biệt giữa các lớp và giới hạn của thông tin cạnh.

---

## 5. Yêu cầu triển khai và giới hạn tài nguyên

### 5.1. Công nghệ sử dụng

| Thành phần | Thư viện/công cụ dự kiến |
| --- | --- |
| Môi trường | Python 3.11, Jupyter Notebook |
| Dữ liệu và tính toán | NumPy, Pandas |
| Trực quan hóa | Matplotlib, Seaborn |
| Thống kê và chuỗi thời gian | SciPy, Statsmodels |
| Xử lý ảnh | OpenCV, Pillow |
| Học máy và giảm chiều | Scikit-learn |
| Đặc trưng ngày lễ | holidays |
| Tải dữ liệu | Kaggle API hoặc thư viện tải Kaggle |
| Báo cáo | Jupyter Book, GitHub Pages |

Phillips-Perron không có sẵn trong `statsmodels` theo API thông dụng; có thể cần thư viện `arch`. Nếu dùng `arch`, `scikit-image` hoặc `ImageHash` cho PP, SSIM/PSNR hay pHash, cần xác nhận với giảng viên vì các thư viện này không được liệt kê rõ trong danh sách thư viện chung của đề. Ưu tiên triển khai pHash, SSIM/PSNR bằng NumPy/OpenCV nếu cần giới hạn dependency. Phiên bản cụ thể sẽ được khóa trong `requirements.txt` sau khi kiểm tra môi trường.

### 5.2. Quy tắc triển khai

- Code Python có tên biến/hàm rõ ràng, chú thích và tổ chức module dễ tái sử dụng. Markdown báo cáo bằng tiếng Việt.
- Mỗi kỹ thuật có ô Markdown giải thích lý thuyết và công thức nếu có, ô code cài đặt, kết quả in trực tiếp và ô Markdown phân tích.
- Mọi score, p-value và kết quả định lượng được lưu trong biến Python và in ra để kiểm tra lại. Không ghi số liệu giả định thành kết quả thực nghiệm.
- Dùng seed cố định (dự kiến 42), lưu manifest dataset, cấu hình, phân chia dữ liệu và phiên bản thư viện.
- Không đưa dữ liệu test hoặc thông tin tương lai vào quá trình học tham số tiền xử lý và lựa chọn mô hình.
- Không commit dữ liệu gốc dung lượng lớn hoặc thông tin xác thực Kaggle. Mã tải/lọc dữ liệu phải nằm trong notebook hoặc được notebook import từ module hỗ trợ.

### 5.3. Giới hạn tính toán

| Hạng mục | Phạm vi dự kiến |
| --- | --- |
| Cổ phiếu | 1 mã phân tích chính, 4 mã cho Granger |
| Ảnh | Khoảng 7.000 ảnh, 6 lớp; xác nhận sau lọc |
| Resize | 3 kích thước |
| Không gian màu | 4 biểu diễn |
| Chuẩn hóa | 4 phương pháp |
| Augmentation | 5 phép biến đổi; tối đa 1 ảnh tăng cường/ảnh train trong thí nghiệm chính |
| PCA | Ảnh đã resize; xử lý theo batch nếu cần, bảo đảm đủ thành phần để đạt ngưỡng 99% |
| t-SNE | Dự kiến khoảng 1.500 ảnh đại diện nếu cần lấy mẫu; dùng chung phép chiếu khi so sánh trước/sau |
| Mô hình đánh giá | Random Forest Regressor và Logistic Regression |

Các yêu cầu phân tích toàn tập vẫn được thực hiện trên toàn bộ tập dữ liệu đã scope. Ảnh kích thước lớn được đọc theo batch thay vì giữ toàn bộ trong RAM. Các giới hạn và tham số trên là quyết định triển khai, không phải các con số bắt buộc của giảng viên.

---

## 6. Cấu trúc repository dự kiến

```text
Data-Preprocessing/
├── README.md
├── requirements.txt
├── .gitignore
├── book/
│   ├── _config.yml
│   ├── _toc.yml
│   ├── index.md
│   ├── time_series/
│   │   ├── 01_eda.ipynb
│   │   ├── 02_interpolation.ipynb
│   │   ├── 03_stationarity.ipynb
│   │   ├── 04_decomposition.ipynb
│   │   ├── 05_feature_extraction.ipynb
│   │   ├── 06_anomaly_detection.ipynb
│   │   ├── 07_forecasting.ipynb
│   │   └── 08_granger_causality.ipynb
│   ├── image/
│   │   ├── 01_eda.ipynb
│   │   ├── 02_resizing.ipynb
│   │   ├── 03_color_space.ipynb
│   │   ├── 04_normalization.ipynb
│   │   ├── 05_augmentation.ipynb
│   │   ├── 06_ablation_study.ipynb
│   │   ├── 07_pca.ipynb
│   │   └── 08_edge_detection.ipynb
│   └── report.md
├── src/
│   ├── data_loader.py
│   ├── time_series/
│   └── image/
├── data/
│   ├── raw/                 # Không commit dữ liệu lớn
│   └── processed/
├── results/
│   ├── figures/
│   └── metrics/
└── .github/
    └── workflows/
        └── deploy.yml
```

Mỗi notebook là nguồn trực tiếp của một chương Jupyter Book. Notebook EDA của mỗi phần chứa phần giới thiệu, mã chuẩn bị dữ liệu hoặc import module tải/lọc dữ liệu. Các notebook còn lại phải có thể thực hiện lại bước chuẩn bị cần thiết hoặc nạp dữ liệu trung gian theo quy trình được mô tả, không phụ thuộc vào biến trong notebook khác.

Cấu trúc sử dụng Jupyter Book 1.x dự kiến để phù hợp với `_config.yml` và `_toc.yml`; có thể thay đổi công cụ dựng book nếu cần, miễn vẫn đáp ứng yêu cầu của đề.

## 7. Hướng dẫn cài đặt và chạy (dự kiến)

Các lệnh dưới đây là hướng dẫn cho cấu trúc dự kiến, chưa được kiểm thử vì mã nguồn và file môi trường chưa được triển khai.

### 7.1. Cài đặt môi trường

Sau khi clone repository cá nhân và mở terminal tại thư mục gốc:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Trên Linux/macOS, kích hoạt bằng `source .venv/bin/activate`. File `requirements.txt` dự kiến khai báo cả Jupyter Notebook, Jupyter Book 1.x (`jupyter-book>=1,<2` trong giai đoạn chọn phiên bản) và các thư viện xử lý, sau đó khóa phiên bản đã kiểm thử.

### 7.2. Tải và chuẩn bị dữ liệu

Cấu hình quyền truy cập Kaggle nếu API yêu cầu; không đưa khóa API vào GitHub. Có thể tải dữ liệu qua giao diện Kaggle hoặc qua API; mã chuẩn bị dữ liệu vẫn phải có trong các notebook hoặc module được import.

```bash
kaggle datasets files -d dgawlik/nyse
kaggle datasets download -d dgawlik/nyse -f prices-split-adjusted.csv -p data/raw/nyse
kaggle datasets files -d nih-chest-xrays/data
```

Với ảnh X-quang, kiểm tra danh sách file và metadata trước khi chọn phương án tải từng gói; không mặc định phải tải toàn bộ bộ ảnh. Mã chuẩn bị dự kiến sẽ lưu danh sách ảnh được chọn, đường dẫn, nhãn và phân chia tập. Nếu cần giải nén file tải xuống, thực hiện trong bước chuẩn bị dữ liệu.

### 7.3. Chạy notebook và build Jupyter Book

```bash
python -m notebook
```

Mở các notebook trong `book/`, kiểm tra chạy sạch các ô code, sau đó build từ thư mục gốc:

```bash
jupyter-book build book --all
```

Cấu hình thực thi notebook trong `_config.yml` để kiểm tra toàn bộ chương; không che giấu lỗi runtime. Với Jupyter Book 1.x, kết quả HTML dự kiến nằm tại `book/_build/html/`. Kiểm tra trang `index.html`, điều hướng, biểu đồ, bảng số liệu và các liên kết trước khi xuất bản.

### 7.4. Xuất bản GitHub Pages

Thiết lập GitHub Actions hoặc cơ chế triển khai GitHub Pages tương ứng, xuất bản thư mục HTML đã build. Có thể thực thi notebook và build trên máy/môi trường đủ tài nguyên trước khi triển khai HTML; không yêu cầu GitHub Actions tải lại toàn bộ bộ ảnh nếu không phù hợp giới hạn runner. Repository vẫn phải chứa nguồn book và hướng dẫn tái lập đầy đủ.

Cập nhật URL website thật vào mục thông tin đồ án sau khi xuất bản thành công.

---

## 8. Yêu cầu báo cáo Jupyter Book

Toàn bộ báo cáo trình bày trong một Jupyter Book duy nhất, gồm phần giới thiệu đồ án, các chương theo mục 3 (chuỗi thời gian), mục 4 (ảnh số), tổng kết so sánh, thảo luận hạn chế và tài liệu tham khảo.

Mỗi mục kỹ thuật cần có đủ:

1. Mục tiêu và lý thuyết, công thức toán nếu có.
2. Code triển khai, tham số và cách thiết kế thực nghiệm.
3. Kết quả định lượng, bảng số liệu và biểu đồ.
4. Phân tích, so sánh phương pháp và giải thích các kết quả, kể cả kết quả không cải thiện.

Các chương phải chạy sạch và build thành công. Báo cáo không yêu cầu PDF IEEE, Word hay PowerPoint riêng theo file đề đang sử dụng.

## 9. Tiêu chí chấm điểm

| Nội dung | Điểm |
| --- | ---: |
| **Phần (a) – Chuỗi thời gian** | **30%** |
| Kiểm định tính dừng và phân rã STL | 10% |
| Phát hiện dị thường và ma trận đặc trưng | 10% |
| Dự báo Random Forest Regressor và Granger Causality | 10% |
| **Phần (b) – Ảnh số** | **30%** |
| Phân tích thống kê, pHash, class imbalance và KS | 10% |
| Kỹ thuật tiền xử lý, SSIM và PSNR | 10% |
| Ablation Study, PCA và t-SNE | 10% |
| **Chất lượng mã nguồn và cuốn book** | **40%** |
| Mã sạch, có chú thích, tái lập được | 10% |
| Repository cá nhân, README và cấu trúc book | 10% |
| Báo cáo, so sánh và thảo luận | 20% |
| **Tổng** | **100%** |

Granger Causality, phân tích PCA chuyên sâu và phát hiện cạnh là các mục được ghi nâng cao trong đề. Kế hoạch thực hiện cả ba; Granger và PCA cũng xuất hiện trong rubric nên không bỏ qua. Bảng điểm trên phản ánh rubric của đề, không giả định số điểm thưởng vượt 100%.

## 10. Checklist hoàn thành và nộp bài

- [ ] Xác nhận scope hai dataset đáp ứng ngưỡng đề bài; hoàn thành mã tải và chuẩn bị dữ liệu.
- [ ] Hoàn thành EDA chuỗi thời gian, nội suy, tính dừng, phân rã, trích đặc trưng, dị thường, dự báo và Granger Causality.
- [ ] Hoàn thành EDA ảnh, pHash, resize, màu, chuẩn hóa, augmentation, Ablation Study, PCA và phát hiện cạnh.
- [ ] Hoàn thành toàn bộ bảng số liệu, biểu đồ và thảo luận trong notebook.
- [ ] Hoàn thiện repository cá nhân, README, requirements và lịch sử commit rõ ràng.
- [ ] Chạy lại tất cả notebook từ đầu đến cuối; kiểm tra Jupyter Book build thành công, không lỗi runtime.
- [ ] Xuất bản một Jupyter Book công khai trên GitHub Pages và kiểm tra link hoạt động ổn định.
- [ ] Nộp **duy nhất một link GitHub Pages của Jupyter Book trên Moodle**, không nộp link repository thay thế link website.
- [ ] Chuẩn bị giải thích lý thuyết, công thức, mã nguồn và kết quả trong phần vấn đáp cuối kỳ.

Theo đề, chương gặp lỗi runtime không được chấm; không chấp nhận nộp trễ. Được sử dụng AI để hỗ trợ code và phân tích nhưng sinh viên phải hiểu và chịu trách nhiệm về bài làm. Deadline cụ thể cần kiểm tra trên Moodle hoặc thông báo của giảng viên.

## 11. Nguồn dữ liệu và tài liệu tham khảo

- [New York Stock Exchange – Kaggle](https://www.kaggle.com/datasets/dgawlik/nyse).
- [NIH Chest X-rays – Kaggle](https://www.kaggle.com/datasets/nih-chest-xrays/data).
- Wang và cộng sự (2017), *ChestX-ray8: Hospital-scale Chest X-ray Database and Benchmarks on Weakly-Supervised Classification and Localization of Common Thorax Diseases*, CVPR. Ghi nhận NIH Clinical Center là đơn vị cung cấp dữ liệu.
- Đề bài Tiền xử lý dữ liệu (`preprocessing.md`) của môn Khai thác dữ liệu.
- Tài liệu thuật toán và thư viện sử dụng sẽ được bổ sung, trích dẫn tại các chương tương ứng trong Jupyter Book.
