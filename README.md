[README.md](https://github.com/user-attachments/files/27638739/README.md)
# 🎮 Gaming & Mental Health Analytics Dashboard

> *Where Gaming Meets Well-Being*

## 📋 Overview

This is a **comprehensive data analytics graduation project** developed as part of the **Digital Egypt Pioneers Initiative (DEPI) — Data Analysis Track**. It explores how gaming habits, behaviors, and platform usage patterns correlate with mental health, sleep quality, physical well-being, and productivity across a large-scale population of gamers.

The full pipeline was built on **Microsoft Fabric**, following the **Medallion Architecture (Bronze → Silver → Gold)**, and culminates in an interactive **Power BI dashboard** designed to surface actionable insights for researchers and the general public alike.

**Key Focus Areas:**

- Gaming behavior analysis (hours, sessions, addiction, night gaming)
- Mental health impact (stress, anxiety, depression, loneliness)
- Sleep & physical health consequences
- Productivity & academic performance
- **ML-powered Addiction Level Predictor (XGBoost — R² = 0.80)**

---

## 👥 Project Team

| Name |
|---|
| Alia Amir Ali |
| Esraa Abu Bakr |
| Manar Magdy |
| Maha Saleh |
| Menna Mohamed |
| Rofyda Sayed |

---

## 📊 Dataset

### Gaming and Mental Health Dataset

| Attribute | Detail |
|---|---|
| **Dataset Name** | Gaming and Mental Health Dataset |
| **Source Platform** | Kaggle |
| **Publisher** | sharmajicoder |
| **Dataset URL** | [kaggle.com/datasets/sharmajicoder/gaming-and-mental-health](https://www.kaggle.com/datasets/sharmajicoder/gaming-and-mental-health) |
| **Data Type** | Synthetic / Simulated Survey Data |
| **Total Records** | 1,000,000 rows |
| **File Format** | CSV (converted to Parquet for processing) |
| **License** | Open / Public (Kaggle) |

The dataset contains **39 raw columns** spanning five thematic domains:

- **Demographics** — age, gender, income
- **Gaming Behavior** — daily hours, weekly sessions, platform ratios, rank
- **Mental Health** — stress, anxiety, depression, loneliness, addiction
- **Physical Health** — sleep, exercise, BMI, eye strain, back pain
- **Social & Productivity** — social interaction, academic performance, work productivity

---

## 🛠️ Tools & Technologies

| Component | Technology | Purpose |
|---|---|---|
| **Data Engineering** | Microsoft Fabric + PySpark | End-to-end Medallion pipeline |
| **Data Architecture** | Medallion Architecture (Bronze → Silver → Gold) | Structured, scalable data pipeline |
| **Machine Learning** | Scikit-learn, XGBoost, MLflow | Addiction Level Prediction |
| **Visualization** | Power BI | Interactive 5-page dashboard |
| **Data Modeling** | Star Schema (1 Fact + 5 Dimensions) | Optimized relational design |
| **Dashboard Design** | Figma | UI/UX wireframing & layout planning |

---

## 🔄 Architecture & Steps

### Step 1: Data Pipeline (Medallion Architecture)

```
Bronze Layer → Silver Layer → Gold Layer
  (Raw)         (Cleaned)      (Refined)
```

#### 🥉 Bronze Layer — [`bronze_layer.ipynb`](https://github.com/cherrys21/DEPI_Project/blob/main/Medallion_architecture/Bronze/bronze_layer.ipynb)

- Raw CSV ingested from Kaggle via `kagglehub`
- Converted to PySpark DataFrame
- Saved as Parquet to `Files/Bronze/Gaming_Bronze` — no transformations applied
- Permanent unmodified archive of source data

#### 🥈 Silver Layer — [`silver_layer.ipynb`](https://github.com/cherrys21/DEPI_Project/blob/main/Medallion_architecture/Silver/silver_layer.ipynb)

- Three targeted cleaning corrections applied (zero rows dropped):

| Column | Issue | Fix | Threshold |
|---|---|---|---|
| `sleep_hours` | Negative values (physically impossible) | Replace with 0 | Min = 0 |
| `daily_gaming_hours` | Values > 18 hrs/day (implausible) | Cap at upper limit | Max = 18 hrs |
| `weekend_gaming_hours` | Values > 36 hrs/weekend (implausible) | Cap at upper limit | Max = 36 hrs |

- Six composite KPI columns engineered (see Feature Engineering below)
- `User_ID` surrogate key assigned via `row_number()` window function
- Output saved to `Files/Silver/Gaming_Silver`

#### 🥇 Gold Layer — [`gold_layer.ipynb`](https://github.com/cherrys21/DEPI_Project/blob/main/Medallion_architecture/Gold/gold_layer.ipynb)

- Star schema built from enriched Silver DataFrame
- All tables saved as **Delta format** for ACID compliance and direct Power BI connectivity

---

### Step 2: Data Modeling (Star Schema)

The Gold layer implements one central **Fact Table** joined to five **Dimension Tables** via the `User_ID` surrogate key:

```
                    ┌─────────────────┐
                    │ fact_gaming_    │
         ┌──────────│    metrics      │──────────┐
         │          └────────┬────────┘          │
         ▼                   │                   ▼
  dim_demographics     dim_health          dim_gaming
  (age, gender,        (sleep, BMI,        (ratios, rank,
   income)              Risk_Level)         experience)

         ┌──────────────────────────────────────────┐
         ▼                                          ▼
    dim_social                                 dim_impact
  (interaction score,                     (impact_category:
   relationships,                          Low/Moderate/High)
   friend counts)
```

| Table | Type | Key Columns | Description |
|---|---|---|---|
| `fact_gaming_metrics` | Fact | `fact_id`, `User_ID`, all FK keys | All quantitative KPI metrics |
| `dim_demographics` | Dimension | `dim_demo_id`, `User_ID` | Age, gender, income |
| `dim_health` | Dimension | `dim_health_id`, `User_ID` | Sleep, exercise, BMI, Risk_Level, is_balanced_gamer |
| `dim_gaming` | Dimension | `dim_gaming_id`, `User_ID` | Gaming behavior ratios, rank, headset, internet quality |
| `dim_social` | Dimension | `dim_social_id`, `User_ID` | Social interaction, relationship satisfaction, friend counts |
| `dim_impact` | Dimension | `dim_impact_id`, `User_ID` | Engineered impact_category (Low / Moderate / High) |

---

### Step 3: Feature Engineering

Six composite KPI columns were engineered in the Silver layer to capture behavioral patterns not observable from any single raw field:

| KPI | Description | Formula / Threshold |
|---|---|---|
| **Mental Health Risk Index** | Z-score composite of stress, anxiety, depression & loneliness | Avg Z-score of 4 components |
| **Risk Level** | Categorical segmentation | Low ≤ -0.5 │ Moderate: -0.5–0.5 │ High > 0.5 |
| **Gaming Impact Score** | 0–100 negative life impact score | Normalized avg of gaming hours + mental risk + inverted sleep + inverted productivity |
| **Impact Category** | Segmentation of impact score | Low: 0–30 │ Moderate: 31–60 │ High: 61–100 |
| **Balanced Gamer Flag** | Users meeting all 4 healthy thresholds simultaneously | ≤4 hrs gaming + ≥7 hrs sleep + ≥1 hr exercise + social score ≥ 5 |
| **User_ID** | Surrogate key | Sequential `row_number()` over full dataset |

---

### Step 4: Machine Learning

Beyond descriptive analytics, three ML experiments were conducted on **Microsoft Fabric** using Python (scikit-learn, XGBoost, MLflow). All models used an 80/20 train/test split (800K / 200K rows). All four notebooks were orchestrated in a single automated **Fabric Data Pipeline** named `Gaming_And_Mental_Health_Pipeline`.

**Pipeline Run Log (09 May 2026):**

| Activity | Status | Duration |
|---|---|---|
| Bronze | ✅ Succeeded | 1m 16s |
| Silver | ✅ Succeeded | 1m 51s |
| Gold | ✅ Succeeded | 1m 21s |
| Addiction_Lvl_ML_Model | ✅ Succeeded | 18m 40s |

#### Experiment Summary

| # | Experiment | Target | Type | Outcome |
|---|---|---|---|---|
| 1 | Risk Level Classification | Risk_Level (Low/Moderate/High) | Classification | Best: XGBoost F1 = 0.55 (after removing data-leakage features + SMOTE) |
| 2 | Happiness Score Regression | happiness_score (1–10) | Regression | **Halted at EDA** — max correlation with all features = 0.00175 (synthetic target) |
| 3 | Addiction Level Regression *(Primary)* | addiction_level (0–10) | Regression | **XGBoost selected** — R² = 0.7984 |

#### Model Results — Addiction Level ([`Addiction_Level_ML_Model.ipynb`](https://github.com/cherrys21/DEPI_Project/blob/main/ML_Models/Addiction_level_ML/Addiction))

| Model | RMSE | MAE | R² | Selected |
|---|---|---|---|---|
| Linear Regression | 0.9543 | 0.7659 | 0.7956 | |
| Random Forest | 0.9545 | 0.7669 | 0.7955 | |
| **XGBoost** | **0.9475** | **0.7608** | **0.7984** | ✅ **Best** |

**Why XGBoost?**
- Explains ~80% of variance in addiction level — strong result for behavioral survey data
- RMSE of 0.95 on a 0–10 scale means predictions are within ~1 point of actuals on average
- Sequential tree building corrects prior errors; well-suited for tabular feature interactions
- L1/L2 regularization reduces overfitting on high-dimensional feature sets
- Final model registered in MLflow as `Gaming_Addiction_XGBoost v1`; predictions saved to `Tables/Gold/ml_addiction_predictions`

#### Top Predictive Features (SHAP Analysis — 10,000-row sample)

| Rank | Feature | Importance Score | Interpretation |
|---|---|---|---|
| 1 | `daily_gaming_hours` | ~0.97 | Dominant predictor — single largest driver by a wide margin |
| 2 | `screen_time_total` | ~0.01 | Secondary contributor |
| 3–N | All remaining features | < 0.01 each | Minimal contribution once gaming hours are accounted for |

#### Forecast / What-If Model

A Power BI DAX what-if simulation measure was implemented to serve as the forward-looking forecast component, enabling stakeholders to ask: *"If users in this segment reduced daily gaming by 20%, by how much would their Balanced Gamer Score improve?"*

```dax
-- Simulated Balanced Score (What-If Forecast Measure)
Simulated_Balanced_Score =
    [Balanced Gamer Score] + ([Avg Gaming Hours] * [Reduction % Value] * 0.5)
```

---

## 📈 Power BI Dashboard (5 Pages)

### Page 1: Executive Overview
High-level project snapshot — Avg Gaming Hours, High Risk %, Mental Risk Index, Gaming Impact Score

### Page 2: Gaming Behavior Analysis
Gaming habits & behavioral patterns — Avg Gaming Hours, High Gaming %, Night Gamers %, Avg Weekly Sessions

### Page 3: Mental Health Analysis
Psychological impact — Mental Risk Index, Avg Depression, High Depression %, Happiness Score, Balanced Gamer Score

### Page 4: Sleep & Physical Health
Physical consequences — Avg Sleep Hours, Sleep Deprivation %, Avg Exercise, Digital Strain Index, Avg BMI

### Page 5: Productivity & Academic Impact
Performance output — Academic Performance, Work Productivity, Productivity Risk

---

## 📐 Key KPIs

### Gaming Behavior
| KPI | Definition |
|---|---|
| Avg Gaming Hours | Mean daily gaming hours across all users |
| High Gaming % | % of users gaming > 5 hrs/day |
| Night Gamers % | % of sessions occurring at night |
| Avg Weekly Sessions | Mean weekly gaming sessions |
| Addiction Category | Low / Moderate / High (from `addiction_level` thresholds) |

### Mental Health
| KPI | Definition |
|---|---|
| Mental Risk Index | Z-score composite of stress, anxiety, depression, loneliness |
| Risk Level | Low / Moderate / High classification |
| High Risk % | % of users classified as High Risk |
| Avg Depression | Mean `depression_score` across population |
| Balanced Gamer Score | % meeting all 4 healthy lifestyle thresholds |

### Sleep & Physical Health
| KPI | Definition |
|---|---|
| Avg Sleep Hours | Mean nightly sleep (corrected negative values → 0) |
| Sleep Deprivation % | % of users sleeping < 6 hrs/night |
| Avg Exercise Hours | Mean weekly physical exercise |
| Digital Strain Index | Composite eye + back pain score, normalized 0–1 |

### Productivity & Academic
| KPI | Definition |
|---|---|
| Gaming Impact Score | 0–100 composite negative life impact score |
| Academic Performance | Mean `academic_performance` score |
| Work Productivity | Mean `work_productivity` score |
| Productivity Risk | Flag for low performance + high gaming simultaneously |

---

## 🔬 SMART Research Questions

| # | Question | KPIs Measured |
|---|---|---|
| 1 | Does average daily gaming exceed 4 hrs, and does it correlate with elevated mental health risk? | Avg Gaming Hours, Mental Risk Index |
| 2 | What % of users show simultaneously elevated addiction, depression, and sleep deprivation — segmented by age/gender? | High Risk %, Avg Depression, Sleep Deprivation % |
| 3 | Is there a measurable productivity difference between users gaming > 5 hrs vs < 2 hrs/day? | Academic Performance, Work Productivity, High Gaming % |
| 4 | What proportion qualify as Balanced Gamers, and which demographic has the highest rate? | Balanced Gamer Score, Night Gamers % |
| 5 | Does Digital Strain Index increase proportionally with screen time, and is this stronger in night gamers? | Digital Strain Index, Night Gamers % |

---

## 📁 Project File Structure

```
DEPI_Project/
├── Medallion_architecture/
│   ├── Bronze/
│   │   └── bronze_layer.ipynb          # Raw ingestion from Kaggle → Parquet
│   ├── Silver/
│   │   └── silver_layer.ipynb          # Cleaning + Feature Engineering
│   └── Gold/
│       └── gold_layer.ipynb            # Star schema Delta tables
├── ML_Models/
│   ├── Risk_level_ML/
│   │   └── Risk_Level_ML_Model.ipynb   # Risk classification + SMOTE
│   ├── Happiness_ML/
│   │   └── Happiness_ML_Model.ipynb    # Halted at EDA (correlation ≈ 0)
│   └── Addiction_level_ML/
│       └── Addiction_Level_ML_Model.ipynb  # Production XGBoost model
└── README.md
```

---

## 🚀 How to Run

### Prerequisites

- Microsoft Fabric workspace (with Spark compute enabled)
- Power BI Desktop
- Python libraries: `kagglehub`, `pyspark`, `scikit-learn`, `xgboost`, `mlflow`, `shap`

### Execution Steps

#### Step 1: Run Bronze Layer
```bash
# Open in Microsoft Fabric Notebook
Medallion_architecture/Bronze/bronze_layer.ipynb
```
Downloads the Kaggle dataset and saves raw Parquet to `Files/Bronze/Gaming_Bronze`.

#### Step 2: Run Silver Layer
```bash
Medallion_architecture/Silver/silver_layer.ipynb
```
Applies 3 cleaning fixes and engineers 6 composite KPI columns. Outputs to `Files/Silver/Gaming_Silver`.

#### Step 3: Run Gold Layer
```bash
Medallion_architecture/Gold/gold_layer.ipynb
```
Builds the star schema (5 dimensions + 1 fact) and writes Delta tables to `Tables/Gold/`.

#### Step 4: Run ML Model
```bash
ML_Models/Addiction_level_ML/Addiction_Level_ML_Model.ipynb
```
Trains Linear Regression, Random Forest, and XGBoost; logs all runs to MLflow; registers the best model; saves predictions to `Tables/Gold/ml_addiction_predictions`.

#### Step 5: Open Power BI Dashboard
```
# Connect Power BI to Microsoft Fabric Gold layer Delta tables
# Open the .pbix file and refresh the data source
```

> **Tip:** All four notebooks can be run automatically via the **`Gaming_And_Mental_Health_Pipeline`** Fabric Data Pipeline in the correct sequence.

---

## 📊 Key Findings & Insights

### Gaming Behavior
- Daily gaming hours are the single dominant predictor of addiction level (SHAP importance ~0.97)
- Night gaming patterns strongly correlate with sleep deprivation metrics

### Mental Health
- The Mental Health Risk Index reveals a meaningful cluster of high-risk users with simultaneously elevated addiction, depression, and loneliness scores
- Only a subset of users qualify as Balanced Gamers — meeting all four healthy thresholds simultaneously

### Sleep & Physical Health
- Negative `sleep_hours` values were corrected to 0; implausible gaming hour extremes were capped before any analysis
- The Digital Strain Index (composite eye + back pain) scales with total screen time, with a stronger effect among night gamers

### Machine Learning
- XGBoost achieves **R² = 0.7984** — approximately 80% of variance in addiction level explained by behavioral features alone
- The Happiness experiment was transparently halted after EDA confirmed zero predictive signal (max correlation 0.00175), demonstrating data integrity over forced results
- Risk Level classification reached F1 = 0.55 after careful removal of data-leakage components and SMOTE balancing for the 67% Moderate Risk class imbalance

---

## 🎓 Learning Outcomes

| Skill | Application |
|---|---|
| **Data Pipeline Design** | Medallion Architecture (Bronze → Silver → Gold) on Microsoft Fabric |
| **Data Modeling** | Star schema with 1 fact table and 5 dimension tables |
| **Data Engineering** | PySpark transformations, Delta format, ACID compliance |
| **Machine Learning** | XGBoost regression, SHAP feature importance, MLflow tracking |
| **Statistical Analysis** | Z-score normalization, KPI derivation, feature leakage detection |
| **Data Visualization** | 5-page interactive Power BI dashboard with slicers and KPI cards |
| **Dashboard Design** | Figma wireframing for layout, color palette, and storytelling flow |
| **Pipeline Orchestration** | Microsoft Fabric Data Pipeline (4-notebook automated run) |

---

## 📄 Data Source & Attribution

**Dataset:** [Gaming and Mental Health Dataset](https://www.kaggle.com/datasets/sharmajicoder/gaming-and-mental-health) by sharmajicoder on Kaggle (Open / Public license)

**Tools:** Microsoft Fabric · Power BI · Figma · Python (PySpark, scikit-learn, XGBoost, MLflow, SHAP)

**Initiative:** Digital Egypt Pioneers Initiative (DEPI) — Data Analysis Track · 2025 / 2026

---

## ✨ Summary

This **Gaming & Mental Health Analytics Dashboard** delivers:

🎯 **Complete 3-Layer Data Pipeline** — Bronze, Silver, Gold via dedicated PySpark notebooks on Microsoft Fabric  
📊 **5 Interactive Power BI Dashboard Pages** — Gaming Behavior, Mental Health, Sleep & Physical Health, Productivity, Executive Overview  
🤖 **ML Addiction Level Predictor** — XGBoost with R² = 0.7984, registered in MLflow  
🔬 **Transparent ML Methodology** — Leakage detection, SMOTE balancing, SHAP analysis, and one experiment halted at EDA  
🚀 **Automated Fabric Pipeline** — All 4 notebooks orchestrated end-to-end in a single run
