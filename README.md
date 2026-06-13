# 🚗 Electric Vehicle Adoption Analysis (Washington State)

Exploratory data analysis and predictive modeling on **200K+ Washington State
electric-vehicle registration records**, uncovering adoption trends, manufacturer
and geographic concentration, and BEV vs. PHEV range differences.

> **Data Source:** Washington State Department of Licensing — EV Population dataset
> (200,048 records, 17 columns). Real-world, publicly available registration data.

![EV Adoption Trend](images/ev-adoption-trend.png)

---

## 🎯 Objectives
- Analyze EV adoption trends over time
- Compare Battery Electric (BEV) vs. Plug-in Hybrid (PHEV) vehicles
- Identify leading manufacturers, models, and counties
- Examine electric-range behavior across vehicle types
- Build baseline models to predict EV type and electric range

---

## 📊 Key Findings (EDA)

| Insight | Result |
|---|---|
| EV registrations | Grew sharply, accelerating after **2015** |
| BEV vs. PHEV split | **52.5% BEV / 47.5% PHEV** (range-valid subset) |
| Top manufacturer | **Tesla** (25,581), then Nissan (10,677), Chevrolet (9,706) |
| Geographic concentration | **King County = 48.6%** of all EVs |
| Avg. electric range | **BEV 197 mi vs. PHEV 31 mi** |
| Dataset coverage | 35 makes · 98 models · 155 counties |

![EV Type Distribution](images/ev-type-distribution.png)
![Top Manufacturers](images/top-ev-manufacturers.png)
![Top Counties](images/top-counties-with-evs.png)
![Range by EV Type](images/electric-range-by-ev-type.png)

---

## 🤖 Predictive Modeling

### Classification — Predicting EV Type (BEV vs. PHEV)
| Model | Accuracy |
|---|---|
| Logistic Regression | 82.3% |
| Decision Tree | 98.8% |
| **Random Forest** | **98.9%** |

![Model Comparison](images/classification-model-comparison.png)
![Feature Importance](images/random-forest-feature-importance.png)

> **Interpretation:** Tree-based models reach very high accuracy largely because
> **Make and Model nearly determine EV type** (Model = 51%, Make = 30% importance).
> This is a textbook case of strong feature–target dependence — the high accuracy
> reflects that relationship rather than a difficult prediction task.

### Regression — Predicting Electric Range (baseline)
A Linear Regression baseline using vehicle attributes explained only part of the
variation in electric range (**R² = 0.27, MAE ≈ 73 mi**), indicating that the
selected categorical features alone are weak predictors of range — a useful
baseline for future feature engineering.

---

## 🛠️ Tools & Technologies
`Python` · `Pandas` · `NumPy` · `Matplotlib` · `Seaborn` · `Scikit-learn` · `Jupyter Notebook`

---

## 🚀 How to Run
1. Open `ev-adoption-analysis-and-prediction.ipynb`
2. Install the required libraries (`pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`)
3. Run all cells to reproduce the analysis and models

---

## 👤 Author
**Krishna Maniyar** — Data Analyst
📧 krishnamaniyarkm22@gmail.com · [LinkedIn](https://www.linkedin.com/in/krishnamaniyar/) · [Portfolio](https://krishnamaniyar2209.github.io/)
