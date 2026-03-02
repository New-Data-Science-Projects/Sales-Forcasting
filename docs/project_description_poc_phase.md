# PoC Project Description – Sales Forecasting with XAI

## 1. Overview

This is a **Proof of Concept (PoC)** project that simulates a real-world retail problem: forecasting daily sales for each product at each store, using historical sales data combined with weather data.

The main objective is not only to build an accurate model, but also to **explain** how the model makes decisions using **Explainable AI (XAI)** techniques, enabling business teams to trust and utilize the results.

Through this project, learners will experience the full lifecycle of an enterprise PoC: from data exploration, feature engineering, model training, evaluation, to presenting results and business recommendations.

---

## 2. PoC Objectives

- Build a sales forecasting model by:
  - Store (`store`)
  - Product (`item`)
  - Date (`date`)
- Analyze relationships between:
  - Temporal features (day, week, month, season, holidays, etc.)
  - Historical sales features (lag, rolling mean/std, etc.)
  - Store and product features
  - Weather features (temperature, dry/wet season, etc.)
- Use **XAI (SHAP)** to:
  - Understand which features are most important.
  - Explain why a specific day has a high/low forecast.
- Deliver actionable insights for:
  - Inventory planning.
  - Staffing allocation.
  - Promotions & marketing planning.

---

## 3. Data Sources

### 3.1. Sales Data

Two main data files:

- `2016_sales.csv`
- `2017_sales.csv`

Key columns:

- `date`: Sale date
- `province`, `store_id`, `store_name`: Store information
- `category`, `item_id`, `item_name`: Product information
- `sales`: Number of units sold per day

### 3.2. Weather Data

File:

- `weather_data.csv`

Key columns:

- `date`: Weather recording date
- `city`: City / region
- `temperature`: Temperature
- `humidity`: Humidity
- `season`: Season (e.g., dry/wet, summer/winter, etc.)

After preprocessing, the data is merged into a single unified table for model training.

---

## 4. Phase 1 – Building the PoC

### 4.1. Data Integration & Cleaning

- Standardize data types (datetime, numeric, category, etc.).
- Handle missing values using:
  - Dropping in safe cases.
  - Imputation based on statistics / business logic.
- Handle outliers in the `sales` column (due to data entry errors or anomalous events).
- Merge sales and weather data by:
  - `date` + `province` / `city`

### 4.2. Exploratory Data Analysis (EDA)

- Plot sales time series:
  - By day, week, month.
  - By each store, each product category.
- Check for patterns:
  - Seasonality.
  - Weekends / holidays.
  - Best-selling / slow-moving products.
- Analyze correlations between:
  - Temperature, humidity, season.
  - Sales over time.

### 4.3. Feature Engineering

Main feature groups:

- **Temporal Features**
  - `day_of_week`, `day_of_month`, `month`, `is_weekend`, `is_holiday`, etc.
- **Historical Sales Features**
  - Sales lags from previous days (1, 7, 14, 21, 28 days, etc.).
  - Rolling mean/min/max/std over windows (7, 14, 28 days, etc.).
  - Exponentially weighted moving average (EWMA) for recent trends.
- **Store & Product Features**
  - Average / total sales by:
    - Store over the last 7 days.
    - Product over the last 7 days.
  - Encoding of `store_id`, `item_id`.
- **Weather Features**
  - Temperature category (`temp_category`).
  - Humidity level (`humidity_level`).
  - Season flags (`season_wet`, `season_winter`, etc.).

The final result is a feature-rich table used for model training.

### 4.4. Modeling

- Build a baseline with:
  - A simple model or LightGBM on a basic feature set.
- Build an improved model:
  - LightGBM on the full feature set (including all temporal, historical, store, product, and weather features).
- Split train/test by time:
  - Avoid future data leakage.
- Evaluate using metrics:
  - MAE (Mean Absolute Error)
  - RMSE (Root Mean Squared Error)
  - WAPE (Weighted Absolute Percentage Error)
  - Time series cross-validation can be added if needed.
- Hyperparameter tuning with **Optuna**.

### 4.5. Explainable AI (XAI)

Use **SHAP (SHapley Additive exPlanations)** to:

- Measure feature importance at:
  - **Global level**: Which features are most important to the model?
  - **Local level**: Why does the model predict high/low for a specific day/store/product?
- Analyze:
  - The influence of short-term vs. long-term sales history.
  - Differences between stores.
  - The actual impact of weather.
- Plot:
  - Summary plot.
  - Feature importance.
  - Dependency plot for key features (e.g., `item_mean_7d`).

For details, see: `docs/shap_analysis_summary_report.md`.

### 4.6. Reporting & Recommendations

In the PoC phase, the focus is:

- **Not just answering "Is the model good?"**, but also:
  - To what extent can the model be explained?
  - Are the insights useful for business operations?
- Finally, the team will:
  - Summarize model accuracy.
  - Present the key factors influencing sales.
  - Recommend:
    - Whether to proceed to the expanded deployment phase (Production/Phase 2).
    - What additional data/features are needed to improve performance.

---

## 5. Learning Outcomes

After completing this PoC project, learners will be able to:

- Understand the **end-to-end** process of an enterprise PoC project:
  - Data collection – exploration – processing.
  - Feature engineering.
  - Model training & tuning.
  - Results interpretation and stakeholder presentation.
- Gain proficiency in:
  - Time series data processing and analysis.
  - Feature design for forecasting problems.
  - Using LightGBM and Optuna for forecasting tasks.
  - Applying SHAP for model explainability.
- Enhance skills in:
  - Writing technical reports.
  - Translating technical language into business language.
  - Providing actionable recommendations.
