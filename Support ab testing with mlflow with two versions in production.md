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

You’re talking about extending your **A/B testing router** so that:

* Sometimes you want **per-request randomization** (e.g., every API call gets a random model).
* Sometimes you want **user-level assignment** (so the same user always sees the same model across sessions).

Let’s design this step by step.

---

## 🧠 Goal

Build an **A/B testing router** (e.g., FastAPI or Flask) that can flexibly choose routing logic:

| Mode               | Description                 | Example                           |
| ------------------ | --------------------------- | --------------------------------- |
| `"request_random"` | Random model for every call | Each API hit is 50/50             |
| `"user_hash"`      | Deterministic per user      | Same user → same model every time |

---

## ⚙️ Step 1: Extend Router Logic

We’ll use a simple FastAPI router that supports both routing strategies via a config or environment variable.

### Example Code

```python
from fastapi import FastAPI, Request
import hashlib, os, random, requests, mlflow

app = FastAPI()

# Two MLflow serving endpoints
MODEL_A_URL = "http://mlflow-v7:8080/invocations"
MODEL_B_URL = "http://mlflow-v8:8080/invocations"

# Choose routing mode: "request_random" or "user_hash"
ROUTING_MODE = os.getenv("ROUTING_MODE", "user_hash")

def choose_model(user_id=None):
    """Return 'A' or 'B' depending on routing mode"""
    if ROUTING_MODE == "request_random":
        return "A" if random.random() < 0.5 else "B"

    elif ROUTING_MODE == "user_hash":
        if not user_id:
            # fallback: random if no user id
            return "A" if random.random() < 0.5 else "B"
        # Hash user_id to ensure deterministic assignment
        h = int(hashlib.sha256(str(user_id).encode()).hexdigest(), 16)
        return "A" if h % 2 == 0 else "B"

@app.post("/predict")
async def predict(request: Request):
    payload = await request.json()
    user_id = payload.get("user_id")

    group = choose_model(user_id)
    model_url = MODEL_A_URL if group == "A" else MODEL_B_URL

    # Call MLflow serving endpoint
    response = requests.post(model_url, json=payload)
    preds = response.json()

    # Log routing + model version to MLflow
    with mlflow.start_run(run_name="ab_inference", nested=True):
        mlflow.log_param("routing_mode", ROUTING_MODE)
        mlflow.log_param("group", group)
        mlflow.log_param("user_id", user_id)
    
    return {"model_group": group, "predictions": preds}
```

---

## 🪄 How It Works

| Mode               | Logic                      | Effect                                                 |
| ------------------ | -------------------------- | ------------------------------------------------------ |
| `"request_random"` | `random.random()` per call | Each request gets A or B independently                 |
| `"user_hash"`      | `hash(user_id) % 2`        | Same user ID always maps to same model (deterministic) |

---

## 🧩 Step 2: Ensure Deterministic User Assignment

Hashing ensures that:

* Same user always gets same model
* Distribution stays roughly balanced
* No need to store state in DB

You can also change assignment ratio easily:

```python
# 70/30 split by hash
return "A" if h % 10 < 7 else "B"
```

---

## 🗂️ Step 3: Add Configurable Routing (Optional)

If you want DS or Ops to switch routing logic dynamically without redeploying, use a config service or database flag.

Example using environment variable:

```bash
export ROUTING_MODE=user_hash
uvicorn router:app --host 0.0.0.0 --port 8080
```

Or dynamically via REST endpoint:

```python
@app.post("/set_mode")
def set_mode(new_mode: str):
    global ROUTING_MODE
    if new_mode in ["request_random", "user_hash"]:
        ROUTING_MODE = new_mode
        return {"status": "ok", "new_mode": ROUTING_MODE}
    return {"status": "error", "msg": "invalid mode"}
```

---

## 📊 Step 4: Tracking User-Level Experiment Results

Log user-level metrics back into MLflow (or another store).

Example (later, when you get real outcomes):

```python
import mlflow

def log_outcome(user_id, model_group, clicked):
    with mlflow.start_run(run_name="ab_user_outcome", nested=True):
        mlflow.log_param("user_id", user_id)
        mlflow.log_param("group", model_group)
        mlflow.log_metric("clicked", int(clicked))
```

Then, you can aggregate by group later to compute CTR, AUC, etc.

---

## 🧮 Step 5: Optional — Persistent Mapping (for Analytics)

If you need to analyze assignment consistency or override user-group assignment:

* Store user → group mapping in a database table or Redis.
* Example schema:

  ```sql
  CREATE TABLE ab_assignments (
      user_id VARCHAR(255),
      model_group CHAR(1),
      assigned_at TIMESTAMP
  );
  ```

You can pre-assign users at experiment start if needed (e.g., for a stable test population).

---

## 🧠 Advanced Extensions

| Feature                 | Approach                                               |
| ----------------------- | ------------------------------------------------------ |
| **Weighted assignment** | Random split 70/30 or 90/10 using thresholds           |
| **Multi-variant test**  | More than 2 models (A/B/C/D) → hash % N                |
| **Time-based rollout**  | Use timestamps to gradually ramp up                    |
| **Sticky sessions**     | Use user_hash or session_id hashing                    |
| **Dynamic config**      | Store routing rules in DynamoDB / Feature Flag service |

---

## ✅ Summary

| Goal                   | Implementation                     |
| ---------------------- | ---------------------------------- |
| Random per request     | `random.random()`                  |
| Same user → same model | `hash(user_id) % 2`                |
| Switch between modes   | Config var or `/set_mode` endpoint |
| Log all events         | MLflow `log_param` + `log_metric`  |
| Rollout A/B safely     | Hash or weighted split logic       |

---

