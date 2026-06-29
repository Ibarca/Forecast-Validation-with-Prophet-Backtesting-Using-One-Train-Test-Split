# How I Validate the Forecast: Backtesting with One Train-Test Split


<p align="center">
  <img src="[https://raw.githubusercontent.com/Ibarca/Forecast-Validation-How-Do-We-Know-If-a-Forecast-Is-Good-/e3ef09854b6591477992778defc652b4c22a6a7e/Images/FCValidation.png](https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/forecast_validation_process%20(1).png)"
       alt="Forecast Validation Process"
       width="600">
</p>


In my previous article, I showed how to build a sales forecast using Prophet. At first glance, it can feel like the hard part is done once the model generates a forecast. After all, we have historical data, we train a model, and it produces predictions for the future.

But there is an important question that comes before we use those predictions to make decisions:

**Can we trust them?**

A forecast is ultimately an estimate of future demand. The numbers may look reasonable, but without evaluating the model's performance, we have no way of knowing whether those predictions are actually useful. A sophisticated model can still produce poor forecasts, and relying on inaccurate forecasts can lead to costly business decisions.


## Why Forecast Validation Matters

Imagine a forecast predicts sales of 1,000 units next month.

* If actual sales are 990 units, the forecast is very accurate.
* If actual sales are 600 units, relying on that forecast could lead to excess inventory, unnecessary purchasing, and poor planning decisions.

Without validation, we are simply trusting the model without evidence. This becomes especially important when you are responsible for the forecasting process. In a business environment, people will rightly question the reliability of your work, especially when forecasted numbers deviate from reality, or when demand is affected by strong seasonality, volatility, promotions, or unusual market behavior.

Therefore, it is not enough to simply generate a forecast. You also need to be able to explain how accurate the forecast is, where the model performs well, where it struggles, and whether the historical data is reliable enough to support future predictions. Forecast validation gives us a quantifiable way to answer these uncomfortable but necessary questions before the forecast is used for planning decisions.



## The Principle of Backtesting


<p align="center">
  <img src="https://raw.githubusercontent.com/Ibarca/Forecast-Validation-How-Do-We-Know-If-a-Forecast-Is-Good-/e3ef09854b6591477992778defc652b4c22a6a7e/Images/FCValidation.png"
       alt="Forecast Validation Process"
       width="600">
</p>

<br>


One of the easiest mistakes to make in forecasting is to train a model with all the historical data available and then check how well the model explains that same past data.

At first, this may look reasonable. The model follows the historical trend, captures some seasonality, and produces a forecast that seems to make sense.

But this can be misleading.

A forecast is not valuable only because it explains what already happened. Its real value comes from its ability to predict periods the model has not seen before.

This is where backtesting becomes important.

Instead of giving the model the full history, I split the data into two parts. The first part is used to train the model. The second part is kept aside as a validation period.

| Period              | Usage           |
| ------------------- | --------------- |
| Jan 2020 - Dec 2023 | Training Data   |
| Jan 2024 - Dec 2024 | Validation Data |

The model learns only from the training period. Then, it is asked to forecast the validation period.

The important detail is that the model does not see the validation data during training. From the model’s perspective, this period represents the future.

Because the validation period already happened in real life, I can compare the forecast with the actual sales and measure how accurate the model would have been.

In other words, backtesting allows me to ask a very practical question:

If I had used this model in the past, how well would it have predicted actual demand?

This makes the validation process much closer to a real business situation. When a planner creates a forecast, future demand is unknown. Backtesting recreates that situation using historical data.

This is especially important in time series forecasting because the order of the data matters. We cannot randomly shuffle past observations into training and test sets. The model must always learn from the past and be tested on a later period.

Only then can we understand whether the forecast is actually useful for predicting what comes next.

---

## Backtesting with Prophet

Prophet provides a built-in backtesting framework through its cross-validation functionality.

Instead of manually creating only one train-test split, Prophet can automatically create multiple historical cutoff points.

At each cutoff point, Prophet trains the model using only the data available up to that date and then forecasts a future period. The forecast is then compared with the actual values.

This approach is also known as **rolling-origin cross-validation**.

Example:

