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


<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/forecast_validation_overview.png?raw=true"
       alt="Forecast validation overview showing the process from train-test split to forecast comparison, metric calculation, and business interpretation"
       width="680">
</p>

<p align="center">
  <em>Figure: Forecast validation overview, from historical data split to metric interpretation.</em>
</p>

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

---
### MAE: Mean Absolute Error

**MAE** measures the average size of the forecast error in absolute terms.  
In simple words, it tells us how far the forecast was from the actual value on average.

$$
MAE = \frac{1}{n}\sum_{i=1}^{n}|A_i - F_i|
$$

Where:

- \(A_i\) = actual value  
- \(F_i\) = forecasted value  
- \(n\) = number of periods  

For example, if MAE is **120 units**, it means that the forecast was wrong by around **120 units per period**, on average.

<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/mae_explainer.png?raw=true"
       alt="MAE explainer showing the average absolute difference between actual and forecasted values"
       width="680">
</p>

<p align="center">
  <em>Figure: MAE measures the average absolute distance between forecasted and actual values.</em>
</p>

MAE is easy to interpret because it uses the same unit as the original data.

| MAE Result | Interpretation |
|---|---|
| Low MAE | The forecast is close to actual demand on average |
| High MAE | The forecast is frequently far away from actual demand |

In demand planning, MAE helps answer:

> On average, how many units could my forecast be wrong by?

However, MAE should always be interpreted together with the sales volume. An error of **100 units** may be small for a product selling **10,000 units**, but very large for a product selling only **200 units**.

One limitation of MAE is that it only measures the size of the error. It does not tell us whether the model tends to over-forecast or under-forecast. For that, we need to look at **Bias**.

---

### RMSE: Root Mean Squared Error

**RMSE** measures forecast error like MAE, but it gives more weight to large mistakes.  
This makes it especially useful when large forecast errors are more costly than small ones.

$$
RMSE = \sqrt{\frac{1}{n}\sum_{i=1}^{n}(A_i - F_i)^2}
$$

Where:

- \(A_i\) = actual value  
- \(F_i\) = forecasted value  
- \(n\) = number of periods  

Because the errors are squared before averaging, large mistakes have a stronger impact on RMSE than small ones.

If RMSE is much higher than MAE, it usually means the model made some large errors in certain periods.

<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/rmse_vs_mae_scenarios.png?raw=true"
       alt="RMSE compared with MAE across different forecast error scenarios"
       width="680">
</p>

<p align="center">
  <em>Figure: RMSE increases more strongly than MAE when a forecast contains large errors.</em>
</p>

Like MAE, RMSE is expressed in the same unit as the original data.

| Situation | Interpretation |
|---|---|
| RMSE close to MAE | Forecast errors are relatively stable |
| RMSE much higher than MAE | Some periods had very large errors |

This is important because large forecast errors can create serious business problems.

A single large under-forecast can lead to stockouts and lost sales. A single large over-forecast can create excess inventory, markdowns, and working capital pressure.

In demand planning, RMSE helps answer:

> Did the model make any large errors that could create operational or financial risk?

One limitation of RMSE is that it can be heavily influenced by a few extreme errors. For that reason, it is best interpreted together with MAE rather than on its own.


---

### MAPE: Mean Absolute Percentage Error

**MAPE** expresses the forecast error as a percentage of actual demand.  
This makes it useful when comparing forecast accuracy across different products, categories, or time periods.

$$
MAPE = \frac{1}{n}\sum_{i=1}^{n}\left|\frac{A_i - F_i}{A_i}\right| \times 100\%
$$

Where:

- \(A_i\) = actual value  
- \(F_i\) = forecasted value  
- \(n\) = number of periods  

If MAPE is **15%**, it means that the forecast was wrong by around **15% on average** compared with actual demand.

<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/mape_chart_with_scale.png?raw=true"
       alt="MAPE interpretation scale for forecast accuracy"
       width="680">
</p>

<p align="center">
  <em>Figure: MAPE expresses forecast error as a percentage, making it easier to compare accuracy across products or periods.</em>
</p>

As a practical guideline, MAPE can often be interpreted as follows:

