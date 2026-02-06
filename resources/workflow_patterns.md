# n8n Workflow Patterns & Complete Examples

This document covers common workflow patterns and includes complete, importable n8n workflow JSON examples.

## Common Patterns

### 1. Webhook → Process → Respond
```
Webhook → Validate Data → Process → Respond to Webhook
```

### 2. Schedule → ETL
```
Schedule → Fetch Source → Transform → Load Destination
                                          ↓
                                    Send Summary Email
```

### 3. Event-Driven Notification
```
Trigger → Process Event → Route (Switch)
                            ├→ Slack
                            ├→ Email
                            └→ Database
```

### 4. Error-Resistant Batch Processing
```
Trigger → Get All Items → Loop
                           ├→ Process Item → Continue
                           └→ (Done) → Summary Report
```

### 5. Parallel API Aggregation
```
Trigger → HTTP Request 1 ──┐
       → HTTP Request 2 ──├→ Merge → Combine Results
       → HTTP Request 3 ──┘
```

### 6. Form → Validate → Store → Notify
```
Form Trigger → Validate → If Valid
                            ├→ (yes) → Store in DB → Send Confirmation
                            └→ (no) → Respond with Error
```

### 7. AI-Powered Processing
```
Chat Trigger → AI Agent → Custom Tool (via sub-workflow) → Response
                  ↑
            Chat Model + Memory
```

---

## Design Principles

### Workflow Organization

1. **Single Responsibility** - Each workflow does one thing well
2. **Error Handling** - Always plan for failures
3. **Idempotency** - Safe to re-run without side effects
4. **Logging** - Track important operations
5. **Sub-Workflows** - Break complex flows into modules

### Naming Conventions

```
[Trigger Type] - [Action] - [Version]
Examples:
  Webhook - Create Customer - v2
  Schedule - Daily Sales Report
  Form - Employee Onboarding
```

### Error Handling Strategy

```
Main Flow → Try Node → Process
              ↓ (error)
           Error Handler → Log → Notify Admin
```

Per-node settings:
```json
{
  "settings": {
    "onError": "continueErrorOutput"
  }
}
```

---

## Complete Importable Workflow Examples

### 1. Webhook API Endpoint

A REST-style API endpoint that processes POST data and returns a response.

```json
{
  "name": "Webhook API Endpoint",
  "nodes": [
    {
      "parameters": {
        "path": "api/data",
        "httpMethod": "POST",
        "responseMode": "responseNode",
        "options": {}
      },
      "id": "webhook-1",
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [250, 300],
      "webhookId": "api-data-webhook"
    },
    {
      "parameters": {
        "mode": "manual",
        "duplicateItem": false,
        "assignments": {
          "assignments": [
            {
              "id": "assign-1",
              "name": "processedAt",
              "value": "={{ $now.toISO() }}",
              "type": "string"
            },
            {
              "id": "assign-2",
              "name": "data",
              "value": "={{ $json.body }}",
              "type": "object"
            },
            {
              "id": "assign-3",
              "name": "status",
              "value": "received",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "id": "set-1",
      "name": "Process Data",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [500, 300]
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={{ JSON.stringify({ success: true, processedAt: $json.processedAt, status: $json.status }) }}",
        "options": { "responseCode": 200 }
      },
      "id": "respond-1",
      "name": "Respond to Webhook",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1.1,
      "position": [750, 300]
    }
  ],
  "connections": {
    "Webhook": { "main": [[{ "node": "Process Data", "type": "main", "index": 0 }]] },
    "Process Data": { "main": [[{ "node": "Respond to Webhook", "type": "main", "index": 0 }]] }
  },
  "settings": { "executionOrder": "v1" }
}
```

---

### 2. Scheduled Data Processing

Daily scheduled task that fetches data from an API, filters, and stores results.

