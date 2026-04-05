---
node_id: "medifus"
title: "Medifus (Visiostation)"
description: "Interact with the Visiostation API — retrieve medical statistics, prescriptions, and consultation insights across regions, provinces, and centers"
category: "integrations"
subcategory: "healthcare"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - medifus
  - visiostation
  - healthcare
  - statistics
  - prescriptions
  - consultations
  - action
related_nodes:
  - function
  - cron
  - postgres-action
---

<!-- SECTION: header -->
# Medifus (Visiostation)

> **Category:** Integrations | **Type:** Action Node

Connect to the Visiostation API and retrieve medical statistics, prescription data, and consultation insights. Supports filtering by date range, region, province, and healthcare center.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Medifus** node (internally named `VisiostationNode`) integrates with the Visiostation healthcare platform API. It authenticates automatically using login/password credentials and performs one of five data retrieval operations. It is designed to power medical reporting workflows, dashboards, and data pipelines within the Fusion automation platform.

Authentication is handled transparently — the node logs in on each request, obtains a JWT access token, and re-authenticates automatically on `401` responses, ensuring uninterrupted operation.

### Key Features

- **Five Operations:** Overview statistics, dashboard statistics, prescriptions, consultation prescriptions, and consultation insights
- **Dashboard Types:** Six dashboard categories for `getAllStatistics` (Administration, MedicalPharmaceutical, ClinicalEpidemiology, Telemedecine, Facturation, Appointement)
- **Flexible Filtering:** Filter all operations by date range (`from` / `to`) and geographic scope (`regionId`, `provinceId`, `centerId`)
- **Auto-Authentication:** JWT token obtained and refreshed automatically — no manual token management required
- **Expression Support:** `from`, `to`, `regionId`, `provinceId`, `centerId`, and `dashbordType` support dynamic expressions

### Use Cases

- Generate daily or weekly medical statistics reports automatically
- Pull prescription data per center for audit and compliance workflows
- Monitor consultation insights by region for administrative dashboards
- Feed Visiostation data into a PostgreSQL or MongoDB database for long-term storage
- Trigger alerts when specific metrics cross a threshold

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `baseURL` | `string` | ✅ Yes | Base URL of the Visiostation API (e.g. `https://api.visiostation.example.com`) |
| `auth.login` | `string` | ✅ Yes | Visiostation account login (username) |
| `auth.password` | `string` | ✅ Yes | Visiostation account password |

> Authentication is performed via `POST /auth/login`. The node retrieves a `Bearer` token automatically and retries once on `401` errors.

### Operation Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `getOverviewStatistics` | Operation to execute (see below) |
| `dashbordType` | `enum` | Conditional | `Administration` | Dashboard type — required for `getAllStatistics` only |

### Available Operations

| Operation | Description |
|-----------|-------------|
| `getOverviewStatistics` | Retrieve high-level overview statistics from the main dashboard |
| `getAllStatistics` | Retrieve detailed statistics for a specific dashboard type |
| `getPrescriptions` | Retrieve prescription data grouped by healthcare center |
| `getConsultationPrescriptions` | Retrieve consultation prescription data by center |
| `getConsultationInsights` | Retrieve consultation insights and analytics |

### Dashboard Types (for `getAllStatistics`)

| Value | ID | Description |
|-------|-----|-------------|
| `Administration` | `1` | Administrative dashboard |
| `MedicalPharmaceutical` | `2` | Medical and pharmaceutical statistics |
| `ClinicalEpidemiology` | `3` | Clinical and epidemiological data |
| `Telemedecine` | `4` | Telemedicine statistics |
| `Facturation` | `5` | Billing and invoicing data |
| `Appointement` | `6` | Appointment and scheduling statistics |

### Filter Parameters

All operations support the following optional filters. They can be provided as static values or dynamic expressions from the incoming workflow data.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `from` | `string` | ❌ No | Start date for the filter range (ISO 8601 or `YYYY-MM-DD`) |
| `to` | `string` | ❌ No | End date for the filter range (ISO 8601 or `YYYY-MM-DD`) |
| `regionId` | `string` | ❌ No | Filter by region identifier |
| `provinceId` | `string` | ❌ No | Filter by province identifier |
| `centerId` | `string` | ❌ No | Filter by healthcare center identifier |

