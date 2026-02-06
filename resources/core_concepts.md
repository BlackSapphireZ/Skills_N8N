# n8n Core Concepts, Nodes & Flow Logic

This document covers the fundamental concepts of n8n workflow automation, including all core node types with their JSON configurations.

## Workflows

A **workflow** is a collection of nodes connected together to automate a process.

### Workflow Lifecycle

```
Create → Configure → Test → Activate → Monitor
```

### Workflow States

| State | Description |
|-------|-------------|
| **Draft** | Work in progress, not executing |
| **Active** | Running on triggers/schedule |
| **Inactive** | Saved but not executing |
| **Error** | Failed, needs attention |

### Core Workflow JSON Structure

Every n8n workflow is a JSON object:

```json
{
  "name": "My Workflow",
  "nodes": [
    {
      "parameters": {},
      "id": "unique-uuid",
      "name": "Node Display Name",
      "type": "n8n-nodes-base.nodeType",
      "typeVersion": 1,
      "position": [250, 300]
    }
  ],
  "connections": {
    "Source Node Name": {
      "main": [
        [
          { "node": "Target Node Name", "type": "main", "index": 0 }
        ]
      ]
    }
  },
  "settings": {
    "executionOrder": "v1"
  }
}
```

**Node positioning**: Start triggers at [250, 300]. Space nodes ~250px horizontally, ~200px vertically for branches.

---

## Trigger Nodes

Every workflow starts with exactly one trigger.

### Manual Trigger
```json
{
  "parameters": {},
  "type": "n8n-nodes-base.manualWorkflowTrigger",
  "typeVersion": 1,
  "name": "Manual Trigger"
}
```

### Schedule Trigger
```json
{
  "parameters": {
    "rule": {
      "interval": [{ "field": "cronExpression", "expression": "0 9 * * 1-5" }]
    }
  },
  "type": "n8n-nodes-base.scheduleTrigger",
  "typeVersion": 1.2,
  "name": "Schedule Trigger"
}
```
Supports: `seconds`, `minutes`, `hours`, `days`, `weeks`, `months`, or `cronExpression`.

### Webhook Trigger
```json
{
  "parameters": {
    "path": "my-webhook-path",
    "httpMethod": "POST",
    "responseMode": "responseNode",
    "options": {}
  },
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 2,
  "name": "Webhook",
  "webhookId": "unique-webhook-uuid"
}
```
- `httpMethod`: GET, POST, PUT, DELETE, PATCH, HEAD
- `responseMode`: `onReceived` (immediate 200), `responseNode` (wait for Respond to Webhook node), `lastNode`
- Test URL vs Production URL: test URL has `/test/` path prefix

### Chat Trigger (for AI workflows)
```json
{
  "parameters": {
    "options": {}
  },
  "type": "@n8n/n8n-nodes-langchain.chatTrigger",
  "typeVersion": 1.1,
  "name": "Chat Trigger"
}
```

### Other Common Triggers
| Type | `type` value | Use case |
|------|-------------|----------|
| Email (IMAP) | `n8n-nodes-base.emailImap` | Trigger on new email |
| Gmail Trigger | `n8n-nodes-base.gmailTrigger` | New Gmail messages |
| n8n Form Trigger | `n8n-nodes-base.formTrigger` | Form submissions |
| RSS Feed Trigger | `n8n-nodes-base.rssFeedReadTrigger` | New RSS items |
| Error Trigger | `n8n-nodes-base.errorTrigger` | Workflow errors |
| Execute Workflow Trigger | `n8n-nodes-base.executeWorkflowTrigger` | Called by sub-workflow |
| App-specific triggers | `n8n-nodes-base.{app}Trigger` | GitHub, Slack, Airtable, etc. |

---

## Flow Logic Nodes