```json
{
  "name": "Scheduled Data Processing",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [{ "field": "cronExpression", "expression": "0 9 * * 1-5" }]
        }
      },
      "id": "schedule-1",
      "name": "Schedule Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [250, 300]
    },
    {
      "parameters": {
        "method": "GET",
        "url": "https://api.example.com/data",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            { "name": "Authorization", "value": "Bearer {{ $env.API_KEY }}" }
          ]
        },
        "options": { "response": { "response": { "responseFormat": "json" } } }
      },
      "id": "http-1",
      "name": "Fetch Data",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [500, 300]
    },
    {
      "parameters": {
        "conditions": {
          "options": { "caseSensitive": true },
          "conditions": [
            {
              "leftValue": "={{ $json.status }}",
              "rightValue": "active",
              "operator": { "type": "string", "operation": "equals" }
            }
          ],
          "combinator": "and"
        }
      },
      "id": "filter-1",
      "name": "Filter Active",
      "type": "n8n-nodes-base.filter",
      "typeVersion": 2.2,
      "position": [750, 300]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO processed_data (data, processed_at) VALUES ($1, $2)",
        "options": {}
      },
      "id": "postgres-1",
      "name": "Store in Database",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2.5,
      "position": [1000, 300],
      "credentials": { "postgres": { "id": "1", "name": "Postgres" } }
    }
  ],
  "connections": {
    "Schedule Trigger": { "main": [[{ "node": "Fetch Data", "type": "main", "index": 0 }]] },
    "Fetch Data": { "main": [[{ "node": "Filter Active", "type": "main", "index": 0 }]] },
    "Filter Active": { "main": [[{ "node": "Store in Database", "type": "main", "index": 0 }]] }
  },
  "settings": { "executionOrder": "v1" }
}
```

---

### 3. Conditional Branching

Routes items based on value with If/Switch logic.

```json
{
  "name": "Conditional Branching",
  "nodes": [
    {
      "parameters": {
        "path": "process-order",
        "httpMethod": "POST",
        "responseMode": "lastNode",
        "options": {}
      },
      "id": "webhook-2",
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [250, 300],
      "webhookId": "process-order"
    },
    {
      "parameters": {
        "conditions": {
          "options": { "caseSensitive": true, "leftValue": "" },
          "conditions": [
            {
              "id": "cond-1",
              "leftValue": "={{ $json.amount }}",
              "rightValue": 1000,
              "operator": { "type": "number", "operation": "gte" }
            }
          ],
          "combinator": "and"
        }
      },
      "id": "if-1",
      "name": "Is High Value",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.2,
      "position": [500, 300]
    },
    {
      "parameters": {
        "mode": "manual",
        "assignments": {
          "assignments": [
            { "id": "a1", "name": "priority", "value": "high", "type": "string" },
            { "id": "a2", "name": "requiresApproval", "value": true, "type": "boolean" }
          ]
        },
        "options": {}
      },
      "id": "set-high",
      "name": "High Value Processing",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [750, 200]
    },
    {
      "parameters": {
        "mode": "manual",
        "assignments": {
          "assignments": [
            { "id": "a3", "name": "priority", "value": "normal", "type": "string" },
            { "id": "a4", "name": "requiresApproval", "value": false, "type": "boolean" }
          ]
        },
        "options": {}
      },
      "id": "set-normal",
      "name": "Normal Processing",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [750, 400]
    }
  ],
  "connections": {
    "Webhook": { "main": [[{ "node": "Is High Value", "type": "main", "index": 0 }]] },
    "Is High Value": {
      "main": [
        [{ "node": "High Value Processing", "type": "main", "index": 0 }],
        [{ "node": "Normal Processing", "type": "main", "index": 0 }]
      ]
    }
  },
  "settings": { "executionOrder": "v1" }
}
```

---

### 4. Loop with HTTP Requests

Batch-process items with rate-limited API calls.

```json
{
  "name": "Loop with HTTP Requests",
  "nodes": [
    {
      "parameters": {},
      "id": "trigger-loop",
      "name": "Manual Trigger",
      "type": "n8n-nodes-base.manualWorkflowTrigger",
      "typeVersion": 1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "batchSize": 5,
        "options": {}
      },
      "id": "loop-1",
      "name": "Loop Over Items",
      "type": "n8n-nodes-base.splitInBatches",
      "typeVersion": 3,
      "position": [500, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=https://api.example.com/process/{{ $json.id }}",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            { "name": "data", "value": "={{ JSON.stringify($json) }}" }
          ]
        },
        "options": {}
      },
      "id": "http-loop",
      "name": "Process Item",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [750, 200]
    },
    {
      "parameters": {
        "resume": "timeInterval",
        "amount": 1,
        "unit": "seconds"
      },
      "id": "wait-1",
      "name": "Rate Limit Wait",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [1000, 200]
    },
    {
      "parameters": {
        "mode": "manual",
        "assignments": {
          "assignments": [
            { "id": "done-1", "name": "completedAt", "value": "={{ $now.toISO() }}", "type": "string" },
            { "id": "done-2", "name": "totalProcessed", "value": "={{ $input.all().length }}", "type": "string" }
          ]
        },
        "options": {}
      },
      "id": "done-1",
      "name": "Done",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [750, 450]
    }
  ],
  "connections": {
    "Manual Trigger": { "main": [[{ "node": "Loop Over Items", "type": "main", "index": 0 }]] },
    "Loop Over Items": {
      "main": [
        [{ "node": "Process Item", "type": "main", "index": 0 }],
        [{ "node": "Done", "type": "main", "index": 0 }]
      ]
    },
    "Process Item": { "main": [[{ "node": "Rate Limit Wait", "type": "main", "index": 0 }]] },
    "Rate Limit Wait": { "main": [[{ "node": "Loop Over Items", "type": "main", "index": 0 }]] }
  },
  "settings": { "executionOrder": "v1" }
}
```

