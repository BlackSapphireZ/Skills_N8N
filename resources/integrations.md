# n8n Integrations

This document covers n8n's integration capabilities including built-in nodes, community nodes, and custom integrations.

## Built-in Nodes Overview

n8n includes 400+ built-in integrations organized by category.

### Productivity & Collaboration

| Node | Description |
|------|-------------|
| **Google Sheets** | Spreadsheet operations |
| **Notion** | Notes and databases |
| **Airtable** | Database management |
| **Slack** | Team messaging |
| **Microsoft Teams** | Team collaboration |
| **Discord** | Community messaging |
| **Trello** | Project boards |
| **Asana** | Task management |
| **Todoist** | To-do lists |

### CRM & Sales

| Node | Description |
|------|-------------|
| **Salesforce** | Enterprise CRM |
| **HubSpot** | Marketing & CRM |
| **Pipedrive** | Sales pipeline |
| **Zoho CRM** | CRM suite |
| **Freshsales** | Sales automation |
| **Close** | Sales platform |

### Marketing & Email

| Node | Description |
|------|-------------|
| **Mailchimp** | Email marketing |
| **SendGrid** | Email delivery |
| **ActiveCampaign** | Marketing automation |
| **Mailjet** | Email API |
| **Brevo** | Marketing platform |
| **ConvertKit** | Creator marketing |

### Development & DevOps

| Node | Description |
|------|-------------|
| **GitHub** | Code hosting |
| **GitLab** | DevOps platform |
| **Jira** | Issue tracking |
| **Bitbucket** | Code hosting |
| **Jenkins** | CI/CD |
| **CircleCI** | CI/CD |
| **Sentry** | Error tracking |

### Databases

| Node | Description |
|------|-------------|
| **PostgreSQL** | Relational DB |
| **MySQL** | Relational DB |
| **MongoDB** | Document DB |
| **Redis** | Key-value store |
| **Elasticsearch** | Search engine |
| **Supabase** | Backend as a service |
| **Firebase** | Google backend |

### Cloud Services

| Node | Description |
|------|-------------|
| **AWS S3** | Object storage |
| **AWS Lambda** | Serverless functions |
| **Google Cloud Storage** | Object storage |
| **Azure Blob Storage** | Object storage |
| **Dropbox** | File storage |
| **Google Drive** | File storage |

### Social Media

| Node | Description |
|------|-------------|
| **Twitter/X** | Social posting |
| **Facebook** | Social platform |
| **LinkedIn** | Professional network |
| **Instagram** | Photo sharing |
| **Telegram** | Messaging |

### E-commerce

| Node | Description |
|------|-------------|
| **Shopify** | E-commerce platform |
| **WooCommerce** | WordPress e-commerce |
| **Stripe** | Payments |
| **PayPal** | Payments |
| **Square** | Payments & POS |

### AI & Machine Learning

| Node | Description |
|------|-------------|
| **OpenAI** | GPT models |
| **Anthropic** | Claude models |
| **Google AI** | Gemini models |
| **Hugging Face** | ML models |
| **Pinecone** | Vector database |
| **Qdrant** | Vector search |

---

## Core Nodes

Essential nodes for any workflow.

### HTTP Request

Make API calls to any service:

```javascript
// Basic GET request
Method: GET
URL: https://api.example.com/data
Headers: {
  "Authorization": "Bearer {{ $env.API_KEY }}"
}

// POST with body
Method: POST
URL: https://api.example.com/create
Body Type: JSON
Body: {
  "name": "{{ $json.name }}",
  "email": "{{ $json.email }}"
}
```

**HTTP Request Options:**
- All HTTP methods (GET, POST, PUT, DELETE, PATCH)
- Custom headers
- Query parameters
- Authentication types
- Response formatting
- Pagination support
- Retry on failure

### Webhook

Create HTTP endpoints:

```javascript
// Configuration
HTTP Method: POST
Path: /my-webhook
Response Mode: Last Node
Response Data: First Entry JSON

// Authentication options
- None
- Basic Auth
- Header Auth
```

**Webhook URL:**
```
Production: https://your-n8n.com/webhook/my-webhook
Test: https://your-n8n.com/webhook-test/my-webhook
```

