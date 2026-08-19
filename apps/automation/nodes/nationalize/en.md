---
node_id: "nationalize"
title: "Nationalize.io"
description: "Predict nationality based on a name using the Nationalize.io API."
category: "Web Search & Information"
subcategory: "Utilities"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - nationalize
  - nationality
  - name
  - prediction
  - api
  - utility
related_nodes:
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# Nationalize.io

> **Category:** Web Search & Information | **Subcategory:** Utilities | **Type:** Action Node

Predict the likely nationality or nationalities associated with a person’s name using the Nationalize.io API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nationalize.io** node sends a name to the Nationalize.io service and returns estimated nationality probabilities based on statistical name data. It is useful for enrichment, demonstration, and exploratory workflows where a probabilistic estimate is appropriate.

### Key Features

- **Name Prediction:** Analyze a supplied first name or name string
- **Probability Scores:** Return an estimated probability for each predicted country
- **Multiple Results:** Receive more than one possible country when the data supports it
- **Observation Count:** Include the number of observations used by the service when available
- **Workflow Ready:** Pass predictions to Log, Function, filtering, or reporting nodes
- **Error Routing:** Send invalid-input and API failures to the error output
- **Progress Visibility:** Supports running status while the API request is processed

### Typical Use Cases

- Enrich sample or demographic datasets
- Explore name and nationality patterns
- Build educational demonstrations of probabilistic APIs
- Route records based on a probability threshold
- Add predicted-country data to a downstream report
- Compare predictions for multiple names

### Important Limitation

Nationality predictions are statistical estimates, not verified facts about an individual. A name can be associated with multiple countries, and results may reflect historical or cultural naming patterns. Do not use the result as proof of a person’s nationality or identity, or as the sole basis for high-impact decisions.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `name` | `string` | Yes | — | Name to analyze with the Nationalize.io API |

### Name Value

Provide a non-empty name string. The example workflow uses:

```text
monica
```

Names may be supplied in lowercase or mixed case. Avoid sending unrelated text, punctuation-only values, or an empty string.

### Dynamic Input

A preceding node can provide the name dynamically through the input connection when supported by the workflow runtime.

Example incoming data:

```json
{
  "name": "monica"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` or `any` | Optional incoming data that can provide the name dynamically |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Nationalize.io prediction response |
| `error` | `object` | Error details when the name is invalid or the API request fails |

### Success Output Structure

A successful response follows the Nationalize.io response model:

```json
{
  "name": "monica",
  "country": [
    {
      "country_id": "US",
      "probability": 0.42
    },
    {
      "country_id": "GB",
      "probability": 0.18
    }
  ],
  "count": 1250
}
```

| Field | Type | Description |
|-------|------|-------------|
| `name` | `string` | Name analyzed by the service |
| `country` | `array` | List of predicted countries |
| `country[].country_id` | `string` | Two-letter country identifier for a prediction |
| `country[].probability` | `number` | Estimated probability for the country |
| `count` | `number` | Number of observations used by the service, when available |

The prediction list may contain multiple countries. Use a Function or Restructure node to select the highest-probability result or to preserve only the fields needed downstream.

### Error Output Example

```json
{
  "success": false,
  "error": "Name is required",
  "details": "Provide a non-empty name value"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Predict a Name’s Nationality

Analyze the name used in the example workflow.

**Configuration:**

```json
{
  "name": "monica"
}
```

Connect the success output to a Log node to inspect the returned country probabilities.

---

### Example 2: Dynamic Name from Input

A previous node can supply the name to analyze.

**Input:**

```json
{
  "name": "alexander"
}
```

The Nationalize.io node uses the incoming name and returns the corresponding prediction response.

---

### Example 3: Select the Most Likely Country

Use a Function node after Nationalize.io to sort the `country` array by `probability` and retain the highest-probability entry.

**Possible transformed result:**

```json
{
  "name": "monica",
  "mostLikelyCountry": "US",
  "probability": 0.42
}
```

Treat this as a statistical estimate and retain the probability if the result is used downstream.

---

### Example 4: Apply a Confidence Threshold

Use a Function or conditional node to continue only when the highest probability meets a workflow-defined threshold.

```text
Nationalize.io → Function → Decision → Log or Notification
```

Do not interpret a threshold as proof of nationality; it only controls workflow behavior based on the prediction score.

---

### Example 5: Handle Multiple Predictions

When the response contains several countries, preserve the complete `country` array instead of assuming the first entry is certain.

```json
{
  "name": "alex",
  "country": [
    { "country_id": "US", "probability": 0.31 },
    { "country_id": "GB", "probability": 0.24 }
  ]
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Predict nationality from a name and inspect the result
```

### Common Patterns

- **Basic Lookup:** Manual Trigger → Nationalize.io → Log
- **Dynamic Enrichment:** Name Input → Nationalize.io → Restructure → Storage
- **Confidence Routing:** Nationalize.io → Function → Decision
- **Batch Analysis:** Iterator → Nationalize.io → Merge → Report
- **API Orchestration:** HTTP Request → Nationalize.io → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Name is missing

**Cause:** The `name` parameter is empty, omitted, or was not passed from the incoming input.

**Solution:** Configure a non-empty `name` value or provide an input object containing `name`.

#### No country predictions are returned

**Cause:** The service may not have enough data for the supplied name.

**Solution:** Check the spelling and try a commonly observed name. An empty prediction list is not evidence that a person has no nationality.

#### Unexpected probability values

**Cause:** Probabilities represent estimates across the returned candidates and can vary by name and available data.

**Solution:** Treat them as estimates, preserve the complete result, and avoid using a single value as a verified fact.

#### API request fails

**Cause:** Network connectivity, service availability, malformed input, or request limits.

**Solution:** Verify the name, check workflow network access, retry safely, and inspect the error output.

#### Downstream mapping fails

**Cause:** The `country` field is an array rather than a single country value.

**Solution:** Iterate over the array or explicitly select an entry using Function or Restructure.

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `Name is required` | No name was supplied | Configure or pass a non-empty name |
| `No prediction data` | No usable match was found | Verify the input and handle an empty result |
| `Invalid request` | Input format is not accepted | Send a valid name string |
| `API request failed` | Network or service failure | Check connectivity and retry safely |
| `Rate limit exceeded` | Too many requests in a short period | Reduce request frequency and retry later |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](../function/en.md) — Transform or filter prediction results
- [Restructure](../restructure/en.md) — Map the response into a downstream schema
- [Merge](../merge/en.md) — Combine predictions from multiple names
- Log — Inspect Nationalize.io responses during workflow development

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial documentation for the Nationalize.io node |

<!-- /SECTION: changelog -->
