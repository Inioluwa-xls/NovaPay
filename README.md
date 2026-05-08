# NovaPay
## Fraudulent Transaction detection for digital money transfer

### Data Cleaning
Data types were fixed and timestamp was converted from object to datetime and amount_src was converted from object to float

#### Filling in missing values for targeted columns

Filling in missing values for amount_usd carried out by calculating exchange rates per currency by selecting non-missing rows in amount_usd and grouping by source currency, then computing the mean of amount_usd/amount_src for each currency and saving the results to a dictionary for easy lookup. The missing values in amount_src are filled by multiplying the values derived for the exchange rate and the source_currency column

Filling in missing values for the fee column is done based on the median for the channel feature since fee is dependent on channel

Filling in the ip_country values done by using the corresponding home country

Filling in the missing values in kyc_tier with the mode

Filling in the missing values in device_trust_score using the group's median value

#### Droppig missing values and duplicated rows
The rows that had missing values for ip_address(305), timestamp(61) and amount_src(4) were dropped as they are unique values that cannot be computed
Duplicated rows which totaled to 194 rows were also dropped

### Sanity Checks for the dataset

This section lists essential sanity checks to validate the dataset after cleaning and imputation

1. Check for invalid numeric values, including negative values in monetary, risk, trust or velocity fields and validate the user age in days is not negative
2. Verify currency-related logic and ensure there are no negative values and that the derived exchange rates fall within reasonable range
3. Validate timestamp integrity and confirm no transaction timestamps occur in the future
4. Review location consistency to ensure the counts in location_mismatch was generated corrrectly and the country features contain plausible country codes
5. Check categorical column consistency to review unique values in channel, source_currency, dest_currency and kyc_tier to ensure there are no unexpected entries
6. Validate risk-score ranges ensuring all values fall within expected numeric range
7. Ensure standardization of formatting used in the features
8. Confirm fraud label integrity ensuring they contain only binary values
9. Validate velocity features

These checks ensure this dataset is thoroughly consistent before the feature engineering and modeling stage.

## Project Summary

This project focuses on developing a robust machine learning solution for detecting fraudulent transactions in digital money transfers, specifically for NovaPay. The goal is to identify fraudulent activities while minimizing false positives to maintain customer trust and operational efficiency.

### 1. Dataset Overview

The project utilized the `nova_pay_combined.csv` dataset, initially containing 11,400 transactions across 26 features. After rigorous cleaning, imputation, and removal of duplicates and invalid entries, the dataset was refined to **10,591 transactions**. The target variable `is_fraud` indicated a significant class imbalance, with approximately **9%** of transactions being fraudulent.

**Key Data Preprocessing Steps:**

- Handling missing values in `timestamp`, `amount_src`, `amount_usd`, `fee`, `ip_address`, `ip_country`, `kyc_tier`, and `device_trust_score`.
- Converting data types (`timestamp` to datetime, `amount_src` to float).
- Removing negative values in `amount_src`, `fee`, `device_trust_score`, and `txn_velocity_1h`.
- Standardizing categorical features like `channel` and `kyc_tier`.

The dataset was chronologically split into 80% training (8,472 samples) and 20% testing (2,119 samples) to prevent data leakage.

### 2. Model Performance Comparison

Several machine learning models were trained and evaluated for fraud detection, including Logistic Regression, Random Forest, XGBoost, and LightGBM. Both initial and hyperparameter-tuned versions were assessed, alongside experiments with SMOTE and Undersampling. The primary evaluation metrics were Accuracy, ROC AUC, Precision, Recall, F1-score for the fraud class, and False Positives/False Negatives.

The following table summarizes the performance of all models on the test set:

```
                    Model  Accuracy  ROC AUC  Precision (Fraud)  Recall (Fraud)  F1-score (Fraud)  False Positives  False Negatives
1           Random Forest    0.9882   0.9739             1.0000          0.9188            0.9575                0               25
5         XGBoost (Tuned)    0.9802   0.9795             0.9318          0.9318            0.9318               21               21
4        LightGBM (Tuned)    0.9750   0.9784             0.9147          0.9221            0.9184               29               24
6  LightGBM (SMOTE+Under)    0.9703   0.9773             0.8734          0.9286            0.9008               41               22
0     Logistic Regression    0.9566   0.9834             0.7951          0.9448            0.8635               75               17
7   XGBoost (SMOTE+Under)    0.9504   0.9792             0.7742          0.9318            0.8454               84               21
2       XGBoost (Initial)    0.9825   0.9700             0.9562          0.9221            0.9388               13               24
3      LightGBM (Initial)    0.9830   0.9737             0.9562          0.9221            0.9388               12               24
```

### 3. Best Model Performance and Choice

Considering the business context of NovaPay, minimizing false positives (incorrectly flagging legitimate transactions as fraud) is crucial to avoid inconveniencing customers. While Logistic Regression achieved the highest recall (0.94), it also had a higher number of false positives (75).

