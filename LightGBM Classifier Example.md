Here's an example of training a LightGBM classifier using a DataFrame `df` with the specified column structure. It includes preprocessing, training, and evaluation by F1 score, AUROC, and average precision.

```
import lightgbm as lgb
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.metrics import f1_score, roc_auc_score, average_precision_score

# Assuming `df` is your DataFrame
# Extract the relevant columns
categorical_cols = [col for col in df.columns if col.startswith('feature_cat_')]
numerical_cols = [col for col in df.columns if col.startswith('feature_num_')]
label_col = [col for col in df.columns if col.startswith('label_')][0]

# Combine feature columns and target column
X = df[categorical_cols + numerical_cols]
y = df[label_col]

# Train-test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

# Handle categorical features (LightGBM requires category dtype for categorical features)
for col in categorical_cols:
    X_train[col] = X_train[col].astype('category')
    X_test[col] = X_test[col].astype('category')

# LightGBM dataset preparation
train_data = lgb.Dataset(X_train, label=y_train, categorical_feature=categorical_cols)
test_data = lgb.Dataset(X_test, label=y_test, categorical_feature=categorical_cols, reference=train_data)

# Set LightGBM parameters
params = {
    'objective': 'binary',
    'metric': ['auc', 'average_precision'],
    'boosting_type': 'gbdt',
    'num_leaves': 31,
    'learning_rate': 0.05,
    'feature_fraction': 0.8,
    'verbose': -1
}

# Train the model
model = lgb.train(
    params,
    train_data,
    valid_sets=[train_data, test_data],
    valid_names=['train', 'eval'],
    num_boost_round=1000,
    early_stopping_rounds=50
)

# Make predictions
y_pred_prob = model.predict(X_test, num_iteration=model.best_iteration)
y_pred = (y_pred_prob > 0.5).astype(int)

# Evaluate the model
f1 = f1_score(y_test, y_pred)
auroc = roc_auc_score(y_test, y_pred_prob)
avg_precision = average_precision_score(y_test, y_pred_prob)

print(f"F1 Score: {f1:.4f}")
print(f"AUROC: {auroc:.4f}")
print(f"Average Precision: {avg_precision:.4f}")
```

### Explanation:

1. **Data Preparation**:
   - Categorical columns are identified and converted to `category` dtype for LightGBM's efficient handling.
   - The `label_col` is extracted as the target variable.

2. **Train-Test Split**:
   - Splits the data into training and testing sets with a stratified split to maintain label distribution.

3. **LightGBM Dataset**:
   - LightGBM requires data to be passed as `lgb.Dataset` for training. It supports `categorical_feature` for specifying categorical columns.

4. **Model Training**:
   - Uses `binary` objective for classification and metrics like `auc` and `average_precision`.
   - Implements early stopping to prevent overfitting.

5. **Evaluation**:
   - Predictions are made as probabilities, then converted to binary predictions using a threshold of 0.5.
```
   - Metrics include F1 score, AUROC, and average precision.
