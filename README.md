# Week-05-Day-26

# Case Study: Pharmalink — B2B Medicine Distribution Platform
**PG Diploma · AI-ML & Agentic AI Engineering · IIT Gandhinagar**

---

## Business Context

Pharmalink connects **1.8M+ retail pharmacies and small hospitals** across 700+ cities with drug manufacturers and distributors. Pharmacies place bulk orders through the app and are given a 21-day rolling credit facility.

**Problem:** 15% of pharmacies active 4 months ago have placed **no orders in the last 30 days**.

A pharmacy is defined as **"churned"** if it does not reorder within 30 days of its last purchase.

**Key Nuance:** Tier-2/3 pharmacies place large but infrequent bulk orders — 30-day inactivity may not indicate churn for non-urban pharmacies.

---

## Questions & Solutions

### Q1 — Problem Identification
Supervised or Unsupervised? Classification or Regression? Which ML category? <br>
[Answer](q1_problem_identification.md)

### Q2 — Map to the 5-Stage ML Pipeline
One specific Pharmalink action per stage using actual column names. <br>
[Answer](q2_ml_pipeline.md)

### Q3 — Evaluation Strategy
Should Pharmalink prioritize Precision or Recall? Why? What trade-off does this represent? <br>
[Answer](q3_evaluation_strategy.md)

### Q4 — Advanced Thinking
How to handle Tier-3's fundamentally different ordering behaviour — global model, separate models, or interaction features? <br>
[Answer](q4_advanced_thinking.md)

---

## Quick Summary

| Question | Answer |
| :--- | :--- |
| **ML Type** | Supervised Learning |
| **Problem Type** | Binary Classification |
| **Primary Metric** | Recall (minimise missed churners) |
| **Tier Strategy** | Separate models per `city_tier` with extended 60-day churn window for Tier-2/3 |
| **Deployment** | Daily batch scoring, priority queue sorted by `avg_order_value` DESC |
