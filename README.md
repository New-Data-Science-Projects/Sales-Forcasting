# Dự án Dự báo Doanh số với Explainable AI (XAI)
- **Author:** Nguyen Van Thuc, Pham Xuan Khang
- **Loại dự án:** Proof of Concept (PoC)
- **Ngăn xếp công nghệ:** Python, LightGBM, SHAP, Optuna, Streamlit

## Tổng quan

- **Sales Forecasting with Explainable AI (XAI)** là một PoC end-to-end mô phỏng bài toán dự báo doanh số bán lẻ ở cấp cửa hàng/sản phẩm theo ngày.
- Mục tiêu không chỉ là xây dựng mô hình dự báo có độ chính xác tốt, mà còn **giải thích được** lý do mô hình đưa ra dự đoán thông qua các kỹ thuật XAI (SHAP).
- Dự án sử dụng **LightGBM** làm mô hình chính, tối ưu tham số với **Optuna**, phân tích giải thích với **SHAP**, và triển khai giao diện khám phá kết quả bằng **Streamlit**.

## Chức năng chính

- **Tiền xử lý & làm sạch dữ liệu**
  - Kết hợp nhiều nguồn dữ liệu (bán hàng, thời tiết).
  - Xử lý giá trị thiếu, nhiễu và ngoại lệ (outlier).

- **Xây dựng đặc trưng (Feature Engineering)**
  - Hơn 50 đặc trưng: thời gian, độ trễ (lag), rolling statistics, đặc trưng theo cửa hàng/sản phẩm và đặc trưng thời tiết.

- **Mô hình chuỗi thời gian**
  - Dự báo doanh số bằng **LightGBM** với chiến lược chia train/test theo thời gian.

- **Tối ưu siêu tham số**
  - Sử dụng **Optuna** để tìm bộ tham số tốt, giảm sai số dự báo.

- **Giải thích mô hình với SHAP**
  - Phân tích **ảnh hưởng toàn cục (global)** và **cục bộ (local)** của từng đặc trưng lên dự đoán.

- **Ứng dụng Streamlit**
  - File `app.py` cung cấp giao diện web cho phép:
    - Xem lịch sử doanh số theo cửa hàng/sản phẩm.
    - Xem kết quả dự báo và các đồ thị quan trọng liên quan.

## Sản phẩm đầu ra

- Hệ thống notebook cho toàn bộ vòng đời PoC (tiền xử lý, EDA, feature engineering, mô hình, giải thích).
- Mô hình LightGBM đã huấn luyện.
- Bộ hình minh họa giải thích mô hình bằng SHAP – xem nhanh tại:
  - 📄 `docs/shap_analysis_summary_report.md`
- Ứng dụng Streamlit phục vụ demo và khám phá kết quả.

## Cấu trúc dự án

```bash
├── app.py                          # Ứng dụng Streamlit
├── check_data/
│   ├── check_data.xlsx             # File Excel để kiểm tra dự đoán
│   └── prediction_results.csv      # Kết quả dự báo từ mô hình
├── data/
│   ├── 2016_sales.csv              # Dữ liệu bán hàng 2016 (thô)
│   ├── 2017_sales.csv              # Dữ liệu bán hàng 2017 (thô)
│   ├── feature_engineered_data_55_features.feather
│   ├── sales_data_preprocessed.csv
│   ├── weather_data.csv
│   └── weather_preprocessed.csv
├── docs/
│   ├── project_description_poc_phase.md  # Mô tả chi tiết PoC
│   └── shap_analysis_summary_report.md   # Tóm tắt kết quả phân tích SHAP
├── environment.yml                 # Môi trường Conda cho đa số hệ thống
├── environment_macm1.yml           # Môi trường Conda cho Mac M1
├── requirements.txt                # Thư viện Python cần thiết
├── figures/                        # Hình EDA và các biểu đồ SHAP
├── models/
│   ├── feature_stats.json
│   └── sales_forecast_model.pkl   # Mô hình đã huấn luyện
├── notebooks/                     # Toàn bộ phần làm việc chính trong giai đoạn PoC
│   ├── 01_preprocessing.ipynb      # Tiền xử lý dữ liệu
│   ├── 02_EDA.ipynb                # Khám phá dữ liệu
│   ├── 03_feature_engineering.ipynb   # Xây dựng đặc trưng
│   ├── 04_modelling.ipynb          # Huấn luyện mô hình (Prophet baseline & LightGBM)
│   └── 05_explain_model.ipynb      # Giải thích mô hình bằng XAI
├── src/                            # Mã nguồn dạng module
│   ├── data_loader/
│   ├── data_generator/
│   ├── ui_builder/
│   ├── ui_predictor/
│   └── utils/
└── README.md
```

