To let the frontend know **where to send user action data**, you need to design a **well-defined API contract** and provide the frontend with the endpoint and instructions. Here’s how it’s usually done in large systems:

---

## **1. Define a REST or gRPC Endpoint**

**Example REST endpoint:**

```
POST https://api.hyatt.com/v1/log_action
```

* This is the **URL the frontend should call** for every action (click, search, book, scroll).
* You can version your API (`v1`) to manage future changes.

---

## **2. Provide a Clear JSON Schema**

Frontend developers need to know **what fields to send**.
Example JSON payload:

```json
{
  "user_id": "12345",
  "session_id": "abcde-67890",
  "timestamp": "2025-11-04T16:00:00Z",
  "action_type": "click",
  "query": "New York hotels",
  "hotel_id": "H123",
  "position": 3,
  "device": "web"
}
```

* Backend should maintain a **schema document** (e.g., OpenAPI/Swagger) so the frontend can validate before sending.

---

## **3. Use an API Gateway or Load Balancer**

* The frontend doesn’t directly talk to internal services.
* API Gateway handles:

  * Authentication/authorization (JWT tokens, API keys)
  * Routing to backend services
  * Rate limiting

**Example flow:**

```
Frontend → HTTPS POST → API Gateway → Logging Service → Kafka → Storage/ETL
```

---

## **4. Provide Environment-Specific Endpoints**

* **Development / staging / production** may have different endpoints.
* Frontend can use **configuration file or environment variable**:

```javascript
const ACTION_API_URL = process.env.ACTION_API_URL || "https://staging-api.hyatt.com/v1/log_action";
```

---

## **5. Handle Failures Gracefully**

* Frontend should **retry** if sending fails.
* Can also **batch multiple actions** to reduce network calls.
* Example:

```javascript
async function sendAction(action) {
    try {
        await fetch(ACTION_API_URL, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(action)
        });
    } catch (err) {
        console.error("Failed to send action, will retry later:", err);
        // Optionally queue for retry
    }
}
```

---

## **6. Document the API**

* Maintain a developer-facing document (Swagger/OpenAPI):

  * Endpoint URL
  * HTTP method
  * JSON fields and types
  * Required vs optional fields
  * Authentication method
  * Example request/response

This way, frontend engineers **don’t have to guess** where or how to send data.

---

✅ **Summary**

1. Backend exposes a **REST/gRPC endpoint** (e.g., `/v1/log_action`).
2. Frontend gets the **URL and JSON schema** from config/documentation.
3. Use **API Gateway** to route, secure, and scale.
4. Frontend sends actions with proper attributes.
5. Backend validates, stores, and processes the data.