---

### 5. AI Agent with Tools

AI Agent with Calculator and HTTP Request tools.

```json
{
  "name": "AI Agent with Tools",
  "nodes": [
    {
      "parameters": { "options": {} },
      "id": "chat-t1",
      "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "options": {
          "systemMessage": "You are a helpful assistant. You can look up information and perform calculations. Always be accurate and cite your sources.",
          "maxIterations": 10
        }
      },
      "id": "agent-tools",
      "name": "AI Agent",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 1.7,
      "position": [500, 300]
    },
    {
      "parameters": {
        "model": "gpt-4o",
        "options": { "temperature": 0.7 }
      },
      "id": "openai-tools",
      "name": "OpenAI Chat Model",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "typeVersion": 1.2,
      "position": [400, 520],
      "credentials": { "openAiApi": { "id": "1", "name": "OpenAI Account" } }
    },
    {
      "parameters": {},
      "id": "calc-1",
      "name": "Calculator",
      "type": "@n8n/n8n-nodes-langchain.toolCalculator",
      "typeVersion": 1,
      "position": [600, 520]
    },
    {
      "parameters": {
        "name": "search_api",
        "description": "Search for information. Input should be a search query.",
        "method": "GET",
        "url": "=https://api.example.com/search?q={{ $fromAI('query', 'search query text') }}",
        "options": {}
      },
      "id": "http-tool-1",
      "name": "Search Tool",
      "type": "@n8n/n8n-nodes-langchain.toolHttpRequest",
      "typeVersion": 1.1,
      "position": [800, 520]
    },
    {
      "parameters": {
        "sessionKey": "default",
        "contextWindowLength": 10
      },
      "id": "memory-tools",
      "name": "Simple Memory",
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [500, 520]
    }
  ],
  "connections": {
    "Chat Trigger": { "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]] },
    "OpenAI Chat Model": { "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]] },
    "Calculator": { "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]] },
    "Search Tool": { "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]] },
    "Simple Memory": { "ai_memory": [[{ "node": "AI Agent", "type": "ai_memory", "index": 0 }]] }
  },
  "settings": { "executionOrder": "v1" }
}
```

---

### 6. Form Submission Handler

Process form submissions with validation and notification.

```json
{
  "name": "Form Submission Handler",
  "nodes": [
    {
      "parameters": {
        "path": "contact-form",
        "httpMethod": "POST",
        "responseMode": "responseNode",
        "options": {}
      },
      "id": "form-webhook",
      "name": "Form Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [250, 300],
      "webhookId": "contact-form"
    },
    {
      "parameters": {
        "conditions": {
          "options": { "caseSensitive": false },
          "conditions": [
            {
              "leftValue": "={{ $json.body.email }}",
              "rightValue": "",
              "operator": { "type": "string", "operation": "notEquals" }
            },
            {
              "leftValue": "={{ $json.body.message }}",
              "rightValue": "",
              "operator": { "type": "string", "operation": "notEquals" }
            }
          ],
          "combinator": "and"
        }
      },
      "id": "validate-1",
      "name": "Validate Input",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.2,
      "position": [500, 300]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO contacts (name, email, message, created_at) VALUES ($1, $2, $3, NOW())",
        "options": {}
      },
      "id": "store-1",
      "name": "Store Submission",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2.5,
      "position": [750, 200],
      "credentials": { "postgres": { "id": "1", "name": "Postgres" } }
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={{ JSON.stringify({ success: true, message: 'Thank you!' }) }}",
        "options": { "responseCode": 200 }
      },
      "id": "respond-ok",
      "name": "Success Response",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1.1,
      "position": [1000, 200]
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": "={{ JSON.stringify({ success: false, message: 'Email and message are required' }) }}",
        "options": { "responseCode": 400 }
      },
      "id": "respond-err",
      "name": "Error Response",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1.1,
      "position": [750, 450]
    }
  ],
  "connections": {
    "Form Webhook": { "main": [[{ "node": "Validate Input", "type": "main", "index": 0 }]] },
    "Validate Input": {
      "main": [
        [{ "node": "Store Submission", "type": "main", "index": 0 }],
        [{ "node": "Error Response", "type": "main", "index": 0 }]
      ]
    },
    "Store Submission": { "main": [[{ "node": "Success Response", "type": "main", "index": 0 }]] }
  },
  "settings": { "executionOrder": "v1" }
}
```

