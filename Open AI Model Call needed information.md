If you mean **“I have an application on Azure that calls Azure OpenAI, and I want to reach/use an OpenAI model—what components do I need?”**, the typical architecture is:

```text
Your Application
     |
     | HTTPS / REST API
     ↓
Azure OpenAI Service
     |
     ↓
Model Deployment
     |
     ↓
GPT / Embedding Model
```

### What you need

| Component                 | Purpose                                                            |
| ------------------------- | ------------------------------------------------------------------ |
| **Azure subscription**    | Azure account/billing                                              |
| **Azure OpenAI resource** | Provides access to OpenAI models                                   |
| **Model deployment**      | Deploys a specific model in your Azure OpenAI resource             |
| **Endpoint**              | URL your application calls                                         |
| **Authentication**        | API key or Microsoft Entra ID                                      |
| **API/SDK**               | Your Python/Java/.NET/etc. application uses this to call the model |

For example, Python can call an Azure OpenAI deployment using the OpenAI SDK:

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    azure_endpoint="https://YOUR-RESOURCE.openai.azure.com/",
    api_key="YOUR_API_KEY",
    api_version="YOUR_API_VERSION"
)

response = client.chat.completions.create(
    model="YOUR_DEPLOYMENT_NAME",
    messages=[
        {
            "role": "user",
            "content": "Explain RAG in simple terms."
        }
    ]
)

print(response.choices[0].message.content)
```

The important distinction is:

**Your application does not normally call the model name directly. It calls your Azure OpenAI endpoint and specifies the deployment.**

```text
Application
     |
     | endpoint + authentication
     ↓
Azure OpenAI Resource
     |
     | deployment name
     ↓
GPT model
```

If you're asking specifically **how to reach an Azure OpenAI model from an Azure-hosted application (App Service, AKS, Function App, etc.)**, I can show the complete setup including **Azure OpenAI resource → model deployment → Entra ID → RBAC → Python application → API call**.
