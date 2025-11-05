**How the backend should receive and process user actions**, and what **attributes to include in the JSON payload** so your ranking/feedback system works efficiently. I’ll break this down carefully.

---

## **1. JSON Payload Design for User Actions**

Each user action (search, click, book, etc.) should be sent as a **structured JSON object**. Here’s a recommended design:

```json
{
  "user_id": "12345",
  "session_id": "abcde-67890",  // tracks user session
  "timestamp": "2025-11-04T15:30:00Z",
  "action_type": "click",        // search, click, book, scroll, rate
  "query": "New York hotels",    // optional, only for search
  "hotel_id": "H123",            // relevant for click/book
  "position": 3,                 // ranking position shown to user
  "price": 250,                  // hotel attribute for context
  "rating": 4.5,                 // hotel rating
  "device": "web",               // web, mobile
  "referrer": "homepage",        // optional
  "duration": 15                 // seconds spent on hotel page, optional
}
```

**Key Points:**

* **user_id**: identify the user (anonymized if needed for privacy).
* **session_id**: groups actions in one session.
* **timestamp**: necessary for ordering and time-based analysis.
* **action_type**: defines the kind of feedback (click/book/scroll).
* **query & hotel_id**: tie the action to search and item.
* **position**: important for **position bias correction**.
* **hotel attributes**: optional, but useful to enrich features for ranking.
* **device/referrer/duration**: optional context for advanced models.

---

## **2. Backend System Design**

We want the backend to **capture actions, validate them, store them, and prepare them for ML pipelines**.

### **Architecture**

```
Frontend --> API Gateway --> Action Service --> Data Pipeline --> Storage --> Training / Analytics
```

**Components:**

1. **API Gateway / Endpoint**

   * Receives JSON payloads from frontend.
   * Validates schema and required attributes.
   * Can throttle or batch requests for efficiency.

2. **Action Service**

   * Processes incoming actions.
   * Adds metadata if missing (e.g., server timestamp, IP-based location).
   * Publishes messages to **message queue** for real-time processing (Kafka, Kinesis).

3. **Data Pipeline**

   * Consumes actions from queue.
   * Cleans and transforms data:

     * Fill missing values
     * Normalize features (e.g., price, rating)
     * Deduplicate events
   * Stores in a data lake / warehouse for training:

     * **Raw logs**: for analytics/debugging
     * **Processed dataset**: for model training (features + labels)

4. **Storage**

   * **Raw events**: S3 / HDFS / BigQuery
   * **Feature dataset**: Redshift / Snowflake / Parquet in S3
   * Supports batch and streaming access for training

5. **Model Training & Feedback Loop**

   * Regular batch jobs or streaming jobs build **learning-to-rank datasets**.
   * Train/update models and deploy to ranking service.

---

## **3. Example Backend Flow in Python (simplified)**

```python
from fastapi import FastAPI, Request
import json
import datetime

app = FastAPI()

@app.post("/log_action")
async def log_action(request: Request):
    data = await request.json()
    
    # Add server timestamp
    data["server_timestamp"] = datetime.datetime.utcnow().isoformat()
    
    # Simple validation
    required_fields = ["user_id", "session_id", "action_type"]
    for f in required_fields:
        if f not in data:
            return {"status": "error", "message": f"Missing {f}"}
    
    # Send to Kafka / Kinesis / S3
    # producer.send('user_actions', json.dumps(data))
    
    return {"status": "success", "message": "Action logged"}
```

**Notes:**

* Use a **message queue** for high throughput instead of writing directly to storage.
* Supports **real-time analytics** and **feedback loop** for ranking.
* Scales horizontally to millions of users.

---

## **4. Capturing Good/Bad Feedback**

* Map `action_type` to **feedback labels**:

  | action_type | label |
  | ----------- | ----- |
  | click       | +1    |
  | book        | +2    |
  | ignore      | 0     |
  | quick_back  | -1    |
* Include **position** for position bias correction.

---

## **5. Summary of Backend Design Principles**

1. **Standardized JSON schema** for all actions.
2. **API endpoint** to collect actions reliably.
3. **Message queue / stream** for real-time handling.
4. **Data pipeline** to clean, enrich, and store data.
5. **Feature dataset** ready for ML training.
6. **Feedback mapping** to generate labels for ranking models.
Do you want me to make that diagram?
