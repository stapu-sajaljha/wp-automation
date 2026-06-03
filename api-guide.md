# OpenWA API Integration Guide (wa.test.stapubox.com)

This guide details the API endpoints, payloads, headers, and `curl` examples to integrate OpenWA with your application.

## Authentication & Headers
All requests to the API must include the following headers:
- **`X-API-Key`**: Your generated production API key (read from `/apps/whatsapp/wp-automation/data/.api-key`).
- **`Content-Type`**: `application/json`

---

## 1. Create a Session
Before automating WhatsApp, you need to create a session container.

* **Method:** `POST`
* **Path:** `/api/sessions`
* **Payload:**
  ```json
  {
    "name": "testing"
  }
  ```

### Curl Example:
```bash
curl -X POST https://wa.test.stapubox.com/api/sessions \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_KEY" \
  -d '{"name": "testing"}'
```
*Note the `"id"` (UUID) returned in the response. You must use that **`id`** (e.g., `8f9ce70c-5b3e-4731-a63d-a138d208f1e0`) in the URL path for all subsequent requests!*

---

## 2. Get All Sessions
Lists all active, idle, or disconnected sessions.

* **Method:** `GET`
* **Path:** `/api/sessions`

### Curl Example:
```bash
curl -X GET https://wa.test.stapubox.com/api/sessions \
  -H "X-API-Key: YOUR_API_KEY"
```

---

## 3. Create Group
Creates a new WhatsApp group with initial participants.

* **Method:** `POST`
* **Path:** `/api/sessions/{session_id}/groups`
* **Payload:**
  ```json
  {
    "name": "Squad Alpha Group",
    "participants": ["919876543210@c.us", "919988776655@c.us"]
  }
  ```
  *(Note: Phone numbers must end with `@c.us`)*

### Curl Example:
```bash
curl -X POST https://wa.test.stapubox.com/api/sessions/{session_id}/groups \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_KEY" \
  -d '{
    "name": "Squad Alpha Group",
    "participants": ["919876543210@c.us"]
  }'
```

---

## How to Get Group ID
There are two ways to retrieve a group's unique ID (which always ends with `@g.us`):

### A. Upon Group Creation
When you call the Create Group endpoint (above), the API response directly returns the created group ID:
```json
{
  "id": "120363409235686382@g.us",
  "name": "Squad Alpha Group"
}
```

### B. List Joined Groups for a Session
If the group already exists, you can query a list of all groups the session is currently in.

* **Method:** `GET`
* **Path:** `/api/sessions/{session_id}/groups`

#### Curl Example:
```bash
curl -X GET https://wa.test.stapubox.com/api/sessions/{session_id}/groups \
  -H "X-API-Key: YOUR_API_KEY"
```

#### Response Example:
```json
[
  {
    "id": "120363409235686382@g.us",
    "name": "Squad Alpha Group"
  },
  {
    "id": "120363999999999999@g.us",
    "name": "Squad Beta Group"
  }
]
```

---

## 4. Get Group Invite Link
Gets the official WhatsApp invite link and code for a group.

* **Method:** `GET`
* **Path:** `/api/sessions/{session_id}/groups/{group_id}/invite-code`
  *(Note: Group IDs end with `@g.us`)*

### Curl Example:
```bash
curl -X GET https://wa.test.stapubox.com/api/sessions/{session_id}/groups/120363409235686382@g.us/invite-code \
  -H "X-API-Key: YOUR_API_KEY"
```
### Response Example:
```json
{
  "inviteCode": "L1J8iKLmn...",
  "inviteLink": "https://chat.whatsapp.com/L1J8iKLmn..."
}
```

---

## 5. Set Group Description & Get Profile Pic

### A. Set Group Description
Updates the description text of the WhatsApp group.

* **Method:** `PUT`
* **Path:** `/api/sessions/{session_id}/groups/{group_id}/description`
* **Payload:**
  ```json
  {
    "description": "Welcome to Squad Alpha! Alternative Link: https://app.stapubox.com/squad/123"
  }
  ```

#### Curl Example:
```bash
curl -X PUT https://wa.test.stapubox.com/api/sessions/{session_id}/groups/120363409235686382@g.us/description \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_KEY" \
  -d '{"description": "Squad Description with Link: https://app.stapubox.com/squad/123"}'
```

### B. Get Group/Contact Profile Picture
*Note: Setting/updating group profile pictures is not exposed by the current gateway API. However, you can retrieve the current profile picture using:*

* **Method:** `GET`
* **Path:** `/api/sessions/{session_id}/contacts/{group_id}/profile-picture`

#### Curl Example:
```bash
curl -X GET https://wa.test.stapubox.com/api/sessions/{session_id}/contacts/120363409235686382@g.us/profile-picture \
  -H "X-API-Key: YOUR_API_KEY"
```

---

## 6. Add & Remove Members

### A. Add Members to Group
* **Method:** `POST`
* **Path:** `/api/sessions/{session_id}/groups/{group_id}/participants`
* **Payload:**
  ```json
  {
    "participants": ["919876543210@c.us"]
  }
  ```

#### Curl Example:
```bash
curl -X POST https://wa.test.stapubox.com/api/sessions/{session_id}/groups/120363409235686382@g.us/participants \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_KEY" \
  -d '{"participants": ["919876543210@c.us"]}'
```

### B. Remove Members from Group
* **Method:** `DELETE`
* **Path:** `/api/sessions/{session_id}/groups/{group_id}/participants`
* **Payload:**
  ```json
  {
    "participants": ["919876543210@c.us"]
  }
  ```

#### Curl Example:
```bash
curl -X DELETE https://wa.test.stapubox.com/api/sessions/{session_id}/groups/120363409235686382@g.us/participants \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_KEY" \
  -d '{"participants": ["919876543210@c.us"]}'
```

---

## 7. Send Message in a Group
Sends a text message to the group chat.

* **Method:** `POST`
* **Path:** `/api/sessions/{session_id}/messages/send-text`
* **Payload:**
  ```json
  {
    "chatId": "120363409235686382@g.us",
    "text": "Hello Everyone! A new event has been created in the squad."
  }
  ```

### Curl Example:
```bash
curl -X POST https://wa.test.stapubox.com/api/sessions/{session_id}/messages/send-text \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_KEY" \
  -d '{
    "chatId": "120363409235686382@g.us",
    "text": "Hello! A new event has been scheduled for tomorrow at 6 PM."
  }'
```