### If (Conditional)
```json
{
  "parameters": {
    "conditions": {
      "options": { "caseSensitive": true, "leftValue": "" },
      "conditions": [
        {
          "id": "unique-id",
          "leftValue": "={{ $json.status }}",
          "rightValue": "active",
          "operator": { "type": "string", "operation": "equals" }
        }
      ],
      "combinator": "and"
    }
  },
  "type": "n8n-nodes-base.if",
  "typeVersion": 2.2,
  "name": "If"
}
```
Outputs: `main[0]` = true branch, `main[1]` = false branch.

Operators: `equals`, `notEquals`, `contains`, `notContains`, `startsWith`, `endsWith`, `regex`, `gt`, `gte`, `lt`, `lte`, `isEmpty`, `isNotEmpty`, `exists`, `notExists`.

### Switch
```json
{
  "parameters": {
    "rules": {
      "values": [
        {
          "outputKey": "order_processing",
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.type }}",
                "rightValue": "order",
                "operator": { "type": "string", "operation": "equals" }
              }
            ]
          }
        }
      ]
    },
    "options": { "fallbackOutput": "extra" }
  },
  "type": "n8n-nodes-base.switch",
  "typeVersion": 3.2,
  "name": "Switch"
}
```
Multiple output branches. `fallbackOutput`: `"none"` or `"extra"` (adds a fallback output).

### Merge
```json
{
  "parameters": {
    "mode": "combine",
    "combineBy": "combineByPosition",
    "options": {}
  },
  "type": "n8n-nodes-base.merge",
  "typeVersion": 3.1,
  "name": "Merge"
}
```
Modes:
- `append`: Combine all items from both inputs
- `combine`: Join items by position (`combineByPosition`), by matching fields (`combineByFields`), or all combinations (`multiplex`)
- `chooseBranch`: Wait for both inputs but only output one

### Loop Over Items (Split In Batches)
```json
{
  "parameters": {
    "batchSize": 10,
    "options": {}
  },
  "type": "n8n-nodes-base.splitInBatches",
  "typeVersion": 3,
  "name": "Loop Over Items"
}
```
Outputs: `main[0]` = batch items (loop body), `main[1]` = done (all batches complete).

### Wait
```json
{
  "parameters": {
    "resume": "timeInterval",
    "amount": 5,
    "unit": "seconds"
  },
  "type": "n8n-nodes-base.wait",
  "typeVersion": 1.1,
  "name": "Wait"
}
```
Resume options: `timeInterval`, `specificTime`, `webhook`, `form`.

### Execute Sub-Workflow
```json
{
  "parameters": {
    "source": "database",
    "workflowId": "={{ $json.workflowId }}",
    "options": {}
  },
  "type": "n8n-nodes-base.executeWorkflow",
  "typeVersion": 1.2,
  "name": "Execute Sub-Workflow"
}
```

### Error Handling
- **Stop And Error**: Stops workflow with custom error message
- **Error Trigger**: Catches errors from other workflows
- Per-node error handling: In node settings, set `"onError": "continueRegularOutput"` or `"continueErrorOutput"`

---

## Data Transformation Nodes

### Edit Fields (Set)
```json
{
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "unique-id",
          "name": "newField",
          "value": "={{ $json.existingField }}",
          "type": "string"
        }
      ]
    },
    "options": {}
  },
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "name": "Edit Fields"
}
```
Types: `string`, `number`, `boolean`, `array`, `object`.

### Filter
```json
{
  "parameters": {
    "conditions": {
      "options": { "caseSensitive": true },
      "conditions": [
        {
          "leftValue": "={{ $json.amount }}",
          "rightValue": 100,
          "operator": { "type": "number", "operation": "gt" }
        }
      ],
      "combinator": "and"
    }
  },
  "type": "n8n-nodes-base.filter",
  "typeVersion": 2.2,
  "name": "Filter"
}
```

### Sort
```json
{
  "parameters": {
    "sortFieldsUi": {
      "sortField": [
        { "fieldName": "createdAt", "order": "descending" }
      ]
    }
  },
  "type": "n8n-nodes-base.sort",
  "typeVersion": 1,
  "name": "Sort"
}
```

