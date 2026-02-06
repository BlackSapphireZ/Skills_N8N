# n8n Code and Expressions

This document covers coding capabilities in n8n including expressions, the Code node, and advanced data manipulation.

## Data Structure

All data between nodes is an **array of items**. Each item has a `json` key and optional `binary` key:

```json
[
  {
    "json": {
      "name": "Alice",
      "email": "alice@example.com",
      "age": 30
    },
    "binary": {
      "data": {
        "mimeType": "image/png",
        "data": "base64string...",
        "fileName": "avatar.png"
      }
    }
  },
  {
    "json": {
      "name": "Bob",
      "email": "bob@example.com",
      "age": 25
    }
  }
]
```

**Key rules:**
- n8n auto-wraps in `json` key if missing (v0.166.0+)
- Each item is processed independently by most nodes
- Binary data (files) uses the `binary` key alongside `json`

---

## Expression Syntax

All expressions use double curly braces: `{{ expression }}`

### Accessing current item data
```
{{ $json.fieldName }}                    // Top-level field
{{ $json.nested.field }}                 // Nested (dot notation)
{{ $json['field with spaces'] }}         // Bracket notation
{{ $json.items[0].name }}                // Array index
{{ $json.field ?? 'default' }}           // Nullish coalescing
```

### Accessing data from other nodes
```
{{ $('Node Name').item.json.field }}     // Specific node's current item
{{ $('Node Name').first().json.field }}  // First item from node
{{ $('Node Name').last().json.field }}   // Last item from node
{{ $('Node Name').all() }}               // All items as array
{{ $('Node Name').all()[0].json.field }} // First item from all
{{ $('Node Name').item.json }}           // Full json of matched item
```

### Input helpers
```
{{ $input.item.json.field }}             // Current input item
{{ $input.first().json.field }}          // First input item
{{ $input.last().json.field }}           // Last input item
{{ $input.all() }}                       // All input items
{{ $input.all().length }}                // Count of input items
```

### Item index and metadata
```
{{ $itemIndex }}                         // Current item index (0-based)
{{ $runIndex }}                          // Current run index in loops
{{ $executionId }}                       // Workflow execution ID
{{ $workflow.id }}                       // Workflow ID
{{ $workflow.name }}                     // Workflow name
{{ $workflow.active }}                   // Is workflow active
```

---

## Global Variables

### Date/Time (Luxon)
```
{{ $now }}                               // Current DateTime (Luxon)
{{ $now.toISO() }}                       // ISO string
{{ $now.toFormat('yyyy-MM-dd') }}        // Custom format
{{ $now.minus({ days: 7 }).toISO() }}   // 7 days ago
{{ $now.plus({ hours: 2 }).toISO() }}   // 2 hours from now
{{ $now.hour }}                          // Current hour (0-23)
{{ $now.weekday }}                       // Day of week (1=Mon, 7=Sun)
{{ $today }}                             // Today at midnight
```

### Environment & Workflow
```
{{ $env.MY_VAR }}                        // Environment variable
{{ $vars.myVariable }}                   // n8n static variable
{{ $execution.id }}                      // Execution ID
{{ $execution.mode }}                    // 'test' or 'production'
{{ $execution.resumeUrl }}               // For Wait node resume
{{ $prevNode.name }}                     // Previous node's name
```

### Built-in Helpers
```
{{ $if(condition, trueVal, falseVal) }}  // Inline conditional
{{ $ifEmpty(value, defaultValue) }}      // Default if empty
{{ $jmespath(data, 'expression') }}      // JMESPath query
{{ Math.round($json.price * 100) / 100 }}// Standard JS Math
{{ JSON.stringify($json) }}              // Serialize to string
{{ JSON.parse($json.stringField) }}      // Parse string to object
{{ encodeURIComponent($json.query) }}    // URL encode
```

---

## Code Node

### Code Node JSON — Run Once for All Items
```json
{
  "parameters": {
    "jsCode": "const items = $input.all();\nconst results = items.map(item => {\n  return {\n    json: {\n      name: item.json.name.toUpperCase(),\n      processed: true\n    }\n  };\n});\nreturn results;",
    "mode": "runOnceForAllItems"
  },
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "name": "Code"
}
```

### Code Node JSON — Run Once for Each Item
```json
{
  "parameters": {
    "jsCode": "const name = $input.item.json.name;\nreturn {\n  json: {\n    greeting: `Hello, ${name}!`,\n    timestamp: new Date().toISOString()\n  }\n};",
    "mode": "runOnceForEachItem"
  },
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "name": "Code"
}
```

### Code Node JSON — Python Mode
```json
{
  "parameters": {
    "language": "python",
    "pythonCode": "results = []\nfor item in _input.all():\n    results.append({\n        'json': {\n            'name': item.json['name'].upper(),\n            'processed': True\n        }\n    })\nreturn results",
    "mode": "runOnceForAllItems"
  },
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "name": "Code"
}
```

### Code Node API Reference

**runOnceForAllItems (JS):**
- `$input.all()` → all items array
- `$input.first()` → first item
- `$input.last()` → last item
- Must return `Array<{ json: object, binary?: object }>`

**runOnceForEachItem (JS):**
- `$input.item` → current item
- `$input.item.json` → current item's json
- Must return `{ json: object, binary?: object }`

