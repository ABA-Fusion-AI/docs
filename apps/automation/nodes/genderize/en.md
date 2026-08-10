---

node_id: "genderize"
title: "Genderize.io"
description: "Predict gender based on a name using the Genderize.io API."
category: "API"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - api
  - genderize
  - gender
  - name
  - prediction
  - gender-prediction
  - data-enrichment
  - action
related_nodes:
  - function
  - if
  - http-request

---

# Genderize.io

> **Category:** api-nodes | **Type:** Action Node

Predict the gender statistically associated with a name using the **Genderize.io API**.

The **Genderize.io** node accepts a name through its configuration or workflow input, sends the name to the Genderize.io API, and returns the predicted gender, probability, and number of data points supporting the prediction.

The node uses the Genderize.io API endpoint directly and does not require an API key in its current implementation.

> **Important:** Genderize.io provides a statistical prediction based on name data. The result should be treated as a prediction and not as definitive information about an individual's gender identity.

### Supported Features

- Predict gender from a name
- Accept a name through node configuration
- Accept a name from workflow input
- Use configured name before workflow input
- URL-encode names before sending API requests
- Return predicted gender
- Return prediction probability
- Return supporting data count
- Handle API errors
- Validate that a name is provided
- Return structured workflow data

### Use Cases

- Enrich customer or contact data
- Analyze lists of names
- Build name-based data enrichment workflows
- Personalize workflow processing
- Create demographic analysis workflows
- Route workflow data using an `If` node
- Process prediction results using a `Function` node
- Connect name data to downstream workflow actions

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `name` | `string` | ❌ No | `""` | Name to predict gender for. |

The `name` parameter is optional because the node can also receive a name through workflow input.

### Example Configuration

```json
{
  "name": "Sarah"
}
```

---

## Inputs & Outputs

### Inputs

The node can receive the name from two sources:

1. The `name` configuration parameter.
2. Workflow input data.

The configured `name` takes priority when provided.

If no configured name is provided and the workflow input is a string, the input string is used as the name.

### Input Example

```text
Sarah
```

The node sends `Sarah` to the Genderize.io API.

### Input Priority

The node uses the following priority:

```text
Configured name
      ↓
Workflow input string
      ↓
Name validation
```

If neither source contains a valid name, the node throws:

```text
Name is required
```

---

## Outputs

The node returns a structured object containing the Genderize.io prediction.

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Indicates that the API request succeeded. |
| `name` | `string` | Name analyzed by Genderize.io. |
| `gender` | `string \| null` | Predicted gender returned by the API. |
| `probability` | `number \| null` | Statistical probability associated with the prediction. |
| `count` | `number` | Number of data points associated with the prediction. |

### Output Example

```json
{
  "success": true,
  "name": "Sarah",
  "gender": "female",
  "probability": 0.99,
  "count": 123456
}
```

The exact `probability` and `count` values depend on the data returned by Genderize.io.

---

## API Request

The node sends a `GET` request to the Genderize.io API:

```text
https://api.genderize.io
```

The name is passed using the `name` query parameter.

For example:

```text
https://api.genderize.io?name=Sarah
```

The node uses `encodeURIComponent()` to safely encode the supplied name before adding it to the request URL.

For example:

```text
John Smith
```

is automatically URL-encoded before being sent to the API.

---

## Gender Prediction

The node uses the following request format:

```text
GET https://api.genderize.io?name=<name>
```

### Example Request

```text
https://api.genderize.io?name=Michael
```

### Example API Response

```json
{
  "name": "Michael",
  "gender": "male",
  "probability": 0.99,
  "count": 250000
}
```

### Node Output

The node converts the API response into:

```json
{
  "success": true,
  "name": "Michael",
  "gender": "male",
  "probability": 0.99,
  "count": 250000
}
```

---

## Probability

The `probability` field represents the statistical probability associated with the returned prediction.

For example:

```json
{
  "gender": "female",
  "probability": 0.98
}
```

A workflow can use the probability value to apply its own confidence threshold.

For example:

```text
probability >= 0.90
```

can be used in an `If` node to continue processing only predictions above a selected confidence level.

---

## Count

The `count` field represents the number of data points associated with the prediction.

Example:

```json
{
  "gender": "male",
  "probability": 0.97,
  "count": 54321
}
```

The count is returned directly from the Genderize.io API.

---

## Unknown Names

Genderize.io may not be able to make a prediction for every name.

For names where insufficient data is available, the API may return:

```json
{
  "name": "UnknownName",
  "gender": null,
  "probability": null,
  "count": 0
}
```

The node returns the result without treating it as an API error:

```json
{
  "success": true,
  "name": "UnknownName",
  "gender": null,
  "probability": null,
  "count": 0
}
```

