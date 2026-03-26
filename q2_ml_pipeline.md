# Question 2 — Map to the 5-Stage ML Pipeline

Each stage below describes the **specific action Pharmalink should take**, using actual column names from the dataset.

---

## Stage 1 — Data Collection

Pull **15 months of transaction and engagement history** for all 1.5 million pharmacies from Pharmalink's internal databases. Ensure the following feature groups are complete:

*   **Behavioural:** `last_order_date`, `order_frequency_60d`, `avg_order_value`, `app_sessions_30d`
*   **Financial:** `credit_utilization_ratio`, `payment_delay_days`, `credit_limit`, `outstanding_amount`
*   **Engagement:** `product_diversity_score`, `return_rate`, `complaints_logged`
*   **Demographic:** `city_tier`, `years_of_operation`, `store_type`
*   **Target:** `will_churn_next_30d` (1 = Churn, 0 = Active) — derived by checking whether each pharmacy reordered within 30 days of its last purchase

Also verify data freshness: `last_order_date` must be populated for all records and the 30-day churn label must be applied consistently across tiers before training.

---

## Stage 2 — Data Preprocessing

*   **Missing values:** Impute missing `payment_delay_days` with the median. Fill missing `last_order_date` gaps by cross-referencing the order transaction table.
*   **Encoding:** Encode `store_type` (independent / chain) as binary (0 / 1). Treat `city_tier` as an ordinal feature (1 < 2 < 3).
*   **Outliers:** `avg_order_value` will be heavily right-skewed — Tier-2/3 bulk orders can be 10–20× larger than Tier-1 orders. Apply log-transformation or cap at the 99th percentile to prevent distortion.
*   **Class imbalance:** With ~15% churn rate, the dataset is imbalanced. Apply `class_weight='balanced'` in the model or use SMOTE oversampling on the minority (churn) class.
*   **Feature scaling:** Standardise all numerical features (`credit_utilization_ratio`, `payment_delay_days`, `app_sessions_30d`, etc.) for Logistic Regression. Tree-based models do not require this.

---

## Stage 3 — Model Training

Train a **Random Forest classifier** using the following most predictive features:

*   `app_sessions_30d` — engagement drop is an early churn signal
*   `payment_delay_days` — financial stress predicts disengagement
*   `credit_utilization_ratio` — high utilisation with no new orders signals exit
*   `order_frequency_60d` — frequency decline is the most direct churn indicator
*   `avg_order_value` — sudden value drop signals reduced commitment
*   `city_tier` — controls for tier-specific ordering patterns

Use an **80/20 stratified train-test split** on `will_churn_next_30d` to preserve the 15% churn ratio in both splits. Train separate models per `city_tier` (see Q4 for reasoning).

---

## Stage 4 — Model Evaluation

Primary metric: **Recall for the churn class (label = 1)**, missing a churner is far more costly than a false alarm (see Q3 for full reasoning).

Supporting metrics:

*   **AUC-ROC** — measures overall discriminative ability across all thresholds
*   **Precision** — tracked to ensure retention team workload stays manageable
*   **Confusion Matrix** — inspect False Negatives specifically (missed churners)

Also validate feature importances: `app_sessions_30d`, `payment_delay_days`, and `credit_utilization_ratio` should rank as top predictors. If `city_tier` dominates, the model may be memorising tier patterns rather than churn signals — investigate further.

---

## Stage 5 — Deployment

*   Run a **daily batch scoring job** each morning on all pharmacies that were active in the last 60 days.
*   Flag pharmacies with `churn_probability > 0.40` (lowered threshold — see Q3) for the retention team.
*   **Priority queue:** Sort flagged pharmacies by `avg_order_value` descending — the retention team calls the highest-revenue accounts first, as directed by the CRO.
*   Integrate churn scores into the **Pharmalink CRM dashboard**, segmented by `city_tier` so regional managers can act on their own accounts.
*   **Monitor drift monthly:** Retrain the model every 4–6 weeks as pharmacy ordering behaviour and credit patterns shift with season and market conditions.
