---
node_id: "clinical-trials-search"
title: "Clinical Trials Search"
description: "Search ClinicalTrials.gov for clinical studies by medical condition and study status."
category: "healthcare-life-sciences"
subcategory: "clinical-trials"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - healthcare
  - clinical-trials
  - medical-research
  - studies
  - recruiting
  - clinicaltrials-gov
related_nodes:
  - clinical-tables-search
  - chembl
  - biorxiv
---

<!-- SECTION: header -->
# Clinical Trials Search

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Search ClinicalTrials.gov for clinical studies by medical condition and study status. The node returns normalized study information, including identifiers, titles, phases, sponsors, locations, eligibility criteria, and direct study URLs.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Clinical Trials Search** node queries the ClinicalTrials.gov API and retrieves clinical studies matching a configured medical condition and study status.

The node builds the API request automatically, retries temporary failures, processes the returned studies, and converts them into a consistent workflow-friendly structure.

### Key Features

- **Condition-Based Search:** Search studies by medical condition.
- **Status Filtering:** Filter by recruiting, completed, active, not recruiting, or all studies.
- **Configurable Result Limit:** Request between 1 and 100 studies.
- **Structured Trial Data:** Returns study identifiers, titles, statuses, phases, sponsors, locations, and eligibility details.
- **Direct Study URLs:** Builds a ClinicalTrials.gov URL for every study with an NCT identifier.
- **Location Limiting:** Returns up to three locations per study.
- **Retry Handling:** Retries temporary network errors, rate limits, and server failures.
- **Timeout Protection:** Uses a 30-second request timeout.
- **No Credentials Required:** Uses the public ClinicalTrials.gov API.

### Returned Trial Information

Each processed study can contain:

- NCT identifier
- Brief title
- Overall status
- Study phase
- Study type
- Medical conditions
- Up to three study locations
- Lead sponsor
- ClinicalTrials.gov URL
- Eligibility information

### Use Cases

- Find recruiting studies for a medical condition
- Search completed clinical trials
- Build clinical research dashboards
- Retrieve study sponsor information
- Review study eligibility criteria
- Find trial locations
- Enrich healthcare research workflows
- Create medical study search tools
- Monitor clinical research activity

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `condition` | `string` | ✅ Yes | — | Medical condition used to search ClinicalTrials.gov. Must contain at least one character. |
| `status` | `enum` | ❌ No | `recruiting` | Study status filter. |
| `maxResults` | `number` | ❌ No | `10` | Maximum number of studies to request. Accepts values from `1` to `100`. |

### Status Values

| Value | ClinicalTrials.gov Filter | Description |
|-------|---------------------------|-------------|
| `recruiting` | `RECRUITING` | Studies currently recruiting participants. |
| `completed` | `COMPLETED` | Completed studies. |
| `active` | `ACTIVE_NOT_RECRUITING` | Active studies that are no longer recruiting. |
| `not_recruiting` | `ACTIVE_NOT_RECRUITING` | Active studies that are no longer recruiting. |
| `all` | No status filter | Search studies regardless of status. |

> **Note:** Both `active` and `not_recruiting` map to `ACTIVE_NOT_RECRUITING` in the current implementation.

### Default Values

| Parameter | Default |
|-----------|---------|
| `status` | `recruiting` |
| `maxResults` | `10` |

### Condition Rules

The node validates the condition before sending the request.

- Leading and trailing spaces are removed.
- Empty conditions are rejected.
- The configured condition is used as the ClinicalTrials.gov condition query.

### Result Limit Handling

The `maxResults` parameter accepts values from `1` to `100`.

The runtime also applies defensive normalization:

- Values below `1` fall back to `10`.
- Values above `100` are limited to `100`.
- Invalid values fall back to `10`.

### Request Behavior

The node:

- sends requests to the ClinicalTrials.gov v2 API;
- requests JSON output;
- uses a 30-second timeout;
- retries temporary errors up to three times;
- applies exponential backoff;
- retries HTTP `429` and server errors;
- returns an empty result for HTTP `404`;
- limits processed locations to three per study.

<!-- /SECTION: configuration -->
---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow data. The node uses its configured parameters to perform the search. |

