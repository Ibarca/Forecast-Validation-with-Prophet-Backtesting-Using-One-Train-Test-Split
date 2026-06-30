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
       alt="Forecast Validation Process"
       width="600">

 align="center">
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

To evaluate the difference between forecast and actual sales, I use several error metrics:

- **MAE** to understand the average forecast error
- **RMSE** to penalize larger mistakes
- **MAPE** to express the error as a percentage
- **Bias** to detect whether the model systematically over-forecasts or under-forecasts

The goal is not only to generate a forecast, but to understand whether the forecast is accurate enough to support business decisions such as inventory planning, purchasing, service levels, and working capital management.
--



The important detail is that the model does not see the test data during training. Because the test period already happened in real life, I can compare the forecast with the actual sales and measure how accurate the model would have been.

In other words, backtesting allows me to ask a practical question:

If I had used this model in the past, how well would it have predicted actual demand?

This makes the validation process much closer to a real business situation. When a planner creates a forecast, future demand is unknown. Backtesting recreates that situation using historical data.

Why the Order of the Data Matters

In time series forecasting, the order of the data is important.

This means we should not randomly shuffle the observations into training and test sets. A forecasting model must always learn from the past and then be tested on a later period.

For example, if I have monthly sales data from January 2020 to December 2024, I could use the earlier period for training and the later period for testing:

Period	Usage
January 2020 – December 2023	Training data
January 2024 – December 2024	Test data

The model learns from the sales history between 2020 and 2023. Then, it generates a forecast for 2024.

Since the actual sales for 2024 are already known, I can compare the forecasted values with the real values and calculate how accurate the model was.

This approach helps answer whether the model would have been useful before trusting it for future forecasting.

Why I Use One Train-Test Split

In this article, I use a simple backtesting approach based on one train-test split.

This means I select one historical cutoff point, train the model on the data before that point, and test it on the data after that point.

This is not the only possible way to do backtesting. More advanced approaches can use multiple rolling validation windows, where the model is tested several times across different historical periods.

However, one train-test split is a good starting point because it is simple, easy to explain, and already gives useful information about how the model performs on unseen data.

For a first validation step, the goal is not to make the process overly complex. The goal is to check whether the model can produce reasonable forecasts outside the data it was trained on.

From Forecast to Evaluation

Once the model has generated predictions for the test period, I compare two values:

Value	Meaning
Actual sales	What really happened
Forecasted sales	What the model predicted

This comparison allows me to measure the forecast error.

If the forecast is close to the actual sales, the model may be useful. If the forecast is far away from the actual sales, the model needs to be reviewed before it can be trusted for business decisions.

This is where forecast accuracy metrics become useful.

Instead of only looking at the forecast visually, I can calculate numerical indicators such as:

Metric	What it helps evaluate
MAE	The average size of the forecast error
RMSE	Whether the model makes large errors
MAPE	The percentage error compared to actual demand
Bias	Whether the model systematically over-forecasts or under-forecasts

These metrics help transform the validation process from a visual check into a measurable evaluation.

Why This Matters in Business

Forecast validation is not only a technical step. It is also a business control.

In demand planning, people will often question the forecast, especially when numbers deviate from reality or when demand is affected by strong seasonality, volatility, promotions, or external events.

This is normal.

A forecast should not be accepted only because the model looks sophisticated. It should be supported by evidence.

Backtesting provides that evidence. It allows us to evaluate the model before using it to make decisions about purchasing, inventory, staffing, or capacity planning.

By testing the forecast on a historical period that was excluded from training, we can better understand whether the model is reliable enough to support business decisions.
Only then can we understand whether the forecast is actually useful for predicting what comes next.
