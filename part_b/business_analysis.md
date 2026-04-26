# B1 - Problem Formulation

## (a) Machine Learning Problem

### Target Variable

The target variable is **`items_sold`**, which represents the number of products sold during a promotion.

### Candidate Input Features

* Promotion type
* Store location type (urban, semi-urban, rural)
* Store size
* Monthly footfall
* Local competition density
* Customer demographics
* Time-based features (month, day of week, etc.)

### Type of ML Problem

This is a **supervised learning regression problem**, as the goal is to predict a continuous numerical value (items sold) based on past data.

### Justification

Regression is appropriate because:

* The output (`items_sold`) is continuous
* We are predicting a quantity rather than a category
* The objective is to estimate the magnitude of sales under different conditions based on previous data

---

## (b) Why Use Items Sold Instead of Revenue

### Explanation

Using **sales revenue** as a target variable can be misleading because revenue is influenced by factors such as pricing, discounts, and product mix. For example, a high discount promotion may increase items sold but reduce total revenue per item.

In contrast, **items sold (sales volume)** directly measures customer response to a promotion, making it a more reliable indicator of promotion effectiveness.

### Broader Principle

This illustrates the principle that:

> *The target variable in a machine learning problem should align closely with the actual business objective and should not be distorted by external or confounding factors.*

Choosing the right target makes it sure that the model learns meaningful patterns and supports correct decision-making.

---

## (c) Alternative Modelling Strategy

### Proposed Approach

Instead of a single global model, use a **segmented or hierarchical modelling approach**, such as:

* Separate models for different store types (urban, semi-urban, rural), or
* A model that includes interaction effects between promotion type and location

### Justification

Stores in different locations have different customer behaviors, demand patterns, and sensitivities to promotions. A single global model may average out these differences and fail to capture location-specific patterns.

By segmenting the model:   

* Predictions become more accurate
* Local patterns are better captured
* Custome business decisions to each store type
 
### Business Insight

This approach enables the company to deploy **location-specific promotion strategies**, improving overall effectiveness and maximizing sales.

---

# B2 - Data and EDA Strategy

## (a) Data Integration and Dataset Design

### Table Joins

The four tables would be joined using appropriate keys:

* **Transactions table** → primary fact table (contains sales records)
* **Store attributes table** → joined using `store_id`
* **Promotion details table** → joined using `promotion_id` (or mapped via date/store if needed)
* **Calendar table** → joined using `transaction_date`

This results in a unified dataset combining transactional, store-level, promotion, and temporal information.

---

### Grain of the Dataset

The final modelling dataset should have the grain:

> **One row = one store per day (or store–promotion–day combination)**

This ensures that the model captures how each store performs under specific conditions on a given day.

---

### Aggregations

Before modelling, transactional data should be aggregated to match the chosen grain:

* Total `items_sold` per store per day
* Total revenue (optional for analysis)
* Average or total footfall (if available at transaction level)
* Count of transactions per store per day

Additional derived features:

* Indicator for whether a promotion was active
* Number of promotion days in a period
* Weekend/festival flags from the calendar

---

## (b) Exploratory Data Analysis (EDA)

### 1. Sales by Promotion Type (Bar Chart)

**What to check:**

* Average items sold across different promotion types

**Why:**

* Identify which promotions perform best

**Impact:**

* Helps prioritize high-performing promotions
* May guide feature importance expectations

---

### 2. Time Series Analysis (Line Plot)

**What to check:**

* Trends in items sold over time (daily/monthly)
* Seasonality patterns (e.g., festivals, weekends)

**Impact:**

* Leads to creation of time-based features (month, weekday, festival flag)
* Helps detect trends or anomalies

---

### 3. Distribution of Target Variable (Histogram)

**What to check:**

* Skewness, outliers, and spread of `items_sold`

**Impact:**

* May require transformation (e.g., log transformation)
* Helps choose appropriate models and evaluation metrics

---

### 4. Correlation Analysis (Heatmap)

**What to check:**

* Relationships between numerical features (footfall, store size, items sold)

**Impact:**

* Identify strong predictors
* Detect multicollinearity
* Guide feature selection

---

### 5. Sales by Store Type / Location (Box Plot)

**What to check:**

* Variation in sales across urban, semi-urban, and rural stores

**Impact:**

* Supports segmentation strategy
* Suggests interaction features (promotion × location)

---

## (c) Handling Imbalance in Promotions

### Problem

Since **80% of transactions have no promotion**, the model may:

