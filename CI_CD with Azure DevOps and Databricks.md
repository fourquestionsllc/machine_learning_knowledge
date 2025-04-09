Setting up CI/CD for **Azure Databricks** using **Azure DevOps (ADO)** is a powerful way to automate deployments and streamline collaboration. Here's a step-by-step guide to help you set up this pipeline.

---

## 🔧 Overview

You’ll be doing:
1. **Version Control**: Store notebooks and code in Azure Repos (Git).
2. **CI/CD Pipeline**: Use Azure Pipelines to automate:
   - Unit tests (CI)
   - Deploy notebooks/libraries/jobs to different workspaces (CD)
3. **Databricks CLI / REST APIs**: Used in pipeline scripts for deployment.

---

## ✅ Prerequisites

- Azure DevOps project with **Repos and Pipelines**.
- Azure Databricks workspace(s).
- Databricks **personal access token (PAT)**.
- Install **Databricks CLI** on agent or use pre-built Docker container.
- Create **Service Principal** if using multi-workspace (recommended).

---

## 🧱 Project Structure Example

```
/my-databricks-project
│
├── notebooks/
│   ├── ETL_Notebook.py
│   └── utils.py
├── tests/
│   └── test_utils.py
├── jobs/
│   └── job-config.json
├── cicd/
│   └── deploy.sh
├── requirements.txt
└── azure-pipelines.yml
```

---

## 1️⃣ Setup Azure Repo

- Create a repo in Azure DevOps and push your Databricks project code there.
- Convert Databricks notebooks to `.py` or `.dbc` using [Databricks CLI](https://docs.databricks.com/dev-tools/cli/index.html) or the Databricks UI export.

---

## 2️⃣ Configure Databricks CLI

Install and configure on your local or DevOps agent:

```bash
pip install databricks-cli

databricks configure --token
```

Set:
- Host: `https://<databricks-instance>#`
- Token: (from User Settings > Developer)

---

## 3️⃣ Create `deploy.sh` Script (CD Step)

```bash
#!/bin/bash

DATABRICKS_HOST=$1
DATABRICKS_TOKEN=$2
NOTEBOOK_PATH=$3
WORKSPACE_PATH=$4

# Install databricks CLI
pip install databricks-cli

# Set environment variables
export DATABRICKS_HOST=$DATABRICKS_HOST
export DATABRICKS_TOKEN=$DATABRICKS_TOKEN

# Deploy notebook
databricks workspace import $NOTEBOOK_PATH $WORKSPACE_PATH -f SOURCE -o
```

---

## 4️⃣ Write Azure Pipeline YAML

Create `.yml` in your root folder:

```yaml
trigger:
- main

variables:
  DATABRICKS_HOST: 'https://<your-databricks-url>'
  DATABRICKS_TOKEN: $(DATABRICKS_TOKEN)

stages:
- stage: Test
  jobs:
  - job: RunTests
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.9'
    - script: |
        pip install -r requirements.txt
        pytest tests/
      displayName: 'Run unit tests'

- stage: Deploy
  dependsOn: Test
  condition: succeeded()
  jobs:
  - job: DeployToWorkspace
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - checkout: self
    - script: |
        bash cicd/deploy.sh \
          $(DATABRICKS_HOST) \
          $(DATABRICKS_TOKEN) \
          notebooks/ETL_Notebook.py \
          /Users/youruser/ETL_Notebook
      displayName: 'Deploy notebook to Databricks'
```

---

## 5️⃣ Store Secrets in Azure DevOps

Go to **Pipelines > Library > New Variable Group**:

- Name: `databricks-secrets`
- Add `DATABRICKS_TOKEN` (make it secret)

In your pipeline YAML, link the variable group:

```yaml
variables:
- group: databricks-secrets
```

---

## 6️⃣ Optional: Multi-Stage Deployment

You can set up **dev → test → prod** Databricks workspaces using:

- Multiple service principals
- Environment-specific variable groups
- Separate stages in YAML

---

## ✅ Done! When you push to `main`, this will:

1. Run unit tests
2. Deploy the notebook to your Databricks workspace

---

## 🧠 Tips

- Use `databricks jobs create` / `databricks jobs reset` to manage production jobs.
- Use `pytest` + `unittest.mock` to mock Databricks functions in tests.
- Convert notebooks using `databricks workspace export`.
