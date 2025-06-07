# Power Outage Analysis
*By Sohan Shingade*

This report presents an end-to-end data analysis project conducted as part of the DSC 80 final project at UC San Diego. The goal is to analyze and predict power outage durations in the United States using a structured dataset and the full data science lifecycle.

## Introduction

Power outages affect millions of people every year. Understanding why some outages last longer than others can help communities, utilities, and policymakers better prepare for and respond to these disruptions.

In this project, we investigate the following research question:

**Do weather-related power outages tend to last longer than outages caused by other reasons?**

This question is important because weather events like storms, hurricanes, and extreme heat are becoming more frequent due to climate change. If such outages are more disruptive, identifying this pattern could inform infrastructure investments and emergency planning.

The dataset contains information on thousands of power outages in the U.S. After cleaning, we analyzed **2,247** outage events. The following columns are particularly relevant:

- `OUTAGE.DURATION`: Duration of the outage in hours.
- `CAUSE.CATEGORY`: Broad cause of the outage.
- `OUTAGE.START`, `OUTAGE.RESTORATION`: Timestamps of the outage start and restoration.
- `U.S._STATE`: State where the outage occurred.
- `CLIMATE.CATEGORY`: Climate classification.
- `POPULATION`, `POPDEN_URBAN`, `POPDEN_RURAL`: Demographic context.
- `RES.PRICE`, `COM.PRICE`: Electricity prices.
- `GDP`: Economic output of the region.

Our objective is to analyze whether weather-related outages are statistically longer and to build a model that predicts outage duration from available predictors.

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The raw dataset required several preprocessing steps to ensure accurate analysis:

- **Metadata Removal**: Skipped the first three non-data rows in the Excel file.
- **Datetime Construction**: Combined separate date and time columns into unified `OUTAGE.START` and `OUTAGE.RESTORATION` timestamps.
- **Duration Calculation**: Created `OUTAGE.DURATION` by subtracting start time from restoration time (in hours).
- **Data Type Correction**: Parsed string-encoded numerical values (e.g., GDP, prices) into floats.
- **Missing Value Handling**: Replaced placeholder strings with `NaN` values.

These steps reflect typical issues encountered in real-world datasets and ensured consistent formatting for downstream analysis.

### Univariate Analysis

<iframe src="assets/univar_1_outage_duration_hist.html" width="800" height="600" frameborder="0"></iframe>

This bar chart shows average outage duration by cause category. Weather-related causes lead to significantly longer outages on average compared to categories such as equipment failure or vandalism.

<iframe src="assets/univar_2_outage_duration_hist.html" width="800" height="600" frameborder="0"></iframe>

The histogram displays the distribution of outage durations (top 2% removed). Most outages last under 20 hours, but a right-skewed tail suggests occasional extreme durations.

### Bivariate Analysis

<iframe src="assets/bivar_1_outage_duration_hist.html" width="800" height="600" frameborder="0"></iframe>

This scatter plot compares population and outage duration (top 2% removed). There is no clear relationship, implying other features may drive outage length.

<iframe src="assets/bivar_4_outage_duration_hist.html" width="800" height="600" frameborder="0"></iframe>

The box plot shows outage duration by month. February and June have longer tails, possibly due to seasonal weather patterns.

### Interesting Aggregates

The table below shows the average outage duration (in minutes) for each combination of `CAUSE.CATEGORY` and `CLIMATE.CATEGORY`:

| CAUSE.CATEGORY        |      cold |   normal |      warm |
|:----------------------|----------:|---------:|----------:|
| equipment failure     |   308.235 | 3201.43  |   505     |
| fuel supply emergency | 17433     | 7658.82  | 22799.7   |
| intentional attack    |   497.282 |  426.818 |   312.557 |
| islanding             |   259.267 |  142.176 |   209.833 |
| public appeal         |  2125.91  | 1376.53  |   596.231 |

Both cause and climate category significantly impact outage duration. Fuel emergencies and severe weather consistently result in longer outages, while events like vandalism are shorter and less climate-sensitive.

## Assessment of Missingness

### NMAR Analysis

We analyzed the missingness of the `CUSTOMERS.AFFECTED` column. Smaller outages in remote areas may go unrecorded, indicating that missingness may depend on the value itself—suggesting Not Missing At Random (NMAR).

### Missingness Dependency Tests

<iframe src="assets/missingness_vs_duration.html" width="800" height="600" frameborder="0"></iframe>

The test for `OUTAGE.DURATION` revealed a mean difference of 773.5 minutes with a p-value of 0.019. We reject the null hypothesis and conclude that shorter outages are more likely to have missing `CUSTOMERS.AFFECTED` values, consistent with NMAR.

<iframe src="assets/missingness_vs_population.html" width="800" height="600" frameborder="0"></iframe>

