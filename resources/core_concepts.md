# n8n Core Concepts

This document covers the fundamental concepts of n8n workflow automation.

## Workflows

A **workflow** is a collection of nodes connected together to automate a process.

### Workflow Lifecycle

```
Create → Configure → Test → Activate → Monitor
```

### Creating Workflows

1. **New Workflow**: Click "New Workflow" button
2. **Add Trigger**: Start with a trigger node
3. **Add Actions**: Connect action nodes
4. **Configure**: Set parameters for each node
5. **Test**: Execute manually to verify
6. **Activate**: Enable for automatic execution

### Workflow States

| State | Description |
|-------|-------------|
| **Draft** | Work in progress, not executing |
| **Active** | Running on triggers/schedule |
| **Inactive** | Saved but not executing |
| **Error** | Failed, needs attention |

### Workflow Templates

Pre-built workflows for common use cases:
- Access via Templates button in editor
- Categories: Marketing, Sales, IT, Finance
- Import and customize as needed
- Share your own templates

---

## Nodes

Nodes are the building blocks of workflows.

### Node Categories

#### Trigger Nodes
Start workflow execution:

| Node | Purpose |
|------|---------|
| **Webhook** | HTTP endpoint trigger |
| **Schedule** | Time-based trigger |
| **Email Trigger** | Email received trigger |
| **Chat Trigger** | Chat message trigger |
| **Manual Trigger** | Manual execution |

#### Action Nodes
Perform operations on data:

| Node | Purpose |
|------|---------|
| **HTTP Request** | API calls |
| **Set** | Set/modify data |
| **Function** | Custom JavaScript |
| **Send Email** | Email sending |
| **Database** | DB operations |

#### Core Nodes
Control workflow flow:

| Node | Purpose |
|------|---------|
| **IF** | Conditional branching |
| **Switch** | Multiple conditions |
| **Merge** | Combine data streams |
| **Split In Batches** | Process in chunks |
| **Wait** | Pause execution |

### Node Configuration

Each node has:
- **Parameters**: Input configuration
- **Credentials**: Authentication (if needed)
- **Settings**: Execution options
- **Notes**: Documentation

### Node Execution Order

1. Nodes execute left to right
2. Parallel branches execute simultaneously
3. Merge nodes wait for all inputs
4. Loops iterate through items

---

## Connections

Connections define data flow between nodes.

### Connection Types

#### Main Connections (Solid Lines)
- Regular data flow
- JSON data transfer
- Support multiple outputs

#### AI Connections (Dashed Lines)
- Connect sub-nodes to root nodes
- Used in AI workflows
- LangChain components

### Creating Connections

1. Drag from node output (right side)
2. Drop on node input (left side)
3. Multiple connections allowed
4. Delete by clicking and pressing Delete

### Data Flow

```
Node A (output) → [items array] → Node B (input)
```

Data flows as arrays of items:
```json
[
  { "json": { "id": 1, "name": "Item 1" } },
  { "json": { "id": 2, "name": "Item 2" } }
]
```

---

## Executions

Executions are records of workflow runs.

### Execution Data

Each execution contains:
- **ID**: Unique identifier
- **Status**: success, error, running
- **Started At**: Timestamp
- **Finished At**: Timestamp
- **Mode**: manual, trigger, webhook
- **Data**: Input/output of each node

### Viewing Executions

1. Click "Executions" tab
2. Filter by status, date, workflow
3. Click execution to view details
4. See data at each node step

### Execution States

| State | Description |
|-------|-------------|
| **Success** | Completed without errors |
| **Error** | Failed with error |
| **Running** | Currently executing |
| **Waiting** | Paused (Wait node) |
| **Canceled** | Manually stopped |

### Data Pinning

Pin data for testing:
1. Execute workflow
2. Click pin icon on node output
3. Pinned data used in subsequent tests
4. Unpin to use live data

### Retry Executions

Retry failed executions:
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

### Creating Credentials

1. Go to Settings → Credentials
2. Click "Add Credential"
3. Select credential type
4. Fill in required fields
5. Test connection
6. Save

### Using Credentials

1. Select node requiring auth
2. Click "Credential" dropdown
3. Choose existing or create new
4. Credential stored securely

### Credential Sharing

- Share credentials with team members
- Role-based access control
- Audit credential usage

---

## Variables

### Workflow Variables

Store values used across workflow:

```javascript
// Access workflow variable
{{ $vars.myVariable }}
```

### Environment Variables

Server-level configuration:

```javascript
// Access environment variable
{{ $env.API_KEY }}
```

### Setting Variables

**Workflow Variables:**
1. Open workflow settings
2. Go to Variables tab
3. Add key-value pairs

**Environment Variables:**
```bash
# Docker
-e MY_VAR=value

# .env file
MY_VAR=value
```

---

## Tags and Organization

### Workflow Tags

Organize workflows with tags:
- Create descriptive tags
- Filter workflows by tag
- Group related workflows

### Folder Structure (Enterprise)

Organize in folders:
- Team-based organization
- Project separation
- Access control

### Naming Conventions

Best practices:
- Use descriptive names
- Include purpose/trigger
- Version if needed

Examples:
- `Slack - New Customer Notification`
- `Daily Sales Report - v2`
- `Webhook - Order Processing`
