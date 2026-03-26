# Question 4 — Advanced Thinking: Handling Tier-3 Ordering Behavior

## The Problem

Tier-2 and Tier-3 pharmacies place **large but infrequent bulk orders**, reordering every 45–60 days is completely normal behaviour for them, not a churn signal. The current 30-day inactivity rule was designed around Tier-1 urban pharmacies (frequent small orders) and incorrectly classifies Tier-3 pharmacies as churned simply because they haven't ordered in 30 days.

A single global model trained on all pharmacies will learn the Tier-1 pattern as the baseline. Features like `order_frequency_60d` and `app_sessions_30d` carry very different meanings across tiers, what looks like churn behaviour for Tier-1 is completely normal for Tier-3.

---

## Recommended Approach: Separate Models by City Tier ✅

### Why NOT a Global Model?

A global model is trained on all pharmacies together, forcing it to find **one decision boundary** where three distinct behavioural patterns exist. The model will:

*   Learn that low `order_frequency_60d` = churn (true for Tier-1, false for Tier-3)
*   Systematically over-predict churn for Tier-2/3 pharmacies
*   Generate massive False Positive rates for bulk buyers, wasting retention team capacity on healthy accounts
*   Underfit the minority churn signals that are specific to each tier (e.g., `credit_utilization_ratio` spiking for a Tier-3 bulk buyer is a much stronger churn signal than for Tier-1)

### Why NOT Just Interaction Features?

Adding interaction features like `city_tier × order_frequency_60d` or `city_tier × app_sessions_30d` is a partial fix but has clear limitations:

*   It still constrains the model to a **single architecture with shared weights**, it can scale feature importance by tier but cannot learn completely different relationships.
*   A Tier-3 pharmacy's churn signal might be driven by `payment_delay_days` and `outstanding_amount` (financial stress before going silent), whereas Tier-1 churn is driven by `app_sessions_30d` dropping to zero. Interaction terms cannot capture this structural difference.
*   Interaction features increase model complexity without the full benefit of tier-specific learning.

### Separate Models — Implementation Plan

| | Tier-1 Model | Tier-2 / Tier-3 Model |
| :--- | :--- | :--- |
| **Churn window** | 30 days (standard) | 60 days (extended for bulk buyers) |
| **Primary signals** | `order_frequency_60d`, `app_sessions_30d` | `payment_delay_days`, `credit_utilization_ratio`, `avg_order_value` trend |
| **Interpretation** | Any frequency or engagement drop is a churn signal | Financial health and volume trajectory matter more than recency |
| **Training data** | Filter `city_tier == 1` | Filter `city_tier IN (2, 3)` |
| **Churn label** | Reorder within 30 days | Reorder within 60 days |
| **Model** | Logistic Regression (interpretable, fast) | Random Forest / LightGBM (captures non-linear bulk order patterns) |

### Deployment

*   Both models run as separate daily batch scoring pipelines.
*   Scores are merged in the **Pharmalink CRM dashboard** by `city_tier`.
*   The retention priority queue still sorts by `avg_order_value` descending, a Tier-3 hospital at risk always ranks above a Tier-1 corner pharmacy.

### Summary

Separate models by `city_tier` is the right choice because the **data generating process is fundamentally different** across tiers, not just different in scale, but different in which features drive churn. Training together averages out these differences and degrades predictive performance for every tier. Separate models respect the domain knowledge embedded in the CRO's own statement about Tier-2/3 ordering behaviour.
