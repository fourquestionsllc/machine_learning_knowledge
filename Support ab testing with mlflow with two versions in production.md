You’ve now got MLflow managing multiple model versions (say, version 7 and version 8 of `hyatt_booking_model`) — and your data scientists want to **run A/B tests** to compare how both models perform **in production** with real user data.

Let’s walk through how to design a **production-grade A/B testing workflow using MLflow**, step-by-step — including **architecture, code, tracking, and rollout strategy**.

---

## 🎯 Goal

Run **two (or more) production model versions simultaneously**, route live traffic between them, collect metrics, and **track performance in MLflow**.

---

## 🧩 Scenario Example

For the **Hyatt booking recommendation service**:

* You have `hyatt_booking_model:v7` (current Production baseline).
* You’ve trained a new `v8` (candidate model).
* You want to send 80% of requests to v7, and 20% to v8.
* You’ll track each model’s performance and choose the winner.

---

## 🧱 Architecture Overview

```
Client (Frontend)
   ↓
Load Balancer / API Gateway
   ↓
A/B Router (custom Python or API Gateway rule)
   ├──> MLflow model v7 endpoint (Production-A)
   └──> MLflow model v8 endpoint (Production-B)
   ↓
Collector → logs user outcomes → MLflow Tracking (metrics)
```

---

## ⚙️ Step 1: Prepare Two Production Models in MLflow

MLflow supports multiple models in **Production** simultaneously — just set tags to differentiate.

```python
from mlflow import MlflowClient
client = MlflowClient()

model_name = "hyatt_booking_model"

# Promote both versions as Production
client.transition_model_version_stage(model_name, 7, stage="Production")
client.transition_model_version_stage(model_name, 8, stage="Production")

# Tag for clarity
client.set_model_version_tag(model_name, 7, "ab_group", "A")
client.set_model_version_tag(model_name, 8, "ab_group", "B")
```

💡 By default, MLflow doesn’t block multiple “Production” models — you can use tags or metadata to manage routing.

---

## 🚦 Step 2: Set Up A/B Routing Layer

You can build a **simple API gateway** or **Flask FastAPI app** that routes incoming requests to one of the two MLflow model endpoints.

### Example (Python FastAPI Router)

```python
from fastapi import FastAPI, Request
import random, requests

app = FastAPI()

MODEL_A_URL = "http://mlflow-service-v7:8080/invocations"
MODEL_B_URL = "http://mlflow-service-v8:8080/invocations"

@app.post("/predict")
async def predict(request: Request):
    payload = await request.json()

    # 80/20 split
    model_group = "A" if random.random() < 0.8 else "B"
    url = MODEL_A_URL if model_group == "A" else MODEL_B_URL

    # Call corresponding MLflow model endpoint
    response = requests.post(url, json=payload)
    result = response.json()

    # Log to MLflow for tracking
    import mlflow
    with mlflow.start_run(run_name=f"ab_test_{model_group}", nested=True):
        mlflow.log_param("model_group", model_group)
        mlflow.log_param("model_version", 7 if model_group=="A" else 8)
        mlflow.log_input(payload)
    
    # Return prediction
    return {"model_group": model_group, "result": result}
```

This router:

* Randomly assigns users to model A or B.
* Sends their request to the correct MLflow model server.
* Logs which model served it.

---

## 🧮 Step 3: Track A/B Test Metrics in MLflow

After serving predictions, you’ll collect **ground truth outcomes** (e.g., user booked hotel or not) and log evaluation metrics per model.

Example offline aggregation:

```python
import mlflow
import pandas as pd
from sklearn.metrics import roc_auc_score

# Suppose we collected A/B results into a DataFrame
df = pd.read_csv("ab_results.csv")  # columns: [user_id, model_version, y_true, y_pred]

for version in [7, 8]:
    subset = df[df.model_version == version]
    auc = roc_auc_score(subset.y_true, subset.y_pred)
    mlflow.log_metric(f"auc_v{version}", auc)
```

Or, track per-request metrics if you’re streaming data back (e.g., via Kafka → Airflow → MLflow).

---

## 📊 Step 4: Compare A/B Models in MLflow UI

Now in your MLflow Tracking UI:

* Search for runs tagged with `run_name like "ab_test_*"`.
* Compare metrics for model A (v7) vs model B (v8).
* Visualize metrics like AUC, CTR, conversion rate.

---

## 🏁 Step 5: Choose Winner & Roll Out

Once the test concludes:

* Use metrics or statistical significance tests (e.g., t-test, chi-squared) to decide.
* Promote the winner as the **single production model** and archive the other.

Example:

```python
client.transition_model_version_stage(
    name=model_name,
    version=8,
    stage="Production",
    archive_existing_versions=True
)
```

This archives v7 automatically.

---

## ⚡️ Step 6: (Optional) Automate the A/B Workflow

Use **Airflow or CI/CD** to automate:

1. Train → Register → Staging
2. Deploy both versions (A/B)
3. Collect metrics for N days
4. Auto-promote best-performing version
5. Archive the other

Example pseudo-Automation DAG:

```text
train_model -> register_model -> deploy_AB_test
        -> collect_metrics -> evaluate_AB -> promote_winner
```

---

## 🧠 Advanced Setup (Production-Grade)

| Component             | Role                  | Tool                          |
| --------------------- | --------------------- | ----------------------------- |
| MLflow Model Registry | Versioning & metadata | MLflow                        |
| Router / API Gateway  | Request routing       | FastAPI / NGINX / API Gateway |
| Experiment Tracking   | Metrics & logs        | MLflow Tracking Server        |
| Data Collection       | Logging outcomes      | Kafka / Firehose / S3         |
| Evaluation            | Periodic analysis     | Airflow / SageMaker Pipeline  |
| Promotion             | Rollout automation    | MLflow + CI/CD pipeline       |

---

## ✅ Summary

| Step               | Purpose                       | Tools                                   |
| ------------------ | ----------------------------- | --------------------------------------- |
| 1. Register models | Keep both versions tracked    | MLflow Model Registry                   |
| 2. Serve both      | Deploy v7 & v8                | MLflow model servers                    |
| 3. Route traffic   | Split traffic 80/20           | FastAPI / Gateway                       |
| 4. Log metrics     | Track per-version performance | MLflow Tracking                         |
| 5. Evaluate        | Pick the winner               | A/B analysis job                        |
| 6. Promote         | Archive loser, keep winner    | MLflow `transition_model_version_stage` |

---
