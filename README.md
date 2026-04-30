# CTA Prediction Case Study

A machine learning case study on optimizing call-to-action (CTA) engagement in a mortgage-style lead generation funnel.  
The project focuses on predicting the probability that a user clicks a CTA, while also analyzing how CTA copy and placement affect downstream business value such as form submissions, appointments, and revenue per impression.

## Project Overview

This project analyzes a randomized CTA experiment in which users were shown different combinations of CTA copy and page placement.  
The modeling task is to predict `Pr(clickedCTA = 1)` for each user session, with **log loss** as the primary evaluation metric. At the same time, the broader business goal is to improve the conversion funnel and maximize downstream value, not just clicks.

## Business Framing

The core question is not only:

- *Will this user click the CTA?*

but also:

- *Which CTA option should be shown to which type of user?*

A key finding from the analysis is that the CTA combination with the highest click-through rate was **not** the same as the one with the highest revenue per impression. That means optimizing only for CTR can leave business value on the table.

## Dataset

The case study uses a structured session-level dataset with:

- user/session context
- referral source
- browser and device type
- estimated annual income and property type
- visit count
- page URL and editorial snippet
- CTA copy and CTA placement
- downstream outcomes such as form submission, appointment scheduling, and revenue

The prediction target is `clickedCTA`, and the deliverable is a predicted probability for each test record.

## Approach

### 1. Data quality and modeling scope
The first step was validating schema consistency, checking duplicates, and separating **decision-time features** from **downstream leakage features**.

For the primary click model, only features plausibly available when the CTA is shown were retained.  
Downstream variables such as submitted forms, scheduled appointments, mortgage variation, and revenue were excluded to avoid leakage. `scrollDepth` was also excluded from the primary model because it is observed after the CTA loads and would not be appropriate in a real-time decision setting.

### 2. Exploratory data analysis
The analysis examined:

- funnel conversion rates
- CTA copy performance
- CTA placement performance
- copy-by-placement interaction effects
- segment-level differences across traffic source, device, browser, property type, and visit frequency

Notable findings included:

- overall CTR: **17.26%**
- form submission rate: **14.79%**
- scheduled appointment rate: **5.52%**
- average revenue per impression: **$12.03**
- best CTR placement: **Top**
- best CTR copy: **“Get Pre-Approved for a Mortgage in 5 Minutes”**
- best CTR combination: **that copy + Top placement**
- best revenue-per-impression combination: **a different CTA + Bottom placement**

### 3. Feature engineering
Feature engineering focused on improving predictive signal while preserving decision-time realism. Techniques included:

- missing-value indicators
- grouped or transformed numerical features
- interaction features such as:
  - `ctaCopy × ctaPlacement`
  - `deviceType × ctaPlacement`
  - `sessionReferrer × ctaCopy`
- contextual features from page content and URL structure

These features were intended to capture user- and page-dependent differences in CTA effectiveness.

### 4. Modeling
The modeling workflow compared:

- a **dummy baseline**
- **regularized logistic regression**
- **random forest**

The primary optimization metric was **log loss**, since the task requires predicted probabilities. Supporting metrics included:

- ROC-AUC for threshold-independent ranking
- PR-AUC for positive-class discrimination
- Brier score for probability quality and calibration awareness

The final model selected was **random forest**, which achieved the best cross-validated log loss and supporting metrics. The interpretation was that tree-based models better captured the non-linearities and interactions in user behavior than the linear baseline.

## Key Insights

- CTA placement was a major driver of click behavior.
- User response varied meaningfully by context, including traffic source, device type, and visit frequency.
- The highest-CTR CTA combination was not the same as the highest-revenue combination.
- These patterns suggest that a **personalized CTA policy** may outperform a single global “champion” CTA.

## Deployment Thinking

If productionized, this project could support a real-time CTA decision system:

1. collect session-level features available at page load
2. score the likely click probability for candidate CTA options
3. select the best CTA for that user context
4. log outcomes for monitoring and retraining

Important post-deployment considerations would include:

- input drift monitoring
- probability calibration checks
- CTR and downstream revenue monitoring
- champion-vs-personalized policy testing through online experimentation

## Repository Structure

```text
.
├── notebooks/
├── src/
├── outputs/
├── README.md
└── requirements.txt
