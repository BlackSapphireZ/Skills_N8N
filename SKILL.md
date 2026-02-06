---
name: n8n Workflow Automation
description: >  
  Comprehensive skill for building, generating, and debugging n8n workflow automation JSON files.
  Use when the user asks to create, edit, debug, or explain n8n workflows, including AI agent workflows,
  LangChain integrations, webhook automations, scheduled tasks, data transformations, or any n8n-related task.
  Includes exact JSON node configurations and importable workflow examples.
---

# n8n Workflow Automation Skill

This skill provides comprehensive knowledge for building and generating workflow automations using n8n, based on official documentation from https://docs.n8n.io/. It includes exact JSON node configurations for direct import into n8n, importable workflow examples, and AI agent integration details.

## Overview

n8n (pronounced n-eight-n) is a fair-code licensed workflow automation tool that combines AI capabilities with business process automation. It connects any app with an API with any other, and manipulates data with little or no code.

### Key Features
- **Customizable**: Highly flexible workflows with option to build custom nodes
- **Deployment Options**: npm, Docker, or Cloud hosting
- **Privacy-focused**: Self-host for privacy and security
- **AI-capable**: Built-in LangChain integration for AI workflows
- **JSON-based**: All workflows are JSON — can be generated, imported, and version-controlled

---

## Core Concepts

### Workflows

A **workflow** is a collection of nodes connected together to automate a process.

```
Trigger Node → Action Node → Action Node → Output
```

**Key Workflow Operations:**
- **Create**: Build new workflows using the visual editor
- **Activate**: Enable workflows to run automatically
- **Execute**: Run workflows manually or on schedule
- **Share**: Share workflows between users
- **Template**: Use pre-built workflow templates

### Nodes

Nodes are the building blocks of workflows. Types include:

| Node Type | Description | Examples |
|-----------|-------------|----------|
| **Trigger Nodes** | Start workflow execution | Webhook, Schedule, Email Trigger |
| **Action Nodes** | Perform operations | HTTP Request, Send Email, Database |
| **Core Nodes** | Flow control & utilities | IF, Switch, Merge, Code |
| **AI Nodes** | AI/LLM operations | AI Agent, Chat Model, Vector Store |

### Connections

Connections define how data flows between nodes:
- **Main connections**: Regular data flow (solid lines)
- **AI connections**: Connect AI sub-nodes to root nodes

### Executions

Executions are records of workflow runs:
- View execution history
- Debug failed executions
- Pin data for testing
- Retry failed executions

---

## Data Handling

### Data Structure

n8n uses a specific JSON structure for data:

```json
[
  {
    "json": {
      "name": "John",
      "email": "john@example.com"
    },
    "binary": {}
  }
]
```

- Each item has a `json` property containing the data
- Optional `binary` property for files/attachments
- Data flows as arrays of items between nodes

### Data Transformation Nodes

| Node | Purpose |
|------|---------|
| **Aggregate** | Group items together |
| **Limit** | Remove items beyond max |
| **Remove Duplicates** | Delete identical items |
| **Sort** | Order items |
| **Split Out** | Separate list into items |
| **Summarize** | Pivot-table style aggregation |

---

## Expressions

Expressions allow dynamic data access and transformation.

### Syntax

```javascript
{{ $json.fieldName }}           // Access current item field
{{ $input.first().json.name }}  // First input item
{{ $node["NodeName"].json }}    // Reference another node
{{ $now }}                      // Current timestamp
{{ $execution.id }}             // Execution ID
{{ $workflow.id }}              // Workflow ID
```

### Built-in Methods

```javascript
// Data access
$json                    // Current item's JSON data
$binary                  // Current item's binary data
$input.all()            // All input items
$input.first()          // First input item
$input.last()           // Last input item
$input.item             // Current item in loop

// Environment
$env.MY_VARIABLE        // Environment variable
$vars.myVariable        // Workflow variable

// Execution context
$execution.id           // Current execution ID
$execution.resumeUrl    // Wait node resume URL
$workflow.id            // Workflow ID
$workflow.name          // Workflow name
$now                    // Current DateTime
$today                  // Start of today
```

### Data Transformation Functions

