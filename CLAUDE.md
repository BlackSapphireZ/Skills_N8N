# CLAUDE.md - N8N Skills for AI Assistants

> AI-optimized documentation for understanding and using n8n workflow automation.

## Quick Reference

**What is n8n?**
- Fair-code licensed workflow automation platform
- Visual node-based editor
- 400+ built-in integrations
- AI/LangChain integration support
- Self-hosting or cloud options

## Core Architecture

```
Workflow = Trigger Node → Action Nodes → Output
             │
          Connections (data flow)
             │
          Expressions (data transformation)
```

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Workflow** | Collection of connected nodes |
| **Node** | Building block (trigger, action, core) |
| **Trigger** | Starts workflow (webhook, schedule, event) |
| **Action** | Performs operation (API call, DB, email) |
| **Expression** | Dynamic data access `{{ $json.field }}` |
| **Execution** | Single workflow run record |
| **Credential** | Stored authentication |

## Essential Expression Patterns

```javascript
// Access data
{{ $json.fieldName }}           // Current item
{{ $input.first().json }}       // First input
{{ $node["NodeName"].json }}    // Other node output

// Environment
{{ $env.API_KEY }}              // Env variable
{{ $now.toISO() }}              // Current time

// Transform
{{ $json.items.filter(i => i.active) }}
{{ $json.name.toUpperCase() }}

// Defaults
{{ $json.value ?? 0 }}
{{ $json.user?.email ?? 'none' }}
```

## Node Type Quick Reference

### Triggers
- `Webhook` - HTTP endpoint
- `Schedule` - Cron/interval
- `Chat Trigger` - AI chat
- `Manual Trigger` - Test execution

### Core Nodes
- `IF` - Conditional branch
- `Switch` - Multi-condition
- `Merge` - Combine data
- `Code` - JavaScript/Python
- `HTTP Request` - API calls
- `Set` - Modify data

### AI Nodes
- `AI Agent` - Autonomous agent
- `Basic LLM Chain` - Simple prompt
- `Vector Store` - RAG retrieval
- `Chat Model` - LLM connection

## Common Workflow Patterns

### 1. Webhook → Process → Respond
```
Webhook → Validate → Process → Respond to Webhook
```

### 2. Schedule → Sync
```
Schedule → Fetch API → Transform → Update Database
```

### 3. AI Chat
```
Chat Trigger → AI Agent → Response
                  ↑
             Chat Model + Tools
```

### 4. Event → Multi-Action
```
Trigger → IF (type?)
           → Create: Create record
           → Update: Update record
           → Delete: Delete record
```

## Data Structure

n8n data format:
```json
[
  { "json": { "field": "value" }, "binary": {} },
  { "json": { "field": "value2" }, "binary": {} }
]
```

## Hosting Options

| Option | Command/Method |
|--------|----------------|
| Docker | `docker run -p 5678:5678 docker.n8n.io/n8nio/n8n` |
| npm | `npm install n8n -g && n8n start` |
| Cloud | https://app.n8n.cloud |

## Key Environment Variables

```bash
N8N_HOST=localhost
N8N_PORT=5678
N8N_PROTOCOL=https
WEBHOOK_URL=https://domain.com/
N8N_ENCRYPTION_KEY=32-char-key
DB_TYPE=postgresdb
```

## API Quick Reference

```bash
# List workflows
GET /api/v1/workflows
Header: X-N8N-API-KEY: your-key

# Activate workflow
POST /api/v1/workflows/{id}/activate

# Execute workflow
POST /api/v1/workflows/{id}/execute
```

## Troubleshooting Quick Fixes

| Issue | Solution |
|-------|----------|
| Webhook not working | Check active + production URL |
| Expression undefined | Use optional chaining `?.` |
| Timeout | Increase `EXECUTIONS_TIMEOUT` |
| Rate limited | Add Wait node between calls |

## File Structure

```
Skills_N8N/
├── SKILL.md              # Main skill file
├── CLAUDE.md             # This file (AI reference)
├── README.md             # Thai documentation
└── resources/
    ├── core_concepts.md
    ├── integrations.md
    ├── advanced_ai.md
    ├── code_and_expressions.md
    ├── hosting_and_deployment.md
    ├── api_reference.md
    ├── workflow_patterns.md
    └── troubleshooting.md
```

## When to Use This Skill

Use when user asks about:
- Creating n8n workflows
- Node configuration
- Expression syntax
- API integrations
- Self-hosting n8n
- AI/LangChain in n8n
- Debugging workflows

## Resources

- **Docs**: https://docs.n8n.io/
- **Templates**: https://n8n.io/workflows/
- **Community**: https://community.n8n.io/
