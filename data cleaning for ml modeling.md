# 📘 Data Cleaning & Preprocessing Handbook for ML

---

# 1. 🔍 Data Understanding & Initial Inspection

### Steps

```python
df.shape
df.head()
df.sample(5)
df.info()
df.describe(include='all')
```

### Goals

* Understand structure
* Identify obvious issues
* Detect data types and anomalies

---

# 2. 🧩 Data Issues Identification (Systematic)

---

## 2.1 Missing Values

### Detection

```python
df.isna().sum()
df.isnull().mean() * 100
```

### Pattern Analysis

* MCAR → random
* MAR → depends on other variables
* MNAR → systematic

---

## 2.2 Data Type Issues

### Detection

```python
df.dtypes
```

### Red Flags

* Numbers stored as strings (`"50,000"`, `"70%"`)
* Dates stored as text
* Mixed types in one column

---

## 2.3 Categorical Inconsistencies

### Detection

```python
df['col'].unique()
df['col'].value_counts()
```

### Issues

* Case mismatch (`Male`, `male`)
* Format mismatch (`USA`, `U.S.A.`)
* Extra spaces

---

## 2.4 Invalid Values

### Detection

```python
df[df['age'] < 0]
df[df['rating'] > 5]
```

---

## 2.5 Duplicates

```python
df.duplicated().sum()
```

---

## 2.6 Outliers

### Statistical Detection

```python
Q1 = df['col'].quantile(0.25)
Q3 = df['col'].quantile(0.75)
IQR = Q3 - Q1
```

---

## 2.7 Text / Formatting Issues

```python
df['col'].str.strip()
```

---

## 2.8 Noise / Irrelevant Columns (IDs, Metadata, Leakage)

### A. ID Columns

**Examples:** `id`, `PassengerId`, `track_id`

```python
df.nunique()
```

➡️ Unique values ≈ number of rows → likely ID

---

### B. Metadata Columns

**Examples:**

* timestamps (`created_at`)
* system logs (`processing_date`)
* tracking fields (`session_id`)

---

### C. Data Leakage Columns ⚠️

**Definition:** Columns that contain future or target-related information

```python
df.corr()['target'].sort_values(ascending=False)
```

---

### D. Constant / Redundant Columns

```python
[col for col in df.columns if df[col].nunique() == 1]
```

---

# 3. 🛠️ Data Cleaning (How to Fix Issues)

---

## 3.1 Handling Missing Values

### Strategy

| Type                | Action        |
| ------------------- | ------------- |
| Numeric             | mean / median |
| Categorical         | mode          |
| High missing (>40%) | drop          |
| Target              | drop rows     |

```python
df['col'].fillna(df['col'].median(), inplace=True)
df.dropna(subset=['target'], inplace=True)
```

---

## 3.2 Fix Data Types

```python
df['income'] = df['income'].str.replace(',', '').astype(float)
df['percent'] = df['percent'].str.replace('%','').astype(float)
df['date'] = pd.to_datetime(df['date'])
```

---

## 3.3 Standardize Categorical Data

```python
df['gender'] = df['gender'].str.lower().str.strip()

mapping = {'m':'male','male':'male','f':'female'}
df['gender'] = df['gender'].map(mapping)
```

---

## 3.4 Handle Invalid Values

```python
df.loc[df['age'] < 0, 'age'] = np.nan
df = df[df['rating'] <= 5]
```

---

## 3.5 Remove Duplicates

```python
df.drop_duplicates(inplace=True)
```

---

## 3.6 Handle Outliers

### Remove

```python
df = df[(df['col'] >= lower) & (df['col'] <= upper)]
```

### Cap

```python
df['col'] = np.clip(df['col'], lower, upper)
```

### Transform

```python
df['col'] = np.log1p(df['col'])
```

---

## 3.7 Clean Text

```python
df['col'] = df['col'].str.strip()
df['col'] = df['col'].str.replace(r'\s+', ' ', regex=True)
```

---

## 3.8 Remove Irrelevant Columns (CRITICAL)

### Step 1: Drop ID Columns

```python
df.drop(columns=['id', 'PassengerId'], inplace=True)
```

---

### Step 2: Drop Metadata / Noise

```python
df.drop(columns=['timestamp', 'source'], inplace=True)
```

---

### Step 3: Remove Leakage ⚠️

```python
df.drop(columns=['final_status'], inplace=True)
```

---

### Step 4: Remove Constant Columns

```python
constant_cols = [col for col in df.columns if df[col].nunique() == 1]
df.drop(columns=constant_cols, inplace=True)
```

---

# 4. ⚙️ Feature Engineering

---

## 4.1 Numerical Features

```python
df['income_per_person'] = df['income'] / df['family_members']
```

---

## 4.2 Categorical Encoding

### One-hot

```python
pd.get_dummies(df['col'])
```

### Label Encoding

```python
from sklearn.preprocessing import LabelEncoder
```

---

## 4.3 Date Features

```python
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
```

---

## 4.4 Text Features (NLP)

```python
from sklearn.feature_extraction.text import TfidfVectorizer
```

---

## 4.5 Aggregation (for relational datasets)

```python
df.groupby('ID')['payment'].mean()
```

---

# 5. 📏 Feature Scaling

```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
df[['col']] = scaler.fit_transform(df[['col']])
```

---

# 6. 🧠 Dataset-Type Specific Guidelines

---

## Regression Datasets

(Boston, Chocolate, Flight, Spotify)

* Remove invalid targets
* Handle skew (log transform)
* Scale features

---

## Classification Datasets

(Titanic, Credit, Heart, Spam)

* Encode target
* Handle imbalance (SMOTE/class weights)
* Normalize binary variables

---

## Text Dataset (Spam)

* Clean text
* Tokenize
* TF-IDF

---

## Time-Series / Sequential (Credit, Spotify)

* Sort by time
* Extract time features
* Aggregate history

---

# 7. 📊 Final Validation Checklist

Before modeling:

* ✅ No missing values (handled)
* ✅ Correct data types
* ✅ No duplicates
* ✅ No invalid values
* ✅ Outliers handled
* ✅ Irrelevant columns removed
* ✅ Features encoded
* ✅ Features scaled
* ✅ No data leakage

---

# 8. 🔄 End-to-End Pipeline

```python
# 1. Load data
# 2. Inspect
# 3. Handle missing values
# 4. Fix data types
# 5. Clean categorical variables
# 6. Remove duplicates
# 7. Handle outliers
# 8. Remove irrelevant columns (IDs, leakage)
# 9. Feature engineering
# 10. Encoding
# 11. Scaling
# 12. Train-test split
```

---

# 9. ⚠️ Common Mistakes to Avoid

* ❌ Keeping ID columns → causes overfitting
* ❌ Ignoring data leakage → unrealistic performance
* ❌ Blind imputation without understanding patterns
* ❌ Removing outliers without domain reasoning
* ❌ Scaling before train/test split

---

# 10. 🧠 Pro Tips

* Always **analyze before cleaning**
* Keep a **data cleaning log**
* Validate against domain logic
* Build reusable pipelines
* Treat preprocessing as part of modeling (not separate)

Just tell me 👍