```javascript
// String functions
{{ $json.name.toUpperCase() }}
{{ $json.email.split('@')[0] }}
{{ $json.text.replace('old', 'new') }}

// Array functions
{{ $json.items.length }}
{{ $json.items.filter(item => item.active) }}
{{ $json.items.map(item => item.name) }}

// Object functions
{{ Object.keys($json) }}
{{ Object.values($json) }}

// Date functions
{{ $now.toFormat('yyyy-MM-dd') }}
{{ $now.plus({ days: 7 }) }}
```

---

## Code Node

The Code node allows JavaScript or Python execution in workflows.

### JavaScript Example

```javascript
// Process all items
const results = [];

for (const item of $input.all()) {
  results.push({
    json: {
      originalName: item.json.name,
      processedName: item.json.name.toUpperCase(),
      timestamp: new Date().toISOString()
    }
  });
}

return results;
```

### Python Example

```python
# Process all items
results = []

for item in _input.all():
    results.append({
        "json": {
            "originalName": item.json["name"],
            "processedName": item.json["name"].upper(),
            "processed": True
        }
    })

return results
```

---

## Flow Logic

### Splitting with Conditionals

**IF Node** - Binary branching:
```
Data → IF → True branch
         → False branch
```

**Switch Node** - Multiple branches:
```
Data → Switch → Case 1
              → Case 2
              → Case 3
              → Default
```

### Merging Data

**Merge Node Modes:**
- **Append**: Combine all items from both inputs
- **Combine**: Match items by position or key
- **Choose Branch**: Select one input based on conditions

### Looping

**Loop Over Items Node:**
- Process items in batches
- Implement retry logic
- Rate limiting

### Sub-workflows

**Execute Workflow Node:**
- Call another workflow
- Pass data between workflows
- Reuse common logic

### Error Handling

**Error Trigger Node:**
- Catch workflow errors
- Send notifications
- Implement fallback logic

**Stop And Error Node:**
- Terminate execution
- Throw custom errors

---

## Integrations

### Built-in Nodes (400+)

Categories include:
- **Productivity**: Google Sheets, Notion, Airtable, Slack
- **CRM**: Salesforce, HubSpot, Pipedrive
- **Marketing**: Mailchimp, ActiveCampaign, SendGrid
- **Development**: GitHub, GitLab, Jira
- **Databases**: PostgreSQL, MySQL, MongoDB, Redis
- **Cloud**: AWS, Google Cloud, Azure
- **AI/ML**: OpenAI, Anthropic, Hugging Face

### HTTP Request Node

For custom API integrations:

```javascript
// Configuration
Method: POST
URL: https://api.example.com/data
Authentication: Header Auth
Headers: {
  "Content-Type": "application/json",
  "Authorization": "Bearer {{ $env.API_KEY }}"
}
Body: {
  "data": {{ $json }}
}
```

### Webhook Node

Create HTTP endpoints:

```javascript
// Webhook settings
HTTP Method: POST
Path: /my-webhook
Response Mode: Last Node
Authentication: Header Auth
```

### Credentials

Credentials store authentication data:
- API Keys
- OAuth2 tokens
- Basic Auth
- Custom authentication

---

## Advanced AI

### LangChain Integration

n8n uses LangChain for AI functionality (v1.19.4+).

### AI Node Types

**Root Nodes:**
- **AI Agent**: Autonomous AI with tools
- **Basic LLM Chain**: Simple LLM call
- **Q&A Chain**: Question answering
- **Summarization Chain**: Text summarization

**Sub-nodes:**
- **Chat Models**: OpenAI, Anthropic, etc.
- **Embeddings**: OpenAI, Cohere
- **Vector Stores**: Pinecone, Qdrant, Supabase
- **Memory**: Buffer, PostgreSQL
- **Tools**: Calculator, Code, HTTP Request

### Example: AI Agent Workflow

```
Chat Trigger → AI Agent → Response
                 ↑
            Chat Model (GPT-4)
                 ↑
            Vector Store (for RAG)
                 ↑
            Embeddings
```

### Chat Trigger

Trigger workflows from chat interactions:
- Built-in chat widget
- External chat integration
- Chatbot frontend

---

## Self-Hosting

### Docker Installation

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

### Docker Compose