## Cài đặt & chạy

### 1. Clone repository

```bash
git clone https://github.com/nguyenhads/sales_forecasting_xai.git
cd sales_forecasting_xai
```

### 2. Thiết lập môi trường

**Dùng virtualenv (Python thuần):**

```bash
python -m venv .venv

# macOS/Linux:
source .venv/bin/activate

# Windows:
.venv\Scripts\activate

pip install -r requirements.txt
```

### 3. Chạy notebook

Sau khi kích hoạt môi trường:

```bash
jupyter lab
```

Mở lần lượt các notebook trong thư mục `notebooks` để xem toàn bộ pipeline.

### 4. Sinh dữ liệu mô phỏng (tùy chọn)

Bạn có thể tùy chỉnh khoảng thời gian dữ liệu, tỉ lệ giá trị thiếu và outlier:

- Chỉnh tham số trong file `src/data_generator/data_generator.py`.
- Tại thư mục gốc `sales_forecasting_xai`, chạy:

```bash
python src/data_generator/data_generator.py
```

### 5. Chạy ứng dụng Streamlit

```bash
streamlit run app.py
```

## Cách hoạt động (Tóm tắt pipeline)

1. **Xử lý dữ liệu**
   - Gộp dữ liệu bán hàng và thời tiết.
   - Sinh các đặc trưng thời gian, cửa hàng, sản phẩm và rolling.

2. **Huấn luyện mô hình**
   - Chia train/test theo trục thời gian.
   - Dùng LightGBM, tinh chỉnh tham số bằng Optuna, đánh giá bằng MAE, RMSE, WAPE.

3. **Giải thích mô hình**
   - Tính SHAP values, vẽ biểu đồ global & local.
   - Rút ra insight về các yếu tố chính ảnh hưởng tới dự báo.

4. **Giao diện người dùng**
   - `app.py` cho phép:
     - Xem lịch sử doanh số.
     - Khảo sát dự báo tương lai để hỗ trợ lập kế hoạch tồn kho, nhân sự, khuyến mãi.

## Tài liệu tham khảo

- LightGBM: `https://lightgbm.readthedocs.io/`
- SHAP: `https://shap.readthedocs.io/`
- Optuna: `https://optuna.org/`
- Streamlit: `https://streamlit.io/`

## Liên hệ

Nếu bạn muốn trao đổi thêm hoặc hợp tác, vui lòng liên hệ:

**📧 thucpkbn@gmail.com**

# Sales Forecasting with Explainable AI (XAI)

- **Author:** Nguyen Van Thuc, Pham Xuan Khang
- **Project Type:** Proof of Concept (PoC)
- **Tech Stack:** Python, LightGBM, SHAP, Optuna, Streamlit

## Overview

- **Sales Forecasting with Explainable AI (XAI)** is a complete end-to-end proof of concept (PoC) that leverages machine learning to forecast store-level sales with transparency and interpretability.

- The project combines time series modeling with explainability tools to provide actionable insights, making it easier for business stakeholders to understand and trust the model’s predictions.

- At its core, this project builds a sales forecasting model using **LightGBM**, optimized with **Optuna**, and explained using **SHAP (SHapley Additive exPlanations)**. It culminates in a **Streamlit web application** that allows users to explore historical sales and prediction results by store.

## Key Features

- **Data Preprocessing & Cleaning:**
  Integration of multiple data sources (sales, weather), missing value handling, outlier detection.

* **Feature Engineering:**
  Over 50 crafted features including date, lag, rolling stats, and weather-based inputs.

