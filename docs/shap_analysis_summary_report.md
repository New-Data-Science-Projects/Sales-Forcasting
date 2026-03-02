# SHAP Analysis Report – Sales Forecasting Model

This report summarizes key insights from the **SHAP values** analysis on the sales forecasting model for 10 stores and 35 products. The content focuses on:

- Global importance of feature groups.
- The role of sales history, store, and weather features.
- Detailed explanations of specific predictions (local explanations).

---

## 1. Global Feature Group Importance

| Feature Group         | Influence (%) | Explanation                                                                 |
| --------------------- | ------------- | --------------------------------------------------------------------------- |
| **Product (Item)**    | 46.4%         | Product-level features such as `item_mean_7d`, `item_sum_7d`, etc. dominate. |
| **Sales History**     | 32.8%         | Rolling features (mean, std, lag) effectively capture recent trends.         |
| **Store**             | 18.1%         | Store performance has a moderate influence on forecasts.                     |
| **Temporal (Date)**   | 2.2%          | Features like `month`, `day_of_week` have a small influence.                |
| **Weather**           | 0.4%          | Very limited influence in the current dataset.                              |
| **Other**             | 0.1%          | Mostly IDs/features with little direct contribution to predictions.         |

<p align="center">
  <img src="../figures/global_feature_importance_1.png" alt="Global Feature Importance" width="600"/>
</p>

### Business Insights

- **1. Product-level features are critical**
  - Nearly **50%** of the model's influence comes from product-level features.
  - Priorities should include:
    - Product segmentation (fast-moving vs. slow-moving items).
    - Inventory, pricing, and promotion management by product group.

- **2. Sales history is foundational**
  - Rolling and lag features effectively represent short-to-medium-term trends.
  - These can be reused in:
    - BI dashboards.
    - Inventory planning systems.

- **3. Store context has moderate but important influence**
  - Useful for:
    - Allocating goods across stores.
    - Identifying stores that need additional support (marketing, promotions, etc.).

- **4. Temporal and weather features have limited influence**
  - In this dataset, seasonality and weather impact are not strong.
  - Possible reasons:
    - Seasonal patterns are not sufficiently clear.
    - Products are not highly sensitive to weather conditions.

---

## 2. Most Influential Features

The most important features according to SHAP:

- **`item_mean_7d`** – Average product sales over the last 7 days:
  - High value → positive SHAP (red) → model increases forecast.
  - Low value → negative SHAP (blue) → model decreases forecast.

- **`sales_mean_28d`** – 28-day average sales:
  - Reflects longer-term trends and product stability.

- **`store_mean_7d`** – Recent store performance:
  - Adjusts forecasts based on individual store context.

- **`sales_mean_14d`**, **`store_sum_7d`**:
  - Combine short-term perspectives (7–14 days) at both store and product levels.

- ID features such as `item_id`, `store_id`:
  - May be encoding fixed effects by product/store.

- Weather features such as `temperature`, `season_wet`:
  - Contribute minimally, with weak impact in this dataset.

**Key Conclusion:**  
The model is particularly sensitive to **recent product sales trends**, more than climate or seasonal factors.

<p align="center">
  <img src="../figures/global_feature_importance.png" alt="Most Influential Features" width="600"/>
</p>

---

## 3. Weather Feature Influence

| Weather Feature   | SHAP Influence | Interpretation                                      |
| ----------------- | -------------- | --------------------------------------------------- |
| `temperature`     | -0.0174        | Lower temperatures tend to decrease forecasts.       |
| `season_winter`   | -0.0108        | Winter is associated with lower demand.              |
| Other features    | ≈ 0            | Almost no contribution to model decisions.           |

- In general:
  - The impact of weather is **weak** compared to sales history.
  - It can be considered a supplementary feature, not a primary driver of the model.

---

## 4. Local Explanations

Below are some representative examples showing how the model "weighs" features to produce forecasts.

### 4.1. Juice – Ba Dinh Supermarket (2017-10-26)

- **Actual**: 60.00  
- **Predicted**: 46.12

#### Features that **increase** the forecast

| Feature           | SHAP Effect |
| ----------------- | ----------- |
| `item_mean_7d`    | +15.73      |
| `sales_mean_28d`  | +4.41       |
| `sales_mean_14d`  | +2.02       |
| `sales_mean_7d`   | +1.02       |
| `item_sum_7d`     | +0.77       |

