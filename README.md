# Rossmann Store Sales Forecasting Using Machine Learning

## Project Overview

This project is an end-to-end machine learning solution for retail sales forecasting using the Rossmann store dataset from Kaggle.

The objective is to predict future store sales and provide business insights using predictive modeling and visualization techniques.

The forecasting horizon is approximately six weeks.

## Project Objectives

- Data preprocessing and cleaning  
- Handling missing values using logical placeholder encoding  
- Feature engineering for temporal and business variables  
- Building a machine learning forecasting model  
- Model evaluation using validation data  
- Generating future sales predictions  
- Creating an interactive business dashboard  

## Methodology

### Data Preprocessing

- Merged store and sales datasets  
- Converted date variables into meaningful temporal features  
- Created missingness indicator variables  
- Applied categorical encoding using one-hot encoding  

Missing values were handled using business-meaningful placeholder values.

### Feature Engineering

The following features were constructed:

- Month, week, and day-based temporal features  
- Promotion indicators  
- Store structural variables  
- Competition duration effects  

These features were used to capture seasonality and business behavior.

## Machine Learning Model

A gradient boosting ensemble model was used for forecasting.

The model was implemented using LightGBM regression.

The objective was to predict continuous sales values.

## Model Evaluation

The dataset was split based on time.  
Training data included observations up to **01.01.2015**, and later dates were used as validation data to preserve the time-series structure and avoid data leakage.

The model was evaluated using a hold-out validation dataset.

Performance was measured using Root Mean Squared Error (RMSE).

Validation RMSE was approximately 716.

Residual distribution and prediction visualization were analyzed.

## Forecasting Output

The trained model was used to generate future sales predictions for the test dataset.

Prediction results were exported for dashboard visualization.

## Dashboard Visualization

An interactive forecasting dashboard was built using Microsoft Power BI.

The dashboard includes:

- Predicted sales trend over time  
- Store type comparison analysis  
- Promotion impact visualization  
- Key performance indicators  

Displayed business metrics include:

- Total forecasted sales  
- Average forecasted sales  
- Number of active stores  

## Technologies Used

- Python  
- Pandas  
- Scikit-learn  
- LightGBM  
- Matplotlib  
- Seaborn  
- Microsoft Power BI  

## Conclusion

This project demonstrates a complete machine learning pipeline for retail sales forecasting using the Rossmann dataset, including data preprocessing, model training, prediction generation, and dashboard visualization.
