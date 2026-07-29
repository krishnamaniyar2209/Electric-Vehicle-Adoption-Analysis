# 🚗 Electric Vehicle Adoption Analysis (Washington State)

Exploratory data analysis and predictive modeling on **200,048 Washington State electric-vehicle registration records**, covering adoption trends, manufacturer and geographic concentration, and BEV vs. PHEV range behavior.

> **Data Source:** [Electric Vehicle Population Data](https://data.wa.gov/Transportation/Electric-Vehicle-Population-Data/f6w7-q2d2) — Washington State Department of Licensing, published on data.wa.gov. 200,048 records, 17 columns. Real-world, publicly available registration data.

![EV Adoption Trend](images/ev-adoption-trend.png)

> Registrations climb steeply from 2015 and peak at model year 2023. The fall-off across 2024–2025 reflects **incomplete reporting for recent model years**, not a decline in adoption.

---

## 🎯 Objectives
- Analyze EV adoption trends over time
- Compare Battery Electric (BEV) vs. Plug-in Hybrid (PHEV) vehicles
- Identify leading manufacturers, models, and counties
- Examine electric-range behavior across vehicle types
- Build baseline models to predict EV type and electric range

---

## 🧹 Dataset & Preparation

| Stage | Records | Notes |
|---|---|---|
| Raw registration data | 200,048 | 17 columns as published by WA DOL |
| After cleaning | 200,044 | Dropped 6 identifier/location columns and 4 null-geography rows → 11 analysis columns |
| **Range-valid subset** | **90,643** | Rows where `Electric Range > 0` |

**109,401 records (54.7%) carry an electric range of `0`.** These are not zero-range vehicles — every one of them is flagged in the source data as *"Eligibility unknown as battery range has not been researched."* The counts match exactly (200,044 − 90,643 = 109,401), confirming the zero-range rows and the unresearched rows are the same set. All range and modeling work therefore runs on the 90,643-record range-valid subset.

### ⚠️ The range-valid subset is not a representative sample

This is the single most important caveat in the project, and it shapes how every subset statistic below should be read.

| | Full dataset | Range-valid subset | Retained |
|---|---|---|---|
| **BEV** | 156,956 | 47,553 | **30.3%** |
| **PHEV** | 43,092 | 43,090 | **~100%** |

The filter keeps virtually every PHEV but discards **70% of all BEVs**, because BEV ranges are far more often left unresearched by WA DOL. The subset is therefore heavily skewed toward PHEVs relative to the real fleet, and any composition statistic drawn from it will understate BEVs.

| Metric | Full dataset (200,044) | Range-valid subset (90,643) |
|---|---|---|
| BEV share | **78.5%** | 52.5% |
| Tesla registrations | **88,083** | 25,581 |
| King County share | **51.4%** | 48.6% |
| Unique makes / models / counties | 42 / 151 / 199 | 35 / 98 / 155 |
| Mean electric range | 53.5 mi | 118.0 mi |

Both columns are correct — they simply answer different questions. Population-level questions ("how many EVs are Teslas?") should be read from the left column; range-dependent questions from the right.

---

## 📊 Key Findings (EDA)

| Insight | Result | Basis |
|---|---|---|
| EV registrations | Grew sharply, accelerating after **2015**, peaking at model year 2023 | Full dataset |
| BEV vs. PHEV split | **78.5% BEV / 21.5% PHEV** | Full dataset |
| Top manufacturer | **Tesla (88,083)** — 44% of all registrations | Full dataset |
| Geographic concentration | **King County = 51.4%** of all EVs | Full dataset |
| Top makes by volume | Tesla (25,581), Nissan (10,677), Chevrolet (9,706) | Range-valid subset |
| Avg. electric range | **BEV 197 mi vs. PHEV 31 mi** | Range-valid subset |
| Longest average range by make | **Tesla 241 mi**, Jaguar 234 mi, Polestar 233 mi | Range-valid subset |

![EV Type Distribution](images/ev-type-distribution.png)
![Top Manufacturers](images/top-ev-manufacturers.png)
![Top Counties](images/top-counties-with-evs.png)
![Range by EV Type](images/electric-range-by-ev-type.png)
![Range Distribution](images/electric-range-distribution.png)

> The EV-type, manufacturer, county, and range charts above are all plotted on the range-valid subset. The 52.5 / 47.5 split shown in the pie chart is a property of that subset, not of Washington's EV fleet.

### Range is strongly bimodal

PHEVs cluster tightly at a median of **30 miles** (IQR 21–38); BEVs center on a median of **215 miles** (IQR 150–238). The two interquartile ranges do not overlap at all, though the extremes do — the shortest-range BEV is 29 miles and the longest-range PHEV reaches 153. That separation is what makes the classification task below far easier than it first appears.

---

## 🤖 Predictive Modeling

### Classification: Predicting EV Type (BEV vs. PHEV)

Features: `Make`, `Model`, `Model Year`, `City`, `County` (label-encoded).
Train/test split: 80/20 on the 90,643-record subset (18,129 test rows).

| Model | Accuracy |
|---|---|
| Logistic Regression | 82.3% |
| Decision Tree | 98.8% |
| **Random Forest** | **98.9%** |

![Model Comparison](images/classification-model-comparison.png)
![Confusion Matrix](images/random-forest-confusion-matrix.png)
![Feature Importance](images/random-forest-feature-importance.png)

**Interpretation — Make and Model very nearly determine EV type.** Random Forest feature importance puts `Model` at 51% and `Make` at 30%, 81% combined. This is a textbook case of strong feature-target dependence: a Tesla Model 3 is a BEV by definition and a Chrysler Pacifica Hybrid is a PHEV by definition, so the model is closer to reading a lookup table than learning a difficult boundary. The headline accuracy reflects that relationship, not model sophistication.

**A note on the logistic baseline.** The 82.3% figure is not a like-for-like comparison. Categorical features here are label-encoded as ordinal integers, which tree models are invariant to but logistic regression is not, since it reads arbitrary category codes as a continuous scale. One-hot encoding would give a fairer baseline. The gap is reported as-is for transparency rather than presented as evidence that trees are inherently stronger on this problem.

### Regression: Predicting Electric Range (baseline)

A Linear Regression baseline on `Model Year`, `Make`, `Model`, `County`, `City`, and `Base MSRP` explained only part of the variation in electric range:

| Metric | Value |
|---|---|
| R² | **0.265** |
| MAE | **73.45 mi** |
| RMSE | 85.05 mi |

Categorical vehicle attributes alone are weak predictors of range. Battery capacity, trim level, and model-year-specific specifications are absent from the source data and are likely where the missing signal sits. This serves as a baseline for future feature engineering rather than a usable predictor.

---

## ⚠️ Limitations & Next Steps

- **Selection bias in the range-valid subset.** The filter retains ~100% of PHEVs but only 30% of BEVs, so subset composition statistics are not fleet statistics. Every model in this project is trained on that skewed subset and inherits the bias.
- **Feature-target leakage in the classification task.** Make and Model effectively encode the label. A more meaningful version would exclude them and predict EV type from geography, model year, and MSRP alone.
- **Label encoding penalizes the linear baseline.** Re-running Logistic Regression with one-hot encoded categories would make the model comparison fair.
- **Range placeholders are excluded rather than imputed.** Any range statistic here describes the researched subset, not the full fleet.
- **No temporal validation.** Adoption is strongly time-trended; a time-based split would test generalization to newer model years more honestly than a random split.
- **Base MSRP is largely zero** in the source data (75th percentile = 0) and contributes almost nothing to the regression, so price effects remain untested.

---

## 🛠️ Tools & Technologies
`Python` · `Pandas` · `NumPy` · `Matplotlib` · `Seaborn` · `Scikit-learn` · `Jupyter Notebook`

---

## 📁 Repository Structure
```
├── ev-adoption-analysis-and-prediction.ipynb   # Full analysis: EDA, modeling, evaluation
├── data/
│   └── EV_Population_WA_Data.csv               # Download separately (see How to Run)
├── images/                                     # Exported charts
│   ├── ev-adoption-trend.png
│   ├── ev-type-distribution.png
│   ├── top-ev-manufacturers.png
│   ├── top-counties-with-evs.png
│   ├── electric-range-by-ev-type.png
│   ├── electric-range-distribution.png
│   ├── classification-model-comparison.png
│   ├── random-forest-confusion-matrix.png
│   └── random-forest-feature-importance.png
└── README.md
```

---

## 🚀 How to Run

1. **Download the dataset.** The CSV is not committed to this repository. Get it from [data.wa.gov — Electric Vehicle Population Data](https://data.wa.gov/Transportation/Electric-Vehicle-Population-Data/f6w7-q2d2) (Export → CSV) and save it as `data/EV_Population_WA_Data.csv`.

2. **Install dependencies:**
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn jupyter
   ```

3. **Set the path.** In the data-loading cell, point to your local copy:
   ```python
   file_path = "data/EV_Population_WA_Data.csv"
   ```

4. **Run all cells** to reproduce the analysis and models.

> The notebook was originally developed in Google Colab, where the load path was `/content/EV_Population_WA_Data.csv`. Update it as shown above to run locally.
>
> **Note on reproducibility:** WA DOL refreshes this dataset regularly, so a newly downloaded copy will contain more records than the 200,048 analyzed here and the exact figures will differ.

---

## 👤 Author
**Krishna Maniyar**, Data Analyst
📧 maniyarkrishnakm22@gmail.com · [LinkedIn](https://www.linkedin.com/in/krishnamaniyar2209/) · [Portfolio](https://krishnamaniyar2209.github.io/)