> Filter parameters are resolved from incoming workflow data first, then from the static node configuration. This allows dynamic filtering based on upstream node outputs.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Incoming workflow data — filter fields (`from`, `to`, `regionId`, `provinceId`, `centerId`) are resolved from here first |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Raw response data from the Visiostation API |
| `error` | `Error` | Emitted if authentication or the API request fails |

### Output Schema

The output structure depends on the Visiostation API response for each operation. Below are representative schemas.

**`getOverviewStatistics`**
```json
{
  "totalConsultations": 14820,
  "totalPrescriptions": 9430,
  "totalPatients": 6210,
  "activeCenters": 42,
  "period": {
    "from": "2026-01-01",
    "to": "2026-03-10"
  }
}
```

**`getAllStatistics` (e.g. Administration)**
```json
{
  "dashboardId": "1",
  "dashboardType": "Administration",
  "data": {
    "centerCount": 42,
    "activeUsers": 318,
    "sessionsToday": 87
  }
}
```

**`getPrescriptions`**
```json
{
  "centers": [
    {
      "centerId": "CTR-01",
      "centerName": "Centre de Santé El Fath",
      "prescriptionCount": 342
    },
    {
      "centerId": "CTR-02",
      "centerName": "Centre Médical Agdal",
      "prescriptionCount": 218
    }
  ]
}
```