| MAPE Result | General Interpretation |
|---:|---|
| Below 10% | Very strong forecast |
| 10–20% | Good or reasonable forecast |
| 20–30% | Acceptable, depending on the business context |
| 30–50% | Forecast needs attention |
| Above 50% | Forecast may be unreliable |

These ranges are not universal. A good MAPE depends on demand volatility, product category, seasonality, promotions, stock availability, and data quality.

In demand planning, MAPE helps answer:

> How large is the forecast error compared with actual demand?

However, MAPE should be interpreted carefully when actual sales are very low. In those cases, even a small absolute error can create a very high percentage error. If actual demand is zero, MAPE cannot be calculated because the formula divides by the actual value.

For this reason, MAPE is useful for business interpretation, but it should not be used alone. It is best read together with MAE, RMSE, and Bias.


---

### Bias: Forecast Direction

**Bias** shows the direction of the forecast error.  
While MAE, RMSE, and MAPE tell us how large the error is, Bias tells us whether the model tends to over-forecast or under-forecast.

$$
Bias = \frac{1}{n}\sum_{i=1}^{n}(F_i - A_i)
$$

Where:

- \(F_i\) = forecasted value  
- \(A_i\) = actual value  
- \(n\) = number of periods  

With this formula, a **positive Bias** means the forecast is too high on average, while a **negative Bias** means the forecast is too low on average.

<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/bias_explainer.png?raw=true"
       alt="Bias explainer showing over-forecasting, under-forecasting, and balanced forecast errors"
       width="680">
</p>

<p align="center">
  <em>Figure: Bias shows whether the forecast is systematically too high or too low.</em>
</p>

| Bias Result | Interpretation | Business Risk |
|---|---|---|
| Bias close to 0 | Forecast is balanced | No strong systematic direction |
| Positive Bias | Forecast is too high on average | Overstock, markdowns, excess working capital |
| Negative Bias | Forecast is too low on average | Stockouts, lost sales, lower service level |

Bias is especially important in demand planning because two forecasts can have a similar average error but create very different business consequences.

A forecast that is consistently too high may lead to excess inventory, higher storage costs, markdowns, and working capital pressure. A forecast that is consistently too low may reduce inventory in the short term, but it can damage availability, service levels, and customer satisfaction.

In demand planning, Bias helps answer:

> Is the model making random errors, or is it systematically wrong in one direction?

One limitation of Bias is that positive and negative errors can cancel each other out. For example, a model may over-forecast in some periods and under-forecast in others, resulting in a Bias close to zero. This is why Bias should always be interpreted together with MAE, RMSE, and MAPE.


## What If the Forecast Validation Is Not Successful?

Sometimes the validation results show that the forecast is not reliable enough. This does not mean the process failed. In fact, this is exactly why backtesting is useful: it helps identify problems before the forecast is used for real business decisions.

If the error metrics are too high, or if the model shows a strong positive or negative Bias, the next step is to understand why the forecast is not performing well.

| Area to Check | Why It Matters |
|---|---|
| Data quality | Missing values, wrong dates, outliers, or stockout periods can distort the model |
| Seasonality | The model may not fully capture monthly, weekly, or yearly demand patterns |
| Trend changes | Demand may have shifted due to assortment changes, pricing, or market conditions |
| Promotions or campaigns | Marketing activity can create demand spikes that the model does not understand |
| External factors | Weather, holidays, availability, or economic effects may influence demand |
| Product behavior | Some SKUs are too volatile or irregular to forecast accurately with a simple model |

After identifying the likely cause, the model can be improved. For example, I could clean the historical data, remove or flag outliers, add relevant regressors such as holidays or promotions, adjust Prophet’s seasonality settings, or test a different forecasting approach.

In some cases, the conclusion may also be that the product is simply difficult to forecast. For example, low-volume or highly irregular SKUs often produce unstable accuracy metrics. In that case, it may be better to evaluate the forecast at a category level, use simpler planning rules, or combine statistical forecasting with business input.

The goal of validation is not only to approve or reject a forecast. The real goal is to create a feedback loop:

```text
Validate forecast
        ↓
Identify errors
        ↓
Understand the cause
        ↓
Improve data or model
        ↓
Validate again

