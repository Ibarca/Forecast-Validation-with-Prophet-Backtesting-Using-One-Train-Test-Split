# How I Validate the Forecast: Backtesting with One Train-Test Split

In my previous article, I showed how to build a sales forecast using Prophet. At first glance, it can feel like the hard part is done once the model generates a forecast. After all, we have historical data, we train a model, and it produces predictions for the future.

But there is an important question that comes before we use those predictions to make decisions:

Can we trust them?

A forecast is ultimately an estimate of future demand. The numbers may look reasonable, but without evaluating the model’s performance, we have no way of knowing whether those predictions are actually useful.

A sophisticated model can still produce poor forecasts, and relying on inaccurate forecasts can lead to costly business decisions. This is especially important in demand planning, where forecast errors can affect inventory levels, purchasing decisions, service levels, and working capital.


<svg width="680" height="500" viewBox="0 0 680 500" xmlns="http://www.w3.org/2000/svg" role="img">
<title>Forecast risk chain</title>
<desc>Diagram showing how one inaccurate demand forecast cascades into understocking or overstocking, then into financial impact, and finally into customer trust and competitive risk.</desc>
<defs>
<marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
<path d="M2 1L8 5L2 9" fill="none" stroke="#444441" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</marker>
</defs>

<rect x="0" y="0" width="680" height="500" fill="#FFFFFF"/>

<!-- Trigger node -->
<rect x="250" y="20" width="180" height="44" rx="8" fill="#F1EFE8" stroke="#5F5E5A" stroke-width="0.5"/>
<text x="340" y="42" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="14" font-weight="500" fill="#000000">Inaccurate forecast</text>

<!-- Branch arrows -->
<path d="M340 64 L340 84 L175 84 L175 100" fill="none" stroke="#444441" stroke-width="1.5" marker-end="url(#arrow)"/>
<path d="M340 64 L340 84 L505 84 L505 100" fill="none" stroke="#444441" stroke-width="1.5" marker-end="url(#arrow)"/>

<!-- Understocking -->
<rect x="40" y="100" width="270" height="56" rx="8" fill="#FAECE7" stroke="#993C1D" stroke-width="0.5"/>
<text x="175" y="122" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="14" font-weight="500" fill="#000000">Understocking</text>
<text x="175" y="140" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="12" fill="#000000">Demand underestimated</text>

<!-- Overstocking -->
<rect x="370" y="100" width="270" height="56" rx="8" fill="#FAECE7" stroke="#993C1D" stroke-width="0.5"/>
<text x="505" y="122" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="14" font-weight="500" fill="#000000">Overstocking</text>
<text x="505" y="140" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="12" fill="#000000">Demand overestimated</text>

<!-- Down arrows -->
<line x1="175" y1="156" x2="175" y2="196" stroke="#444441" stroke-width="1.5" marker-end="url(#arrow)"/>
<line x1="505" y1="156" x2="505" y2="196" stroke="#444441" stroke-width="1.5" marker-end="url(#arrow)"/>

<!-- Lost sales -->
<rect x="40" y="196" width="270" height="56" rx="8" fill="#FAECE7" stroke="#993C1D" stroke-width="0.5"/>
<text x="175" y="218" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="14" font-weight="500" fill="#000000">Lost sales</text>
<text x="175" y="236" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="12" fill="#000000">Stockout costs, expediting</text>

<!-- Markdowns -->
<rect x="370" y="196" width="270" height="56" rx="8" fill="#FAECE7" stroke="#993C1D" stroke-width="0.5"/>
<text x="505" y="218" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="14" font-weight="500" fill="#000000">Markdowns</text>
<text x="505" y="236" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="12" fill="#000000">Holding costs, write-offs</text>

<!-- Converge arrows -->
<path d="M175 252 L175 280 L340 280 L340 292" fill="none" stroke="#444441" stroke-width="1.5" marker-end="url(#arrow)"/>
<path d="M505 252 L505 280 L340 280 L340 292" fill="none" stroke="#444441" stroke-width="1.5" marker-end="url(#arrow)"/>

<!-- Financial impact -->
<rect x="190" y="292" width="300" height="56" rx="8" fill="#FAEEDA" stroke="#854F0B" stroke-width="0.5"/>
<text x="340" y="314" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="14" font-weight="500" fill="#000000">Margin &amp; cash flow impact</text>
<text x="340" y="332" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="12" fill="#000000">Profitability and liquidity squeezed</text>

<line x1="340" y1="348" x2="340" y2="388" stroke="#444441" stroke-width="1.5" marker-end="url(#arrow)"/>

<!-- Trust and strategic risk -->
<rect x="170" y="388" width="340" height="56" rx="8" fill="#FCEBEB" stroke="#A32D2D" stroke-width="0.5"/>
<text x="340" y="410" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="14" font-weight="500" fill="#000000">Customer trust &amp; competitive risk</text>
<text x="340" y="428" text-anchor="middle" dominant-baseline="central" font-family="Arial, sans-serif" font-size="12" fill="#000000">Churn, reputation, market share loss</text>

</svg>




## The Principle of Backtesting








<br>


One of the easiest mistakes in forecasting is to train a model using all available historical data and then evaluate how well it fits that same data.

At first, this may look reasonable. The model follows the historical trend, captures some seasonality, and produces a forecast that seems to make sense.

But this can be misleading.

A forecast is not valuable only because it explains what already happened. Its real value comes from its ability to predict periods the model has not seen before.

This is where backtesting and the train-test split become important.

| 💡 **Definition: Train-test split**                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A **train-test split** is a validation technique used in machine learning where the available data is divided into two parts: a **training set** and a **test set**.<br><br>• The **training set** is used to teach the model and allow it to learn patterns from historical data.<br><br>• The **test set** is kept separate and used only to evaluate how well the model performs on data it has not seen before. |





Instead of giving the model the full history, I split the data into two parts. The first part is used to train the model. The second part is kept aside as a validation period.


<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/prophet_5yr_split_with_full_dataset%20(1).png"
       alt="Forecast Validation Process"
       width="600">
</p>


## How the Validation Process Works

Instead of giving the model the full history, I split the data into two parts.

The first part is used as the **training period**. This is the data the model is allowed to learn from.

The second part is kept aside as the **test period**. This period is not shown to the model during training. From the model’s perspective, this test period represents the future.

The process works like this:



<p align="center">
  <img src="https://github.com/Ibarca/forecast-validation-and-backtesting-/blob/main/Images/forecast_validation_process%20(1).png"
       alt="Forecast Validation Process"
       width="600">
</p>





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