* Become biased toward predicting non-promotion outcomes
* Underestimate the true impact of promotions

---

### Steps to Address

1. **Feature Engineering**

   * Create a binary feature: `is_promotion`
   * Add interaction features (promotion × store type, promotion × season)

2. **Resampling or Weighting**

   * Oversample promotion cases or assign higher importance (weights) to them

3. **Stratified Analysis**

   * Evaluate model performance separately for promotion vs non-promotion cases

---

### Business Insight

Addressing this imbalance ensures the model properly learns the **true effect of promotions**, which is critical for making accurate marketing decisions.

---

# B3 - Model Evaluation and Deployment 

## (a) Train-Test Split and Evaluation Metrics

### Train-Test Split Strategy

Since the data is **time-based (monthly over three years)**, a **temporal split** should be used:

* Train on the **first ~80% of time period** (e.g., first 2.4 years)
* Test on the **most recent ~20%** (last 7–8 months)


---

### Why Random Split is Inappropriate

A random split mixes past and future data, causing **data leakage**:

* The model may learn patterns from future data
* Leads to overly optimistic performance
* Does not reflect real-world forecasting (where only past data is available)

---

### Evaluation Metrics

#### 1. RMSE (Root Mean Squared Error)

* Penalizes large errors more heavily
* Useful for detecting poor performance during **high-sales months**

**Interpretation:**
A high RMSE indicates the model struggles with peak demand periods, which can lead to poor inventory planning.

---

#### 2. MAE (Mean Absolute Error)

* Measures average prediction error

**Interpretation:**
If MAE = 50, predictions are off by ~50 items on average, helping assess day-to-day accuracy.

---

#### 3. (Optional) MAPE (Mean Absolute Percentage Error)

* Measures error in percentage terms

**Interpretation:**
Useful for comparing performance across stores with different sales volumes.

---

### Business Perspective

* **MAE** → operational accuracy (typical error)
* **RMSE** → risk of large mistakes (important for peak seasons)
* **MAPE** → relative performance across stores

---

## (b) Explaining Different Recommendations Using Feature Importance

### Problem

The model recommends:

* **Loyalty Points Bonus** in December
* **Flat Discount** in March
  for the same store

---

### Investigation Approach

1. **Global Feature Importance**

   * Identify key drivers such as:

     * Month/seasonality
     * Promotion type
     * Footfall
     * Store characteristics

2. **Local Explanation (Instance-Level)**

   * Analyze feature contribution for Store 12 in:

     * December
     * March

3. **Compare Feature Values**

   * December:

     * Likely high seasonal demand (festivals, holidays)
     * Customers respond to loyalty incentives
   * March:

     * Lower seasonal demand
     * Price-sensitive behavior → discounts more effective

---

### Communication to Marketing Team

> “The model recommends different promotions because customer behavior changes depending on season. In December, higher seasonal demand due to festivals and repeat customers make loyalty-based incentives more effective. In contrast, March shows lower demand, where direct price discounts drive higher sales. The model captures these seasonal and behavioral patterns to optimize promotion effectiveness.”

---

### Business Insight

* Promotions should be **seasonally adaptive**
* Same store ≠ same strategy year-round

---

## (c) Model Deployment and Monitoring

### 1. Model Saving

After training, save the pipeline (preprocessing + model):

```python
import joblib
joblib.dump(rf_pipeline, "promotion_model.pkl")
```

---

### 2. Monthly Prediction Workflow

At the start of each month:

1. Collect new data:

   * Store attributes
   * Planned promotions
   * Calendar features

2. Prepare input data:

   * Apply same feature engineering steps
   * Ensure schema matches training data

3. Load model and predict:

```python
model = joblib.load("promotion_model.pkl")
predictions = model.predict(new_data)
```

4. Generate recommendations:

   * Select promotion with highest predicted `items_sold` per store

---

### 3. Monitoring and Performance Tracking

#### a. Prediction Monitoring

* Track predicted vs actual sales monthly
* Monitor RMSE / MAE over time

#### b. Data Drift Detection

* Check if input features (footfall, promotions, etc.) change significantly
* Example: sudden change in customer behavior

#### c. Concept Drift

* If relationship between features and sales changes (e.g., promotions become less effective)

---

### 4. Retraining Strategy

Retrain model when:

* Performance drops beyond threshold
* Significant data drift is detected
* Periodically (e.g., every 6–12 months)

---

### Business Insight

This deployment ensures:

* Scalable monthly recommendations across all stores
* Consistent decision-making
* Early detection of performance degradation, maintaining model reliability


