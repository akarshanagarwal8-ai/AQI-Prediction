# Air Quality Prediction (PM 2.5) 🌫️

A machine learning project that predicts **PM 2.5 air quality levels** using meteorological data scraped from [tutiempo.net](https://en.tutiempo.net/climate). Multiple regression models are trained, compared, and the best model is deployed via a Flask web application.

---

## 📁 Project Structure

```
├── Data/
│   ├── AQI/                    # Raw hourly PM 2.5 AQI CSV files (2013–2016)
│   ├── Html_Data/              # Scraped HTML weather pages (by year/month)
│   └── Real-Data/              # Processed CSVs per year + combined dataset
│
├── app.py                      # Flask web app for PM 2.5 prediction
├── dataextractfile.py          # Web scraper for weather HTML pages
├── Html_script.py              # Alternate HTML retrieval script
├── Extract_combine.py          # Combines meteorological + AQI data into CSVs
├── Plot_AQI.py                 # Computes daily average PM 2.5 from hourly data
│
├── Linear_Regression.ipynb
├── Ridge_and_Lasso_Regression.ipynb
├── Decision_Tree_.ipynb
├── Random_Forest__Xgboost_and_Knn_Regressor.ipynb
│
└── random_forest_regression_model.pkl   # Saved best model (Random Forest)
```

---

## 📊 Dataset

- **Source:** [tutiempo.net](https://en.tutiempo.net/climate/ws-421820.html) (weather) + Government AQI data (PM 2.5)
- **Period:** 2013 – 2016
- **Target Variable:** `PM 2.5` (daily average, µg/m³)

### Features (Independent Variables)

| Column | Description |
|--------|-------------|
| T | Average temperature (°C) |
| TM | Maximum temperature (°C) |
| Tm | Minimum temperature (°C) |
| SLP | Sea-level pressure (hPa) |
| H | Average relative humidity (%) |
| VV | Average visibility (km) |
| V | Average wind speed (km/h) |
| VM | Maximum wind speed (km/h) |

---

## ⚙️ Pipeline

### Step 1 — Data Collection
Run `Html_script.py` (or `dataextractfile.py`) to scrape monthly weather HTML pages from tutiempo.net for 2013–2018 and save them under `Data/Html_Data/<year>/<month>.html`.

```bash
python Html_script.py
```

### Step 2 — AQI Processing
`Plot_AQI.py` reads hourly PM 2.5 CSV files and computes a **daily average** for each year (2013–2016).

### Step 3 — Data Combination
`Extract_combine.py` parses the HTML files with BeautifulSoup, merges meteorological readings with the daily PM 2.5 averages, and writes annual CSVs and a combined `Real_Combine.csv` to `Data/Real-Data/`.

```bash
python Extract_combine.py
```

### Step 4 — Model Training
Open the notebooks in order to train and evaluate models:

1. `Linear_Regression.ipynb`
2. `Ridge_and_Lasso_Regression.ipynb`
3. `Decision_Tree_.ipynb`
4. `Random_Forest__Xgboost_and_Knn_Regressor.ipynb`

Each notebook follows the same workflow:
- Load `Real_Combine.csv`
- Handle null values
- Correlation heatmap & feature importance (ExtraTreesRegressor)
- Train/test split (70/30)
- Model training + cross-validation
- Hyperparameter tuning (GridSearchCV / RandomizedSearchCV)
- Evaluation: MAE, MSE, RMSE

### Step 5 — Deployment
```bash
python app.py
```
Opens a Flask server at `http://127.0.0.1:5000`. Enter the 8 meteorological values to get a predicted PM 2.5 concentration.

---

## 🤖 Models Compared

| Model | Notes |
|-------|-------|
| Linear Regression | Baseline model |
| Ridge Regression | L2 regularization; best alpha via GridSearchCV |
| Lasso Regression | L1 regularization; best alpha via GridSearchCV |
| Decision Tree | Prone to overfitting; tuned via GridSearchCV |
| **Random Forest** ⭐ | **Best performer**; tuned via RandomizedSearchCV |
| XGBoost | Gradient boosting; tuned via RandomizedSearchCV |
| KNN Regressor | Tuned for optimal K (1–39) |

The **Random Forest Regressor** (after hyperparameter tuning) gave the best results and is saved as `random_forest_regression_model.pkl` for deployment.

---

## 🚀 Running the Web App

### Prerequisites

```bash
pip install flask numpy scikit-learn xgboost beautifulsoup4 pandas matplotlib seaborn lxml
```

### Start the server

```bash
python app.py
```

Navigate to `http://127.0.0.1:5000`, fill in the weather parameters, and click **Predict** to see the estimated PM 2.5 value.

---

## 📦 Dependencies

| Package | Purpose |
|---------|---------|
| `pandas`, `numpy` | Data manipulation |
| `scikit-learn` | All regression models + evaluation |
| `xgboost` | XGBoost regressor |
| `matplotlib`, `seaborn` | Visualization |
| `beautifulsoup4`, `lxml` | HTML parsing |
| `requests` | Web scraping |
| `flask` | Web deployment |
| `pickle` | Model serialization |

---

## 📈 Evaluation Metrics

All models are evaluated using:
- **MAE** — Mean Absolute Error
- **MSE** — Mean Squared Error  
- **RMSE** — Root Mean Squared Error
- **R²** — Coefficient of Determination

---

## 📝 Notes

- The AQI data files (`aqi2013.csv` – `aqi2016.csv`) must be present in `Data/AQI/` before running `Extract_combine.py`.
- The scraper targets weather station `ws-421820` (New Delhi area).
- Invalid AQI readings (`NoData`, `PwrFail`, `---`, `InVld`) are excluded from the daily average calculation.
- The Flask app expects the saved model at `random_forest_regression_model.pkl` in the root directory.