### Limit
```json
{
  "parameters": { "maxItems": 10 },
  "type": "n8n-nodes-base.limit",
  "typeVersion": 1,
  "name": "Limit"
}
```

### Remove Duplicates
```json
{
  "parameters": {
    "compare": "selectedFields",
    "fieldsToCompare": {
      "fields": [{ "fieldName": "email" }]
    }
  },
  "type": "n8n-nodes-base.removeDuplicates",
  "typeVersion": 1,
  "name": "Remove Duplicates"
}
```

### Split Out (unnest arrays)
```json
{
  "parameters": {
    "fieldToSplitOut": "items",
    "options": {}
  },
  "type": "n8n-nodes-base.splitOut",
  "typeVersion": 1,
  "name": "Split Out"
}
```

### Aggregate (group items)
```json
{
  "parameters": {
    "aggregate": "aggregateAllItemData",
    "options": {}
  },
  "type": "n8n-nodes-base.aggregate",
  "typeVersion": 1,
  "name": "Aggregate"
}
```

### Summarize
```json
{
  "parameters": {
    "fieldsToSummarize": {
      "values": [
        { "field": "amount", "aggregation": "sum" }
      ]
    },
    "options": {}
  },
  "type": "n8n-nodes-base.summarize",
  "typeVersion": 1,
  "name": "Summarize"
}
```
Aggregations: `sum`, `count`, `countUnique`, `min`, `max`, `average`, `concatenate`, `append`.

---

## Action / App Nodes

### HTTP Request (universal API calls)
```json
{
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/data",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        { "name": "Authorization", "value": "Bearer {{ $json.token }}" }
      ]
    },
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        { "name": "key", "value": "={{ $json.value }}" }
      ]
    },
    "options": { "response": { "response": { "responseFormat": "json" } } }
  },
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "name": "HTTP Request"
}
```

### Respond to Webhook
```json
{
  "parameters": {
    "respondWith": "json",
    "responseBody": "={{ JSON.stringify({ success: true, data: $json }) }}",
    "options": { "responseCode": 200 }
  },
  "type": "n8n-nodes-base.respondToWebhook",
  "typeVersion": 1.1,
  "name": "Respond to Webhook"
}
```

### Popular App Nodes (type identifiers)
| App | Type | Common Actions |
|-----|------|---------------|
| Gmail | `n8n-nodes-base.gmail` | send, getAll, get, reply, addLabels |
| Google Sheets | `n8n-nodes-base.googleSheets` | append, read, update, clear |
| Slack | `n8n-nodes-base.slack` | postMessage, update, getAll |
| Notion | `n8n-nodes-base.notion` | create, get, getAll, update, archive |
| Airtable | `n8n-nodes-base.airtable` | create, get, getAll, update, delete |
| Telegram | `n8n-nodes-base.telegram` | sendMessage, sendPhoto, sendDocument |
| Discord | `n8n-nodes-base.discord` | sendMessage |
| GitHub | `n8n-nodes-base.github` | create, get, getAll |
| Postgres | `n8n-nodes-base.postgres` | executeQuery, insert, update, select |
| MySQL | `n8n-nodes-base.mysql` | executeQuery, insert, update, select |
| MongoDB | `n8n-nodes-base.mongodb` | find, insert, update, delete |
| Redis | `n8n-nodes-base.redis` | get, set, delete, push, pop |
| OpenAI | `@n8n/n8n-nodes-langchain.openAi` | text, image, audio, assistant |
| Anthropic | `@n8n/n8n-nodes-langchain.anthropic` | message |
| Google Gemini | `@n8n/n8n-nodes-langchain.googleGemini` | message |

---

## Utility Nodes

