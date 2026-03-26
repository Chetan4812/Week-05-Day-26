# Question 3 — Evaluation Strategy

## Should Pharmalink Prioritize Precision or Recall?

**Answer: Recall**

---

## Why?

The CRO's quote defines the cost asymmetry directly:

> *"We can afford to call 500 pharmacies who were never going to churn. But if we lose a high-revenue hospital account placing ₹8 lakh monthly orders without warning, that's unacceptable."*

In ML terms, there are two types of errors the model can make:

| Error Type | What it means | Business consequence |
| :--- | :--- | :--- |
| **False Negative (FN)** | Model predicts "Active" — pharmacy actually churns | Account lost silently. No retention call made. Revenue gone permanently. |
| **False Positive (FP)** | Model predicts "Churn" — pharmacy was actually fine | Unnecessary retention call made. Costs ~₹500 of team time. |

**Recall = TP / (TP + FN)**

Maximising Recall directly minimises False Negatives — which is exactly what the CRO is asking for. Every missed churner is a lost account; every unnecessary call is just a minor operational cost.

---

## What Business Trade-off Does This Represent?

Choosing high Recall means accepting lower Precision:

*   **High Recall → More false alarms** → The retention team calls pharmacies that were never going to churn.
*   This increases the team's workload but each wasted call costs ~₹500.
*   Missing one high-revenue hospital account costs ₹8,00,000+/month — potentially ₹96L/year.

**The trade-off is justified** because the cost ratio is approximately **1:1600** (₹500 vs ₹8,00,000). Even if the model generates 1,600 false alarms, the cost is equivalent to losing just one high-value account.

### Recommended Threshold Adjustment

The default classification threshold of **0.50** optimises for balanced accuracy. For Pharmalink, the threshold should be **lowered to 0.35–0.40**:

*   Any pharmacy with `churn_probability > 0.35` is flagged for retention outreach.
*   This catches more true churners (higher Recall) at the cost of more unnecessary calls (lower Precision).
*   The exact threshold should be tuned on a validation set by plotting the Precision-Recall curve and selecting the point where Recall ≥ 0.85 while Precision stays above ~0.40.

### Tiered Escalation by Revenue

Not all false positives cost the same, and not all true positives are equally urgent. A practical deployment strategy:

*   `avg_order_value > ₹2L/month` + `churn_probability > 0.35` → **Immediate senior account manager call**
*   `avg_order_value < ₹50K/month` + `churn_probability > 0.60` → **Automated WhatsApp nudge / discount offer**

This keeps Recall high for high-revenue accounts while managing retention team capacity efficiently.
