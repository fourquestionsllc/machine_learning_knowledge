Here's an example of training a LightGBM classifier using the specified structure in your dataframe (`df`) and evaluating it with the requested metrics (`F1 score`, `AUROC`, and `Average Precision`) on both training and test datasets:

### Code Example

```python
import lightgbm as lgb
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.metrics import f1_score, roc_auc_score, average_precision_score

# Assuming `df` is already defined
# Identify columns by prefix
categorical_cols = [col for col in df.columns if col.startswith('feature_cat_')]
numerical_cols = [col for col in df.columns if col.startswith('feature_num_')]
label_col = [col for col in df.columns if col.startswith('label_')][0]  # Assuming only one label column

# Features and target
X = df[categorical_cols + numerical_cols]
y = df[label_col]

# Split into training and test sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42, stratify=y)

# LightGBM Dataset
train_data = lgb.Dataset(X_train, label=y_train, categorical_feature=categorical_cols)
test_data = lgb.Dataset(X_test, label=y_test, categorical_feature=categorical_cols, reference=train_data)

# LightGBM Parameters
params = {
    'objective': 'binary',  # Assuming binary classification
    'metric': ['auc', 'average_precision'],  # Metrics for evaluation
    'boosting_type': 'gbdt',
    'learning_rate': 0.05,
    'num_leaves': 31,
    'max_depth': -1,
    'verbose': -1,
    'seed': 42
}

# Train the model
model = lgb.train(
    params,
    train_data,
    valid_sets=[train_data, test_data],
    valid_names=['train', 'test'],
    num_boost_round=1000,
    early_stopping_rounds=50
)

# Predictions
y_train_pred = model.predict(X_train)
y_test_pred = model.predict(X_test)

# Convert probabilities to binary predictions for F1 score
y_train_pred_binary = (y_train_pred >= 0.5).astype(int)
y_test_pred_binary = (y_test_pred >= 0.5).astype(int)

# Evaluation Metrics
train_metrics = {
    'F1 Score': f1_score(y_train, y_train_pred_binary),
    'AUROC': roc_auc_score(y_train, y_train_pred),
    'Average Precision': average_precision_score(y_train, y_train_pred)
}

test_metrics = {
    'F1 Score': f1_score(y_test, y_test_pred_binary),
    'AUROC': roc_auc_score(y_test, y_test_pred),
    'Average Precision': average_precision_score(y_test, y_test_pred)
}

# Report metrics
print("Training Metrics:")
for metric, value in train_metrics.items():
    print(f"{metric}: {value:.4f}")

print("\nTest Metrics:")
for metric, value in test_metrics.items():
    print(f"{metric}: {value:.4f}")
```

### Explanation
1. **Feature Selection**: Columns are split into categorical, numerical, and label based on prefixes.
2. **Data Splitting**: Data is split into training and test sets with a 70-30 split.
3. **LightGBM Dataset**: Categorical columns are explicitly specified for LightGBM to handle them efficiently.
4. **Model Training**: The model uses `early_stopping_rounds` to prevent overfitting.
5. **Evaluation Metrics**:
   - **F1 Score**: Requires binary predictions, so probabilities are converted using a threshold of 0.5.
   - **AUROC** and **Average Precision**: Use raw predicted probabilities.

### Example Output
```
Training Metrics:
F1 Score: 0.9123
AUROC: 0.9785
Average Precision: 0.9612

Test Metrics:
F1 Score: 0.8951
AUROC: 0.9724
Average Precision: 0.9523
``` 

Adjust the `params` dictionary for better performance based on your data. 
