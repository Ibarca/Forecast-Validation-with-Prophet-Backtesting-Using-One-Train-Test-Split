# forecast-validation-and-backtesting-

Forecast Validation and Backtesting

Building a forecast is only the first step. Before using a forecasting model to support business decisions, we need to determine whether it can accurately predict demand.

This process is known as forecast validation.

Rather than immediately forecasting the future, we first test the model on historical data. We deliberately hide a portion of the dataset, train the model on older observations, and then ask it to predict the period that was excluded.

Because we already know the actual sales for that period, we can compare the model's predictions against reality and objectively evaluate its performance.

This technique is commonly known as backtesting.

Train-Test Split

In this example, the dataset is divided into two periods:

Dataset	Purpose
Training Set	Used to train the Prophet model
Validation Set	Used to evaluate forecasting performance
Historical Data

|------------------------|------------|
      Training Data        Test Data
       (Model learns)    (Model tested)



The model never sees the validation period during training. This simulates a real forecasting scenario where future demand is unknown.

The key question becomes:

If this model had been used in the past, how accurately would it have predicted actual sales?

If the answer is satisfactory, we can have greater confidence when using the model to forecast future periods.

Why One Validation Period Is Not Enough

A single train-test split provides a useful first indication of model quality, but it may not tell the whole story.

Perhaps the selected validation period was unusually easy to predict. Perhaps demand was unusually stable during that time.

To obtain a more robust estimate of performance, Prophet provides a built-in cross-validation framework.

Instead of testing the model on only one period, Prophet repeatedly creates multiple train-test splits across the historical data.

Historical Data

Fold 1: [Train========][Test]

Fold 2: [Train==============][Test]

Fold 3: [Train====================][Test]

Fold 4: [Train==========================][Test]

The model is retrained for each fold and evaluated on data it has never seen before.

This provides a much more reliable assessment of forecasting performance across different periods and business conditions.

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
