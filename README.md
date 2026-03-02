# Sales Forecasting with Explainable AI (XAI)
- **Authors:** Nguyen Van Thuc, Pham Xuan Khang
- **Project Type:** Proof of Concept (PoC)
- **Tech Stack:** Python, LightGBM, SHAP, Optuna, Streamlit

## Overview

- **Sales Forecasting with Explainable AI (XAI)** is an end-to-end PoC that simulates a retail sales forecasting problem at the store/product level on a daily basis.
- The goal is not only to build an accurate forecasting model, but also to **explain** why the model makes certain predictions using XAI techniques (SHAP).
- The project uses **LightGBM** as the main model, optimizes hyperparameters with **Optuna**, performs explainability analysis with **SHAP**, and deploys an interactive results exploration interface with **Streamlit**.

## Key Features

- **Data Preprocessing & Cleaning**
  - Combines multiple data sources (sales, weather).
  - Handles missing values, noise, and outliers.

- **Feature Engineering**
  - Over 50 features: temporal, lag, rolling statistics, store/product-level, and weather features.

- **Time Series Modeling**
  - Forecasts sales using **LightGBM** with a time-based train/test split strategy.

- **Hyperparameter Optimization**
  - Uses **Optuna** to find optimal parameters and reduce forecasting error.

- **Model Explainability with SHAP**
  - Analyzes **global** and **local** feature influence on predictions.

- **Streamlit Application**
  - `app.py` provides a web interface that allows users to:
    - View historical sales by store/product.
    - View forecast results and key related charts.

## Deliverables

- A complete notebook pipeline covering the entire PoC lifecycle (preprocessing, EDA, feature engineering, modeling, explainability).
- A trained LightGBM model.
- A set of SHAP-based model explanation visualizations — see quick summary at:
  - 📄 `docs/shap_analysis_summary_report.md`
- A Streamlit application for demo and results exploration.

## Project Structure

```bash
├── app.py                          # Streamlit application
├── check_data/
│   ├── check_data.xlsx             # Excel file for prediction verification
│   └── prediction_results.csv      # Model prediction results
├── data/
│   ├── 2016_sales.csv              # 2016 sales data (raw)
│   ├── 2017_sales.csv              # 2017 sales data (raw)
│   ├── feature_engineered_data_55_features.feather
│   ├── sales_data_preprocessed.csv
│   ├── weather_data.csv
│   └── weather_preprocessed.csv
├── docs/
│   ├── project_description_poc_phase.md  # Detailed PoC description
│   └── shap_analysis_summary_report.md   # SHAP analysis summary report
├── requirements.txt                # Required Python libraries
├── figures/                        # EDA and SHAP charts
├── models/
│   ├── feature_stats.json
│   └── sales_forecast_model.pkl   # Trained model
├── notebooks/                     # All main work for the PoC phase
│   ├── 01_preprocessing.ipynb      # Data preprocessing
│   ├── 02_EDA.ipynb                # Exploratory data analysis
│   ├── 03_feature_engineering.ipynb   # Feature engineering
│   ├── 04_modelling.ipynb          # Model training (Prophet baseline & LightGBM)
│   └── 05_explain_model.ipynb      # Model explainability with XAI
├── src/                            # Source code modules
│   ├── data_loader/
│   ├── data_generator/
│   ├── ui_builder/
│   ├── ui_predictor/
│   └── utils/
└── README.md
```

## Installation & Running

### 1. Clone the Repository

```bash
git clone https://github.com/nguyenhads/sales_forecasting_xai.git
cd sales_forecasting_xai
```

### 2. Set Up the Environment

**Using virtualenv (pure Python):**

```bash
python -m venv .venv

# macOS/Linux:
source .venv/bin/activate

# Windows:
.venv\Scripts\activate

pip install -r requirements.txt
```

### 3. Run the Notebooks

After activating the environment:

```bash
jupyter lab
```

Open the notebooks in the `notebooks` folder sequentially to view the entire pipeline.

### 4. Generate Simulated Data (Optional)

You can customize the data time range, missing value ratio, and outlier ratio:

- Adjust parameters in `src/data_generator/data_generator.py`.
- From the project root directory `sales_forecasting_xai`, run:

```bash
python src/data_generator/data_generator.py
```

### 5. Run the Streamlit Application

```bash
streamlit run app.py
```

## How It Works (Pipeline Summary)

1. **Data Processing**
   - Merges sales and weather data.
   - Generates temporal, store, product, and rolling features.

2. **Model Training**
   - Splits train/test along the time axis.
   - Uses LightGBM, tunes hyperparameters with Optuna, evaluates with MAE, RMSE, WAPE.

3. **Model Explainability**
   - Computes SHAP values, plots global & local charts.
   - Extracts insights about key factors influencing forecasts.

4. **User Interface**
   - `app.py` allows users to:
     - View historical sales.
     - Explore future forecasts to support inventory planning, staffing, and promotions.

## References

- LightGBM: `https://lightgbm.readthedocs.io/`
- SHAP: `https://shap.readthedocs.io/`
- Optuna: `https://optuna.org/`
- Streamlit: `https://streamlit.io/`

## Contact

For further discussion or collaboration, please reach out:

**📧 thucpkbn@gmail.com**