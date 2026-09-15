Since your **ADK agent works locally**, the next step is **deploy it to Agent Runtime, then register that deployed agent in Gemini Enterprise**.

The key point is:

```text
Your laptop
   │
   │ deploy
   ▼
Agent Runtime
   │
   │ register
   ▼
Gemini Enterprise
   │
   ▼
Enterprise users
```

You **do not deploy your local Python code directly into the Gemini Enterprise UI**. Google requires ADK agents used by Gemini Enterprise to be hosted on Agent Runtime. ([Google Cloud Documentation][1])

## 1. Prepare the GCP project

In the project where you will host the agent:

```bash
gcloud config set project YOUR_PROJECT_ID

gcloud auth application-default login
```

Make sure you have the relevant Agent Platform permissions, including `roles/aiplatform.user` and storage permissions for the deployment approach you're using. ([Google Cloud Documentation][2])

Install/update the SDK:

```bash
pip install --upgrade \
  "google-cloud-aiplatform[agent_engines,adk]>=1.112"
```

Create a Cloud Storage bucket for staging:

```bash
gcloud storage buckets create gs://YOUR_AGENT_BUCKET \
    --location=us-central1
```

---

# 2. Your local ADK code becomes an `AdkApp`

You probably already have something similar to:

```python
from google.adk.agents import Agent
from vertexai import agent_engines

root_agent = Agent(
    name="financial_agent",
    model="gemini-3.5-flash",
    instruction="""
    You are a financial research assistant.
    Answer questions using the available tools.
    """,
    tools=[
        # your tools
    ],
)

app = agent_engines.AdkApp(
    agent=root_agent
)
```

The important part for deployment is:

```python
app = agent_engines.AdkApp(
    agent=root_agent
)
```

Google's current Agent Runtime quickstart uses this `AdkApp` structure for both local testing and deployment. ([Google Cloud Documentation][2])

---

# 3. Create the Vertex AI client

Create something like `deploy.py`:

```python
import vertexai

client = vertexai.Client(
    project="YOUR_PROJECT_ID",
    location="us-central1",
)
```

Then import your ADK app:

```python
from agent import app
```

---

# 4. Deploy the ADK agent

Then:

```python
from vertexai import types

remote_agent = client.agent_engines.create(
    agent=app,
    config={
        "requirements": [
            "google-cloud-aiplatform[agent_engines,adk]"
        ],
        "staging_bucket": "gs://YOUR_AGENT_BUCKET",
        "identity_type": types.IdentityType.AGENT_IDENTITY,
    },
)

print(remote_agent.api_resource.name)
```

Run:

```bash
python deploy.py
```

Google creates an Agent Runtime resource for your ADK application. ([Google Cloud Documentation][2])

You'll get something similar to:

```text
projects/123456789/
locations/us-central1/
reasoningEngines/1234567890123456789
```

**Save this resource path.**

You'll need it when registering the agent with Gemini Enterprise.

---

# 5. Test the cloud deployment BEFORE Gemini Enterprise

This is an important step.

Don't immediately register it with Gemini Enterprise.

First test:

```python
async for event in remote_agent.async_stream_query(
    user_id="developer-001",
    message="What was Citi revenue in Asia?"
):
    print(event)
```

If this works, your architecture is now:

```text
VS Code
   │
   ▼
ADK
   │
   ▼
Agent Runtime
   │
   ├── Gemini
   ├── RAG
   ├── BigQuery
   ├── APIs
   └── MCP
```

Google recommends testing the deployed agent through the Agent Platform SDK before using it through Gemini Enterprise. ([Google Cloud Documentation][2])

---

# 6. Now connect it to Gemini Enterprise

Now go to the **Gemini Enterprise** application in Google Cloud.

You need:

* Gemini Enterprise app
* Gemini Enterprise Admin permissions
* Discovery Engine API enabled
* ADK agent already deployed to Agent Runtime

Google's registration flow is:

**Gemini Enterprise → select your app → Agents → Add agent → Custom agent via Agent Runtime**. ([Google Cloud Documentation][1])

Then configure:

### Agent name

For example:

```text
Citi Financial Research Agent
```

### Description

This is important.

Gemini Enterprise uses the description to help determine when it should invoke your agent.

For example:

```text
Answers questions about Citi financial
metrics by sector, region, and country.
Searches financial reports using semantic
search and queries financial databases using
SQL. Returns answers with supporting sources.
```

### Agent Runtime resource path

Paste the resource you got from deployment:

