# n8n Workflow Patterns

This document covers common workflow patterns and best practices for building automations in n8n.

## Flow Logic Patterns

### Conditional Branching

#### IF Node Pattern

Binary true/false routing:

```
Input → IF → True branch → Action A
           → False branch → Action B
```

**Configuration:**
```javascript
// IF conditions
Value 1: {{ $json.status }}
Operation: equals
Value 2: active
```

#### Switch Node Pattern

Multiple branch routing:

```
Input → Switch → Case: new → Create action
              → Case: update → Update action
              → Case: delete → Delete action
              → Default → Log/Error
```

**Configuration:**
```javascript
// Switch on field
Mode: Rules
Routing Rules:
  - Output 0: {{ $json.action }} equals "new"
  - Output 1: {{ $json.action }} equals "update"
  - Output 2: {{ $json.action }} equals "delete"
```

---

### Merging Data

#### Append Merge

Combine all items from multiple sources:

```
Source A → Merge (Append) → Combined output
Source B ↗
```

#### Key-Based Merge

Join data by matching field:

```
Customers → Merge (Combine) → Enriched data
Orders    ↗   (by customer_id)
```

**Configuration:**
```javascript
Mode: Combine
Combination Mode: Merge By Key
Property to Match On:
  - Input 1: customerId
  - Input 2: customer_id
```

#### Choose Branch

Select one path based on condition:

```
Source A → Merge (Choose Branch) → Selected output
Source B ↗
```

---

### Looping Patterns

#### Process Items in Batches

```
Get All Items → Loop Over Items → Process Batch → Next Batch
                      ↑__________________________|
```

**Configuration:**
```javascript
Batch Size: 10
Options:
  Reset: false
```

#### Retry Pattern

```
Action → IF (success?) → Continue
              ↓
          Wait → Retry (loop back)
              ↓
          Max retries → Error
```

#### While Loop Pattern

```
Start → Set Counter → Action → IF (continue?)
                         ↑         ↓
                         └── Increment Counter
                               ↓ (no)
                              End
```

---

### Sub-Workflow Patterns

#### Modular Workflows

Main workflow calls reusable sub-workflows:

```
Trigger → Get Data → Execute Workflow (Process) → Output
                           ↓
              [Sub-workflow handles processing]
```

**Execute Workflow Node:**
```javascript
Workflow: Process Data Workflow
Mode: Execute in Current Workflow
```

#### Parent-Child Pattern

```
Parent Workflow
    │
    ├── Execute Workflow (Child A)
    │         ↓
    │   [Child A Workflow]
    │
    └── Execute Workflow (Child B)
              ↓
        [Child B Workflow]
```

---

### Error Handling Patterns

#### Try-Catch Pattern

```
Main Flow → Action → Success path
              ↓ (on error)
         Error Handler → Notification/Log
```

**Configuration:**
- Enable "Continue On Fail" on action nodes
- Check for errors in subsequent IF node

#### Error Trigger Pattern

```
[Any workflow error]
        ↓
Error Trigger → Format Error → Send Notification
```

#### Graceful Degradation

```
Primary API → IF (failed?) → Fallback API → Continue
                   ↓ (no)
              Continue with primary
```

---

## Data Transformation Patterns

### Flatten Nested Data

```javascript
// Code node
const items = $input.all();
const flattened = [];

for (const item of items) {
  for (const order of item.json.orders) {
    flattened.push({
      json: {
        customerId: item.json.id,
        customerName: item.json.name,
        orderId: order.id,
        orderTotal: order.total
      }
    });
  }
}

return flattened;
```

### Aggregate Data

```javascript
// Summarize node or Code node
const items = $input.all();

const summary = {
  totalItems: items.length,
  totalAmount: items.reduce((sum, i) => sum + i.json.amount, 0),
  averageAmount: 0
};
summary.averageAmount = summary.totalAmount / summary.totalItems;

return [{ json: summary }];
```

### Data Enrichment

```
Base Data → HTTP Request (get details) → Merge → Enriched Data
      └─────────────────────────────────────↗
```

### Filter and Clean

```
Raw Data → Remove Duplicates → Filter (valid only) → Clean Data
```

---

## Integration Patterns

### Webhook → Process → Response

```
Webhook → Validate → Process → Format Response
   └──────────────────────────────↲ (Response)
```

**Webhook Configuration:**
```javascript
Response Mode: Using 'Respond to Webhook' node
```

### Scheduled Sync

```
Schedule → Fetch Source → Transform → Upsert Destination
   (hourly)    ↓
          Compare with last run
```

### Event-Driven Updates

```
Webhook (event) → IF (event type?)
                    → Created: Create in system
                    → Updated: Update in system
                    → Deleted: Remove from system
```

### API Aggregation

```
Trigger → HTTP (API 1) → 
        → HTTP (API 2) → Merge → Transform → Output
        → HTTP (API 3) →
```

---

## Best Practice Patterns

### Idempotent Operations

Ensure operations can be safely retried:

```javascript
// Check if already processed
const existing = await getFromDB(item.json.id);
if (existing) {
  return [{ json: { skipped: true, reason: 'Already processed' }}];
}
// Proceed with processing
```

### Rate Limiting

Prevent API overload:

```
Items → Loop Over Items → Wait → API Call → Collect Results
           (batch: 10)   (1s)
```

### Logging Pattern

Track workflow execution:

```
Start → Log Start → Main Process → Log End
            ↓                 ↓ (error)
       [activity log]    Log Error
```

### Validation Pattern

```
Input → Validate → IF (valid?)
                     → Yes: Process
                     → No: Return Error
```

---

## Common Workflow Examples

### Contact Sync

```
Schedule (daily)
    ↓
Get CRM Contacts
    ↓
Transform Data
    ↓
Upsert to Marketing Platform
    ↓
Log Results
```

### Order Processing

```
Webhook (new order)
    ↓
Validate Order
    ↓
Check Inventory
    ↓
IF (in stock?)
    → Yes: Create Order → Send Confirmation
    → No: Add to Waitlist → Send Backorder Notice
```

### Lead Scoring

```
New Lead Trigger
    ↓
Enrich Data (Clearbit)
    ↓
Calculate Score (Code node)
    ↓
IF (score > threshold?)
    → High: Assign to Sales → Slack Notification
    → Low: Add to Nurture Campaign
```

### Document Processing

```
File Upload Webhook
    ↓
Extract Text (PDF/OCR)
    ↓
AI Summarization
    ↓
Store in Database
    ↓
Notify User
```

### Multi-Channel Notification

```
Event Trigger
    ↓
Format Message
    ↓
├── Send Email
├── Send Slack
├── Send SMS
└── Log to Database
```

---

## Performance Tips

1. **Minimize data early** - Filter before processing
2. **Use batch operations** - Bulk updates vs individual
3. **Parallel when possible** - Independent operations
4. **Cache lookups** - Store frequently used data
5. **Limit API calls** - Aggregate requests
