### Python Program for Ordinal Regression with `OrdinalXGBoost`

Here's the complete code for using `OrdinalXGBoost` to train a model that accounts for missing values, categorical and numerical features, and the imbalanced nature of the labels.

#### Python Code
```python
import pandas as pd
import numpy as np
from ordinal_xgboost import OrdinalXGBClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_absolute_error, cohen_kappa_score, confusion_matrix

# Sample data frame (replace with your actual data)
data = pd.read_csv("your_data.csv")

# Separate features and label
features = data.filter(regex="feature_.*")  # Select columns with prefix `feature_`
label = data.filter(regex="label__").squeeze()  # Select the label column

# Split categorical and numerical features
categorical_features = features.filter(regex="feature_cat__").columns.tolist()
numerical_features = features.filter(regex="feature_num__").columns.tolist()

# Handle missing values in categorical features by replacing NaN with a placeholder
features[categorical_features] = features[categorical_features].fillna("missing")

# Split the data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(features, label, test_size=0.2, stratify=label, random_state=42)

# Define sample weights to address label imbalance
class_weights = {0: 1.0, 1: 80 / 14, 2: 80 / 6}  # Adjust weights inversely proportional to class frequency
sample_weights = y_train.map(class_weights)

# Train the Ordinal XGBoost model
model = OrdinalXGBClassifier(
    random_state=42,
    n_estimators=200,
    learning_rate=0.1,
    max_depth=6,
    min_child_weight=2,
    subsample=0.8,
    colsample_bytree=0.8,
    importance_type="weight"
)

# Fit the model with sample weights
model.fit(X_train, y_train, sample_weight=sample_weights)

# Predict on the test set
y_pred = model.predict(X_test)

# Evaluate the model
mae = mean_absolute_error(y_test, y_pred)
kappa = cohen_kappa_score(y_test, y_pred, weights="quadratic")
conf_matrix = confusion_matrix(y_test, y_pred)

# Print results
print("Mean Absolute Error (MAE):", mae)
print("Quadratic Weighted Kappa (QWK):", kappa)
print("Confusion Matrix:\n", conf_matrix)
```

---

### Explanation of the Code

#### 1. **Handling Missing Values**
- **Categorical Features**: Missing values are replaced with a placeholder `"missing"`. This ensures that the `OrdinalXGBClassifier` treats missing categorical values as a unique category.
- **Numerical Features**: Missing numerical values are handled natively by `OrdinalXGBClassifier`.

#### 2. **Imbalanced Classes**
- Classes are imbalanced (80% for level 0, 6% for level 2). To handle this, we assign **sample weights** inversely proportional to class frequencies using:
  ```python
  class_weights = {0: 1.0, 1: 80 / 14, 2: 80 / 6}
  ```
- These weights are passed to the `fit` method via the `sample_weight` parameter.

#### 3. **Model Training**
- `OrdinalXGBClassifier` from the `ordinal-xgboost` library is used. It handles the ordinal nature of the target by using specialized loss functions (explained below).

#### 4. **Evaluation Metrics**
- **Mean Absolute Error (MAE)**: Measures the average absolute difference between predicted and true levels. Lower MAE indicates better performance.
- **Quadratic Weighted Kappa (QWK)**: Measures the agreement between predictions and true labels, with more weight given to larger disagreements.
- **Confusion Matrix**: Summarizes the distribution of predicted vs. actual levels.

---

### Loss Function in Ordinal Regression

Ordinal regression in `OrdinalXGBoost` relies on **Cumulative Logit** or **Cumulative Probit Models**, where the loss function is designed to preserve the order in the labels.

#### Mechanism:
1. **Threshold Model**:
   - The model predicts probabilities for each ordinal category using cumulative probabilities:
     \[
     P(y \leq k) = \sigma(f(x) - b_k)
     \]
     where:
     - \( \sigma \): Sigmoid function.
     - \( f(x) \): Model's raw score.
     - \( b_k \): Threshold for category \( k \).

2. **Likelihood-based Loss**:
   - The model minimizes the negative log-likelihood:
     \[
     L = -\sum \log(P(y = k))
     \]
   - Ensures that the probability distribution over the ordinal classes respects the ordering.

3. **Order Constraint**:
   - During optimization, the thresholds \( b_k \) are constrained to ensure \( b_1 \leq b_2 \leq \dots \leq b_K \).

---

### Popular Performance Metrics for Ordinal Regression

1. **Mean Absolute Error (MAE)**:
   - Measures the average absolute error between the predicted and true levels.
   - Formula:
     \[
     MAE = \frac{1}{n} \sum_{i=1}^n |y_i - \hat{y}_i|
     \]
   - Lower MAE indicates better performance.

2. **Quadratic Weighted Kappa (QWK)**:
   - Captures agreement between predicted and actual labels, emphasizing the ordinal nature of the target.
   - Formula:
     \[
     QWK = 1 - \frac{\sum_{i,j} W_{i,j} O_{i,j}}{\sum_{i,j} W_{i,j} E_{i,j}}
     \]
     where:
     - \( O_{i,j} \): Observed matrix.
     - \( E_{i,j} \): Expected matrix (based on marginal distributions).
     - \( W_{i,j} \): Weight matrix (quadratic).

3. **Confusion Matrix**:
   - Helps visualize the distribution of true vs. predicted levels.
   - Especially useful for ordinal regression to assess how often the model predicts neighboring classes.

---

### Final Notes

- **Handling Imbalance**: Using sample weights improves predictions for minority classes (levels 1 and 2).
- **Loss Function**: The cumulative logit/probit model ensures the output respects ordinal constraints.
- **Evaluation Metrics**: Use MAE and QWK to evaluate ordinal models, with QWK being particularly insightful for imbalanced and ordered targets.
