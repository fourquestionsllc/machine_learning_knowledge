# ✅ **End-to-End SageMaker ML Pipeline (Production-Ready)**

## **0. Prerequisites**

* AWS account
* S3 bucket for data
* IAM roles for SageMaker
* Training script (train.py)
* (Optional) requirements.txt for dependencies

---

# **1. Data Ingestion**

### **Source options**

* S3
* Redshift / Athena
* Glue ETL
* Kinesis streaming
* API / Batch uploads

### **Pipeline step example**

```python
from sagemaker.processing import ScriptProcessor, ProcessingInput, ProcessingOutput

processor = ScriptProcessor(
    image_uri=processing_image,
    command=['python3'],
    role=role,
    instance_type='ml.m5.large',
    instance_count=1
)

processor.run(
    inputs=[ProcessingInput(source='s3://bucket/raw/', destination='/opt/ml/processing/input')],
    outputs=[ProcessingOutput(source='/opt/ml/processing/output', destination='s3://bucket/processed/')],
    code='preprocess.py'
)
```

---

# **2. Data Processing / Feature Engineering**

* Clean missing values
* Create derived features
* Encode categorical features
* Embed text using BERT/embeddings
* Normalize numerical columns
* Split train/val/test

### **Tools**

* SageMaker Processing jobs
* SageMaker Feature Store (optional)

---

# **3. Model Training**

SageMaker supports:

* XGBoost (built-in)
* LightGBM
* Deep Learning (PyTorch/TensorFlow)
* Custom container

### **XGBoost Example**

```python
from sagemaker.estimator import Estimator

xgb = Estimator(
    image_uri=xgboost_image,
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    output_path='s3://bucket/output/'
)

xgb.set_hyperparameters(
    objective="binary:logistic",
    max_depth=8,
    eta=0.1,
    num_round=200
)

xgb.fit({'train': train_s3_path, 'validation': validation_s3_path})
```

---

# **4. Model Evaluation**

Run evaluation using a **Processing job**.

### **Metrics**

* Accuracy / F1 / ROC-AUC
* Search recommendation: NDCG, MAP, Recall@K
* Revenue metrics: Expected GMV / conversion

### **Evaluation script output**

Save metrics to `/opt/ml/metrics/metrics.json`:

```json
{
  "metrics": {
    "ndcg": 0.82,
    "recall_at_20": 0.65
  }
}
```

Pipeline can **stop deployment if metrics degrade**.

---

# **5. Model Registration (SageMaker Model Registry)**

Models are versioned with:

* Metadata
* Artifacts
* Metrics
* Approval status (Pending, Approved, Rejected)

### Example

```python
model_package = xgb.register(
    content_types=['text/csv'],
    response_types=['text/csv'],
    model_package_group_name='SearchRelevanceGroup',
    approval_status='PendingManualApproval'
)
```

---

# **6. Model Deployment**

### **Options**

* Real-time endpoint
* Serverless endpoint
* Batch transform
* Async inference
* Multi-model endpoint (MME)

### Real-time example

```python
predictor = xgb.deploy(
    initial_instance_count=2,
    instance_type='ml.m5.large'
)
```

---

# **7. MLOps Automation (CI/CD)**

Use:

* **SageMaker Pipelines** (ML pipeline orchestration)
* **CodePipeline** + **CodeBuild** (CI/CD)
* **CloudWatch Events** (trigger pipeline on data changes)

### Typical CI/CD flow

1. Commit code → CodePipeline triggers
2. Run unit tests
3. Build training image
4. Execute SageMaker Pipeline
5. Produce new model version
6. Await human approval
7. Auto-deploy to staging ⇒ prod

---

# **8. Monitoring**

* **Model Drift Monitoring**

  * Data quality
  * Feature drift
  * Label drift
* **Endpoint auto-scaling**
* **CloudWatch logs / metrics**
* **Shadow / A/B testing** for rollout safety

---

# **9. Continual Retraining (Auto Pipeline Trigger)**

Triggered by:

* New data arrival (S3 event)
* Weekly schedule
* Drift detection

This ensures the model stays fresh.

---

# ⭐ Summary Table

| Stage               | SageMaker Component        | Purpose            |
| ------------------- | -------------------------- | ------------------ |
| Ingest              | Processing job / Glue      | Load data          |
| Feature engineering | Processing / Feature Store | Transform data     |
| Training            | Estimator                  | Train model        |
| Evaluation          | Processing job             | Compute metrics    |
| Registry            | Model Registry             | Version + Approval |
| Deployment          | Endpoint                   | Serve predictions  |
| Monitoring          | Model Monitor              | Drift / quality    |
| CI/CD               | CodePipeline + Pipelines   | Automation         |