The **Random Forest Classifier (untuned)** emerged as the best model for this objective, demonstrating **perfect precision (1.00)** for the fraud class, meaning **zero false positives**. This is highly desirable in a production environment where customer experience is paramount. It achieved an Accuracy of **0.9882** and an ROC AUC Score of **0.9739**, with a recall of **0.9188** (25 false negatives).

Although LightGBM (Tuned) and XGBoost (Tuned) showed comparable overall performance and ROC AUC scores, the Random Forest's ability to eliminate false positives without significantly sacrificing recall makes it the most suitable choice for NovaPay's fraud detection system.

### 4. Top 5 Fraud Indicators (from SHAP Analysis)

SHAP (SHapley Additive exPlanations) analysis provided insights into the features most influential in the Random Forest model's predictions. The top 5 global fraud indicators, based on mean absolute SHAP values, are:

```
                   Feature  Importance
0            ip_risk_score    0.055760
1         txn_velocity_24h    0.054066
2        high_velocity_24h    0.049834
3      risk_score_internal    0.042258
4         account_age_days    0.039569
```

These features consistently had the largest impact on whether a transaction was predicted as fraudulent or legitimate. Specifically:

- **`ip_risk_score`** and **`risk_score_internal`**: Higher risk scores significantly increase the likelihood of fraud.
- **`txn_velocity_24h`** and **`txn_velocity_1h`**: High transaction frequency within short periods is a strong indicator of fraudulent activity.
- **`account_age_days`**: Newer accounts (lower `account_age_days`) are more susceptible to fraud.

### 5. Key Business Insights

Based on the Exploratory Data Analysis (EDA) and model interpretability, several actionable business insights were derived:

- **Behavioral Indicators are Crucial**: Transaction velocity (`txn_velocity_1h`, `txn_velocity_24h`), internal risk scores (`risk_score_internal`), and IP risk scores (`ip_risk_score`) are the strongest predictors of fraud, highlighting the importance of real-time behavioral monitoring.
- **Customer Journey Stages**: Newer accounts (less than 90 days old) exhibit significantly higher fraud rates, suggesting a need for enhanced scrutiny during customer onboarding and early transaction periods.
- **Channel Vulnerabilities**: The 'web' channel has a substantially higher fraud rate compared to 'ATM' and 'mobile', indicating potential weaknesses in online transaction security or authentication processes.
- **KYC Effectiveness**: Lower KYC tiers ('low') are associated with much higher fraud rates, confirming that robust KYC procedures are effective in mitigating fraud risk. Customers with 'enhanced' KYC tiers show significantly lower fraud.
- **Geographic Risk**: Transactions originating from Canada (CA) and the UK showed higher fraud rates compared to the US.
- **Location Mismatch**: A mismatch between `home_country` and `ip_country` strongly correlates with fraudulent activity.
- **Device Trust**: Low device trust scores are highly indicative of fraud, emphasizing the need for device fingerprinting and anomaly detection.
- **Transaction Amount**: Fraud rates peak for transactions between \$1,000 and \$5,000 USD.
- **Time of Day**: A spike in fraudulent transactions occurs during the early hours of the day (3-7 AM), suggesting late-night monitoring or heightened security during these times.

### 6. Deliverables Completed

1.  **Data Loading and Initial Exploration**: Loaded and performed initial sanity checks on the `nova_pay_combined.csv` dataset.
2.  **Data Cleaning and Preprocessing**: Handled missing values, corrected data types, removed duplicates, and standardized categorical features.
3.  **Exploratory Data Analysis (EDA)**: Identified key correlations, fraud rates across different categories (channel, KYC tier, country, time), and visualized distributions of numerical features by fraud status.
4.  **Feature Engineering**: Created new time-based (hour, day of week, is_weekend) and threshold-based (high velocity, IP risk, device trust, account age, web channel, KYC tier, risky home country, high amount, night hours, location mismatch) features.
5.  **Data Preparation for Modeling**: Performed one-hot encoding for categorical features, standardized numerical features, and conducted a chronological train-test split.
6.  **Model Training and Evaluation**: Trained and evaluated Logistic Regression, Random Forest, XGBoost, and LightGBM models.
7.  **Hyperparameter Tuning**: Optimized XGBoost and LightGBM models using `GridSearchCV`.
8.  **Class Imbalance Handling**: Experimented with `class_weight='balanced'` and `SMOTE` + `RandomUnderSampler`.
9.  **Model Comparison**: Structured comparison of all models based on key performance metrics.
10. **Model Interpretability (SHAP)**: Used SHAP values to identify global feature importance and provide individual transaction explanations.

### 7. Project Outcomes

This project successfully developed and evaluated multiple machine learning models for fraudulent transaction detection, culminating in the selection of an optimal model and the extraction of valuable business insights. The Random Forest Classifier was identified as the best model, offering **perfect precision for fraud detection**, which is a critical outcome for minimizing false alarms and maintaining customer trust. The SHAP analysis provided transparency into the model's decision-making process, pinpointing key fraud indicators that can inform future prevention strategies and rule-based systems. The comprehensive data cleaning, feature engineering, and rigorous model evaluation pipeline established a strong foundation for deployment and continuous improvement in NovaPay's fraud detection capabilities.
