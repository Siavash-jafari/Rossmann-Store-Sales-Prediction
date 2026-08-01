# Rossmann Store Sales Forecasting Using Machine Learning

## Project Overview

This project implements an end-to-end machine learning solution for retail sales forecasting using the [Rossmann Store Sales dataset](https://www.kaggle.com/competitions/rossmann-store-sales/data) from Kaggle.  
The goal is to predict daily store sales for a six-week horizon and provide business insights through an interactive dashboard.

## Objectives

- Clean and preprocess historical sales and store data  
- Model business-relevant missing values instead of dropping data  
- Engineer temporal, promotion, competition, and customer-demand features  
- Train a gradient boosting model for sales forecasting  
- Evaluate performance on a time-based validation set  
- Generate future sales predictions for all stores  
- Build a Power BI dashboard for business users  

## Data and Feature Engineering

Key steps:

- Merging store metadata into the sales data on `Store`  
- Extracting calendar features from `Date` (year, month, week, day of year, weekend flag)  
- Modeling competition using a continuous **competition duration** feature (months since the competitor opened)  
- Encoding promotion information, including ongoing promo campaigns and their timing  
- Creating a demand proxy feature **`ExpectedCustomers`**, computed as the average number of customers per store and weekday, and set to zero for closed days  

Missing competition and promotion information is represented with explicit indicator flags, so the model can learn the effect of unknown values rather than losing those rows.

**Dataset:** Download the Kaggle dataset [here](https://www.kaggle.com/competitions/rossmann-store-sales/data) and place the files in the `data/` folder (`train.csv`, `test.csv`, `store.csv`).

## Modeling Approach

- Algorithm: LightGBM regression (gradient boosting)  
- Target: daily store `Sales`  
- Input features: store identifiers, temporal features, promo and competition variables, missingness indicators, and `ExpectedCustomers`  

To respect the time structure and avoid leakage, the dataset is split chronologically:

- Training period: 2013‑01‑01 to 2015‑06‑18  
- Validation period: 2015‑06‑19 to 2015‑07‑31  

### Evaluation Metrics

Model performance is assessed on the validation period using three complementary metrics rather than RMSE alone:

- **RMSE (Root Mean Squared Error):** ≈ **1028.08** — penalizes large errors more heavily, but is dominated by high-volume stores and is harder to interpret in isolation.
- **MAE (Mean Absolute Error):** ≈ **553.54** — the average absolute error in raw sales units, giving a more intuitive, scale-robust sense of typical prediction error.
- **RMSPE (Root Mean Squared Percentage Error):** ≈ **0.128** — the official metric used to score the original Kaggle competition. Closed-store days (`Sales == 0`) are excluded to avoid division by zero, matching the competition's own scoring methodology. This makes the result directly comparable to published Kaggle leaderboard scores for this dataset.

After evaluation, the model is retrained on the full history (train + validation) before forecasting the test period.

## Forecasting Pipeline

For the Kaggle test set, the same preprocessing and feature engineering steps are applied:

- Merge store information and derive the same calendar, competition, promotion, and `ExpectedCustomers` features  
- Apply the trained LightGBM model to predict daily sales for each store and date  

Predictions are exported as:

- `prediction.csv`: contains `Id` and predicted `Sales` for submission or further analysis  

## Dashboard Dataset and Power BI

A separate dashboard dataset is created from the test predictions:

- `Store`, `Date`, `Open`, `Promo`, `StoreType`, `Assortment`  
- `ExpectedCustomers`, `CompetitionDuration`, `PredictedSales`

This dataset is exported as `dashboard_data.csv` and visualized in Microsoft Power BI.  
The dashboard highlights:

- Key KPIs (total and average predicted sales, number of active stores)  
- Predicted sales trends over time, broken down by store type  
- Promo impact on predicted sales  
- “Efficiency” views such as sales per expected customer by store type, promo status, and top stores

## Technologies Used

- Python, Jupyter Notebook  
- Pandas, NumPy  
- Scikit-learn  
- LightGBM  
- Matplotlib, Seaborn  
- Microsoft Power BI  

## Conclusion

The project demonstrates a complete machine learning pipeline for retail sales forecasting on the Rossmann dataset, from data preparation and feature engineering through time-aware model training and evaluation, to producing forecasts and a dashboard-ready dataset for business decision-making.