```yaml
version: '3.8'
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=secure_password
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://n8n.example.com/
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

### Key Environment Variables

| Variable | Description |
|----------|-------------|
| `N8N_HOST` | Host for n8n |
| `N8N_PORT` | Port number |
| `N8N_PROTOCOL` | http or https |
| `WEBHOOK_URL` | Public webhook URL |
| `N8N_ENCRYPTION_KEY` | Data encryption key |
| `DB_TYPE` | Database type |
| `N8N_BASIC_AUTH_ACTIVE` | Enable basic auth |

### Scaling

**Queue Mode** for high volume:
- Redis for job queue
- Multiple worker processes
- Webhook processor

---

## REST API

### Authentication

```bash
# API Key in header
curl -X GET https://n8n.example.com/api/v1/workflows \
  -H "X-N8N-API-KEY: your-api-key"
```

### Common Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/workflows` | GET | List workflows |
| `/api/v1/workflows/{id}` | GET | Get workflow |
| `/api/v1/workflows` | POST | Create workflow |
| `/api/v1/workflows/{id}/activate` | POST | Activate workflow |
| `/api/v1/executions` | GET | List executions |
| `/api/v1/executions/{id}` | GET | Get execution |

---

## Best Practices

### Workflow Design

1. **Use descriptive names** for nodes and workflows
2. **Add sticky notes** to document complex logic
3. **Split large workflows** into sub-workflows
4. **Handle errors** with Error Trigger nodes
5. **Test thoroughly** before activating

### Performance

1. **Limit data volume** - Process only needed data
2. **Use pagination** for large datasets
3. **Implement rate limiting** for external APIs
4. **Cache frequently used data**
5. **Use queue mode** for high volume

### Security

1. **Use credentials** - Never hardcode secrets
2. **Enable authentication** on self-hosted instances
3. **Use HTTPS** for webhooks
4. **Restrict access** to sensitive workflows
5. **Audit execution logs** regularly

---

## Workflow JSON Generation

### JSON Structure

Every n8n workflow is a JSON object with this structure:

```json
{
  "name": "Workflow Name",
  "nodes": [
    {
      "parameters": { /* node-specific config */ },
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
        [{ "node": "Target Node Name", "type": "main", "index": 0 }]
      ]
    }
  },
  "settings": { "executionOrder": "v1" }
}
```

### Workflow Building Process

1. **Pick Trigger**: Choose appropriate trigger node (manual, webhook, schedule, chat)
2. **Add Logic**: Add flow control (If, Switch, Loop, Merge)
3. **Transform Data**: Use Set, Filter, Sort, Code nodes
4. **Connect Actions**: Add app/API nodes (HTTP Request, Database, etc.)
5. **Wire Connections**: Connect nodes via `connections` object
6. **Position Nodes**: Start at [250, 300], space ~250px horizontally

### Critical Rules for Generated JSON

- Always include `"settings": { "executionOrder": "v1" }`
- Use correct `type` identifiers (see `resources/core_concepts.md` for full list)
- Use correct `typeVersion` for each node
- Every `name` must be unique within the workflow
- Webhook nodes need a `webhookId` field
- AI sub-nodes use special connection types (`ai_languageModel`, `ai_memory`, `ai_tool`)

### $fromAI() Function

Use in AI Agent tool parameters to let the AI dynamically provide values:

```
{{ $fromAI('paramName', 'description of what this param is', 'string') }}
```

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Webhook not receiving data | Check URL, firewall, and webhook path |
| Expression not working | Verify syntax and data structure |
| Node timeout | Increase timeout or optimize operation |
| Memory issues | Reduce batch size, enable streaming |
| Authentication failed | Check credential configuration |

### Debugging Tips

1. **Check execution data** - View input/output at each node
2. **Use Set node** - Add data inspection points
3. **Enable debug logging** - Set LOG_LEVEL=debug
4. **Test manually** - Execute workflow step by step
5. **Check n8n community** - Search for similar issues

---

## Resources

- **Official Docs**: https://docs.n8n.io/
- **Workflow Templates**: https://n8n.io/workflows/
- **Community Forum**: https://community.n8n.io/
- **GitHub**: https://github.com/n8n-io/n8n
- **Discord**: https://discord.gg/n8n

---

## Quick Reference

### Essential Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Open node panel |
| `Ctrl/Cmd + S` | Save workflow |
| `Ctrl/Cmd + Enter` | Execute workflow |
| `Ctrl/Cmd + Z` | Undo |

### Common Patterns

```javascript
// Get all items from input
$input.all()

// Access current item
$json.fieldName

// Reference node output
$('NodeName').item.json

// Environment variable
$env.MY_VAR

// Current timestamp
$now.toISO()

// Inline conditional
$if(condition, trueVal, falseVal)

// JMESPath query
$jmespath($json, 'expression')
```
