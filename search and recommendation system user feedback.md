Design a **hotel search and recommendation system** (like for Hyatt) that can **learn from user feedback** to improve ranking. Let’s break this down **step by step**, with focus on **capturing good/bad results and building the feedback loop**.

---

## **1. Overall Architecture**

```
User Query → Search Engine → Ranked Hotel List → User Interaction → Feedback Capture → Model Training → Ranking Update
```

**Components:**

1. **Frontend**

   * Web/mobile interface where users search for hotels.
   * Users see results and interact (click, book, scroll).

2. **Search & Ranking Engine**

   * Returns hotel results based on query + ranking model.
   * Ranking could use:

     * Rule-based factors (distance, price, rating)
     * ML model (learning-to-rank, neural network, gradient boosting)

3. **Feedback Capture**

   * Captures signals of good/bad outcomes from user actions.

4. **Data Pipeline**

   * ETL to process interaction logs.
   * Label training data (good/bad results) for ranking model.

5. **Model Training & Deployment**

   * Use captured feedback to train ranking model.
   * Periodically update the model to improve search results.

---

## **2. How to Capture Good/Bad Feedback**

You need **implicit and explicit feedback** from user interactions:

### **Implicit Feedback (most common)**

| Signal                   | Interpretation         | Good/Bad  |
| ------------------------ | ---------------------- | --------- |
| Click on a hotel         | User is interested     | Good      |
| Book a hotel             | Strong positive signal | Very good |
| Dwell time on hotel page | Longer = more engaged  | Good      |
| Scroll past / ignore     | User not interested    | Bad       |
| Quick back navigation    | Dissatisfied           | Bad       |

> These signals are **implicit labels** for ML models.

### **Explicit Feedback (optional)**

* Ratings, thumbs-up/down, survey responses.
* Harder to get at scale but very reliable.

---

## **3. How to Build the Feedback Loop**

1. **Log User Interactions**

   * Capture all actions: search query, hotel list shown, clicks, bookings.
   * Store in a centralized data store (S3, BigQuery, Redshift, etc.).

   ```python
   log = {
       "user_id": "123",
       "query": "New York hotels",
       "hotel_id": "H456",
       "position": 3,
       "clicked": True,
       "booked": False,
       "timestamp": "2025-11-04T12:00:00"
   }
   ```

2. **Generate Labels for Training**

   * For **learning-to-rank**, typical approach:

     * Clicked/Booked → positive label
     * Not clicked → negative label (or lower weight)
   * Normalize by position to account for **position bias**:

     * Hotels at top are more likely to be clicked.
     * Use **inverse propensity scoring** to correct bias.

3. **Aggregate Data for Model Training**

   * Build a dataset like:

   ```
   features: hotel features (price, distance, rating, amenities)
   labels: user engagement score (click/book)
   ```

   * This becomes the **training data for your ranking model**.

4. **Train Ranking Model**

   * Algorithms:

     * Gradient Boosted Decision Trees (XGBoost, LightGBM)
     * Neural Learning-to-Rank models (LambdaMART, BERT for ranking)
   * Objective: maximize user engagement / bookings.

5. **Deploy Updated Ranking**

   * Push model to search engine.
   * Serve ranked results for new queries.

6. **Repeat Loop**

   * Continuously log interactions → retrain → improve ranking.

---

## **4. Example: Feedback to Training Dataset**

```python
import pandas as pd

# Example raw logs
logs = pd.DataFrame([
    {"hotel_id":"H1","position":1,"clicked":True,"booked":False},
    {"hotel_id":"H2","position":2,"clicked":False,"booked":False},
    {"hotel_id":"H3","position":3,"clicked":True,"booked":True}
])

# Convert to training labels
def label(row):
    if row['booked']:
        return 2  # strong positive
    elif row['clicked']:
        return 1  # weak positive
    else:
        return 0  # negative

logs['label'] = logs.apply(label, axis=1)

# Now logs can be used to train ranking model
print(logs)
```

**Output:**

| hotel_id | position | clicked | booked | label |
| -------- | -------- | ------- | ------ | ----- |
| H1       | 1        | True    | False  | 1     |
| H2       | 2        | False   | False  | 0     |
| H3       | 3        | True    | True   | 2     |

---

## **5. Additional Tips**

* **A/B Testing:** Always test new ranking models live with a fraction of users to validate improvements.
* **Cold Start:** For new hotels with no interactions, use content-based ranking.
* **Bias Handling:** Correct for position bias and popularity bias in feedback.
* **Real-Time Feedback:** For large sites, consider streaming interactions with **Kafka/Kinesis** to update rankings faster.


