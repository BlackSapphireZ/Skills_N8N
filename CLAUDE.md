# CLAUDE.md - N8N Skills for AI Assistants

> AI-optimized documentation for understanding and using n8n workflow automation. Includes JSON workflow generation capabilities.

## Quick Reference

**What is n8n?**
- Fair-code licensed workflow automation platform
- Visual node-based editor
- 400+ built-in integrations
- AI/LangChain integration support
- Self-hosting or cloud options
- **All workflows are JSON** — can be generated, imported, and version-controlled

## Core Architecture

```
Workflow = Trigger Node → Action Nodes → Output
             │
          Connections (main + AI special types)
             │
          Expressions ({{ $json.field }})
```

## Workflow JSON Structure

Every n8n workflow is a JSON object:

```json
{
  "name": "Workflow Name",
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
    "Source Node": {
      "main": [[{ "node": "Target Node", "type": "main", "index": 0 }]]
    }
  },
  "settings": { "executionOrder": "v1" }
}
```

### Critical Rules for JSON Generation
- Always include `"settings": { "executionOrder": "v1" }`
- Use correct `type` identifiers (see resources/core_concepts.md)
- Every node `name` must be unique
- Webhook nodes need `webhookId`
- AI sub-nodes use special connection types (`ai_languageModel`, `ai_memory`, `ai_tool`)
- Start positioning at [250, 300], space ~250px horizontally

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Workflow** | Collection of connected nodes (JSON) |
| **Node** | Building block (trigger, action, core, AI) |
| **Trigger** | Starts workflow (webhook, schedule, event) |
| **Action** | Performs operation (API call, DB, email) |
| **Expression** | Dynamic data access `{{ $json.field }}` |
| **Execution** | Single workflow run record |
| **Credential** | Stored authentication |
| **$fromAI()** | Let AI Agent fill parameter values |

## Node Type Quick Reference (with JSON type identifiers)

### Triggers
| Node | Type Identifier |
|------|----------------|
| Manual | `n8n-nodes-base.manualWorkflowTrigger` |
| Webhook | `n8n-nodes-base.webhook` |
| Schedule | `n8n-nodes-base.scheduleTrigger` |
| Chat Trigger | `@n8n/n8n-nodes-langchain.chatTrigger` |

### Core / Flow Logic
| Node | Type Identifier |
|------|----------------|
| If | `n8n-nodes-base.if` |
| Switch | `n8n-nodes-base.switch` |
| Merge | `n8n-nodes-base.merge` |
| Loop Over Items | `n8n-nodes-base.splitInBatches` |
| Wait | `n8n-nodes-base.wait` |
| Execute Sub-Workflow | `n8n-nodes-base.executeWorkflow` |
| Code | `n8n-nodes-base.code` |
| HTTP Request | `n8n-nodes-base.httpRequest` |
| Set / Edit Fields | `n8n-nodes-base.set` |
| Filter | `n8n-nodes-base.filter` |
| Sort | `n8n-nodes-base.sort` |
| Respond to Webhook | `n8n-nodes-base.respondToWebhook` |

### AI Nodes
| Node | Type Identifier |
|------|----------------|
| AI Agent | `@n8n/n8n-nodes-langchain.agent` |
| OpenAI Chat Model | `@n8n/n8n-nodes-langchain.lmChatOpenAi` |
| Anthropic Chat Model | `@n8n/n8n-nodes-langchain.lmChatAnthropic` |
| Google Gemini | `@n8n/n8n-nodes-langchain.lmChatGoogleGemini` |
| Ollama | `@n8n/n8n-nodes-langchain.lmChatOllama` |
| Buffer Memory | `@n8n/n8n-nodes-langchain.memoryBufferWindow` |
| Postgres Memory | `@n8n/n8n-nodes-langchain.memoryPostgresChat` |
| Workflow Tool | `@n8n/n8n-nodes-langchain.toolWorkflow` |
| HTTP Request Tool | `@n8n/n8n-nodes-langchain.toolHttpRequest` |
| Code Tool | `@n8n/n8n-nodes-langchain.toolCode` |
| MCP Client | `@n8n/n8n-nodes-langchain.mcpClient` |

### AI Connection Types
| Type | Purpose |
|------|---------|
| `main` | Standard data flow |
| `ai_languageModel` | Chat/LLM model |
| `ai_memory` | Conversation memory |
| `ai_tool` | Agent tools |
| `ai_outputParser` | Response formatting |
| `ai_retriever` | RAG retriever |
| `ai_document` | Document input |
| `ai_embedding` | Embedding model |

## Essential Expression Patterns

```javascript
// Current item
{{ $json.fieldName }}
{{ $json.nested?.field ?? 'default' }}

// Other node's data
{{ $('Node Name').item.json.field }}
{{ $('Node Name').first().json.field }}
{{ $('Node Name').all() }}

// Environment
{{ $env.API_KEY }}
{{ $vars.myVariable }}
{{ $now.toISO() }}
{{ $now.toFormat('yyyy-MM-dd') }}

// Helpers
{{ $if(condition, trueVal, falseVal) }}
{{ $ifEmpty(value, defaultValue) }}
{{ $jmespath($json, 'expression') }}

// $fromAI() - for AI Agent tool params
{{ $fromAI('paramName', 'description', 'string') }}
```

## Common Workflow Patterns

### 1. Webhook → Process → Respond
```
Webhook → Validate → Process → Respond to Webhook
```

### 2. Schedule → ETL with Error Handling
```
Schedule → Fetch API → Transform → Filter → Load DB
```

### 3. AI Chat Agent with Tools
```
Chat Trigger → AI Agent → Response
                  ↑
             Chat Model + Memory + Tools
```

### 4. Loop with Rate Limiting
```
Trigger → Loop Over Items → Process → Wait (1s) → Back to Loop
                          ↓ (done)
                        Summary
```

## Data Structure

n8n data format:
```json
[
  { "json": { "field": "value" }, "binary": {} }
]
```

## File Structure

```
Skills_N8N/
├── SKILL.md              # Main skill file (comprehensive)
├── CLAUDE.md             # This file (AI quick reference)
├── README.md             # Thai documentation
└── resources/
    ├── core_concepts.md        # Nodes, triggers, flow logic (with JSON configs)
    ├── integrations.md         # 400+ integrations, credentials
    ├── advanced_ai.md          # AI/LangChain (with JSON configs)
    ├── code_and_expressions.md # Code node, expressions, JMESPath
    ├── hosting_and_deployment.md # Docker, scaling, env vars
    ├── api_reference.md        # REST API endpoints
    ├── workflow_patterns.md    # Patterns + importable workflow JSON
    └── troubleshooting.md      # Common issues & solutions
```

## When to Use This Skill

Use when user asks about:
- **Creating n8n workflows** (generate importable JSON)
- **Node configuration** (exact type identifiers and params)
- **AI Agent workflows** (LangChain, Chat Trigger, tools)
- **Expression syntax** ($json, $fromAI, $jmespath)
- **API integrations** (HTTP Request, webhooks)
- **Self-hosting n8n** (Docker, scaling, env vars)
- **Debugging workflows** (troubleshooting, common errors)

## Resources

- **Docs**: https://docs.n8n.io/
- **Templates**: https://n8n.io/workflows/
- **Community**: https://community.n8n.io/
