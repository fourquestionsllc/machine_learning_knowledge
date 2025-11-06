you’ve trained and registered a model version, and now you want to **promote that version to production**, **serve it**, and **track its lifecycle**.

Let’s go through **the exact steps, commands, and code** to do this — both from the **MLflow UI** and **Python API**, plus how to track everything automatically.

---

## ⚙️ Prerequisites

You should already have:

* A **central MLflow Tracking Server** (with backend DB + S3 artifact store).
* A **registered model** (e.g. `hyatt_booking_model`).
* A **model version** (e.g. version 3).

---

## 🪜 Step 1 — Register Your Model (if not already)

You can register a trained model from a run:

```python
import mlflow

# Example: registering model after training
run_id = "abcd1234"  # your training run ID
model_name = "hyatt_booking_model"

result = mlflow.register_model(
    model_uri=f"runs:/{run_id}/model",
    name=model_name
)

print(f"Registered model version: {result.version}")
```

Now MLflow creates a **new model version** under the given name (e.g., version 3).

---

## 🚀 Step 2 — Push a Model Version to Production

You can promote (transition) a model version to **Production** using the MLflow **Python API** or **CLI**.

### ✅ Option 1: Python API

```python
from mlflow import MlflowClient

client = MlflowClient()
model_name = "hyatt_booking_model"
version = 3

# Move to "Production" stage
client.transition_model_version_stage(
    name=model_name,
    version=version,
    stage="Production",
    archive_existing_versions=True  # Optional: archive previous Production model
)

print(f"Model {model_name} version {version} is now in Production.")
```

This updates the model’s **stage** in the MLflow Model Registry:

* `None` → `Staging` → `Production` (or archived).

The `archive_existing_versions=True` ensures only one version is “Production” at a time.

---

### ✅ Option 2: MLflow CLI

```bash
mlflow models transition --name hyatt_booking_model --version 3 --stage Production
```

---

### ✅ Option 3: MLflow UI

* Go to your MLflow UI → **Models** tab → select model.
* Click your version → “Stage” → choose **Production**.
* Optionally add a comment like:

  > “Promoted after passing evaluation on validation dataset v5.”

---

## 📊 Step 3 — Track Promotion in MLflow

Every model transition (e.g., Staging → Production) is **automatically logged** in MLflow’s backend database.
You can also **log metadata** to record who/why the promotion happened.

Example:

```python
client.set_model_version_tag(
    name=model_name,
    version=version,
    key="promoted_by",
    value="julian.wang"
)

client.set_model_version_tag(
    name=model_name,
    version=version,
    key="reason",
    value="Passed AUC threshold > 0.92 on test data v2"
)
```

You can view these tags and comments directly in the UI under that model version.

---

## 🧠 Step 4 — Deploy the Production Model

You have several ways to **serve** the Production model.

### 🔹 Option A: MLflow Model Serving (local or on server)

```bash
mlflow models serve -m "models:/hyatt_booking_model/Production" -p 8080
```

Then send prediction requests:

```bash
curl -X POST -H "Content-Type: application/json" \
     -d '{"columns":["feature1","feature2"], "data":[[10, 5]]}' \
     http://127.0.0.1:8080/invocations
```

---

### 🔹 Option B: Deploy as Docker Container (for EKS or SageMaker)

```bash
mlflow models build-docker \
    -m "models:/hyatt_booking_model/Production" \
    -n hyatt_booking_api:latest
```

Then push to your registry (e.g., ECR) and deploy via ECS/EKS.

---

### 🔹 Option C: Deploy Programmatically (e.g., in Flask/Streamlit app)

```python
import mlflow.pyfunc

model = mlflow.pyfunc.load_model("models:/hyatt_booking_model/Production")
preds = model.predict(df)
```

---

## 🧾 Step 5 — Automatically Track Deployments (Best Practice)

To keep everything **traceable**, log deployment events as part of MLflow runs:

```python
with mlflow.start_run(run_name="deploy_hyatt_booking_model"):
    mlflow.log_param("model_name", model_name)
    mlflow.log_param("model_version", version)
    mlflow.log_metric("deployment_latency_ms", 245)
    mlflow.set_tag("stage", "Production")
    mlflow.set_tag("deployment_env", "EKS")
```

That way, you have a **deployment run** that links the production model version with metadata like latency, environment, etc.

---

## 🧭 End-to-End Lifecycle Summary

| Stage    | Action                 | API/Command                               | Tracking                       |
| -------- | ---------------------- | ----------------------------------------- | ------------------------------ |
| Train    | Run experiment         | `mlflow.start_run()`                      | Parameters, metrics, artifacts |
| Register | Save model to registry | `mlflow.register_model()`                 | Model version created          |
| Validate | Evaluate model         | `mlflow.log_metric()`                     | Metrics logged                 |
| Promote  | Move to Production     | `client.transition_model_version_stage()` | Version history                |
| Deploy   | Serve for inference    | `mlflow.models.serve`                     | Deployment run logged          |
| Monitor  | Track performance      | `mlflow.log_metric("latency", …)`         | Production metrics             |