* **Time Series Modeling:**
  Sales forecasting using LightGBM with careful temporal train/test splitting.

* **Hyperparameter Tuning:**
  Efficient model optimization via **Optuna** for enhanced performance.

* **Explainability with SHAP:**
  Interpretable model predictions with local and global SHAP value analysis.

* **Interactive Streamlit App:**
  A web interface (`app.py`) that enables users to explore store-level forecasts and historical trends.

## Deliverables

- 5 comprehensive notebooks for data processing, feature engineering, modelling and evaluation
- Trained LightGBM model
- SHAP explainability visuals - 📄 [SHAP Analysis Summary Report](docs/shap_analysis_summary_report.md)
- Streamlit app for predictions

## Project Structure

```bash
├── app.py                          # Streamlit web app for user interaction
├── check_data/
│   ├── check_data.xlsx             # Excel file for checking prediction
│   └── prediction_results.csv      # Model prediction output
├── data/
│   ├── 2016_sales.csv              # Raw sales data for 2016
│   ├── 2017_sales.csv              # Raw sales data for 2017
│   ├── feature_engineered_data_55_features.feather
│   ├── sales_data_preprocessed.csv
│   ├── weather_data.csv
│   └── weather_preprocessed.csv
├── docs/
│   ├── project_description_poc_phase.md  # Project detail description
│   └── shap_analysis_summary_report.md   # Quick summary of SHAP results
├── environment.yml                 # Environment for most systems
├── environment_macm1.yml           # Environment for Mac M1 chip
├── requirements.txt                # Nessesary libraries
├── figures/                        # SHAP plots and EDA visuals
├── models/
│   ├── feature_stats.json
│   └── sales_forecast_model.pkl   # Trained model
├── notebooks/                     # Main work for PoC phase is based on Notebooks
│   ├── 01_preprocessing.ipynb      # Proprocessing notebook
│   ├── 02_EDA.ipynb                # EDA notebook
│   ├── 03_feature_engineering.ipynb   # Feature engineer
│   ├── 04_modelling.ipynb          # Model training (base line: Prophet and better: Light GBM)
│   └── 05_explain_model.ipynb      # Explainable AI
├── src/                            # Modular source code
│   ├── data_loader/
│   ├── data_generator/
│   ├── ui_builder/
│   ├── ui_predictor/
│   └── utils/
└── README.md
```

## Installation

1. **Clone the Repository**

   ```bash
   git clone https://github.com/nguyenhads/sales_forecasting_xai.git
   cd sales_forecasting_xai
   ```

2. **Set Up Environment**

  _You need to install Anaconda for this setup. If not, please use the below setup instead._

- Create a virtual environment using pure python

  ```
  python -m venv .venv

  # On macOS/Linux:
  source .venv/bin/activate

  # On Windows:
  .venv\Scripts\activate

  pip install -r requirements.txt
  ```

3. **Run the notebooks**

- After activating virtual enviroments

  ```bash
  jupyter lab
  ```

4. **Generate your all dataset**

- If you preferer generating your all dataset, you can change the range of data as well as the outlier and nan values ratio.
- In this case, modify `src/data_generator/data_generator.py `, and in below `sales_forecasting_xai` folder, run the below command

  ```bash
  python src/data_generator/data_generator.py
  ```

5. **Run the Streamlit App**
   ```bash
   streamlit run app.py
   ```

## How It Works

1. **Data Pipeline**
   Sales and weather data are preprocessed and merged. Features are engineered and saved for model training.

2. **Model Training**
   LightGBM is trained using time-aware train/test split. Optuna tunes the model for best performance.

3. **Explainability**
   SHAP values are calculated and visualized to explain predictions at both global and local levels.

4. **User Interface**

- `app.py` allows users to:
  - View historical sales
  - Make a predictions of future sales to properly arrange the resources

## References

- [LightGBM Documentation](https://lightgbm.readthedocs.io/)
- [SHAP Documentation](https://shap.readthedocs.io/)
- [Optuna Documentation](https://optuna.org/)
- [Streamlit](https://streamlit.io/)

## Contact

For questions or collaboration opportunities, please reach out at:
**📧 datasciencelab.ai@gmail.com**