**Both modes:**
- `$('NodeName').all()` → items from another node
- `$env.VAR` → environment variable
- `$execution.id` → execution ID
- `$now` → Luxon DateTime
- `console.log()` → logs visible in execution data
- External npm modules NOT available (use HTTP Request for APIs)

---

## Data Transformation Functions

### String Functions

```javascript
{{ $json.name.toUpperCase() }}
{{ $json.name.toLowerCase() }}
{{ $json.name.trim() }}
{{ $json.text.replace(/\n/g, ' ') }}
{{ $json.email.split('@')[1] }}
{{ $json.name.includes('John') }}
{{ `Hello ${$json.name}, welcome!` }}
{{ $json.description.substring(0, 100) + '...' }}
{{ $json.tags.join(', ') }}
```

### Number Functions

```javascript
{{ Math.round($json.price * 100) / 100 }}
{{ parseInt($json.stringNumber) }}
{{ parseFloat($json.decimal) }}
{{ $json.total.toFixed(2) }}
{{ Math.max(...$input.all().map(i => i.json.score)) }}
```

### Array Functions

```javascript
{{ $json.items.length }}
{{ $json.items.map(i => i.name) }}
{{ $json.items.filter(i => i.active) }}
{{ $json.items.find(i => i.id === 123) }}
{{ $json.items.some(i => i.type === 'premium') }}
{{ $json.items.reduce((sum, i) => sum + i.price, 0) }}
{{ [...new Set($json.tags)] }}
{{ $json.items.sort((a, b) => a.price - b.price) }}
```

### Object Functions

```javascript
{{ Object.keys($json) }}
{{ Object.values($json.data) }}
{{ Object.entries($json).length }}
{{ { ...$json, newField: 'value' } }}
{{ JSON.stringify($json, null, 2) }}
```

### Date Functions (Luxon)

```javascript
{{ DateTime.fromISO($json.date).toFormat('dd/MM/yyyy') }}
{{ DateTime.fromISO($json.date).diff(DateTime.now(), 'days').days }}
{{ DateTime.fromMillis($json.timestamp).toISO() }}
{{ $now.startOf('day').toISO() }}
{{ $now.endOf('month').toISO() }}
{{ DateTime.fromFormat($json.date, 'dd/MM/yyyy') }}
{{ $now.setZone('Asia/Bangkok').toISO() }}
```

---

## Conditional Expressions

### Ternary & Defaults
```javascript
{{ $json.status === 'active' ? 'Yes' : 'No' }}
{{ $json.count > 0 ? $json.count + ' items' : 'empty' }}
{{ $json.email ?? 'no-email@example.com' }}
{{ $json.data?.nested?.field ?? 'default' }}
{{ $json.name || 'Unknown' }}
```

---

## JMESPath Queries

Use `$jmespath()` for complex JSON querying:

```
{{ $jmespath($json, 'orders[*].total') }}           // All order totals
{{ $jmespath($json, 'orders[?status==`active`]') }}  // Filter active
{{ $jmespath($json, 'users[*].{n: name, e: email}') }} // Project fields
{{ $jmespath($json, 'max_by(items, &price)') }}      // Max by price
{{ $jmespath($json, 'sort_by(items, &date)') }}      // Sort by date
{{ $jmespath($json, 'length(items)') }}              // Count
```

---

## Advanced Code Examples

### Data Transformation
```javascript
// Flatten nested array
const items = $input.all();
const flattened = items.flatMap(item => 
  item.json.orders.map(order => ({
    json: {
      customerId: item.json.id,
      customerName: item.json.name,
      ...order
    }
  }))
);
return flattened;
```

### API Integration in Code
```javascript
// Make HTTP request in Code node
const response = await this.helpers.httpRequest({
  method: 'GET',
  url: 'https://api.example.com/data',
  headers: {
    'Authorization': `Bearer ${$env.API_KEY}`
  }
});

return [{
  json: response.body
}];
```

### Working with Binary Data
```javascript
// Access binary data
const binaryData = $input.first().binary.data;

// Create binary output
const items = $input.all();
for (const item of items) {
  item.binary = {
    data: await this.helpers.prepareBinaryData(
      Buffer.from(item.json.content),
      'output.txt'
    )
  };
}
return items;
```

### Error Handling in Code
```javascript
try {
  const items = $input.all();
  // Processing logic
  return items;
} catch (error) {
  return [{
    json: {
      error: true,
      message: error.message
    }
  }];
}
```

---

## Expression Tips

### Performance
1. **Avoid complex expressions** - Use Code node for heavy logic
2. **Reference specific fields** - Not entire objects
3. **Cache repeated computations** - Store in variables

### Debugging
1. **Use Set node** - Display expression results
2. **Check execution data** - View actual values
3. **Test incrementally** - Build complex expressions step by step

### Common Patterns
```javascript
// Default values
{{ $json.name || 'Unknown' }}
{{ $json.count ?? 0 }}

// Conditional inclusion
{{ $json.active ? $json.email : '' }}

// Safe navigation
{{ $json.user?.address?.city ?? 'N/A' }}

// String interpolation
{{ `Hello, ${$json.name}!` }}

// Array to string
{{ $json.items.join(', ') }}

// Extract IDs
{{ $json.users.map(u => u.id) }}
```
