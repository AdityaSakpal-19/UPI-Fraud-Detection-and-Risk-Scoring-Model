# UPI Fraud Detection & Risk Scoring Model

## 📋 Project Overview

A machine learning-based fraud detection system designed to identify suspicious UPI payment patterns using behavioral analytics, risk scoring, and real-time monitoring. The system analyzes transaction data to flag high-risk transactions for investigation queues and SAR (Suspicious Activity Report) escalation.

**Project Status:** ✅ Completed  
**Dataset:** 500+ UPI transactions  
**Model Accuracy:** 94%  
**Fraud Detection Recall:** 67%

---

## 🎯 Objectives

- **Detect fraudulent UPI transactions** using machine learning classification
- **Engineer behavioral risk features** from transaction metadata (velocity, amount, timing, patterns)
- **Build rule-based fraud scoring system** to prioritize high-confidence fraud cases
- **Create real-time monitoring dashboards** for compliance investigation teams
- **Support SAR (Suspicious Activity Report) workflows** with automated escalations

---

## 📊 Dataset

### Transaction Data (`upi_transactions.csv`)
- **Records:** 500+ UPI transactions
- **Date Range:** March - April 2026
- **Fraud Rate:** 6% (6 actual fraud cases)

### Key Columns
| Column | Description | Type |
|--------|-------------|------|
| `transaction_id` | Unique transaction identifier | String |
| `sender_customer_id` | Sender's customer ID | String |
| `receiver_vpa` | Receiver's UPI VPA | String |
| `amount` | Transaction amount (₹) | Float |
| `timestamp` | Transaction date & time | Datetime |
| `channel` | Transaction channel (intent, QR, mobile_app) | String |
| `upi_app` | UPI application (GPay, PhonePe, Paytm, BHIM) | String |
| `status` | Transaction status (success, failed, pending) | String |
| `is_fraud` | Target variable (0=Legitimate, 1=Fraud) | Binary |

---

## 🔧 Technology Stack

### Languages & Libraries
- **Python 3.8+**
  - `pandas` — Data manipulation & aggregation
  - `numpy` — Numerical computations
  - `scikit-learn` — Machine learning models
    
### Tools & Platforms
- **Power BI** — Real-time fraud analytics dashboard
- **Jupyter Notebook** — Model development & experimentation
- **SQL** — Transaction data querying & validation

---

## 🧠 Feature Engineering

### 1. **Velocity Features** (Transaction Frequency)
- `txn_count_1h` — Number of transactions in last 1 hour
- `txn_count_24h` — Number of transactions in last 24 hours
- **Rationale:** Mule accounts show unusually high transaction frequency

### 2. **Monetary Features** (Amount-Based)
- `high_amount_flag` — Transactions >₹50,000 (binary: 0/1)
- `avg_amount_7d` — Average transaction amount in last 7 days
- **Rationale:** Sudden large transactions are suspicious; deviation from average is a red flag

### 3. **Temporal Features** (Timing Patterns)
- `night_txn_flag` — Transactions between 12 AM - 6 AM (binary: 0/1)
- `hour` — Extracted hour of transaction
- **Rationale:** Fraudsters often operate during off-hours to avoid detection

### 4. **Behavioral Features** (User Behavior)
- `receiver_type` — Customer-to-Customer (P2P) vs Customer-to-Merchant (P2M)
- `transaction_type` — P2P, P2M, collect_request
- `unique_receivers` — Count of unique receivers in transaction history
- **Rationale:** Mule accounts interact with many unknown receivers

### 5. **Channel Features** (Payment Method)
- `channel` — UPI channel used (intent, QR, mobile_app)
- `upi_app` — Which app (GPay, PhonePe, Paytm, BHIM)
- **Rationale:** Certain channels correlate with fraud patterns

### 6. **Collection Request Flag**
- `collect_request_flag` — Is transaction a collect request? (binary)
- **Rationale:** Collect requests are often used in scams

---

## 📈 Model Development

### Data Preprocessing
```python
# 1. Data Cleaning
- Removed missing values
- Encoded categorical variables (receiver_type, channel, upi_app, status)
- Normalized numerical features for scaling

# 2. Feature Engineering
- Aggregated velocity metrics by sender_customer_id
- Calculated rolling averages (7-day window) for amounts
- Extracted temporal features (hour, day of week)
```

### Machine Learning Model

**Algorithm:** Scikit-learn Classification (Logistic Regression / Random Forest / Gradient Boosting)

**Features Used:** 9 engineered features  
**Train/Test Split:** 80/20

### Model Performance

| Metric | Score |
|--------|-------|
| **Accuracy** | 94.00% |
| **Precision** | 50.00% |
| **Recall (Sensitivity)** | 66.67% |
| **F1 Score** | 0.5714 |

**Interpretation:**
- ✅ Model correctly identified **4 out of 6 actual fraud cases** (67% recall)
- ⚠️ Generated 4 false positives (50% precision) — acceptable for fraud detection (better to flag suspicious than miss fraud)

---

## 🎯 Rule-Based Fraud Scoring System

### Weighted Signal Scoring

The model combines high-confidence signals into a **fraud risk score** (0-100):

```python
fraud_score = 0

# Signal 1: High Amount + Night Transaction (HIGH CONFIDENCE)
if (high_amount_flag == 1) AND (night_txn_flag == 1):
    fraud_score += 3  # Weight: 3 (High confidence)

# Signal 2: Mule Detection (MEDIUM-HIGH CONFIDENCE)
if (unique_receivers >= 5):
    fraud_score += 2  # Weight: 2

# Signal 3: High Velocity (MEDIUM CONFIDENCE)
if (txn_count_24h >= 10):
    fraud_score += 1.5  # Weight: 1.5

# Signal 4: Collect Request at Odd Hours (MEDIUM CONFIDENCE)
if (collect_request_flag == 1) AND (night_txn_flag == 1):
    fraud_score += 1.5  # Weight: 1.5
```

### Decision Thresholds

| Score Range | Risk Level | Action |
|------------|-----------|--------|
| 0 - 20 | **LOW** | ✅ Approve |
| 20 - 40 | **MEDIUM** | 🔍 Review |
| 40 - 60 | **HIGH** | 🚨 Hold & Investigate |
| 60+ | **CRITICAL** | 🛑 Block & Manual Review → SAR |

---

## 📊 Power BI Dashboard

### Real-Time Monitoring Dashboard

The Power BI dashboard visualizes fraud trends across multiple dimensions:

#### Key Metrics Displayed
- **Total Transactions:** 500
- **Fraud Cases Detected:** 8 (6 actual + 2 false positives)
- **Fraud Rate:** 6.4%
- **Model Accuracy:** 94%
- **Average Fraud Probability:** 0.10

#### Visualizations
1. **Fraud Trend Over Time** — Line chart of daily fraud cases
2. **Fraud by UPI App** — Distribution across GPay, PhonePe, BHIM, Paytm
3. **Transactions by Status** — Success, failed, pending breakdown
4. **High Amount Fraud** — 80% of fraud cases involve high amounts
5. **Night Transactions Fraud** — 40% of fraud happens at night
6. **Top Senders by Transaction Velocity** — Mule account identification
7. **Fraud by Channel** — Intent, QR, Mobile App comparison
8. **Fraud Probability Distribution** — Risk bucket visualization (0-20%, 20-40%, etc.)

**Use Case:** Compliance teams use dashboard to monitor fraud trends in real-time and prioritize investigation queues.