```python
from prophet.diagnostics import cross_validation, performance_metrics

df_cv = cross_validation(
    model,
    initial='1095 days',
    period='180 days',
    horizon='365 days'
)

df_performance = performance_metrics(df_cv)
```

In this example:

| Parameter | Meaning                                                         |
| --------- | --------------------------------------------------------------- |
| `initial` | Minimum amount of historical data used to train the first model |
| `period`  | Distance between each validation cutoff                         |
| `horizon` | Forecasting window to evaluate                                  |

For example, with `horizon='365 days'`, Prophet evaluates how well the model predicts one year into the future.

With `period='180 days'`, Prophet creates a new validation cutoff approximately every six months.

This allows the model to be tested across different points in time instead of relying on only one validation period.

---

## Why One Validation Period Is Not Enough

A single train-test split provides a useful first indication of model quality, but it may not tell the whole story.

Perhaps the selected validation period was unusually easy to predict. Perhaps demand was very stable during that time. Or perhaps the validation period contained unusual events such as promotions, stockouts, price changes, holidays, or market disruptions.

Because of this, one validation period may give either an overly optimistic or overly pessimistic view of the model.

To obtain a more robust estimate of performance, Prophet repeatedly creates multiple train-test splits across the historical data.

```text
Historical Data

Fold 1: [Train========][Test]

Fold 2: [Train==============][Test]

Fold 3: [Train====================][Test]

Fold 4: [Train==========================][Test]
```

For each fold, the model is retrained using only the data available before the cutoff date. It then forecasts the following period and compares the prediction against actual demand.

This gives a more reliable assessment of forecasting performance across different periods and business conditions.

---

## What Backtesting Helps Us Understand

Backtesting helps answer several practical business questions:

| Question                                                | Why It Matters                                                           |
| ------------------------------------------------------- | ------------------------------------------------------------------------ |
| Is the model consistently accurate?                     | Helps determine whether the forecast is reliable over time               |
| Does the model perform worse during volatile periods?   | Helps identify risk during unstable demand conditions                    |
| Is the forecast systematically too high or too low?     | Helps detect bias                                                        |
| Are large errors occasional or frequent?                | Helps understand variance and forecast stability                         |
| Does accuracy change depending on the forecast horizon? | Helps assess whether short-term or long-term forecasts are more reliable |

This is why backtesting is more than a technical validation step.

For demand planning, inventory management, and supply chain decisions, it provides evidence that the forecast can support real business decisions before it is used to plan purchasing, stock levels, capacity, or promotions.

A forecast should not be trusted only because the model looks sophisticated.

It should be trusted because it has been tested against historical reality.








Forecast Accuracy Metrics

Once forecasts have been generated for the validation periods, we need a way to quantify their accuracy.

Several metrics are commonly used.

MAE (Mean Absolute Error)

Measures the average absolute difference between actual and forecasted values.

Example:

Actual Sales    = 20,000
Forecast Sales  = 19,000

Error = 1,000

MAE tells us the average forecasting error in the same unit as the forecast.

Best for: Understanding average forecast error.

MAPE (Mean Absolute Percentage Error)

Measures forecast error as a percentage of actual demand.

Example:

Actual Sales    = 20,000
Forecast Sales  = 19,000

MAPE = 5%

Because it is expressed as a percentage, MAPE makes it easier to compare forecast accuracy across products with different sales volumes.

Best for: Comparing accuracy across products and categories.

RMSE (Root Mean Squared Error)

RMSE is similar to MAE but penalizes large mistakes more heavily.

This makes it particularly useful when large forecasting errors are especially costly.

Best for: Detecting occasional large forecasting misses.

Interpreting the Results

Metrics tell us how large the forecasting errors are, but they do not explain why those errors occur.

For example:

Observation	Possible Cause
High MAE	Forecast generally inaccurate
High RMSE	Large forecasting misses
High MAPE	Poor percentage accuracy
Consistent over-forecasting	High Forecast Bias
Unstable forecasts	High Variance

Understanding the root causes of forecasting errors leads to one of the most important concepts in predictive modeling: the Bias-Variance Tradeoff.

This concept helps explain why some forecasting models consistently underperform and how they can be improved.