For `POPULATION`, the p-value was 0.321. We do not find statistical evidence of dependence, suggesting missingness is MCAR with respect to population.

### Conclusion

- `CUSTOMERS.AFFECTED` is NMAR with respect to outage duration.
- It appears MCAR with respect to population.

## Hypothesis Testing

### Question

Do weather-related outages last longer than non-weather-related ones?

### Hypotheses

- **Null (H₀):** Mean durations are equal.
- **Alternative (H₁):** Weather-related outages have longer mean durations.

### Test Setup

- **Test**: One-sided Welch’s t-test  
- **Observed difference**: 798.56 minutes  
- **p-value**: 0.012  
- **Significance level**: 0.05

<iframe src="assets/hypothesis_duration_by_weather.html" width="800" height="600" frameborder="0"></iframe>

The box plot shows weather-related outages generally last longer. Outliers were excluded (top 2%).

### Conclusion

We reject the null hypothesis. There is statistical evidence that weather-related outages tend to last longer than others.

## Framing a Prediction Problem

We now attempt to predict the duration of an outage using available data.

### Prediction Problem and Type

This is a **regression** problem:  
**Given contextual and situational data at the start of an outage, predict how long the outage will last (in minutes).**

### Response Variable

- **Target**: `OUTAGE.DURATION`  
  This variable is both operationally and economically meaningful, and is computable from known start and end times.

### Evaluation Metric

- **Metric**: Root Mean Squared Error (RMSE)  
  RMSE is appropriate because it emphasizes large errors and is interpretable in minutes.

### Features Used

Only features known **at the time of prediction** were used, including:

- `CAUSE.CATEGORY`, `CLIMATE.CATEGORY`, `ANOMALY.LEVEL`
- `U.S._STATE`, `YEAR`, `MONTH`
- `GDP`, `POPULATION`, `POPDEN_URBAN`, `POPDEN_RURAL`
- `RES.PRICE`, `COM.PRICE`

This ensures temporal validity and makes the model feasible for real-time prediction.

## Baseline Model
## Baseline Model

To establish a performance benchmark for predicting outage duration, we constructed a baseline regression model using a linear approach with minimal feature engineering.

### Model Description

We implemented a **Linear Regression** model using a `sklearn` pipeline that included preprocessing for both numerical and categorical variables:

- **Numerical features** were standardized using `StandardScaler`.
- **Categorical features** were encoded using `OneHotEncoder` with `handle_unknown="ignore"`.

This pipeline allows for consistent transformation and easy reproducibility across training and test sets.

### Features Used

The model used the following features:

#### Quantitative (6):
- `YEAR`
- `MONTH`
- `ANOMALY.LEVEL`
- `POPULATION`
- `POPDEN_URBAN`
- `POPDEN_RURAL`

#### Nominal (6):
- `U.S._STATE`
- `NERC.REGION`
- `CAUSE.CATEGORY`
- `CAUSE.CATEGORY.DETAIL`
- `CLIMATE.REGION`
- `CLIMATE.CATEGORY`

There were no ordinal variables in this model. All categorical features were one-hot encoded. Only rows with no missing values in these features or in the target (`OUTAGE.DURATION`) were used.

---

### Performance

- **Root Mean Squared Error (RMSE) on Test Set:** **6,728.98 minutes** (approximately 4.7 days)

This value reflects the average error between predicted and actual durations and provides a reference for future models.

---

### Residual Diagnostics

<iframe src="assets/baseline_residuals_vs_predicted.html" width="800" height="600" frameborder="0"></iframe>

This plot shows the residuals versus predicted outage durations. Residuals tend to grow in variance for longer predictions, indicating **heteroscedasticity** — a violation of the constant error variance assumption in linear regression.

<iframe src="assets/baseline_avg_residual_by_year.html" width="800" height="600" frameborder="0"></iframe>

This line plot displays the **average residual per year**. The model tends to **underpredict** durations in earlier years and **overpredict** around 2009, suggesting possible **temporal drift** or shifts in recording practices.

<iframe src="assets/baseline_residual_hist.html" width="800" height="600" frameborder="0"></iframe>

The histogram of residuals is roughly centered around 0 but shows a **right-skewed distribution**, indicating that the model often **underestimates very long outages**.

---

### Interpretation

The baseline linear model captures broad trends but shows several limitations:

- It struggles with extreme durations.
- Residuals are heteroscedastic, violating assumptions of linear regression.
- It shows bias over time, particularly in older records.

Despite these limitations, the model serves as a valid and interpretable **baseline** against which more complex models (e.g., Random Forests) can be measured. A significant drop in RMSE in future models would demonstrate better fit to the true data-generating process.


## Final Model
...

## Fairness Analysis
...
