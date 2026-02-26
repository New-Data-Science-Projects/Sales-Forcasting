# Mô tả Dự án PoC – Dự báo Doanh số với XAI

## 1. Tổng quan

Đây là một dự án **Proof of Concept (PoC)** mô phỏng bài toán thực tế trong bán lẻ: dự báo doanh số bán hàng theo ngày cho từng sản phẩm tại từng cửa hàng, sử dụng dữ liệu lịch sử bán hàng kết hợp với dữ liệu thời tiết.

Mục tiêu chính không chỉ là xây dựng mô hình có độ chính xác tốt, mà còn phải **giải thích được** cách mô hình ra quyết định bằng các kỹ thuật **Explainable AI (XAI)**, giúp nhóm nghiệp vụ có thể tin tưởng và sử dụng kết quả.

Thông qua dự án này, người học sẽ trải nghiệm đầy đủ vòng đời của một PoC trong doanh nghiệp: từ khám phá dữ liệu, xây dựng đặc trưng, huấn luyện mô hình, đánh giá đến trình bày kết quả và khuyến nghị kinh doanh.

---

## 2. Mục tiêu của PoC

- Xây dựng mô hình dự báo doanh số theo:
  - Cửa hàng (`store`)
  - Sản phẩm (`item`)
  - Ngày (`date`)
- Phân tích mối quan hệ giữa:
  - Các đặc trưng thời gian (ngày, tuần, tháng, mùa, ngày nghỉ,…)
  - Đặc trưng lịch sử bán hàng (lag, rolling mean/std,…)
  - Đặc trưng cửa hàng và sản phẩm
  - Đặc trưng thời tiết (nhiệt độ, mùa khô/mưa,…)
- Sử dụng **XAI (SHAP)** để:
  - Hiểu đặc trưng nào quan trọng nhất.
  - Giải thích tại sao một ngày cụ thể lại có dự báo cao/thấp.
- Đưa ra insight có giá trị cho:
  - Lập kế hoạch tồn kho.
  - Bố trí nhân sự.
  - Lên kế hoạch khuyến mãi & marketing.

---

## 3. Dữ liệu sử dụng

### 3.1. Dữ liệu bán hàng (Sales)

Hai file dữ liệu chính:

- `2016_sales.csv`
- `2017_sales.csv`

Các cột quan trọng:

- `date`: Ngày bán hàng
- `province`, `store_id`, `store_name`: Thông tin cửa hàng
- `category`, `item_id`, `item_name`: Thông tin sản phẩm
- `sales`: Số lượng bán được trong ngày

### 3.2. Dữ liệu thời tiết (Weather)

File:

- `weather_data.csv`

Các cột quan trọng:

- `date`: Ngày ghi nhận thời tiết
- `city`: Thành phố / khu vực
- `temperature`: Nhiệt độ
- `humidity`: Độ ẩm
- `season`: Mùa (ví dụ: dry/wet, summer/winter,…)

Sau khi tiền xử lý, dữ liệu sẽ được gộp lại thành một bảng chung phục vụ cho việc huấn luyện mô hình.

---

## 4. Pha 1 – Xây dựng PoC

### 4.1. Tích hợp & làm sạch dữ liệu

- Chuẩn hóa kiểu dữ liệu (datetime, numeric, category,…).
- Xử lý giá trị thiếu (missing values) bằng:
  - Loại bỏ (drop) trong những trường hợp an toàn.
  - Hoặc điền (impute) dựa trên thống kê / logic kinh doanh.
- Xử lý outlier trong cột `sales` (do lỗi nhập liệu hoặc sự kiện bất thường).
- Gộp dữ liệu bán hàng và thời tiết theo:
  - `date` + `province` / `city`

### 4.2. Khám phá dữ liệu (EDA)

- Vẽ chuỗi thời gian doanh số:
  - Theo ngày, tuần, tháng.
  - Theo từng cửa hàng, từng nhóm sản phẩm.
- Kiểm tra các pattern:
  - Mùa vụ (seasonality).
  - Ngày cuối tuần / ngày lễ.
  - Sản phẩm bán chạy / bán chậm.
- Phân tích tương quan giữa:
  - Nhiệt độ, độ ẩm, mùa.
  - Doanh số theo thời gian.

### 4.3. Xây dựng đặc trưng (Feature Engineering)

Nhóm đặc trưng chính:

- **Đặc trưng thời gian**
  - `day_of_week`, `day_of_month`, `month`, `is_weekend`, `is_holiday`,…
