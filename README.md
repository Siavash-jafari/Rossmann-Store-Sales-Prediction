# Rossmann Store Sales Forecasting Using Machine Learning

## Project Overview

This project implements an end-to-end machine learning solution for retail sales forecasting using the Rossmann store dataset from Kaggle.  
The objective is to predict future daily store sales over a forecasting horizon of approximately six weeks and to provide actionable business insights via model-based predictions and an interactive dashboard.

## Project Objectives

- Data preprocessing and cleaning of store and sales data  
- Handling missing values using logical, business-meaningful placeholder encodings  
- Feature engineering for temporal, promotion, competition, and customer-demand related variables  
- Building and tuning a gradient boosting regression model (LightGBM)  
- Time-aware model evaluation using a hold-out validation period  
- Generating aligned test-set predictions for all stores  
- Creating a Power BI dashboard dataset for business visualization  

## Methodology

### Data Preprocessing

- Sales (`train`) and store metadata (`store`) datasets are merged on `Store` to create a unified modeling table.  
- The `Date` column is converted to a datetime type and decomposed into several temporal features: `Year`, `Month`, ISO `Week`, `DayOfYear`, and a weekend flag `IsWeekend` based on `DayOfWeek`.  
- Store-level competition timing is converted into a continuous `CompetitionDuration` (months since competition opened), using the difference between each observation date and a derived `CompetitionStartDate` and clipping negative values to zero.  
- Categorical attributes (e.g., `StoreType`, `Assortment`, promotion interval fields) are kept as categories and later one-hot encoded.  
- Missing values in competition and promotion timing variables are encoded via indicator flags (e.g., `CompetitionOpenSinceMonthmissing`, `Promo2SinceYearmissing`, `PromoIntervalmissing`) instead of dropping rows.

### ExpectedCustomers Feature Engineering

Because the Kaggle test set does not include actual daily customer counts, a proxy demand signal called **ExpectedCustomers** is engineered from the training data.

- Compute the average number of customers by `Store` and `DayOfWeek` on the training set:  
  `storedowavg = train_df.groupby(["Store", "DayOfWeek"])["Customers"].mean()`.  
- Merge these averages back into both train/validation and test data as `ExpectedCustomers`.  
- For days when a store is closed (`Open == 0`), `ExpectedCustomers` is set to 0 to reflect no customer traffic.

This feature carries store- and weekday-specific demand intensity into both validation and test sets without using unavailable test customers.

### Handling Missing Values and Encodings

- Competition and promotion timing missingness is represented through dedicated binary indicators, allowing the model to learn effects of “unknown” versus “known” business configurations.  
- Original competition timing fields (`CompetitionOpenSinceYear`, `CompetitionOpenSinceMonth`, `CompetitionStartDate`) are dropped from the test set after constructing `CompetitionDuration` and indicator columns, keeping the feature space consistent with training.  
- Categorical variables are one-hot encoded using `pd.get_dummies(..., drop_first=True)` applied consistently to train, validation, and test features.  
- After encoding, feature matrices (`X_train`, `X_val`, `X_test`) are aligned so they share identical column names and order, filling any missing columns in the test set with zeros.  
- Special characters are removed from feature names to ensure model compatibility (for example by stripping non-alphanumeric characters from column labels).

## Machine Learning Model

A **LightGBM regression model** (gradient boosting ensemble) is used to predict continuous daily sales per store.

Key configuration:

- Algorithm: `LGBMRegressor`  
- Number of estimators: 500  
- Learning rate: 0.05  
- Max depth: -1 (no explicit maximum depth)  
- Number of leaves: 31  
- Subsample: 0.8  
- Column subsample by tree: 0.8  
- Random state: 42  

Targets and features:

- Target variable: `Sales`  
- Dropped from features: `Sales`, `Date`, and `Customers`, ensuring only features available at prediction time are used  
- Final feature set includes store identifiers, operational flags, temporal variables, competition structure, promotion flags and durations, missingness indicators, and the engineered `ExpectedCustomers` feature  

## Time-Based Validation Strategy

To respect temporal ordering and avoid data leakage, a time-based split is used.

- The last six weeks of available historical data (from 2015‑06‑19 to 2015‑07‑31) form the **validation set**.  
- All earlier dates (from 2013‑01‑01 up to 2015‑06‑18) form the **training set**.  

This simulates a realistic forecasting scenario where the model is trained on past data and evaluated on a future hold-out period.

## Model Evaluation

Evaluation metric:

- Root Mean Squared Error (RMSE) on the validation period  

Observed result:

- The LightGBM model trained on the time-based split achieved a **validation RMSE of approximately 1028.08**.  

After validation, the training and validation sets are concatenated into a **final training dataset**, and LightGBM is retrained on this full history to maximize the information used before generating test predictions.

## Forecasting and Test Predictions

Forecasting pipeline on the Kaggle test set:

- Merge store metadata into the test set and compute the same temporal features as in training (year, month, week, day of year, weekend flag).  
- Compute `CompetitionDuration` in months for each test date, using the same logic as the training data and clipping negative values to zero.  
- Add `ExpectedCustomers` to the test data by merging the precomputed store–weekday averages and setting it to 0 when `Open == 0`.  
- Apply the same preprocessing: missingness indicators, categorical encodings with `get_dummies`, feature name cleaning, and column alignment with the training feature matrix.  
- Predict daily sales for each row using the retrained LightGBM model, yielding `y_test_pred`.  

The predictions are stored as:

- A submission-style file `prediction.csv` containing `Id` (store identifier used as ID) and `Sales` (predicted sales).

## Dashboard Dataset and Visualization

For business users, a dedicated dataset is exported for Power BI visualization.

- A `dashboarddf` DataFrame is created from the test data, including:  
  - `Store`  
  - `Open`  
  - `Promo`  
  - `StoreType`  
  - `Assortment`  
  - `ExpectedCustomers`  
  - `CompetitionDuration`  
  - `PredictedSales` (from `y_test_pred`)  
- This dataset is saved as `dashboarddata.csv` for import into Microsoft Power BI.

The Power BI dashboard can display:

- Predicted sales trends over time at store and chain level  
- Comparisons across different store types and assortments  
- Visual analysis of promotion impact using `Promo` and `ExpectedCustomers`  
- Key performance indicators such as total predicted sales, average predicted sales, and number of active (open) stores over the forecast horizon  

## Technologies Used

- **Python**, Jupyter Notebook  
- **Pandas**, NumPy for data manipulation  
- **Scikit-learn** for preprocessing and evaluation utilities  
- **LightGBM** (`LGBMRegressor`) for gradient boosting regression  
- **Matplotlib**, Seaborn for exploratory data analysis and visualization  
- **Microsoft Power BI** for building the interactive forecasting dashboard  

## Conclusion

This project demonstrates a **complete machine learning pipeline** for retail sales forecasting on the Rossmann dataset, from raw data preprocessing and robust feature engineering (including the `ExpectedCustomers` demand proxy) through time-based validation, gradient-boosted modeling with LightGBM, generation of aligned test predictions, and preparation of a Power BI-ready dashboard dataset for business insight and decision support.
