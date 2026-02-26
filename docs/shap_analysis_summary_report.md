# Báo cáo Phân tích SHAP – Mô hình Dự báo Doanh số

Báo cáo này tóm tắt các insight chính rút ra từ phân tích **SHAP values** trên mô hình dự báo doanh số cho 10 cửa hàng và 35 sản phẩm. Nội dung tập trung vào:

- Tầm quan trọng toàn cục của nhóm đặc trưng.
- Vai trò của lịch sử bán hàng, cửa hàng và thời tiết.
- Giải thích chi tiết một số dự đoán cụ thể (local explanations).

---

## 1. Tầm quan trọng toàn cục của nhóm đặc trưng

| Nhóm đặc trưng      | Độ ảnh hưởng (%) | Giải thích                                                                 |
| ------------------- | ---------------- | -------------------------------------------------------------------------- |
| **Sản phẩm (Item)** | 46.4%            | Các đặc trưng theo sản phẩm như `item_mean_7d`, `item_sum_7d`,… chi phối lớn. |
| **Lịch sử bán hàng**| 32.8%            | Các đặc trưng rolling (mean, std, lag) nắm bắt xu hướng gần rất hiệu quả.  |
| **Cửa hàng (Store)**| 18.1%            | Hiệu suất cửa hàng ảnh hưởng ở mức trung bình đến dự báo.                 |
| **Thời gian (Date)**| 2.2%             | Các đặc trưng `month`, `day_of_week` có ảnh hưởng nhỏ.                    |
| **Thời tiết**       | 0.4%             | Ảnh hưởng rất hạn chế trên dữ liệu hiện tại.                              |
| **Khác**            | 0.1%             | Chủ yếu là ID/feature ít liên quan trực tiếp đến dự đoán.                 |

<p align="center">
  <img src="../figures/global_feature_importance_1.png" alt="Global Feature Importance" width="600"/>
</p>

### Nhận định nghiệp vụ

- **1. Đặc trưng theo sản phẩm là then chốt**
  - Gần **50%** ảnh hưởng của mô hình đến từ nhóm đặc trưng sản phẩm.
  - Cần ưu tiên:
    - Phân khúc sản phẩm (hàng bán chạy, bán chậm).
    - Quản lý tồn kho, giá và khuyến mãi theo từng nhóm sản phẩm.

- **2. Lịch sử bán hàng là nền tảng**
  - Các đặc trưng rolling và lag thể hiện xu hướng ngắn–trung hạn rất tốt.
  - Có thể được tái sử dụng trong:
    - Dashboard BI.
    - Hệ thống lập kế hoạch tồn kho.

- **3. Cửa hàng có ảnh hưởng trung bình nhưng quan trọng**
  - Hữu ích cho:
    - Phân bổ hàng hóa giữa các cửa hàng.
    - Xác định cửa hàng cần hỗ trợ thêm (marketing, khuyến mãi,…).

- **4. Thời gian và thời tiết ít ảnh hưởng**
  - Trong dataset này, seasonality và tác động thời tiết không mạnh.
  - Có thể do:
    - Chu kỳ mùa vụ chưa đủ rõ.
    - Sản phẩm không quá nhạy cảm với thời tiết.

---

## 2. Các đặc trưng có ảnh hưởng cao nhất

Những đặc trưng quan trọng nhất theo SHAP:

- **`item_mean_7d`** – Doanh số trung bình của sản phẩm trong 7 ngày gần nhất:
  - Giá trị cao → SHAP dương (màu đỏ) → mô hình tăng dự báo.
  - Giá trị thấp → SHAP âm (màu xanh) → mô hình giảm dự báo.

- **`sales_mean_28d`** – Doanh số trung bình 28 ngày:
  - Phản ánh xu hướng dài hơi, độ ổn định của sản phẩm.

- **`store_mean_7d`** – Hiệu suất gần đây của cửa hàng:
  - Điều chỉnh dự báo theo bối cảnh từng cửa hàng.

- **`sales_mean_14d`**, **`store_sum_7d`**:
  - Kết hợp góc nhìn ngắn hạn (7–14 ngày) ở cả mức cửa hàng và sản phẩm.

- Các đặc trưng ID như `item_id`, `store_id`:
  - Có thể đang encode hiệu ứng cố định theo sản phẩm/cửa hàng.

- Các đặc trưng thời tiết như `temperature`, `season_wet`:
  - Góp phần nhỏ, tác động yếu trong dataset này.

**Kết luận chính:**  
Mô hình đặc biệt nhạy với **xu hướng bán hàng gần đây của sản phẩm**, hơn là yếu tố khí hậu hay mùa vụ.

<p align="center">
  <img src="../figures/global_feature_importance.png" alt="Most Influential Features" width="600"/>
</p>

---

## 3. Ảnh hưởng của đặc trưng thời tiết

| Đặc trưng thời tiết | Ảnh hưởng SHAP | Diễn giải                                      |
| ------------------- | -------------- | ---------------------------------------------- |
| `temperature`       | -0.0174        | Nhiệt độ thấp có xu hướng làm giảm dự báo.    |
| `season_winter`     | -0.0108        | Mùa đông gắn với nhu cầu thấp hơn.            |
| Các đặc trưng khác  | ≈ 0            | Gần như không đóng góp vào quyết định mô hình.|

- Nhìn chung:
  - Tác động của thời tiết là **yếu** so với lịch sử bán hàng.
  - Có thể coi như feature phụ, không phải driver chính của mô hình.

---

## 4. Giải thích cục bộ (Local Explanations)

Dưới đây là một số ví dụ điển hình, cho thấy cách mô hình “cân” các đặc trưng để đưa ra dự báo.

### 4.1. Juice – Ba Đình Supermarket (2017-10-26)

