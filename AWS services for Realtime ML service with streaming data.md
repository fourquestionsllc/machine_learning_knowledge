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