#### Features that **decrease** the forecast

| Feature          | SHAP Effect |
| ---------------- | ----------- |
| `store_sum_7d`   | -3.31       |
| `store_mean_7d`  | -0.74       |
| `month`          | -0.47       |
| `store_id`       | -0.27       |
| `item_id`        | -0.08       |

**Insight:**  
The product has very strong performance (strong item-level), but the store is not particularly outstanding (weak store-level), causing the model to "pull down" the forecast. This could indicate a **high-potential product at a store that has not been fully utilized**.

<p align="center">
  <img src="../figures/local_explainations1.png" alt="Juice - Ba Dinh" width="2000"/>
</p>

---

### 4.2. Vitamins – Tay Ho Store (2017-12-08)

- **Actual**: 14.00  
- **Predicted**: 14.07

#### Features that **increase** the forecast

| Feature          | SHAP Effect |
| ---------------- | ----------- |
| `store_sum_7d`   | +0.041      |
| `sales_min_28d`  | +0.011      |
| `season_wet`     | +0.010      |
| `sales_std_28d`  | +0.008      |
| `sales_min_7d`   | +0.007      |

#### Features that **decrease** the forecast

| Feature           | SHAP Effect |
| ----------------- | ----------- |
| `item_mean_7d`    | -7.40       |
| `sales_mean_28d`  | -3.19       |
| `sales_mean_14d`  | -0.98       |
| `sales_mean_7d`   | -0.35       |
| `store_id`        | -0.21       |

**Insight:**  
The product has a relatively low recent sales history (weak item-level), so the model estimates at a low level that closely matches reality. This shows the model **accurately reflects the limited demand** for this product.

<p align="center">
  <img src="../figures/local_explainations2.png" alt="Vitamins - Tay Ho" width="2000"/>
</p>

---

### 4.3. Noodles – Phu Nhuan Store (2017-11-21)

- **Actual**: 19.00  
- **Predicted**: 20.78

#### Features that **increase** the forecast

| Feature            | SHAP Effect |
| ------------------ | ----------- |
| `store_mean_7d`    | +1.86       |
| `store_id`         | +0.48       |
| `store_sum_7d`     | +0.36       |
| `sales_mean_14d`   | +0.17       |
| `sales_mean_7d`    | +0.17       |

#### Features that **decrease** the forecast

| Feature            | SHAP Effect |
| ------------------ | ----------- |
| `item_mean_7d`     | -8.49       |
| `sales_mean_28d`   | -0.31       |
| `item_sum_7d`      | -0.26       |
| `month`            | -0.06       |
| `season_wet`       | -0.02       |

**Insight:**  
The product is not particularly strong, but the store has good overall performance, so the model forecasts slightly above actuals. This example shows that **store-level factors can "push up" forecasts**, even when item-level performance is not outstanding.

<p align="center">
  <img src="../figures/local_explainations3.png" alt="Noodles - Phu Nhuan" width="2000"/>
</p>

---

## 5. SHAP Dependency Plot – `item_mean_7d`

The dependency plot for `item_mean_7d` reveals:

- A nearly linear relationship:
  - As `item_mean_7d` increases → sales forecast increases accordingly.
- Some notable thresholds:
  - **`item_mean_7d` < 20** → the model tends to decrease the forecast.
  - **`item_mean_7d` > 30** → the model increases the forecast significantly.
- Suggestions:
  - A **threshold of ~30** can be used to identify **fast-moving products** for inventory management & display prioritization.

<p align="center">
  <img src="../figures/dependency_plots.png" alt="Dependency Plot" width="600"/>
</p>

---

## 6. Conclusions & Recommendations

1. **Product-level sales trends are the most important driver** of the model.
2. **7–28 day sales history** is the "core logic" of the model and requires high data quality to maintain.
3. **Store context** plays an adjusting role, especially for stores with notably strong or weak performance.
4. **Weather and seasonality** currently do not show a strong role and can be considered supplementary features.
5. Accuracy is higher on **weekends and in December**, suggesting:
   - Increased inventory preparation and promotional campaigns during these periods.

The SHAP report demonstrates that the model is **not a complete "black box"** and can be explained intuitively, thereby helping business teams make more confident decisions.
