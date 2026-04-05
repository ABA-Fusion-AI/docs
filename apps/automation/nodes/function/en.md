---
node_id: "function"
title: "Function"
description: "Execute custom JavaScript code for data transformation"
category: "utilities"
subcategory: "data"
version: "1.0.0"
language: "en"
last_updated: "2026-01-31"
author: "Fusion Team"
tags:
  - utility
  - function
  - javascript
  - code
  - transform
related_nodes:
  - filter
  - set-variable
---

<!-- SECTION: header -->
# Function

> **Category:** Utilities | **Type:** Action Node

Execute custom JavaScript code to transform, filter, or process data. The most flexible node for custom logic.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Function** node lets you write custom JavaScript code that executes on incoming data. Transform data structures, perform calculations, apply business logic, or create complex data manipulations.

### Key Features

- **Custom JavaScript:** Write any JavaScript logic
- **Sandboxed Execution:** Runs in isolated VM for security
- **Full Context Access:** Access variables, secrets, and previous outputs
- **Return Values:** Output becomes input for next node
- **Error Handling:** Caught errors flow to error output

### Use Cases

- Transform API response formats
- Calculate derived values
- Implement custom business logic
- Parse and restructure complex data
- Data validation and cleaning

### Available Context

| Variable | Type | Description |
|----------|------|-------------|
| `input` | `any` | Incoming data from previous node |
| `variables` | `object` | Workflow variables |
| `secrets` | `object` | Workflow secrets |
| `outputs` | `object` | Outputs from previous nodes |

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `code` | `string` | ❌ No | `return input;` | JavaScript code to execute |

### Code Guidelines

```javascript
// Your code has access to:
// - input: the incoming data
// - variables: workflow variables
// - secrets: workflow secrets  
// - outputs: previous node outputs

// Must return a value (becomes output)
return transformedData;
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Data to process |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `any` | Return value of the function |
| `error` | `Error` | Emitted on execution error |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Transform Object

Rename and restructure fields.

**Configuration:**
```javascript
// Transform user object
return {
  fullName: `${input.firstName} ${input.lastName}`,
  email: input.email.toLowerCase(),
  age: input.age,
  isAdult: input.age >= 18
};
```

**Input:**
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "JOHN@EXAMPLE.COM",
  "age": 25
}
```

**Output:**
```json
{
  "fullName": "John Doe",
  "email": "john@example.com",
  "age": 25,
  "isAdult": true
}
```

---

### Example: Filter and Map Array

Process array data.

**Configuration:**
```javascript
// Filter active users and extract emails
return input.users
  .filter(user => user.status === 'active')
  .map(user => ({
    id: user.id,
    email: user.email
  }));
```

---

### Example: Calculations

Perform mathematical operations.

**Configuration:**
```javascript
// Calculate order totals
const subtotal = input.items.reduce((sum, item) => 
  sum + (item.price * item.quantity), 0);
const tax = subtotal * 0.1;
const total = subtotal + tax;

return {
  subtotal: subtotal.toFixed(2),
  tax: tax.toFixed(2),
  total: total.toFixed(2),
  itemCount: input.items.length
};
```

---

### Example: Using Variables and Secrets

Access workflow context.

**Configuration:**
```javascript
// Use workflow variables and secrets
const apiUrl = variables.apiBaseUrl;
const authToken = secrets.apiToken;

return {
  url: `${apiUrl}/users/${input.userId}`,
  headers: {
    'Authorization': `Bearer ${authToken}`
  },
  data: input
};
```

---

### Example: Date Formatting

Work with dates.

**Configuration:**
```javascript
// Format dates
const date = new Date(input.timestamp);
return {
  ...input,
  formattedDate: date.toISOString().split('T')[0],
  formattedTime: date.toTimeString().split(' ')[0],
  dayOfWeek: ['Sun','Mon','Tue','Wed','Thu','Fri','Sat'][date.getDay()]
};
```

---

### Example: Data Validation

Validate and clean data.

**Configuration:**
```javascript
// Validate email format
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

if (!input.email || !emailRegex.test(input.email)) {
  throw new Error('Invalid email format');
}

return {
  ...input,
  email: input.email.toLowerCase().trim(),
  validated: true
};
```

---

### Example: Grouping Data

Group array items by property.

**Configuration:**
```javascript
// Group orders by status
const grouped = input.orders.reduce((acc, order) => {
  const status = order.status;
  if (!acc[status]) acc[status] = [];
  acc[status].push(order);
  return acc;
}, {});

return {
  grouped,
  summary: Object.keys(grouped).map(status => ({
    status,
    count: grouped[status].length
  }))
};
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: API Response Transformation

```json
{
  "nodes": [
    {
      "id": "fetch-data",
      "type": "http-request",
      "config": {
        "url": "https://api.example.com/orders",
        "method": "GET"
      }
    },
    {
      "id": "transform",
      "type": "function",
      "config": {
        "code": "return input.data.map(order => ({ id: order.id, total: order.amount, customer: order.customer_name }));"
      }
    },
    {
      "id": "store",
      "type": "google-sheets-append-row",
      "config": {
        "spreadsheetId": "xxx",
        "range": "Orders!A:C"
      }
    }
  ]
}
```

### Common Patterns

- **ETL Pipeline:** Fetch → Function (transform) → Store
- **Data Enrichment:** Get Base Data → Function (add calculated fields) → Use
- **Format Conversion:** Receive Data → Function (convert format) → Send
- **Aggregation:** Collect Items → Function (aggregate/summarize) → Report

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "undefined is not a function"

**Cause:** Calling method on null/undefined value.

**Solution:** Add null checks: `input?.items?.map()` or `if (input.items) { ... }`

#### Timeout error

**Cause:** Code execution exceeded time limit (2 seconds).

**Solution:** Optimize code, reduce data size, or split into multiple function nodes.

#### Memory limit exceeded

**Cause:** Processing too much data (>128MB).

**Solution:** Process data in smaller chunks or filter before processing.

#### Return value is undefined

**Cause:** Missing return statement.

**Solution:** Ensure your code ends with `return value;`

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `SyntaxError` | Invalid JavaScript | Fix code syntax |
| `ReferenceError` | Unknown variable | Check variable names |
| `TypeError` | Wrong data type | Add type validation |
| `TimeoutError` | Execution too long | Optimize or split code |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Filter](./filter.md) - Simple condition-based filtering
- [Set Variable](./set-variable.md) - Store values in workflow variables
- [Switch](./switch.md) - Conditional routing

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-31 | Initial release |

<!-- /SECTION: changelog -->
