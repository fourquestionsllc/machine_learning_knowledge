Below is an example of training a **CatBoost Classifier** using Python. The example includes dataset preparation, model training, and evaluation.

### Example: Training a CatBoost Classifier
```python
# Install CatBoost library if not already installed
# pip install catboost

from catboost import CatBoostClassifier, Pool
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report
from sklearn.datasets import load_breast_cancer

# Load a dataset (Breast Cancer dataset in this case)
data = load_breast_cancer()
X = data.data  # Features
y = data.target  # Target labels

# Split the data into training and test sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Initialize the CatBoostClassifier
model = CatBoostClassifier(
    iterations=500,      # Number of boosting iterations
    learning_rate=0.1,   # Learning rate
    depth=6,             # Depth of the tree
    loss_function='Logloss',  # Loss function for binary classification
    eval_metric='Accuracy',   # Evaluation metric
    verbose=50           # Print training progress every 50 iterations
)

# Train the model
model.fit(X_train, y_train, eval_set=(X_test, y_test), early_stopping_rounds=50)

# Make predictions
y_pred = model.predict(X_test)

# Evaluate the model
accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy:.2f}")
print("Classification Report:")
print(classification_report(y_test, y_pred))

# Feature importance (optional)
feature_importances = model.get_feature_importance()
print("Feature Importances:", feature_importances)
```

---

### Key Points:
1. **Data Loading and Preprocessing**:
   - Used `load_breast_cancer` from `sklearn` as a sample dataset.
   - Use real datasets by loading data from files (e.g., CSV, Parquet) and preprocessing it accordingly.

2. **CatBoost Specifics**:
   - CatBoost can handle categorical features directly without one-hot encoding or label encoding. You can specify categorical features using the `cat_features` parameter.

3. **Model Parameters**:
   - `iterations`: Number of boosting iterations.
   - `learning_rate`: Controls the step size of optimization.
   - `depth`: Tree depth.
   - `loss_function`: Determines the objective, e.g., `Logloss` for binary classification.

4. **Early Stopping**:
   - The `early_stopping_rounds` parameter ensures that training stops when the validation metric does not improve for a specified number of rounds.

---

### Output Example:
```
0:	learn: 0.9423661	test: 0.9561404	best: 0.9561404 (0)	total: 48ms	remaining: 23.9s
50:	learn: 0.9957746	test: 0.9561404	best: 0.9649123 (36)	total: 781ms	remaining: 7.06s
100:	learn: 0.9957746	test: 0.9649123	best: 0.9649123 (36)	total: 1.49s	remaining: 5.92s
...
Accuracy: 0.97
Classification Report:
              precision    recall  f1-score   support
           0       0.96      0.96      0.96        42
           1       0.98      0.98      0.98        72
    accuracy                           0.97       114
```
