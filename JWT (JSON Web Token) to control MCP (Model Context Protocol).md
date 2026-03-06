Using **JWT (JSON Web Token)** to control **MCP (Model Context Protocol) tools/functions for agents** is a good way to implement **authentication + authorization** between the **agent client and MCP server**.

Below is a **simple architecture and example implementation**.

---

# 1. Architecture: JWT for MCP Access Control

In an agent system, the flow typically looks like this:

```
User
 │
 ▼
Auth Service
 │
 │ issues JWT
 ▼
Agent Client
 │
 │ sends JWT
 ▼
MCP Server
 │
 ├── Verify JWT
 ├── Extract user + roles
 ├── Check tool permissions
 │
 ▼
Execute MCP Tool
```

JWT ensures:

* Only **authenticated clients** can call MCP tools
* The server knows **who the user is**
* The server can enforce **tool-level permissions**

---

# 2. JWT Payload Example

A JWT token may contain **identity and permissions**.

Example payload:

```json
{
  "sub": "user123",
  "role": "analyst",
  "permissions": [
    "query_documents",
    "run_simulation"
  ],
  "exp": 1741500000
}
```

Fields:

| Field       | Purpose           |
| ----------- | ----------------- |
| sub         | user ID           |
| role        | user role         |
| permissions | allowed MCP tools |
| exp         | expiration        |

---

# 3. Agent Sends JWT to MCP Server

Example request from agent:

```
POST /mcp/tools/query_documents
Authorization: Bearer <jwt-token>
```

The MCP server validates the token before executing the tool.

---

# 4. Python Example – Verify JWT in MCP Server

Example using **FastAPI**.

```python
from fastapi import FastAPI, Header, HTTPException
import jwt

app = FastAPI()

SECRET = "my_secret"

def verify_jwt(token):
    try:
        payload = jwt.decode(token, SECRET, algorithms=["HS256"])
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

---

# 5. Tool-Level Authorization

Check if the user can access the MCP tool.

```python
def authorize(payload, tool_name):
    permissions = payload.get("permissions", [])
    return tool_name in permissions
```

---

# 6. Protected MCP Tool Example

```python
@app.post("/tools/query_documents")
def query_documents(authorization: str = Header(...)):

    token = authorization.split(" ")[1]
    payload = verify_jwt(token)

    if not authorize(payload, "query_documents"):
        raise HTTPException(status_code=403, detail="Access denied")

    return {"result": "documents returned"}
```

---

# 7. Agent Calling MCP Tool

Agent sends the token.

```python
import requests

token = "JWT_TOKEN"

headers = {
    "Authorization": f"Bearer {token}"
}

response = requests.post(
    "http://localhost:8000/tools/query_documents",
    headers=headers
)

print(response.json())
```

---

# 8. Fine-Grained Control for Agent Tools

JWT can include **tool scope**.

Example:

```json
{
  "sub": "agent123",
  "scope": [
    "vector.search",
    "simulation.run"
  ]
}
```

Server checks scope:

```
vector.search → allowed
admin.delete_data → denied
```

---

# 9. Recommended Security Additions

For **production MCP servers**:

### Token expiration

Short-lived tokens.

### Audience validation

```json
"aud": "mcp-server"
```

### Issuer validation

```json
"iss": "auth-service"
```

### HTTPS only

Never transmit JWT over HTTP.

### Logging

Record:

* user
* tool called
* timestamp

---

# 10. Secure MCP Architecture for Agents

```
User
 │
 ▼
Identity Provider
 │
 ▼
Agent
 │
 ▼
API Gateway
 │
 ▼
MCP Server
 │
 ├── JWT verification
 ├── Role check
 ├── Tool access control
 │
 ▼
Tool Execution
```

---

✅ **Key idea**

JWT enables:

* **authentication** (who is calling MCP)
* **authorization** (which tools the caller can use)
* **stateless security** (no session storage
