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



---

You’re asking about the **controlled “approval → production” process** in MLflow, which is how teams move from *model trained* → *validated* → *approved for production* with governance.

This is a key MLOps workflow for real organizations (like Hyatt, in your example).
Let’s walk through it **step-by-step** with **code, UI actions, and automation best practices**.

---

## 🧭 Goal

Ensure that **only approved models** move to the **Production** stage in the MLflow Model Registry — with proper review, tagging, and tracking.

---

## ⚙️ Precondition

* You already have a registered model in MLflow (e.g., `hyatt_booking_model`).
* A data scientist has trained a new version (e.g., `version 5`).
* Evaluation metrics are logged in MLflow (AUC, accuracy, etc.).

---

## 🪜 Step 1: Review Model in “Staging”

After registration, the model version should first move to **“Staging”**, where it can be validated.

```python
from mlflow import MlflowClient
client = MlflowClient()

model_name = "hyatt_booking_model"
version = 5

client.transition_model_version_stage(
    name=model_name,
    version=version,
    stage="Staging"
)
```

### ✅ What happens:

* Version 5 is now marked as **Staging** in MLflow Model Registry.
* The Staging stage is used for **QA, testing, and validation**.

---

## 🔍 Step 2: Validate the Model (Offline Evaluation)

Perform model validation — for example, comparing its performance to the current Production version.

```python
import mlflow
from sklearn.metrics import roc_auc_score

# Load candidate (staging) and current production models
candidate = mlflow.pyfunc.load_model("models:/hyatt_booking_model/Staging")
current = mlflow.pyfunc.load_model("models:/hyatt_booking_model/Production")

# Evaluate on holdout data
candidate_auc = roc_auc_score(y_test, candidate.predict(X_test))
current_auc = roc_auc_score(y_test, current.predict(X_test))

mlflow.log_metric("candidate_auc", candidate_auc)
mlflow.log_metric("current_auc", current_auc)
```

If the new model performs better or meets threshold, mark it **ready for approval**.

---

## 🧾 Step 3: Approve the Model (Add Approval Metadata)

Before pushing to production, you can require **human approval**.
Use MLflow tags and comments for auditability.

```python
client.set_model_version_tag(
    name=model_name,
    version=version,
    key="approval_status",
    value="approved"
)

client.set_model_version_tag(
    name=model_name,
    version=version,
    key="approved_by",
    value="julian.wang"
)

client.set_model_version_tag(
    name=model_name,
    version=version,
    key="approval_comment",
    value="Validated on dataset v3; AUC improved from 0.88 → 0.91"
)
```

These tags show up in the MLflow UI for governance and traceability.

---

## 🚀 Step 4: Promote Approved Model to Production

Once approved (manually or via automation), promote to Production:

```python
client.transition_model_version_stage(
    name=model_name,
    version=version,
    stage="Production",
    archive_existing_versions=True  # archive old version
)
```

### ✅ Now:

* The model is officially **in Production**.
* The previous Production model (if any) is **archived**.
* The promotion is logged automatically with timestamps and user info.

---

## 🔄 Step 5: Automate Approval Workflow (MVP Automation)

To make this smoother, you can automate part of the process with **CI/CD or Airflow**:

### Example: Simple CI/CD Logic

```python
from mlflow import MlflowClient

client = MlflowClient()
model_name = "hyatt_booking_model"
version = 5

# Get validation metrics
run_id = client.get_model_version(model_name, version).run_id
metrics = client.get_run(run_id).data.metrics
if metrics["val_auc"] > 0.9:
    client.set_model_version_tag(name=model_name, version=version, key="approval_status", value="auto_approved")
    client.transition_model_version_stage(name=model_name, version=version, stage="Production", archive_existing_versions=True)
```

### Orchestration idea:

* **Airflow DAG or GitHub Action** runs this logic nightly:

  1. Train new model.
  2. Evaluate.
  3. If metrics pass threshold → approve → promote.
  4. Notify approvers in Slack or email.

---

## 📊 Step 6: Track Everything in MLflow

Every transition (Staging → Production) and every tag (approval info) is **automatically logged**.
You can view:

* **Who promoted it**
* **When**
* **Why** (approval comment)
* **Performance metrics**

This gives you a full **audit trail** and **governance visibility**.

---

## 🔒 Optional: Approval Gate via REST or Role-Based Access

For enterprise setups:

* Restrict Production transitions to specific roles (e.g. MLOps lead only).
* Use MLflow REST API + your IAM/SSO integration to enforce access control.
* Example (REST):

  ```bash
  curl -X POST https://mlflow.hyatt.com/api/2.0/mlflow/transition \
       -H "Authorization: Bearer $TOKEN" \
       -d '{"name":"hyatt_booking_model","version":5,"stage":"Production"}'
  ```

---

## 🧠 Summary — “Approval → Production” Lifecycle

| Stage    | Description              | MLflow Action                             |
| -------- | ------------------------ | ----------------------------------------- |
| Train    | DS trains and logs model | `mlflow.log_model()`                      |
| Register | New version created      | `mlflow.register_model()`                 |
| Validate | Test on holdout          | custom eval run                           |
| Approve  | Add tags/comments        | `client.set_model_version_tag()`          |
| Promote  | Move to Production       | `client.transition_model_version_stage()` |
| Serve    | Deploy for inference     | `mlflow models serve` or EKS              |

---


