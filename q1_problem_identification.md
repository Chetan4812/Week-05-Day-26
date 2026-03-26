# Question 1 — Problem Identification

## Supervised or Unsupervised Learning?

**Answer: Supervised Learning**

The dataset contains a clearly defined target variable `will_churn_next_30d` (1 = Churn, 0 = Active). Every pharmacy record is labelled with its known outcome, allowing the model to learn a direct input → output mapping from historical data. There is no need to discover hidden structure, the goal is to predict a known label for unseen pharmacies using past labelled examples.

---

## Classification or Regression?

**Answer: Classification**

The target variable `will_churn_next_30d` is binary — it takes exactly two discrete values: **1 (Churn)** or **0 (Active)**. The model's job is to assign each pharmacy to one of these two classes. Predicting a continuous quantity (e.g., exact days-to-next-order or predicted revenue) would be regression; predicting a discrete class label is classification.

---

## Which Specific ML Category?

**Answer: Supervised Binary Classification**

This maps directly to the **binary classification** sub-type of supervised learning from this week's curriculum. The output space has exactly two classes (churn / active), the training data is labelled, and the goal is to learn a decision boundary. Appropriate algorithms include:

*   Logistic Regression — interpretable baseline
*   Random Forest — handles non-linear patterns and feature interactions
*   Gradient Boosting (XGBoost / LightGBM) — high performance on tabular data with class imbalance