```text
projects/123456789/
locations/us-central1/
reasoningEngines/1234567890123456789
```

Then click **Create**. ([Google Cloud Documentation][1])

---

# 7. Now the architecture changes

Before registration:

```text
User
 │
 ▼
Your local ADK
 │
 ▼
Gemini
```

After registration:

```text
                    Gemini Enterprise
                           │
                           │ user question
                           ▼
                    Agent selection
                           │
                           ▼
                    Agent Runtime
                           │
                           ▼
                       ADK Agent
                           │
              ┌────────────┼─────────────┐
              ▼            ▼             ▼
           Gemini         RAG           SQL
              │            │             │
              ▼            ▼             ▼
         Vertex AI    Vector Search   BigQuery
```

The employee interacts with **Gemini Enterprise**, not directly with your ADK endpoint.

Gemini Enterprise sends the request to Agent Runtime, and Agent Runtime executes your ADK agent. ([Google Cloud Documentation][1])

---

# 8. One important thing: project separation

You can have:

```text
Project A
└── Gemini Enterprise
```

and:

```text
Project B
└── Agent Runtime
    └── ADK Agent
```

They don't have to be the same GCP project.

If they are different projects, you need to grant the Gemini Enterprise service agent the required permission in the project hosting the ADK agent. Google currently documents granting:

```text
roles/discoveryengine.serviceAgent
```

to the Gemini Enterprise service agent in the agent-hosting project. ([Google Cloud Documentation][3])

---

# 9. Watch the region

This can cause a surprisingly common deployment/registration error.

For Gemini Enterprise:

| Gemini Enterprise app | Agent Runtime        |
| --------------------- | -------------------- |
| `global`              | Any supported region |
| `us`                  | `us-*` region        |
| `eu`                  | `europe-*` region    |

So, for example:

```text
Gemini Enterprise: eu
Agent Runtime: europe-west1
```

is compatible.

But:

```text
Gemini Enterprise: eu
Agent Runtime: us-central1
```

is not. ([Google Cloud Documentation][1])

---

# 10. Your complete development lifecycle

For you as the developer, I'd structure it like this:

```text
                 DEVELOPMENT

VS Code
   │
   ▼
ADK Agent
   │
   ├── Tools
   ├── RAG
   ├── SQL
   ├── MCP
   └── Sub-agents
   │
   ▼
Gemini / Vertex AI
   │
   ▼
Local ADK Playground
   │
   ▼
pytest / evaluation
   │
   │
   │ DEPLOY
   ▼
────────────────────────────────
             GCP
────────────────────────────────
   │
   ▼
Agent Runtime
   │
   ▼
Cloud test
   │
   │ REGISTER
   ▼
Gemini Enterprise
   │
   ▼
Enterprise employees
```

### In practice, your next 5 commands/actions are:

```text
1. Create/choose GCP project
          ↓
2. Create staging bucket
          ↓
3. Deploy AdkApp → Agent Runtime
          ↓
4. Test remote_agent
          ↓
5. Gemini Enterprise → Agents
   → Add agent
   → Custom agent via Agent Runtime
   → paste resource path
```

Google also now supports several Agent Runtime deployment methods—including source files, Dockerfile, container image, Developer Connect, and the Agent Platform SDK—so for a real team with Git/CI/CD, you don't necessarily have to deploy manually from your laptop every time. ([Google Cloud Documentation][4])

For your **first deployment**, though, I would use the **Agent Platform SDK** above because it makes the relationship between your local `AdkApp` and the deployed Agent Runtime agent very clear.

[1]: https://docs.cloud.google.com/gemini/enterprise/docs/register-and-manage-an-adk-agent?authuser=2&utm_source=chatgpt.com "Register and manage ADK agents hosted on Agent Runtime  |  Gemini Enterprise  |  Google Cloud Documentation"
[2]: https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/runtime/quickstart-adk?utm_source=chatgpt.com "Quickstart: Develop and deploy agents on Agent Runtime with Agent Development Kit  |  Gemini Enterprise Agent Platform  |  Google Cloud Documentation"
[3]: https://docs.cloud.google.com/gemini/enterprise/docs/configure-cross-project-adk-agents?authuser=610&utm_source=chatgpt.com "Configure cross-project ADK agent access  |  Gemini Enterprise  |  Google Cloud Documentation"
[4]: https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale/runtime/deploy-an-agent?utm_source=chatgpt.com "Deploy an agent  |  Gemini Enterprise Agent Platform  |  Google Cloud Documentation"