- **Đặc trưng lịch sử bán hàng**
  - Lag doanh số các ngày trước (1, 7, 14, 21, 28 ngày,…).
  - Rolling mean/min/max/std trong các cửa sổ (7, 14, 28 ngày,…).
  - Exponentially weighted moving average (EWMA) cho xu hướng gần.
- **Đặc trưng cửa hàng & sản phẩm**
  - Doanh số trung bình / tổng theo:
    - Cửa hàng trong 7 ngày gần nhất.
    - Sản phẩm trong 7 ngày gần nhất.
  - Mã hóa (encoding) `store_id`, `item_id`.
- **Đặc trưng thời tiết**
  - Phân loại mức nhiệt độ (`temp_category`).
  - Phân loại độ ẩm (`humidity_level`).
  - Cờ (flag) cho mùa (`season_wet`, `season_winter`,…).

Kết quả cuối cùng là một bảng đặc trưng giàu thông tin dùng để huấn luyện mô hình.

### 4.4. Mô hình hóa

- Xây dựng baseline với:
  - Mô hình đơn giản hoặc LightGBM trên tập đặc trưng cơ bản.
- Xây dựng mô hình cải tiến:
  - LightGBM trên full feature set (bao gồm tất cả các đặc trưng thời gian, lịch sử, cửa hàng, sản phẩm, thời tiết).
- Chia tập train/test theo thời gian:
  - Tránh rò rỉ dữ liệu tương lai (data leakage).
- Đánh giá bằng các chỉ số:
  - MAE (Mean Absolute Error)
  - RMSE (Root Mean Squared Error)
  - WAPE (Weighted Absolute Percentage Error)
  - Có thể thêm cross-validation theo time series nếu cần.
- Tối ưu siêu tham số (hyperparameter tuning) bằng **Optuna**.

### 4.5. Explainable AI (XAI)

Sử dụng **SHAP (SHapley Additive exPlanations)** để:

- Đo tầm quan trọng của từng đặc trưng ở mức:
  - **Toàn cục (global)**: đặc trưng nào quan trọng nhất với mô hình?
  - **Cục bộ (local)**: tại sao mô hình dự báo cao/thấp cho một ngày/cửa hàng/sản phẩm cụ thể?
- Phân tích:
  - Ảnh hưởng của lịch sử bán hàng ngắn hạn so với dài hạn.
  - Khác biệt giữa các cửa hàng.
  - Mức độ ảnh hưởng thực tế của thời tiết.
- Vẽ các biểu đồ:
  - Summary plot.
  - Feature importance.
  - Dependency plot cho các đặc trưng quan trọng (ví dụ: `item_mean_7d`).

Chi tiết xem trong file: `docs/shap_analysis_summary_report.md`.

### 4.6. Báo cáo & khuyến nghị

Trong pha PoC, trọng tâm là:

- **Không chỉ trả lời “mô hình có tốt không?”**, mà còn:
  - Mô hình giải thích được đến mức nào?
  - Insight có hữu ích cho nghiệp vụ không?
- Cuối cùng, nhóm sẽ:
  - Tóm tắt độ chính xác của mô hình.
  - Trình bày các yếu tố chính ảnh hưởng tới doanh số.
  - Đề xuất:
    - Có nên đưa mô hình vào giai đoạn triển khai mở rộng (Production/Phase 2) hay không.
    - Cần thêm dữ liệu/đặc trưng gì để cải thiện.

---

## 5. Kết quả học tập (Learning Outcomes)

Sau khi hoàn thành dự án PoC này, người học có thể:

- Hiểu quy trình **end-to-end** của một dự án PoC trong doanh nghiệp:
  - Thu thập – khám phá – xử lý dữ liệu.
  - Xây dựng đặc trưng.
  - Huấn luyện & tinh chỉnh mô hình.
  - Giải thích kết quả và trình bày cho stakeholder.
- Thành thạo hơn trong:
  - Xử lý và phân tích dữ liệu chuỗi thời gian.
  - Thiết kế đặc trưng cho bài toán forecasting.
  - Sử dụng LightGBM và Optuna cho bài toán dự báo.
  - Ứng dụng SHAP để giải thích mô hình.
- Nâng cao kỹ năng:
  - Viết báo cáo kỹ thuật.
  - Chuyển ngôn ngữ kỹ thuật sang ngôn ngữ kinh doanh.
  - Đề xuất khuyến nghị có thể hành động (actionable recommendations).

