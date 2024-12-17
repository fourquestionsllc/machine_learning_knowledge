To train a model using **OrdinalXGBoost** with the specified dataframe, we need to handle categorical and numerical features, deal with missing values, and account for label imbalance. Below is the complete Python program along with an explanation.

---

### **Python Program**

```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report
import ordinal_xgboost as oxgb

# Example: Load your dataframe (replace this with your actual dataframe)
# df = pd.read_csv("your_data.csv")

# Split features and labels
features = df.filter(like="feature_")  # Extract feature columns
categorical_cols = features.filter(like="feature_cat__").columns  # Categorical columns
numerical_cols = features.filter(like="feature_num__").columns  # Numerical columns
label_col = df.filter(like="label__").columns[0]  # Label column

# Handle categorical features: OrdinalXGBoost supports numerical encoding
# Encode categorical features as integers (assuming missing values are NaN)
for col in categorical_cols:
    features[col] = features[col].astype("category").cat.codes

# Separate features and target variable
X = features
y = df[label_col]

# Split data into train and test sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

# Handle missing values: OrdinalXGBoost handles missing values natively, so no imputation is needed.

# Handle class imbalance: Use class weights in the training
class_weights = {0: 1.0, 1: 80 / 6, 2: 80 / 14}  # Example weights inversely proportional to class frequency
sample_weights = y_train.map(class_weights)

# Train OrdinalXGBoost model
model = oxgb.OrdinalXGBClassifier()
model.fit(X_train, y_train, sample_weight=sample_weights)

# Predict on the test set
y_pred = model.predict(X_test)

# Evaluate the model
print("Classification Report:")
print(classification_report(y_test, y_pred, target_names=["Level 0", "Level 1", "Level 2"]))
```

---

### **Explanation**

#### **1. Feature Preparation**
- Extracted **categorical** and **numerical** columns based on their prefixes (`feature_cat__` and `feature_num__`).
- **Categorical Encoding**:
  - OrdinalXGBoost expects all features to be numeric. Categorical features are converted to integers using `.astype("category").cat.codes`.
  - Missing values in categorical features remain `-1`, which OrdinalXGBoost can handle natively.

#### **2. Handling Missing Values**
- No explicit imputation is performed as OrdinalXGBoost handles `NaN` values internally by learning the optimal split for missing values.

#### **3. Handling Class Imbalance**
- The class distribution is highly imbalanced (80% `Level 0`, 6% `Level 2`).
- Applied **class weights** inversely proportional to class frequencies. This ensures the model gives more importance to minority classes.
- `sample_weight` is passed to `fit()` for balanced learning.

#### **4. Train-Test Split**
- Used `train_test_split()` with `stratify=y` to maintain the class distribution in both training and test sets.

#### **5. Training the Model**
- **OrdinalXGBoost** (`oxgb.OrdinalXGBClassifier`) is used to train an ordinal regression model directly.
- `fit()` accepts `sample_weight` to address class imbalance.

#### **6. Evaluation**
- Predictions are made on the test set.
- The **classification report** provides precision, recall, and F1-score for each level (`0`, `1`, `2`).

---

### **Class Imbalance Note**
If the performance on minority classes is still suboptimal, consider:
- **Oversampling**: Use techniques like SMOTE to balance the dataset.
- **Undersampling**: Reduce the majority class samples.
- **Hyperparameter tuning**: Use `scale_pos_weight` for further balancing.