### Schedule Trigger

Time-based execution:

```javascript
// Cron expression
Trigger Times: Custom (Cron)
Cron Expression: 0 9 * * 1-5  // 9 AM weekdays

// Simple schedule
Trigger Times: Minutes
Minutes Between Executions: 30
```

### Set Node

Create or modify data:

```javascript
// Set values
Keep Only Set: false
Values:
  - Name: processedAt
    Value: {{ $now.toISO() }}
  - Name: status
    Value: processed
```

### Code Node

Execute JavaScript or Python:

```javascript
// JavaScript mode
const items = $input.all();
const results = items.map(item => ({
  json: {
    ...item.json,
    processed: true
  }
}));
return results;
```

### Function Node (Legacy)

```javascript
// Process items
return items.map(item => {
  item.json.newField = 'value';
  return item;
});
```

---

## Community Nodes

Install community-built integrations.

### Installing Community Nodes

**In n8n UI:**
1. Go to Settings → Community Nodes
2. Click "Install a community node"
3. Enter npm package name
4. Click Install

**Via npm:**
```bash
# In n8n installation directory
npm install n8n-nodes-package-name
```

**Via Docker:**
```dockerfile
FROM docker.n8n.io/n8nio/n8n
RUN npm install n8n-nodes-package-name
```

### Popular Community Nodes

| Package | Description |
|---------|-------------|
| `n8n-nodes-aws-bedrock` | AWS Bedrock AI |
| `n8n-nodes-google-analytics` | GA integration |
| `n8n-nodes-imap` | Email fetching |
| `n8n-nodes-puppeteer` | Browser automation |

### Security Considerations

- Community nodes are not officially verified
- Review code before installing
- Check update frequency
- Monitor for security issues

---

## Credential-Only Nodes

Nodes that provide authentication for HTTP Request:

### Using Credential-Only Nodes

1. Add HTTP Request node
2. Select service credential type
3. n8n handles authentication
4. Make direct API calls

### Available Credential-Only Types

Many services have credential-only support:
- Various API services
- Custom OAuth providers
- Enterprise applications

---

## Custom Integrations

### HTTP Request for Custom APIs

For unsupported services:

```javascript
// Example: Custom CRM API
Method: POST
URL: https://custom-crm.com/api/v1/contacts
Authentication: Header Auth
Headers: {
  "X-API-Key": "{{ $env.CRM_API_KEY }}",
  "Content-Type": "application/json"
}
Body: {
  "firstName": "{{ $json.firstName }}",
  "lastName": "{{ $json.lastName }}",
  "email": "{{ $json.email }}"
}
```

### Building Custom Nodes

Create your own nodes for:
- Internal APIs
- Proprietary systems
- Specialized integrations

See [Creating Nodes documentation](https://docs.n8n.io/integrations/creating-nodes/overview/)

---

## Credentials Management

### Supported Auth Types

| Type | Use Case |
|------|----------|
| **API Key** | Simple API access |
| **OAuth2** | User authorization |
| **Basic Auth** | Username/password |
| **Header Auth** | Custom headers |
| **Query Auth** | URL parameters |
| **Digest Auth** | HTTP Digest |

### OAuth2 Configuration

```javascript
// OAuth2 settings (example)
Grant Type: Authorization Code
Authorization URL: https://service.com/oauth/authorize
Access Token URL: https://service.com/oauth/token
Client ID: your-client-id
Client Secret: your-client-secret
Scope: read write
Auth URI Query Parameters: {}
```

### Credential Best Practices

1. **Never hardcode secrets** - Always use credentials
2. **Use environment variables** - For sensitive data
3. **Rotate regularly** - Update API keys periodically
4. **Limit permissions** - Principle of least privilege
5. **Audit usage** - Monitor credential access

---

## Integration Patterns

### Webhook → Action Pattern

```
Webhook → Process Data → Action (CRM/Email/DB)
```

### Scheduled Sync Pattern

```
Schedule → Fetch Source → Transform → Update Destination
```

### Event-Driven Pattern

```
Trigger (Email/Chat/Form) → Process → Multi-Action
```

### API Aggregation Pattern

```
Trigger → Parallel HTTP Requests → Merge → Output
```