| Node | Type | Purpose |
|------|------|---------|
| Code | `n8n-nodes-base.code` | Custom JS/Python code |
| HTML | `n8n-nodes-base.html` | Parse/generate HTML |
| XML | `n8n-nodes-base.xml` | Parse/convert XML |
| Markdown | `n8n-nodes-base.markdown` | Convert markdown |
| Crypto | `n8n-nodes-base.crypto` | Hash, HMAC, encrypt |
| Date & Time | `n8n-nodes-base.dateTime` | Format/calculate dates |
| Convert to File | `n8n-nodes-base.convertToFile` | Convert data to file |
| Extract From File | `n8n-nodes-base.extractFromFile` | Extract data from file |
| Compression | `n8n-nodes-base.compression` | Zip/unzip files |
| Send Email | `n8n-nodes-base.sendEmail` | SMTP email send |
| SSH | `n8n-nodes-base.ssh` | Remote SSH commands |
| FTP | `n8n-nodes-base.ftp` | FTP file operations |
| GraphQL | `n8n-nodes-base.graphQl` | GraphQL queries |
| RSS Read | `n8n-nodes-base.rssFeedRead` | Read RSS feeds |

---

## Connection Patterns

### Linear (A → B → C)
```json
"connections": {
  "Trigger": { "main": [[{ "node": "Process", "type": "main", "index": 0 }]] },
  "Process": { "main": [[{ "node": "Output", "type": "main", "index": 0 }]] }
}
```

### Branch (If node → true/false)
```json
"connections": {
  "If": {
    "main": [
      [{ "node": "True Path", "type": "main", "index": 0 }],
      [{ "node": "False Path", "type": "main", "index": 0 }]
    ]
  }
}
```

### Multi-input Merge
```json
"connections": {
  "Branch A": { "main": [[{ "node": "Merge", "type": "main", "index": 0 }]] },
  "Branch B": { "main": [[{ "node": "Merge", "type": "main", "index": 1 }]] }
}
```

### Loop pattern
```json
"connections": {
  "Loop Over Items": {
    "main": [
      [{ "node": "Process Batch", "type": "main", "index": 0 }],
      [{ "node": "Done", "type": "main", "index": 0 }]
    ]
  },
  "Process Batch": { "main": [[{ "node": "Loop Over Items", "type": "main", "index": 0 }]] }
}
```

### AI Agent Sub-node Connections
AI nodes use special connection types beyond `main`:
```json
"connections": {
  "Chat Trigger": { "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]] },
  "OpenAI Chat Model": { "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]] },
  "Simple Memory": { "ai_memory": [[{ "node": "AI Agent", "type": "ai_memory", "index": 0 }]] },
  "Calculator Tool": { "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]] }
}
```

---

## Executions

Executions are records of workflow runs.

### Execution Data

Each execution contains:
- **ID**: Unique identifier
- **Status**: success, error, running, waiting, canceled
- **Started At**: Timestamp
- **Finished At**: Timestamp
- **Mode**: manual, trigger, webhook
- **Data**: Input/output of each node

### Data Pinning

Pin data for testing:
1. Execute workflow
2. Click pin icon on node output
3. Pinned data used in subsequent tests
4. Unpin to use live data

### Retry Executions

1. Open failed execution
2. Click "Retry" button
3. Choose from failed node or start
4. New execution created

---

## Credentials

Credentials store authentication data securely.

### Credential Types

| Type | Description |
|------|-------------|
| **API Key** | Simple key authentication |
| **OAuth2** | Token-based auth flow |
| **Basic Auth** | Username/password |
| **Header Auth** | Custom header |
| **Query Auth** | URL parameter |

### Variables

```javascript
// Access workflow variable
{{ $vars.myVariable }}

// Access environment variable
{{ $env.API_KEY }}
```

---

## Node Execution Order

1. Nodes execute left to right
2. Parallel branches execute simultaneously
3. Merge nodes wait for all inputs
4. Loops iterate through items

---

## Tags and Organization

### Naming Conventions

Best practices:
- Use descriptive names
- Include purpose/trigger
- Version if needed

Examples:
- `Slack - New Customer Notification`
- `Daily Sales Report - v2`
- `Webhook - Order Processing`