A `null` gender indicates that no prediction was available.

---

## Input Validation

The node validates the name before making the API request.

An empty configured name:

```json
{
  "name": ""
}
```

results in:

```text
Name is required
```

The same validation is applied when the workflow input does not contain a usable name.

Whitespace-only names are also rejected.

---

## Workflow Integration

### Sample Workflow: Predict Gender

```json
{
  "nodes": [
    {
      "id": "genderize",
      "type": "genderize",
      "config": {
        "name": "Sarah"
      }
    }
  ]
}
```

### Sample Workflow: Input → Genderize

The node can receive a name from an upstream workflow node.

Example input:

```text
Sarah
```

Node configuration:

```json
{
  "config": {}
}
```

The node uses the incoming `Sarah` value because no configured name was provided.

### Sample Workflow: Genderize → If

```json
{
  "nodes": [
    {
      "id": "genderize",
      "type": "genderize",
      "config": {
        "name": "Sarah"
      }
    },
    {
      "id": "check-gender",
      "type": "if"
    }
  ]
}
```

The `If` node can inspect:

```text
gender
```

or:

```text
probability
```

to determine which workflow branch should execute.

### Sample Workflow: Genderize → Function

```json
{
  "nodes": [
    {
      "id": "genderize",
      "type": "genderize",
      "config": {
        "name": "Sarah"
      }
    },
    {
      "id": "process-result",
      "type": "function"
    }
  ]
}
```

The `Function` node can process fields such as:

```text
name
gender
probability
count
```

### Common Patterns

- Input → Genderize.io → If
- Input → Genderize.io → Function
- Database → Genderize.io → Database
- CSV → Genderize.io → Function
- Name → Genderize.io → Notification
- Contact Data → Genderize.io → Data Processing

---

## Error Handling

If the Genderize.io API returns a non-success HTTP status, the node throws an error.

The initial API error uses the following format:

```text
Genderize.io API error: <status>
```

The node then wraps the error as:

```text
Genderize.io request failed: <error message>
```

### Example

If the API returns HTTP status `429`, the resulting error is:

```text
Genderize.io request failed: Genderize.io API error: 429
```

The error is thrown so that the workflow execution can handle the failed request using the platform's error-handling behavior.

---

## Troubleshooting

### Name Is Required

**Cause**

No name was provided through the configuration or workflow input.

**Solution**

Provide a name:

```json
{
  "name": "Sarah"
}
```

Alternatively, provide a string through workflow input.

---

### API Request Failed

**Cause**

Genderize.io returned a non-success HTTP status.

**Solution**

Check the HTTP status returned by the API and verify that the Genderize.io service is available.

The node includes the HTTP status in the error message.

---

### No Gender Prediction

**Cause**

Genderize.io does not have sufficient data for the supplied name.

**Solution**

Check the returned `gender` field.

A `null` value is a valid API response and means that no prediction was available.

Example:

```json
{
  "success": true,
  "name": "UnknownName",
  "gender": null,
  "probability": null,
  "count": 0
}
```

---

### Low Probability

**Cause**

The supplied name has a weaker statistical association with the returned prediction.

**Solution**

Inspect the `probability` field before using the prediction in an automated workflow.

For example:

```text
probability >= 0.90
```

can be used as a confidence threshold.

---

## Privacy Considerations

The node sends the supplied name to the Genderize.io API.

Only provide the information necessary for the prediction.

The returned value is a statistical prediction associated with the supplied name and should not be interpreted as verified information about a person's gender identity.

---

## API Endpoint

The Genderize.io endpoint used by this node is:

```text
https://api.genderize.io
```

The endpoint is defined directly in the node implementation and is not exposed as a configuration parameter.

---

## Implementation Behavior

The node performs the following steps:

1. Reads the configured `name`.
2. Checks whether a configured name was provided.
3. If no configured name exists, checks the workflow input.
4. Converts string workflow input into the name value.
5. Validates that the name is not empty.
6. URL-encodes the name.
7. Sends a `GET` request to Genderize.io.
8. Checks the HTTP response status.
9. Parses the JSON response.
10. Returns the prediction as structured workflow data.

### Processing Flow

```text
Node Configuration
       ↓
Configured name?
       ↓
      Yes ──────────────┐
       ↓                │
      No                │
       ↓                │
Workflow Input          │
       ↓                │
Name Validation ←───────┘
       ↓
URL Encode Name
       ↓
Genderize.io API
       ↓
Parse JSON Response
       ↓
Return Prediction
```

---

## Related

- [Function](./function.md) – Transform and process prediction results
- [If](./if.md) – Filter or route workflow data
- [HTTP Request](./http-request.md) – Make requests to external APIs

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-10 | Initial release |

---