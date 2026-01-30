# n8n Code and Expressions

This document covers coding capabilities in n8n including expressions and the Code node.

## Expressions Overview

Expressions allow dynamic data access and transformation within node parameters.

### Basic Syntax

Expressions are wrapped in double curly braces:

```javascript
{{ expression }}
```

### Accessing Data

```javascript
// Current item's JSON data
{{ $json }}
{{ $json.fieldName }}
{{ $json['field name'] }}

// Nested data
{{ $json.user.name }}
{{ $json.items[0].title }}

// Binary data
{{ $binary }}
{{ $binary.data }}
```

---

## Built-in Variables

### Item Data

| Variable | Description |
|----------|-------------|
| `$json` | Current item's JSON data |
| `$binary` | Current item's binary data |
| `$input` | Input data helpers |
| `$node` | Reference other nodes |

### Input Methods

```javascript
// All items from input
{{ $input.all() }}

// First input item
{{ $input.first() }}

// Last input item
{{ $input.last() }}

// Current item (in loop)
{{ $input.item }}

// Item by index
{{ $input.item(0) }}

// Item count
{{ $input.all().length }}
```

### Referencing Other Nodes

```javascript
// Node output by name
{{ $node["Node Name"].json }}
{{ $node["Node Name"].json.fieldName }}

// First item from node
{{ $node["Node Name"].first().json }}

// All items from node
{{ $node["Node Name"].all() }}
```

### Execution Context

```javascript
// Execution info
{{ $execution.id }}           // Execution ID
{{ $execution.mode }}         // manual, trigger, webhook
{{ $execution.resumeUrl }}    // Wait node resume URL

// Workflow info
{{ $workflow.id }}            // Workflow ID
{{ $workflow.name }}          // Workflow name
{{ $workflow.active }}        // Is workflow active

// Run info
{{ $runIndex }}               // Current run index
{{ $itemIndex }}              // Current item index
```

### Environment

```javascript
// Environment variables
{{ $env.MY_VARIABLE }}
{{ $env.API_KEY }}

// Workflow variables
{{ $vars.myVariable }}

// Timestamp
{{ $now }}                    // Current DateTime
{{ $today }}                  // Start of today
```

---

## Data Transformation Functions

### String Functions

```javascript
// Case conversion
{{ $json.name.toUpperCase() }}
{{ $json.name.toLowerCase() }}

// Substring
{{ $json.text.substring(0, 10) }}
{{ $json.text.slice(5) }}

// Replace
{{ $json.text.replace('old', 'new') }}
{{ $json.text.replaceAll('old', 'new') }}

// Split/Join
{{ $json.tags.split(',') }}
{{ $json.items.join(', ') }}

// Trim
{{ $json.text.trim() }}

// Includes
{{ $json.text.includes('search') }}

// Length
{{ $json.text.length }}
```

### Number Functions

```javascript
// Math operations
{{ $json.price * 1.1 }}
{{ Math.round($json.value) }}
{{ Math.floor($json.number) }}
{{ Math.ceil($json.number) }}

// Parsing
{{ parseInt($json.text) }}
{{ parseFloat($json.text) }}

// Formatting
{{ $json.price.toFixed(2) }}
```

### Array Functions

```javascript
// Length
{{ $json.items.length }}

// Access
{{ $json.items[0] }}
{{ $json.items.at(-1) }}

// Filter
{{ $json.items.filter(i => i.active) }}
{{ $json.items.filter(i => i.price > 100) }}

// Map
{{ $json.items.map(i => i.name) }}
{{ $json.items.map(i => ({ ...i, processed: true })) }}

// Find
{{ $json.items.find(i => i.id === 5) }}

// Reduce
{{ $json.items.reduce((sum, i) => sum + i.price, 0) }}

// Sort
{{ $json.items.sort((a, b) => a.price - b.price) }}

// Includes
{{ $json.tags.includes('featured') }}

// Join
{{ $json.items.map(i => i.name).join(', ') }}
```

### Object Functions

```javascript
// Keys and values
{{ Object.keys($json) }}
{{ Object.values($json) }}
{{ Object.entries($json) }}

// Spread
{{ { ...$json, newField: 'value' } }}

// Access
{{ $json.hasOwnProperty('field') }}
```

### Date Functions

```javascript
// Current time
{{ $now }}
{{ $now.toISO() }}              // ISO format
{{ $now.toFormat('yyyy-MM-dd') }}  // Custom format

// Today
{{ $today }}
{{ $today.toISODate() }}

// Date math
{{ $now.plus({ days: 7 }) }}
{{ $now.minus({ hours: 2 }) }}
{{ $now.startOf('month') }}
{{ $now.endOf('week') }}

// Parse date
{{ DateTime.fromISO($json.date) }}
{{ DateTime.fromFormat($json.date, 'dd/MM/yyyy') }}

// Comparison
{{ $now > DateTime.fromISO($json.deadline) }}
```

---

## Conditional Expressions

### Ternary Operator

```javascript
{{ $json.status === 'active' ? 'Yes' : 'No' }}
{{ $json.price > 100 ? 'Expensive' : 'Affordable' }}
```

### Nullish Coalescing

```javascript
{{ $json.name ?? 'Unknown' }}
{{ $json.user?.email ?? 'no-email@example.com' }}
```

### Optional Chaining

```javascript
{{ $json.user?.profile?.name }}
{{ $json.items?.[0]?.title }}
```

---

## Code Node

### Overview

The Code node allows JavaScript or Python execution for complex data manipulation.

### JavaScript Mode

```javascript
// Access input items
const items = $input.all();

// Process and return
const results = [];

for (const item of items) {
  results.push({
    json: {
      original: item.json,
      processed: true,
      timestamp: new Date().toISOString()
    }
  });
}

return results;
```

### Python Mode

```python
# Access input items
items = _input.all()

# Process and return
results = []

for item in items:
    results.append({
        "json": {
            "original": item.json,
            "processed": True,
            "timestamp": datetime.now().isoformat()
        }
    })

return results
```

### Execute Once Mode

Process all items together:

```javascript
// Execute once for all items
const allItems = $input.all();

// Aggregate example
const total = allItems.reduce((sum, item) => sum + item.json.amount, 0);

return [{
  json: {
    totalAmount: total,
    itemCount: allItems.length,
    average: total / allItems.length
  }
}];
```

### Execute Per Item Mode

Process items individually:

```javascript
// Runs for each item
const item = $input.item;

return {
  json: {
    ...item.json,
    processed: true
  }
};
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

### Error Handling

```javascript
try {
  const items = $input.all();
  // Processing logic
  return items;
} catch (error) {
  // Return error info
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
