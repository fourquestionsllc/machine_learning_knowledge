Below is a **practical, production-ready blueprint** for taking a **GenAI model (LLM / RAG / agentic system)** to production using **CI/CD + containers**, tailored to what you’ve worked on before (RAG, agents, Azure/AWS-scale systems).

---

## 1. Production Architecture (High Level)

```
Dev → Git → CI → Container Registry → CD → Runtime (K8s)
                         |
                    Model Registry
```

**Core components**

* GenAI service (API)
* Prompt + orchestration logic
* Model endpoint (OpenAI / Azure OpenAI / Bedrock / local)
* Vector DB (FAISS, Pinecone, Weaviate, Azure AI Search)
* Observability + guardrails

---

## 2. Repo Structure (Recommended)

```
genai-service/
│
├── app/
│   ├── main.py              # FastAPI entry
│   ├── agents/
│   ├── rag/
│   ├── prompts/
│   ├── safety/
│   └── config.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── eval/                # hallucination, consistency tests
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── pyproject.toml
│
├── .github/workflows/
│   ├── ci.yml
│   └── cd.yml
│
└── helm/
    └── genai-service/
```

---

## 3. Containerization (Docker)

### Dockerfile (Optimized for GenAI APIs)

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app ./app

ENV PYTHONUNBUFFERED=1
ENV PORT=8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Best practices**

* Never bake API keys into images
* Use multi-stage builds if using torch / transformers
* Pin dependency versions

---

## 4. CI Pipeline (GitHub Actions Example)

### `.github/workflows/ci.yml`

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  test-build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: "3.11"

    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest ruff

    - name: Lint
      run: ruff check app/

    - name: Run tests
      run: pytest tests/unit

    - name: Build Docker Image
      run: docker build -t genai-service:${{ github.sha }} .
```

---

## 5. CI for GenAI (Non-Traditional Tests)

### What to test beyond unit tests

| Type              | Example                       |
| ----------------- | ----------------------------- |
| Prompt regression | Golden Q/A checks             |
| Hallucination     | Answer must cite source       |
| Safety            | PII leakage tests             |
| Latency           | < X ms under load             |
| Determinism       | Same input → bounded variance |

Example **LLM eval test**:

```python
def test_no_hallucination(llm_client):
    answer = llm_client.ask("What is refund policy?")
    assert "I don't know" in answer or "Source:" in answer
```

---

## 6. Model & Prompt Versioning

### What to version

* Model name (gpt-4.1, gpt-4o, llama3)
* Prompt templates
* Retrieval configs (k, filters)
* Agent graph

**Store in**

* Git (prompts, configs)
* MLflow / Azure ML / Weights & Biases (metrics)

---

## 7. CD Pipeline (Deploy to Kubernetes)

### `.github/workflows/cd.yml`

```yaml
name: CD

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Login to Registry
      run: echo $CR_PAT | docker login ghcr.io -u USERNAME --password-stdin

    - name: Build & Push
      run: |
        docker build -t ghcr.io/org/genai:${{ github.sha }} .
        docker push ghcr.io/org/genai:${{ github.sha }}

    - name: Deploy to K8s
      run: |
        helm upgrade --install genai helm/genai-service \
        --set image.tag=${{ github.sha }}
```

---

## 8. Kubernetes Deployment Pattern

### Deployment YAML (Key parts)

```yaml
env:
- name: OPENAI_API_KEY
  valueFrom:
    secretKeyRef:
      name: openai-secret
      key: api-key

resources:
  requests:
    cpu: "500m"
    memory: "1Gi"
  limits:
    cpu: "2"
    memory: "4Gi"
```

### Scaling

* **HPA based on RPS + latency**
* Separate:

  * API layer
  * Retrieval workers
  * Embedding jobs

---

## 9. Secrets & Security

✅ Use:

* Kubernetes Secrets
* Azure Key Vault / AWS Secrets Manager
* GitHub OIDC → cloud auth

❌ Avoid:

* `.env` files in repos
* Logging prompts with PII
* Returning raw context

---

## 10. Observability (Must-Have)

| Signal        | Tool                      |
| ------------- | ------------------------- |
| Logs          | OpenTelemetry             |
| Traces        | Jaeger                    |
| Metrics       | Prometheus                |
| LLM analytics | LangSmith / Azure Monitor |
| Feedback loop | Thumbs up/down            |

**Log**

* prompt version
* model
* token usage
* retrieval doc IDs
* latency

---

## 11. Rollback & Safety

* **Blue-Green or Canary**
* Prompt rollback without redeploy
* Kill-switch for agent actions
* Human approval nodes (for agentic flows)

---

## 12. Production Checklist

✅ Stateless containers
✅ Prompt + model versioned
✅ CI tests include hallucination checks
✅ Secrets externalized
✅ Rate limits + retries
✅ Monitoring + alerting
✅ Rollback in <5 min

---

## 13. Common Production Mistakes

❌ Treating LLM as deterministic
❌ No evals in CI
❌ Scaling embeddings inline
❌ Overloading one pod with agents
❌ No prompt change audit trail
