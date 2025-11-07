How to **design a real-time hotel recommendation system for Hyatt** on **AWS**, with **streaming data** (e.g., live user clicks, bookings, feedback).

Let’s design this as a **production-grade architecture** used by enterprises like Hyatt, Expedia, or Marriott.

---

# 🏨 Real-Time Recommendation System on AWS — Hyatt Example

## 🎯 Goal

Deliver **personalized hotel recommendations in real-time** using:

* Streaming user interactions (searches, clicks, bookings)
* Machine learning models (ranking, embeddings)
* Continuous feedback loop (online learning)

---

## 🧱 1. High-Level Architecture Overview

```
         ┌───────────────────────────────┐
         │   Web / Mobile Frontend       │
         │ (Search, Browse, Booking)     │
         └──────────────┬────────────────┘
                        │ JSON events
                        ▼
       ┌────────────────────────────────────┐
       │     Amazon Kinesis Data Streams     │  🔄  (real-time data ingestion)
       └──────────────────┬─────────────────┘
                          │
                 ┌─────────┴─────────┐
                 │                   │
                 ▼                   ▼
       ┌─────────────────┐   ┌─────────────────┐
       │ AWS Lambda /    │   │ Kinesis Data    │
       │ Kinesis Firehose│   │ Analytics (SQL) │  🔍  feature enrichment / filtering
       └─────────────────┘   └─────────────────┘
                          │
                          ▼
           ┌──────────────────────────────┐
           │   Feature Store (Amazon SageMaker Feature Store) │  📦  store user & item features
           └──────────────────────────────┘
                          │
                          ▼
           ┌──────────────────────────────┐
           │ SageMaker Real-Time Endpoint │  🤖  model inference (PyTorch / XGBoost)
           └──────────────────────────────┘
                          │
                          ▼
           ┌──────────────────────────────┐
           │   API Gateway + Lambda / ECS │  🌐  REST API for app
           └──────────────────────────────┘
                          │
                          ▼
           ┌──────────────────────────────┐
           │   DynamoDB / Elasticsearch   │  📊  store rec results + logs
           └──────────────────────────────┘
```

---

## ⚙️ 2. AWS Services Breakdown

| Layer               | AWS Service                                                               | Purpose                                                          |
| ------------------- | ------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Event Ingestion** | **Amazon Kinesis Data Streams**                                           | Collect live user behavior (clicks, searches, bookings).         |
| **Transformation**  | **AWS Lambda** / **Kinesis Data Analytics**                               | Enrich or filter event data (user_id, timestamp, location).      |
| **Feature Store**   | **SageMaker Feature Store**                                               | Maintain real-time user/item embeddings and contextual features. |
| **Model Serving**   | **SageMaker Real-Time Endpoint**                                          | Serve ML models (PyTorch, XGBoost, or Factorization Machines).   |
| **API Layer**       | **API Gateway + Lambda or ECS (FastAPI)**                                 | Provide REST endpoint `/recommendations?user_id=123`.            |
| **Data Storage**    | **S3** (raw data), **DynamoDB** (session state), **Redshift** (analytics) | Persist logs, offline training data.                             |
| **Monitoring**      | **CloudWatch + SageMaker Model Monitor**                                  | Track latency, drift, and model quality.                         |
| **A/B Testing**     | **MLflow or SageMaker Experiments**                                       | Track different model versions.                                  |

---

## 📡 3. Data Flow (End-to-End)

1. **Frontend sends events**

   ```json
   {
     "event": "hotel_view",
     "user_id": "U123",
     "hotel_id": "H456",
     "timestamp": "2025-11-06T15:30:00Z"
   }
   ```

   → Sent via **Kinesis Producer SDK** (from web/app).

2. **Kinesis Stream receives events**

   * Buffers and scales automatically.
   * Delivers to consumers in near real-time (<1s latency).