We’re talking about **enabling Data Scientists (DS)** to *self-deploy* and run **A/B test APIs** — while you (as MLOps) keep consistency, governance, and observability.

Let’s build a **clear, practical workflow** for that. 👇

---

## 🧠 Goal

Let DS:

* Train models → log to MLflow
* Propose new versions → register
* Deploy two models for A/B testing
* Automatically get an API endpoint
* You (MLOps) control rollout & tracking

---

## 🏗️ 1. Define the A/B Deployment Architecture

```text
         +-------------------+
         |  Frontend / Client |
         +-------------------+
                    |
                    v
        +---------------------------+
        | A/B Router (MLOps Owned) |
        +---------------------------+
          /                       \
         /                         \
+-------------------+     +-------------------+
|  MLflow Model A   |     |  MLflow Model B   |
| (v7)              |     | (v8)              |
+-------------------+     +-------------------+
```

The **A/B Router** is a lightweight API (e.g., FastAPI, Flask) that:

* Receives inference requests
* Randomly (or deterministically) routes to v7 or v8
* Logs which model handled the call
* Returns prediction results transparently

---

## 🧰 2. Use MLflow Model Registry to Manage Versions

Data Scientists handle:

```python
import mlflow
from mlflow.tracking import MlflowClient

mlflow.set_tracking_uri("http://mlflow-server:5000")
client = MlflowClient()

# Register model
model_uri = "runs:/<RUN_ID>/model"
model_details = mlflow.register_model(model_uri, "hyatt_recommendation_model")

# Move model to "Staging" (ready for A/B test)
client.transition_model_version_stage(
    name="hyatt_recommendation_model",
    version=model_details.version,
    stage="Staging"
)
```

They only need to train → log → register.
Your MLOps pipeline handles **deployment logic** automatically.

---

## ⚙️ 3. MLOps Deployment Pipeline

Your side (MLOps) defines a **GitOps or CI/CD workflow**:

### Example: GitHub Actions or Jenkins

```yaml
on:
  workflow_dispatch:
    inputs:
      model_name:
      version_a:
      version_b:

jobs:
  deploy-ab:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy A/B API
        run: |
          python deploy_ab_api.py \
            --model_name ${{ inputs.model_name }} \
            --version_a ${{ inputs.version_a }} \
            --version_b ${{ inputs.version_b }}
```

### `deploy_ab_api.py`

```python
import mlflow
import os
import json

model_a = mlflow.pyfunc.get_model_version_download_uri("hyatt_recommendation_model", "7")
model_b = mlflow.pyfunc.get_model_version_download_uri("hyatt_recommendation_model", "8")

# generate config for router
config = {
  "model_a": model_a,
  "model_b": model_b,
  "routing_mode": "user_hash"
}

with open("ab_config.json", "w") as f:
    json.dump(config, f)

os.system("docker-compose up -d")  # starts A/B router + MLflow serving
```

Now DS can “request A/B test” via CI input or MLflow tag.

---

## 🧑‍🔬 4. Data Scientist Workflow (Simple & Safe)

1. Train model in notebook
2. `mlflow.log_model()`
3. `mlflow.register_model()`
4. Tag the model:

   ```python
   client.set_model_version_tag(
       name="hyatt_recommendation_model",
       version=8,
       key="ab_test_candidate",
       value="true"
   )
   ```
5. MLOps pipeline detects tag → triggers A/B deployment
6. DS gets API endpoint (e.g. `/predict`) and can start testing

---

## 🔁 5. Unified Inference API (FastAPI Example)

Your **router service** exposes a single `/predict` endpoint.

```python
from fastapi import FastAPI, Request
import os, random, requests

app = FastAPI()

MODEL_A_URL = os.getenv("MODEL_A_URL")
MODEL_B_URL = os.getenv("MODEL_B_URL")
ROUTING_MODE = os.getenv("ROUTING_MODE", "user_hash")

@app.post("/predict")
async def predict(request: Request):
    data = await request.json()
    user_id = data.get("user_id")
    use_a = random.random() < 0.5 if ROUTING_MODE == "request_random" else hash(user_id) % 2 == 0
    model_url = MODEL_A_URL if use_a else MODEL_B_URL

    response = requests.post(model_url, json=data)
    return {"model": "A" if use_a else "B", "prediction": response.json()}
```

Now both models are live, but hidden behind one endpoint.

---

## 🧩 6. Tracking & Reporting

MLOps adds:

* `mlflow.log_param("model", "A" or "B")` at inference
* `mlflow.log_metric("accuracy", ...)` after offline validation
* Combine inference logs + outcome metrics in dashboard (e.g., Superset, Grafana)

---

## ✅ 7. MVP Version (Minimum Viable Product)

| Phase       | Capability                                                                           | Owner      |
| ----------- | ------------------------------------------------------------------------------------ | ---------- |
| **MVP**     | Deploy 2 models behind one router; random traffic split; MLflow registry integration | MLOps      |
| **Phase 2** | Deterministic user hashing; config control; auto-deploy via tags                     | Shared     |
| **Phase 3** | Dynamic rollout (e.g., 10% → 50% → 100%), real-time dashboards                       | MLOps      |
| **Phase 4** | Online feedback loops, continuous retraining                                         | DS + MLOps |

---

## 🗣️ 8. How to *Propose It* to DS

Frame it as **enablement**, not control.
Sample pitch:

> “We’ll handle deployment through MLflow, so you can focus on model performance.
> When you’re ready to test a new version, just tag it as `ab_test_candidate=true`.
> Our A/B router will automatically deploy it next to the current production model — same API, same payload.
> You’ll get real traffic and metrics, and we’ll track results through MLflow.”

This earns DS trust while enforcing consistency.

---





