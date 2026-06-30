# How I Validate the Forecast: Backtesting with One Train-Test Split

In my previous article, I showed how to build a sales forecast using Prophet. At first glance, it can feel like the hard part is done once the model generates a forecast. After all, we have historical data, we train a model, and it produces predictions for the future.

But there is an important question that comes before we use those predictions to make decisions:

Can we trust them?

A forecast is ultimately an estimate of future demand. The numbers may look reasonable, but without evaluating the model’s performance, we have no way of knowing whether those predictions are actually useful.

A sophisticated model can still produce poor forecasts, and relying on inaccurate forecasts can lead to costly business decisions. This is especially important in demand planning, where forecast errors can affect inventory levels, purchasing decisions, service levels, and working capital.

<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/forecast_risk_chain%20(1).png"
       alt="Forecast Validation Process"
       width="600">
</p>

This is why forecast validation is an essential step before using a model for real business decisions. Instead of assuming that the forecast is reliable because it looks reasonable, we need a structured way to test how the model would have performed in the past. **Backtesting** allows us to do exactly that: we simulate a real forecasting situation by training the model on one part of the historical data and then testing it on a later period that the model has not seen before.




## The Principle of Backtesting




**Backtesting** is a method used to evaluate a forecasting model by testing it on past data as if it were predicting the future. The model is trained on an earlier period and then used to forecast a later period that has already happened. By comparing the forecast with the actual results, we can measure how reliable the model would have been in a real business situation



<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/prophet_5yr_split_with_full_dataset%20(1).png"
       alt="Forecast Validation Process"
       width="600">
</p>


<br>


To run the backtest, I used a simple **train-test split**.

In time series forecasting, the data should not be split randomly. The order of time matters. A forecasting model should only learn from the past and then be tested on a later period that represents the future.

In this example, the historical dataset is divided into two parts:

| Period | Purpose |
|---|---|
| First 4 years | Training data |
| Last 1 year | Test data |

The model is trained only on the first four years of data. The final year is kept aside and is not shown to the model during training.

After the model has learned from the training period, it generates a forecast for the test period. Because the test period already happened in reality, I can compare the forecasted values with the actual sales values. This is a very practical technique that allows us to evaluate the model in a realistic way, according to the following logic:

> If I had used this model one year ago, how close would its forecast have been to the actual demand?

This simple train-test split is the foundation of the validation process. It helps move the forecast from something that only looks reasonable to something that can be measured and evaluated.

| 💡 **Definition: Train-test split**                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A **train-test split** is a validation technique used in machine learning where the available data is divided into two parts: a **training set** and a **test set**.<br><br> |




## How the Validation Process Works

Before going into the metrics, it is useful to visualize the full validation process.

The diagram below shows the logic of the backtesting approach. The historical dataset is split chronologically into two parts: a training period and a validation period. The model learns only from the training data and then generates a forecast for the validation period, which represents the “future” from the model’s perspective.

<br>

<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/forecast_validation_dfd.png"
       alt="Forecast validation process"
       width="680">
</p>

<p align="center">
  <em>Figure: Forecast validation process using a chronological train-test split.</em>
</p>

<br>

In this example, the first four years of sales data are used for training. This is the period where the Prophet model identifies patterns such as trend, seasonality, and recurring demand behavior.

The final year is kept aside as validation data. The model does not see this period during training. After the model has been fitted, it is asked to forecast this final year.

Because the validation period has already happened in reality, I can compare two values for each period:

| Value | Meaning |
|---|---|
| Actual sales | What really happened |
| Forecasted sales | What the model predicted |

This comparison is the core of forecast validation. It allows me to calculate error metrics such as MAE, RMSE, MAPE, and Bias, and decide whether the forecast is reliable enough to support business decisions.

## Measuring Forecast Accuracy

Once the forecasted values are compared with the actual sales, the next step is to measure the size and direction of the error.

A forecast will almost never be perfectly correct. Some difference between forecast and actual demand is normal. The important question is whether the error is small enough, stable enough, and unbiased enough to support business decisions.

For this validation, I use four metrics:

| Metric | What it tells us | Why it matters |
|---|---|---|
| MAE | Average size of the error | Easy to understand in units sold |
| RMSE | Size of the error with higher penalty for large mistakes | Useful when big errors are especially costly |
| MAPE | Error as a percentage | Useful to compare accuracy across products or periods |
| Bias | Direction of the error | Shows whether the model tends to over-forecast or under-forecast |

Each metric answers a slightly different question. For that reason, I do not rely on only one number to judge the model. A forecast can have a reasonable average error but still be systematically too high or too low. In demand planning, this distinction matters because over-forecasting and under-forecasting create different business risks.

For example, a model that consistently overestimates demand may lead to excess inventory, markdowns, and higher working capital. A model that consistently underestimates demand may lead to stockouts, lost sales, and lower service levels.

This is why forecast validation should look not only at how large the errors are, but also at the direction and consistency of those errors.

Then after this section, you can create one subsection per metric:
