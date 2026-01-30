# n8n REST API Reference

This document covers the n8n public REST API for programmatic access.

## Overview

The n8n API allows you to programmatically:
- Manage workflows (create, update, activate)
- Execute workflows
- View executions
- Manage credentials
- Access user information

**Note**: API access requires an API key and may not be available during free trial.

---

## Authentication

### API Key Authentication

```bash
# Header authentication
curl -X GET https://your-n8n.com/api/v1/workflows \
  -H "X-N8N-API-KEY: your-api-key"
```

### Creating API Keys

1. Go to Settings → API
2. Click "Create API Key"
3. Set expiration (optional)
4. Copy and store securely

### API Key Permissions

Keys inherit user permissions. Create keys for specific users to limit access.

---

## Base URL

```
https://your-n8n-instance.com/api/v1
```

For cloud:
```
https://your-account.app.n8n.cloud/api/v1
```

---

## Workflows

### List Workflows

```bash
GET /api/v1/workflows
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | number | Max items to return |
| `cursor` | string | Pagination cursor |
| `tags` | string | Filter by tag IDs |
| `active` | boolean | Filter by active state |

**Response:**
```json
{
  "data": [
    {
      "id": "1",
      "name": "My Workflow",
      "active": true,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-02T00:00:00.000Z",
      "tags": []
    }
  ],
  "nextCursor": "abc123"
}
```

### Get Workflow

```bash
GET /api/v1/workflows/{id}
```

**Response:**
```json
{
  "id": "1",
  "name": "My Workflow",
  "active": true,
  "nodes": [...],
  "connections": {...},
  "settings": {...},
  "staticData": null,
  "tags": [],
  "createdAt": "2024-01-01T00:00:00.000Z",
  "updatedAt": "2024-01-02T00:00:00.000Z"
}
```

### Create Workflow

```bash
POST /api/v1/workflows
```

**Request Body:**
```json
{
  "name": "New Workflow",
  "nodes": [
    {
      "parameters": {},
      "id": "node-1",
      "name": "Start",
      "type": "n8n-nodes-base.start",
      "typeVersion": 1,
      "position": [250, 300]
    }
  ],
  "connections": {},
  "settings": {
    "saveManualExecutions": true,
    "executionOrder": "v1"
  }
}
```

### Update Workflow

```bash
PATCH /api/v1/workflows/{id}
```

**Request Body:**
```json
{
  "name": "Updated Workflow",
  "nodes": [...],
  "connections": {...}
}
```

### Delete Workflow

```bash
DELETE /api/v1/workflows/{id}
```

### Activate Workflow

```bash
POST /api/v1/workflows/{id}/activate
```

### Deactivate Workflow

```bash
POST /api/v1/workflows/{id}/deactivate
```

---

## Executions

### List Executions

```bash
GET /api/v1/executions
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `workflowId` | string | Filter by workflow |
| `status` | string | Filter by status (waiting, running, success, error) |
| `limit` | number | Max items |
| `cursor` | string | Pagination |

**Response:**
```json
{
  "data": [
    {
      "id": "123",
      "finished": true,
      "mode": "manual",
      "startedAt": "2024-01-01T00:00:00.000Z",
      "stoppedAt": "2024-01-01T00:00:01.000Z",
      "workflowId": "1",
      "status": "success"
    }
  ],
  "nextCursor": "abc123"
}
```

### Get Execution

```bash
GET /api/v1/executions/{id}
```

**Response:**
```json
{
  "id": "123",
  "finished": true,
  "mode": "manual",
  "startedAt": "2024-01-01T00:00:00.000Z",
  "stoppedAt": "2024-01-01T00:00:01.000Z",
  "workflowId": "1",
  "data": {
    "resultData": {
      "runData": {...}
    }
  },
  "status": "success"
}
```

### Delete Execution

```bash
DELETE /api/v1/executions/{id}
```

### Retry Execution

```bash
POST /api/v1/executions/{id}/retry
```

---

## Credentials

### List Credentials

```bash
GET /api/v1/credentials
```

**Response:**
```json
{
  "data": [
    {
      "id": "1",
      "name": "My API Key",
      "type": "httpHeaderAuth",
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-02T00:00:00.000Z"
    }
  ]
}
```

### Create Credential

```bash
POST /api/v1/credentials
```

**Request Body:**
```json
{
  "name": "New API Key",
  "type": "httpHeaderAuth",
  "data": {
    "name": "Authorization",
    "value": "Bearer token123"
  }
}
```

### Delete Credential

```bash
DELETE /api/v1/credentials/{id}
```

---

## Tags

### List Tags

```bash
GET /api/v1/tags
```

### Create Tag

```bash
POST /api/v1/tags
```

**Request Body:**
```json
{
  "name": "production"
}
```

### Update Tag

```bash
PATCH /api/v1/tags/{id}
```

### Delete Tag

```bash
DELETE /api/v1/tags/{id}
```

---

## Webhook Execution

### Execute Workflow via Webhook

For workflows with Webhook trigger:

```bash
POST https://your-n8n.com/webhook/{webhook-path}
```

**Request Body:**
```json
{
  "data": "your payload"
}
```

### Execute Workflow via API

```bash
POST /api/v1/workflows/{id}/execute
```

**Request Body:**
```json
{
  "data": {
    "inputData": "value"
  }
}
```

---

## Pagination

### Cursor-based Pagination

```bash
# First request
GET /api/v1/workflows?limit=10

# Response includes nextCursor
{
  "data": [...],
  "nextCursor": "abc123"
}

# Next page
GET /api/v1/workflows?limit=10&cursor=abc123
```

---

## Error Handling

### Error Response Format

```json
{
  "code": 400,
  "message": "Bad Request",
  "hint": "Check your request body"
}
```

### Common Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 429 | Too Many Requests |
| 500 | Server Error |

---

## Rate Limiting

- Limits vary by plan
- Check `X-RateLimit-*` headers
- Implement exponential backoff

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
```

---

## API Playground

Self-hosted n8n includes an API playground:

```
https://your-n8n.com/api/v1/docs
```

Features:
- Interactive documentation
- Try requests directly
- View response formats

---

## Usage Examples

### Python Example

```python
import requests

API_KEY = "your-api-key"
BASE_URL = "https://your-n8n.com/api/v1"

headers = {
    "X-N8N-API-KEY": API_KEY,
    "Content-Type": "application/json"
}

# List workflows
response = requests.get(
    f"{BASE_URL}/workflows",
    headers=headers
)
workflows = response.json()

# Activate workflow
response = requests.post(
    f"{BASE_URL}/workflows/1/activate",
    headers=headers
)

# Execute workflow
response = requests.post(
    f"{BASE_URL}/workflows/1/execute",
    headers=headers,
    json={"data": {"key": "value"}}
)
```

### JavaScript Example

```javascript
const API_KEY = 'your-api-key';
const BASE_URL = 'https://your-n8n.com/api/v1';

const headers = {
  'X-N8N-API-KEY': API_KEY,
  'Content-Type': 'application/json'
};

// List workflows
const workflows = await fetch(`${BASE_URL}/workflows`, {
  headers
}).then(r => r.json());

// Create workflow
const newWorkflow = await fetch(`${BASE_URL}/workflows`, {
  method: 'POST',
  headers,
  body: JSON.stringify({
    name: 'API Created Workflow',
    nodes: [],
    connections: {}
  })
}).then(r => r.json());
```

---

## n8n Node for API

Use the built-in n8n node:

```javascript
// In n8n workflow
// n8n Node → API Operations

// List workflows
Operation: Get All Workflows

// Execute workflow
Operation: Execute Workflow
Workflow ID: {{ $json.workflowId }}
```
