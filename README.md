# How I Validate the Forecast: Backtesting with One Train-Test Split



In my previous article, I showed how to build a sales forecast using Prophet. At first glance, it can feel like the hard part is done once the model generates a forecast. After all, we have historical data, we train a model, and it produces predictions for the future.

But there is an important question that comes before we use those predictions to make decisions:

**Can we trust them?**

A forecast is ultimately an estimate of future demand. The numbers may look reasonable, but without evaluating the model's performance, we have no way of knowing whether those predictions are actually useful. A sophisticated model can still produce poor forecasts, and relying on inaccurate forecasts can lead to costly business decisions.



## The Principle of Backtesting


<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/forecast_validation_process%20(1).png"
       alt="Forecast Validation Process"
       width="600">
</p>






<br>


One of the easiest mistakes to make in forecasting is to train a model with all the historical data available and then check how well the model explains that same past data. At first, this may look reasonable. The model follows the historical trend, captures some seasonality, and produces a forecast that seems to make sense, but this can be misleading.

A forecast is not valuable only because it explains what already happened. Its real value comes from its ability to predict periods the model has not seen before.

This is where backtesting and the train-test split become important.

| 💡 **Definition: Train-test split**  |                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A **train-test split** is a validation technique used in machine learning where the available data is divided into two parts: a **training set** and a **test set**.<br><br>• The **training set** is used to teach the model and allow it to learn patterns from historical data.<br><br>• The **test set** is kept separate and used only to evaluate how well the model performs on data it has not seen before. |





Instead of giving the model the full history, I split the data into two parts. The first part is used to train the model. The second part is kept aside as a validation period.


<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/prophet_5yr_split_with_full_dataset%20(1).png"
       alt="Forecast Validation Process"
       width="600">
</p>


The model learns only from the training period. Then, it is asked to forecast the validation period.

The important detail is that the model does not see the validation data during training. From the model’s perspective, this period represents the future. Because the validation period already happened in real life, I can compare the forecast with the actual sales and measure how accurate the model would have been. In other words, backtesting allows me to ask a very practical question:

**If I had used this model in the past, how well would it have predicted actual demand?**

This makes the validation process much closer to a real business situation. When a planner creates a forecast, future demand is unknown. Backtesting recreates that situation using historical data. This is especially important in time series forecasting because the order of the data matters. We cannot randomly shuffle past observations into training and test sets. The model must always learn from the past and be tested on a later period.

Only then can we understand whether the forecast is actually useful for predicting what comes next.

---

d.
