Amazon SageMaker’s **Unified Studio** provides an integrated environment for building, training, deploying, and managing ML models. Below is a complete example of how to use all major **components** of SageMaker Unified Studio:

- **Data Preparation (Data Wrangler)**
- **Model Building (Jupyter Notebooks, SageMaker Experiments)**
- **Model Training (SageMaker Training Jobs)**
- **Model Deployment (SageMaker Endpoints)**
- **Model Monitoring (SageMaker Model Monitor)**

---

### **1️⃣ Prerequisites: Setup AWS Credentials and SageMaker Studio**
Before running the code, ensure you have **AWS CLI, Boto3, and SageMaker SDK** installed and configured.

```bash
pip install boto3 sagemaker pandas numpy
aws configure
```

Then, open **SageMaker Studio** from the AWS Console.

---

## **Step 1: Data Preparation (Using Data Wrangler)**
First, we'll import and process data using **SageMaker Data Wrangler**.

```python
import sagemaker
import boto3
import pandas as pd

session = sagemaker.Session()
s3_bucket = session.default_bucket()
s3_prefix = "sagemaker-studio-example"
role = sagemaker.get_execution_role()

# Load sample data
df = pd.read_csv("https://raw.githubusercontent.com/jbrownlee/Datasets/master/pima-indians-diabetes.data.csv")

# Save to S3
data_file = "diabetes.csv"
df.to_csv(data_file, index=False)
s3_path = f"s3://{s3_bucket}/{s3_prefix}/{data_file}"

s3 = boto3.client("s3")
s3.upload_file(data_file, s3_bucket, f"{s3_prefix}/{data_file}")

print(f"Data uploaded to: {s3_path}")
```

✅ **Next actions:**
- Open **SageMaker Studio** → **Data Wrangler**
- Select **S3 Data Source** and choose the dataset uploaded.
- Perform transformations (scaling, feature engineering) and export the transformed dataset.

---

## **Step 2: Model Building (Using SageMaker Studio Notebooks)**
Now, we create a **SageMaker Experiment** to track model training runs.

```python
from sagemaker.experiments.run import Run
from sagemaker.experiments.experiment import Experiment

experiment_name = "diabetes-experiment"
sagemaker_experiment = Experiment.create(experiment_name=experiment_name)

with Run(experiment_name=experiment_name) as run:
    run.log_parameter("learning_rate", 0.01)
    run.log_parameter("epochs", 100)
    run.log_metric("accuracy", 0.85)
    
    print("Experiment logged successfully")
```

✅ **Next actions:**
- Go to **SageMaker Studio** → **Experiments and Trials**
- Monitor experiment performance.

---

## **Step 3: Model Training (Using SageMaker Training Jobs)**
Now, we train a **Scikit-learn model** using a SageMaker Training Job.

```python
from sagemaker.sklearn.estimator import SKLearn

script_path = "train.py"  # Create a training script (train.py) in the same directory

sklearn_estimator = SKLearn(
    entry_point=script_path,
    role=role,
    instance_count=1,
    instance_type="ml.m5.large",
    framework_version="0.23-1",
    output_path=f"s3://{s3_bucket}/{s3_prefix}/output/",
)

sklearn_estimator.fit({"train": s3_path})
```

✅ **Next actions:**
- In **SageMaker Studio**, go to **Training Jobs** → Monitor logs.
- View training metrics.

---

## **Step 4: Model Deployment (Using SageMaker Endpoints)**
After training, deploy the model as a real-time endpoint.

```python
predictor = sklearn_estimator.deploy(
    initial_instance_count=1,
    instance_type="ml.m5.large",
    endpoint_name="diabetes-endpoint"
)
```

✅ **Next actions:**
- In **SageMaker Studio**, go to **Inference** → **Endpoints**
- Monitor deployed endpoints.

---

## **Step 5: Model Monitoring (Using SageMaker Model Monitor)**
Enable monitoring to track data drift and anomalies.

```python
from sagemaker.model_monitor import DefaultModelMonitor

monitor = DefaultModelMonitor(
    role=role,
    instance_count=1,
    instance_type="ml.m5.large",
    volume_size_in_gb=5,
)

monitor.create_monitoring_schedule(
    endpoint_name="diabetes-endpoint",
    output_s3_uri=f"s3://{s3_bucket}/{s3_prefix}/monitoring/",
)
```

✅ **Next actions:**
- Go to **SageMaker Studio** → **Model Monitor**
- Check the logs for data drift.

---

### **Final Steps**
- Use the endpoint to make real-time predictions.

```python
import json

test_data = [[6, 148, 72, 35, 0, 33.6, 0.627, 50]]
response = predictor.predict(test_data)
print(json.loads(response))
```

- Delete the endpoint after testing.

```python
predictor.delete_endpoint()
```

---

### **✅ Summary of SageMaker Unified Studio Components Used**
| SageMaker Component | Action Taken |
|---------------------|-------------|
| **Data Wrangler** | Uploaded and processed data |
| **SageMaker Notebooks** | Tracked experiments and hyperparameters |
| **Training Jobs** | Trained a Scikit-learn model on AWS infrastructure |
| **Endpoints** | Deployed the trained model for real-time inference |
| **Model Monitor** | Enabled monitoring for data drift detection |

---

This example demonstrates a **complete machine learning workflow** in **SageMaker Unified Studio**, integrating **data prep, training, deployment, and monitoring**. 🚀