- **Thực tế**: 60.00  
- **Dự báo**: 46.12

#### Nhóm đặc trưng làm **tăng** dự báo

| Đặc trưng        | SHAP Effect |
| ---------------- | ----------- |
| `item_mean_7d`   | +15.73      |
| `sales_mean_28d` | +4.41       |
| `sales_mean_14d` | +2.02       |
| `sales_mean_7d`  | +1.02       |
| `item_sum_7d`    | +0.77       |

#### Nhóm đặc trưng làm **giảm** dự báo

| Đặc trưng       | SHAP Effect |
| ----------------| ----------- |
| `store_sum_7d`  | -3.31       |
| `store_mean_7d` | -0.74       |
| `month`         | -0.47       |
| `store_id`      | -0.27       |
| `item_id`       | -0.08       |

**Insight:**  
Sản phẩm có hiệu suất rất tốt (item-level mạnh), nhưng cửa hàng lại không quá nổi bật (store-level yếu), khiến mô hình “kéo xuống” dự báo. Đây có thể là **sản phẩm tiềm năng ở một cửa hàng chưa khai thác hết**.

<p align="center">
  <img src="../figures/local_explainations1.png" alt="Juice - Ba Dinh" width="2000"/>
</p>

---

### 4.2. Vitamins – Cửa hàng Tây Hồ (2017-12-08)

- **Thực tế**: 14.00  
- **Dự báo**: 14.07

#### Đặc trưng làm **tăng** dự báo

| Đặc trưng       | SHAP Effect |
| --------------- | ----------- |
| `store_sum_7d`  | +0.041      |
| `sales_min_28d` | +0.011      |
| `season_wet`    | +0.010      |
| `sales_std_28d` | +0.008      |
| `sales_min_7d`  | +0.007      |

#### Đặc trưng làm **giảm** dự báo

| Đặc trưng        | SHAP Effect |
| ---------------- | ----------- |
| `item_mean_7d`   | -7.40       |
| `sales_mean_28d` | -3.19       |
| `sales_mean_14d` | -0.98       |
| `sales_mean_7d`  | -0.35       |
| `store_id`       | -0.21       |

**Insight:**  
Sản phẩm có lịch sử bán khá thấp trong thời gian gần (item-level yếu), nên mô hình ước lượng ở mức thấp và sát với thực tế. Điều này cho thấy mô hình **phản ánh đúng mức cầu hạn chế** của mặt hàng này.

<p align="center">
  <img src="../figures/local_explainations2.png" alt="Vitamins - Tay Ho" width="2000"/>
</p>

---

### 4.3. Noodles – Cửa hàng Phú Nhuận (2017-11-21)

- **Thực tế**: 19.00  
- **Dự báo**: 20.78

#### Đặc trưng làm **tăng** dự báo

| Đặc trưng         | SHAP Effect |
| ----------------- | ----------- |
| `store_mean_7d`   | +1.86       |
| `store_id`        | +0.48       |
| `store_sum_7d`    | +0.36       |
| `sales_mean_14d`  | +0.17       |
| `sales_mean_7d`   | +0.17       |

#### Đặc trưng làm **giảm** dự báo

| Đặc trưng         | SHAP Effect |
| ----------------- | ----------- |
| `item_mean_7d`    | -8.49       |
| `sales_mean_28d`  | -0.31       |
| `item_sum_7d`     | -0.26       |
| `month`           | -0.06       |
| `season_wet`      | -0.02       |

**Insight:**  
Sản phẩm không quá mạnh, nhưng cửa hàng có hiệu suất tổng thể tốt nên mô hình dự báo hơi cao hơn thực tế một chút. Đây là ví dụ cho thấy **store-level có thể “đẩy” dự báo lên**, ngay cả khi item-level không quá nổi bật.

<p align="center">
  <img src="../figures/local_explainations3.png" alt="Noodles - Phu Nhuan" width="2000"/>
</p>

---

## 5. Biểu đồ phụ thuộc SHAP – `item_mean_7d`

Biểu đồ dependency plot cho `item_mean_7d` cho thấy:

- Quan hệ gần như tuyến tính:
  - Khi `item_mean_7d` tăng → dự báo doanh số tăng theo.
- Một số ngưỡng đáng chú ý:
  - **`item_mean_7d` < 20** → mô hình có xu hướng giảm dự báo.
  - **`item_mean_7d` > 30** → mô hình tăng dự báo khá mạnh.
- Gợi ý:
  - Có thể dùng **ngưỡng ~30** để nhận diện các **sản phẩm bán chạy (fast-moving)** cho mục tiêu quản lý tồn kho & ưu tiên trưng bày.

<p align="center">
  <img src="../figures/dependency_plots.png" alt="Dependency Plot" width="600"/>
</p>

---

## 6. Kết luận & khuyến nghị

1. **Xu hướng bán hàng theo sản phẩm là driver quan trọng nhất** của mô hình.
2. **Lịch sử bán hàng 7–28 ngày** là “lõi logic” của mô hình, cần được duy trì/chất lượng dữ liệu tốt.
3. **Bối cảnh cửa hàng** có vai trò điều chỉnh, nhất là với các cửa hàng có hiệu suất nổi bật hoặc yếu.
4. **Thời tiết và mùa vụ** hiện chưa thể hiện vai trò mạnh, có thể xem là feature bổ sung.
5. Độ chính xác cao hơn vào **cuối tuần và tháng 12**, gợi ý:
   - Tăng cường chuẩn bị tồn kho và chiến dịch khuyến mãi trong các giai đoạn này.

Báo cáo SHAP cho thấy mô hình **không phải “black box” hoàn toàn** mà có thể được giải thích một cách trực quan, từ đó hỗ trợ đội ngũ kinh doanh ra quyết định tự tin hơn.