The node primarily uses the configured `condition`, `status`, and `maxResults` values.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `condition` | `string` | Normalized condition used for the search. |
| `search_status` | `string` | Configured status value. |
| `total_results` | `number` | Total study count reported by the API, or the number of processed studies when no total count is provided. |
| `returned_results` | `number` | Number of studies returned by the node. |
| `trials` | `array` | Processed clinical trial records. |

### Trial Structure

Each item in `trials` contains:

| Field | Type | Description |
|-------|------|-------------|
| `nct_id` | `string` | ClinicalTrials.gov NCT identifier. |
| `title` | `string` | Brief study title. |
| `status` | `string` | Overall study status returned by the API. |
| `phase` | `string[]` | Study phases. |
| `study_type` | `string` | Study type. |
| `conditions` | `string[]` | Conditions associated with the study. |
| `locations` | `array` | Up to three study locations. |
| `sponsor` | `string` | Lead sponsor name. |
| `url` | `string` | Direct ClinicalTrials.gov study URL. |
| `eligibility` | `object` | Eligibility criteria summary. |

### Location Structure

Each item in `locations` contains:

| Field | Type | Description |
|-------|------|-------------|
| `facility` | `string` | Facility name. |
| `city` | `string` | City. |
| `state` | `string` | State or region. |
| `country` | `string` | Country. |

### Eligibility Structure

| Field | Type | Description |
|-------|------|-------------|
| `gender` | `string` | Eligible sex returned by the API. |
| `min_age` | `string` | Minimum eligible age. |
| `max_age` | `string` | Maximum eligible age. |
| `healthy_volunteers` | `string` | `Yes` when healthy volunteers are accepted; otherwise `No`. |

### Successful Response

> Example response. Actual studies, counts, statuses, and locations depend on the current ClinicalTrials.gov data.

```json
{
  "condition": "diabetes",
  "search_status": "recruiting",
  "total_results": 10,
  "returned_results": 10,
  "trials": [
    {
      "nct_id": "NCT01234567",
      "title": "Example Diabetes Clinical Study",
      "status": "RECRUITING",
      "phase": [
        "PHASE2"
      ],
      "study_type": "INTERVENTIONAL",
      "conditions": [
        "Diabetes Mellitus"
      ],
      "locations": [
        {
          "facility": "Example Medical Center",
          "city": "Boston",
          "state": "Massachusetts",
          "country": "United States"
        }
      ],
      "sponsor": "Example Research Organization",
      "url": "https://clinicaltrials.gov/study/NCT01234567",
      "eligibility": {
        "gender": "ALL",
        "min_age": "18 Years",
        "max_age": "75 Years",
        "healthy_volunteers": "No"
      }
    }
  ]
}
```

### Empty Results

When no studies are found, the node returns a valid empty result.

```json
{
  "condition": "unknown-condition",
  "search_status": "recruiting",
  "total_results": 0,
  "returned_results": 0,
  "trials": []
}
```

### Error Response

The node throws an error when validation or the API request fails.

Examples:

```text
Condition is required
```

```text
Error searching clinical trials: <error message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Search Recruiting Diabetes Trials

Search for recruiting clinical trials related to diabetes.

**Configuration**

```text
Condition: diabetes
Status: recruiting
Max Results: 10
```

The node returns recruiting studies related to diabetes.

---

### Example: Search Completed Cancer Trials

Search completed cancer studies.

**Configuration**

```text
Condition: cancer
Status: completed
Max Results: 20
```

---

### Example: Search Active Studies

Search studies that are active but no longer recruiting.

**Configuration**

```text
Condition: breast cancer
Status: active
Max Results: 10
```

The `active` value maps to `ACTIVE_NOT_RECRUITING`.

---

### Example: Search Not Recruiting Studies

**Configuration**

```text
Condition: heart disease
Status: not_recruiting
Max Results: 10
```

The `not_recruiting` value also maps to `ACTIVE_NOT_RECRUITING`.

---

### Example: Search All Statuses

Search without applying a status filter.

**Configuration**

```text
Condition: asthma
Status: all
Max Results: 25
```

---

### Example: Limit Results

Request only five studies.

**Configuration**

```text
Condition: diabetes
Status: recruiting
Max Results: 5
```

---

### Example: Review Study Locations

Search studies and inspect their location data.

**Configuration**

```text
Condition: alzheimer disease
Status: recruiting
Max Results: 10
```

Each study returns at most three locations.

---

### Example: Review Eligibility Information

**Configuration**

```text
Condition: hypertension
Status: recruiting
Max Results: 10
```

The output includes:

```json
{
  "eligibility": {
    "gender": "ALL",
    "min_age": "18 Years",
    "max_age": "80 Years",
    "healthy_volunteers": "No"
  }
}
```

---

### Example: Missing Condition

If the condition is empty, the node throws:

```text
Condition is required
```

<!-- /SECTION: examples -->
---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Recruiting Clinical Trials
```