**`getConsultationInsights`**
```json
{
  "totalConsultations": 5820,
  "averageDuration": 18.4,
  "bySpecialty": [
    { "specialty": "Généraliste", "count": 3100 },
    { "specialty": "Pédiatrie", "count": 980 }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Get Overview Statistics

Retrieve the global overview for a specific date range.

**Configuration:**
```json
{
  "baseURL": "https://api.visiostation.example.com",
  "auth": {
    "login": "admin@fusion.local",
    "password": "secret"
  },
  "operation": "getOverviewStatistics",
  "from": "2026-01-01",
  "to": "2026-03-10"
}
```

---

### Example: Get All Statistics for a Dashboard Type

Retrieve detailed telemedicine statistics for a specific region.

**Configuration:**
```json
{
  "baseURL": "https://api.visiostation.example.com",
  "auth": {
    "login": "admin@fusion.local",
    "password": "secret"
  },
  "operation": "getAllStatistics",
  "dashbordType": "Telemedecine",
  "from": "2026-03-01",
  "to": "2026-03-10",
  "regionId": "REG-02"
}
```

---

### Example: Get Prescriptions for a Center

Retrieve prescription data filtered by a specific healthcare center.

**Configuration:**
```json
{
  "baseURL": "https://api.visiostation.example.com",
  "auth": {
    "login": "admin@fusion.local",
    "password": "secret"
  },
  "operation": "getPrescriptions",
  "centerId": "CTR-07",
  "from": "2026-02-01",
  "to": "2026-02-28"
}
```

---

### Example: Dynamic Filters from Incoming Data

Use values from the upstream node as filter parameters.

**Configuration:**
```json
{
  "baseURL": "https://api.visiostation.example.com",
  "auth": {
    "login": "admin@fusion.local",
    "password": "secret"
  },
  "operation": "getConsultationInsights",
  "from": "{{input.startDate}}",
  "to": "{{input.endDate}}",
  "regionId": "{{input.regionId}}"
}
```

---

### Example: Consultation Prescriptions by Province

**Configuration:**
```json
{
  "baseURL": "https://api.visiostation.example.com",
  "auth": {
    "login": "admin@fusion.local",
    "password": "secret"
  },
  "operation": "getConsultationPrescriptions",
  "provinceId": "PROV-04",
  "from": "2026-01-01",
  "to": "2026-03-10"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Weekly Medical Report

Every Monday, pull overview statistics and store them in PostgreSQL.

```json
{
  "nodes": [
    {
      "id": "weekly-trigger",
      "type": "cron",
      "position": { "x": 100, "y": 100 },
      "config": {
        "cronExpression": "0 6 * * 1"
      }
    },
    {
      "id": "compute-dates",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "const now = new Date(); const from = new Date(now - 7 * 86400000).toISOString().slice(0,10); const to = now.toISOString().slice(0,10); return { from, to };"
      }
    },
    {
      "id": "fetch-stats",
      "type": "medifus",
      "position": { "x": 500, "y": 100 },
      "config": {
        "baseURL": "https://api.visiostation.example.com",
        "auth": { "login": "admin@fusion.local", "password": "secret" },
        "operation": "getOverviewStatistics",
        "from": "{{input.from}}",
        "to": "{{input.to}}"
      }
    },
    {
      "id": "store-report",
      "type": "postgres-action",
      "position": { "x": 700, "y": 100 },
      "config": {
        "host": "db.example.com",
        "database": "reports",
        "tableName": "weekly_stats",
        "operation": "Insert",
        "insertParams": {
          "rows": [
            { "rowName": "week_start", "rowValue": "{{input.period.from}}" },
            { "rowName": "consultations", "rowValue": "{{input.totalConsultations}}" },
            { "rowName": "prescriptions", "rowValue": "{{input.totalPrescriptions}}" },
            { "rowName": "patients", "rowValue": "{{input.totalPatients}}" }
          ]
        }
      }
    }
  ]
}
```

### Common Patterns

- **Scheduled Reports:** Cron → Medifus (fetch stats) → Store in DB or send by email
- **Dynamic Dashboards:** HTTP Trigger (with region/date params) → Medifus → Return data
- **Cross-Dashboard Comparison:** Multiple Medifus nodes in parallel (one per `dashbordType`) → Function (merge) → Output
- **Alerting:** Cron → Medifus → Function (check threshold) → Notification node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Authentication failed`

**Cause:** Invalid `auth.login` or `auth.password`, or the Visiostation API is unreachable.

**Solution:** Verify credentials. Confirm that `baseURL` points to the correct API endpoint and is reachable from the workflow engine. Ensure the account has API access permissions.

#### `Authentication succeeded but tokens are missing`

**Cause:** The `/auth/login` endpoint responded with `200` but the `tokens.accessToken` field is absent — possible API version mismatch.

**Solution:** Check the Visiostation API version. Contact the Visiostation team to confirm the authentication response format.

#### `dashbordType is required for getAllStatistics`

**Cause:** The `getAllStatistics` operation was selected but `dashbordType` was left empty.

**Solution:** Select a dashboard type from the available options: `Administration`, `MedicalPharmaceutical`, `ClinicalEpidemiology`, `Telemedecine`, `Facturation`, or `Appointement`.

#### `Invalid dashboard type`

**Cause:** The `dashbordType` value does not match any of the supported types (case-sensitive).

**Solution:** Use one of the exact values listed in the Dashboard Types table. Dynamic expressions must resolve to one of these exact strings.

#### Empty or unexpected response data

**Cause:** The filter parameters (`from`, `to`, `regionId`, etc.) may be too narrow, or the API returned no data for that combination.

**Solution:** Broaden the date range or remove geographic filters. Validate that the `centerId`, `regionId`, and `provinceId` values exist in the Visiostation system.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `401` | Unauthorized | Re-check credentials — node will retry once automatically |
| `404` | Endpoint not found | Verify `baseURL` and API version |
| `ECONNREFUSED` | Connection refused | Check network access to the Visiostation API |
| `ENOTFOUND` | DNS resolution failed | Verify `baseURL` hostname |
| `Operation failed` | Generic API error | Check response body for details from the Visiostation API |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Cron](./cron.md) - Schedule periodic Medifus data pulls
- [Function](./function.md) - Transform or aggregate Visiostation API responses
- [PostgreSQL Action](./postgres.md) - Store fetched statistics in a relational database
- [MongoDB Action](./mongodb.md) - Store fetched statistics in a document database

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