3. **Lambda or Kinesis Analytics processes events**

   * Join with user profile or hotel metadata.
   * Compute aggregated features (e.g., “#clicks last 10 min”).

4. **Feature Store updates**

   * Write enriched user/hotel features to **SageMaker Feature Store (online store)**.

5. **Model Inference**

   * `API Gateway` → `Lambda` → `SageMaker Endpoint`
   * Model loads user & item features from Feature Store.
   * Returns top-N recommendations (hotel IDs).

6. **Feedback Loop**

   * User clicks/booking data streamed back to Kinesis → retraining in batch (daily) via **SageMaker Pipelines** or **Airflow on MWAA**.

---

## 🧮 4. Real-Time Inference Example (Lambda + SageMaker)

```python
import boto3, json

runtime = boto3.client("sagemaker-runtime")

def lambda_handler(event, context):
    user_id = event["queryStringParameters"]["user_id"]
    
    payload = json.dumps({"user_id": user_id})
    response = runtime.invoke_endpoint(
        EndpointName="hyatt-recommendation-rt",
        ContentType="application/json",
        Body=payload
    )
    
    result = json.loads(response["Body"].read().decode())
    return {"statusCode": 200, "body": json.dumps(result)}
```

---

## 🧠 5. Model Side (SageMaker Endpoint)

Your model can be trained offline using batch data:

* Collaborative filtering
* Neural ranking model (BERT-style hotel embedding)
* Contextual bandit (for personalization)

Then deployed as:

```python
mlflow.sagemaker.deploy(
    model_uri="models:/hyatt_recommendation_model/Production",
    region_name="us-east-1",
    mode="realtime"
)
```

or via SageMaker `predictor.deploy()` in PyTorch.

---

## 🔁 6. Retraining Pipeline

Use **SageMaker Pipelines** + **Step Functions**:

* Daily retraining with historical logs from S3
* Evaluate on offline metrics (NDCG, CTR)
* Register new model to MLflow / SageMaker Registry
* Auto-deploy to A/B test via blue-green routing

---

## 🧩 7. Optional Enhancements

| Feature                       | AWS Service                                        |
| ----------------------------- | -------------------------------------------------- |
| **Real-Time Feature Updates** | SageMaker Feature Store (online) + DynamoDB stream |
| **Search / Filtering Layer**  | Amazon OpenSearch (for geo + text search)          |
| **Personalization API**       | AWS Personalize (managed recommender)              |
| **Experiment Management**     | MLflow / SageMaker Experiments                     |
| **Monitoring**                | CloudWatch + Model Monitor + Data Quality Reports  |

---

## 🧠 8. Example Real-Time Feedback Loop

```text
User clicks → event → Kinesis Stream → Lambda updates Feature Store
→ SageMaker Endpoint gets new feature state → predictions adapt live
→ Kinesis Firehose → S3 → retraining pipeline (daily)
```

This gives a **true feedback loop** for personalization at Hyatt.

---

## 🚀 9. MVP vs Production

| Stage         | MVP                       | Full Production                              |
| ------------- | ------------------------- | -------------------------------------------- |
| Ingestion     | API Gateway + Lambda → S3 | Kinesis Streams + Firehose                   |
| Feature Store | DynamoDB                  | SageMaker Feature Store                      |
| Model         | XGBoost                   | Deep Learning Re-Ranker                      |
| Deployment    | Single SageMaker endpoint | A/B Routing + Canary Deploy                  |
| Monitoring    | Logs                      | CloudWatch + Model Monitor + Drift Detection |

---

## ✅ Summary

**AWS Services Used:**

* **Kinesis Streams / Firehose** → real-time data ingestion
* **Lambda / Kinesis Analytics** → stream processing
* **SageMaker Feature Store** → real-time features
* **SageMaker Endpoint** → online inference
* **API Gateway + Lambda / ECS** → REST API
* **S3 + Redshift** → offline storage & analytics
* **CloudWatch + Model Monitor** → observability

---

> 🧩 **Kinesis is not a request–response system.**
> It’s a **streaming pipeline** — one-way, asynchronous, for event ingestion and fan-out processing.

So:

* Kinesis **does not “return” predictions or responses** directly.
* It’s used to **send events** (e.g., user clicks, actions, features) downstream to be processed by consumers like **Lambda**, **Kinesis Analytics**, or **SageMaker**.
* Any **real-time prediction responses** must go through a **synchronous API layer** (e.g., API Gateway → Lambda → SageMaker Endpoint).

Let’s break it down properly 👇

---

## 🧠 1. What Kinesis Actually Does

Think of Kinesis as **“real-time Kafka”** — it handles continuous event ingestion:

```text
Frontend/App  --->  Kinesis Stream  --->  Consumers
```

Each record you send (`PutRecord`) is appended to a stream.
Consumers (Lambda, Kinesis Data Analytics, Firehose, etc.) **read** the stream asynchronously.

Example:

```python
import boto3, json
kinesis = boto3.client("kinesis")

event = {
    "user_id": "U123",
    "event_type": "hotel_view",
    "hotel_id": "H456",
    "timestamp": "2025-11-06T15:35:00Z"
}

kinesis.put_record(
    StreamName="hyatt-events",
    Data=json.dumps(event),
    PartitionKey="U123"
)
```

✅ It **writes** the event.
⛔ It **does not** return a model result or response — just a write confirmation:

```json
{
  "ShardId": "shardId-000000000001",
  "SequenceNumber": "49572839239520394"
}
```

---

## ⚙️ 2. Where the Response Comes From

When you need **real-time recommendations**, you use a **separate synchronous layer**:

```
Frontend → API Gateway → Lambda → SageMaker Endpoint
```

That Lambda can:

1. Fetch latest features (possibly updated via Kinesis → Feature Store)
2. Call SageMaker Endpoint for inference
3. Return hotel recommendations to the user immediately

Example:

```python
import boto3, json

runtime = boto3.client("sagemaker-runtime")

def lambda_handler(event, context):
    user_id = event["queryStringParameters"]["user_id"]
    payload = json.dumps({"user_id": user_id})

    # Synchronous inference call
    response = runtime.invoke_endpoint(
        EndpointName="hyatt-recommender-rt",
        ContentType="application/json",
        Body=payload
    )

    result = json.loads(response["Body"].read())
    return {"statusCode": 200, "body": json.dumps(result)}
```

So:

* **Kinesis** keeps the user’s event stream (for feature updates, logs, retraining).
* **SageMaker endpoint** handles the **real-time response** path.

---

## 🧩 3. How Kinesis *Still Supports* Real-Time Personalization

Even though Kinesis doesn’t directly return responses, it **makes the responses smarter** because:

1. Every click, booking, or search is streamed into Kinesis.
2. A **Lambda consumer** reads these events in near-real time (milliseconds).
3. It updates the user’s **feature vector** (e.g., last destination viewed, most-clicked brand).
4. That updated feature is stored in **SageMaker Feature Store (online)** or **DynamoDB**.
5. When the next API request comes in, the model uses the **latest features**.

So the **sequence** looks like this:

```
(1) User views hotel → frontend sends event to Kinesis
(2) Lambda consumes event → updates feature store
(3) Next request → Lambda → SageMaker endpoint uses fresh features
(4) SageMaker returns recommendation
```

That’s how you achieve *near real-time adaptation* — the feedback arrives via Kinesis, the model inference remains synchronous.

---

## 🔁 4. Optional Hybrid Design: Async Inference via Kinesis

If you ever need **asynchronous inference** (e.g., thousands of recs in parallel):

* You can **send inference jobs to a Kinesis stream**.
* A **consumer Lambda** processes them, runs inference via SageMaker, and **writes results** to another stream (or DynamoDB table).
* The frontend **polls** for results later.

But that’s **not real-time interactive** — it’s used for background scoring.

Example pipeline:

```
Frontend → Kinesis:requests
          → Lambda:inference
               → SageMaker → write to Kinesis:responses
```

The **frontend polls** `responses` stream or API to fetch results later.

---

## ✅ Summary Table

| Use Case                     | How It Works                                       | Response?   |
| ---------------------------- | -------------------------------------------------- | ----------- |
| Real-time API recommendation | API Gateway → Lambda → SageMaker                   | ✅ Immediate |
| Real-time feature updates    | Frontend → Kinesis Stream → Lambda → Feature Store | ❌ No        |
| Batch / async scoring        | Producer → Kinesis → Lambda → SageMaker → S3       | ❌ Delayed   |
| Feedback logging             | Inference result → Kinesis → S3 / Redshift         | ❌ No        |

---

## 💡 Design Rule of Thumb

| Requirement                       | Use                                       |
| --------------------------------- | ----------------------------------------- |
| Need *response immediately*       | API Gateway + Lambda + SageMaker Endpoint |
| Need *stream events continuously* | Kinesis Streams / Firehose                |
| Need *process streams*            | Kinesis Analytics / Lambda Consumer       |
| Need *store features online*      | SageMaker Feature Store (online store)    |
| Need *offline retraining*         | S3 + SageMaker Pipelines                  |

---

So, **Kinesis never returns a model response** —
it’s the **event backbone**, not the prediction service.
The **real-time inference** must be served via **API + SageMaker Endpoint**.