### Common Patterns

- **Recruiting Study Search:** Manual Trigger → Clinical Trials Search → Log
- **Completed Study Search:** Manual Trigger → Clinical Trials Search → Log
- **Condition Lookup:** HTTP Request → Clinical Trials Search → Log
- **Clinical Research Workflow:** Clinical Trials Search → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "Condition is required"

**Cause**

The `condition` parameter is empty.

**Solution**

Provide a non-empty medical condition, such as:

```text
diabetes
```

```text
breast cancer
```

```text
asthma
```

---

#### No Studies Returned

**Cause**

No studies match the selected condition and status filter.

Example:

```json
{
  "total_results": 0,
  "returned_results": 0,
  "trials": []
}
```

**Solution**

- Verify the condition spelling.
- Try a broader condition.
- Change the status to `all`.

---

#### Unexpected Status Results

**Cause**

The configured status is mapped to a ClinicalTrials.gov API status.

Current mappings:

```text
recruiting       → RECRUITING
completed        → COMPLETED
active           → ACTIVE_NOT_RECRUITING
not_recruiting   → ACTIVE_NOT_RECRUITING
all              → no status filter
```

**Solution**

Use `all` when you do not want a status filter.

---

#### Fewer Locations Than Expected

**Cause**

The node intentionally limits study locations to the first three returned by the API.

**Solution**

This is the current implementation behavior. The node does not return more than three locations per study.

---

#### Missing Trial Fields

**Cause**

Some ClinicalTrials.gov studies do not provide all optional fields.

The node returns empty strings or empty arrays when values such as phase, sponsor, location, or eligibility details are unavailable.

**Solution**

Check values before using them in downstream workflow steps.

---

#### API Timeout

**Cause**

ClinicalTrials.gov did not respond within 30 seconds.

**Solution**

Retry the workflow. The node automatically retries retryable failures before returning an error.

---

#### Rate Limit Error

**Cause**

ClinicalTrials.gov returned HTTP `429`.

**Solution**

Wait before retrying. The node automatically retries rate-limited requests using exponential backoff.

---

#### Server Error

**Cause**

ClinicalTrials.gov returned HTTP `500`, `502`, or `503`.

**Solution**

Retry later. The node automatically retries temporary server failures.

---

### Error Messages

| Error | Description |
|-------|-------------|
| `Condition is required` | No medical condition was configured. |
| `Error searching clinical trials: Bad Request: Invalid search parameters` | ClinicalTrials.gov rejected the request parameters. |
| `Error searching clinical trials: Rate Limited: Too many requests. Retrying...` | The API rate limit was reached after the retry attempts. |
| `Error searching clinical trials: Server Error: ClinicalTrials.gov is experiencing issues` | The remote API returned a temporary server failure. |
| `Error searching clinical trials: Request timeout after 30000ms` | The request exceeded the configured timeout. |
| `Error searching clinical trials: Network error: Unable to connect to ClinicalTrials.gov (DNS resolution failed)` | DNS resolution failed. |
| `Error searching clinical trials: Network error: Connection refused by ClinicalTrials.gov` | The remote service refused the connection. |
| `Error searching clinical trials: <error message>` | The node wrapped the underlying request error. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- Clinical Tables NLM Search
- ChEMBL
- BioRxiv

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial release of the Clinical Trials Search documentation. |

<!-- /SECTION: changelog -->