---

### 7. Data ETL Pipeline

Extract from multi-source → Transform → Load to destination.

```json
{
  "name": "Data ETL Pipeline",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [{ "field": "cronExpression", "expression": "0 2 * * *" }]
        }
      },
      "id": "etl-schedule",
      "name": "Schedule (2AM Daily)",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [250, 300]
    },
    {
      "parameters": {
        "method": "GET",
        "url": "https://api.source.com/data",
        "options": { "response": { "response": { "responseFormat": "json" } } }
      },
      "id": "etl-fetch",
      "name": "Extract Data",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [500, 300]
    },
    {
      "parameters": {
        "jsCode": "const items = $input.all();\nreturn items.map(item => ({\n  json: {\n    id: item.json.id,\n    name: item.json.name?.trim(),\n    email: item.json.email?.toLowerCase(),\n    amount: parseFloat(item.json.amount) || 0,\n    processedAt: new Date().toISOString()\n  }\n}));",
        "mode": "runOnceForAllItems"
      },
      "id": "etl-transform",
      "name": "Transform",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [750, 300]
    },
    {
      "parameters": {
        "conditions": {
          "conditions": [
            {
              "leftValue": "={{ $json.email }}",
              "rightValue": "",
              "operator": { "type": "string", "operation": "notEquals" }
            }
          ],
          "combinator": "and"
        }
      },
      "id": "etl-filter",
      "name": "Filter Valid",
      "type": "n8n-nodes-base.filter",
      "typeVersion": 2.2,
      "position": [1000, 300]
    },
    {
      "parameters": {
        "compare": "selectedFields",
        "fieldsToCompare": { "fields": [{ "fieldName": "email" }] }
      },
      "id": "etl-dedup",
      "name": "Remove Duplicates",
      "type": "n8n-nodes-base.removeDuplicates",
      "typeVersion": 1,
      "position": [1250, 300]
    },
    {
      "parameters": {
        "operation": "executeQuery",
        "query": "INSERT INTO synced_data (id, name, email, amount, synced_at) VALUES ($1, $2, $3, $4, $5) ON CONFLICT (id) DO UPDATE SET name = $2, email = $3, amount = $4, synced_at = $5",
        "options": {}
      },
      "id": "etl-load",
      "name": "Load to DB",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2.5,
      "position": [1500, 300],
      "credentials": { "postgres": { "id": "1", "name": "Postgres" } }
    }
  ],
  "connections": {
    "Schedule (2AM Daily)": { "main": [[{ "node": "Extract Data", "type": "main", "index": 0 }]] },
    "Extract Data": { "main": [[{ "node": "Transform", "type": "main", "index": 0 }]] },
    "Transform": { "main": [[{ "node": "Filter Valid", "type": "main", "index": 0 }]] },
    "Filter Valid": { "main": [[{ "node": "Remove Duplicates", "type": "main", "index": 0 }]] },
    "Remove Duplicates": { "main": [[{ "node": "Load to DB", "type": "main", "index": 0 }]] }
  },
  "settings": { "executionOrder": "v1" }
}
```

---

## Best Practices

### Performance
- Filter early to reduce data volume
- Use batch processing for large datasets
- Add Wait nodes for rate-limited APIs
- Split into sub-workflows for modularity

### Reliability
- Set execution timeouts
- Implement retry logic
- Add error handling per node
- Use idempotent operations (ON CONFLICT)

### Maintainability
- Use descriptive node names
- Add sticky notes for documentation
- Version your workflows
- Tag workflows by category
