---
node_id: "random-user"
title: "RandomUser"
description: "Generate random user data using the RandomUser API."
category: "devops-cloud-management"
subcategory: "developer-tools"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - randomuser
  - random-user
  - mock-data
  - test-data
  - fake-data
  - users
  - developer-tools
  - testing
related_nodes:
  - json-placeholder
  - lorem-picsum
  - barcode-generator
  - quick-chart
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# RandomUser

> **Category:** Developer Tools | **Type:** Action Node

Generate realistic, randomized user profiles, contact information, avatars, login credentials, and demographic test data using the [Random User Generator](https://randomuser.me/) API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **RandomUser** node integrates with the free and open [Random User Generator API](https://randomuser.me/) (`https://randomuser.me/api/`) to generate mock user profiles for development, testing, UI prototyping, and database seeding workflows.

Each generated user profile contains realistic and structured data, including full names, physical addresses, geocoordinates, timezone information, email addresses, phone/cell numbers, date of birth, age, login credentials (UUID, MD5/SHA256 password hashes), high-resolution portrait photos, and nationality codes.

The node allows filtering by gender (`male`, `female`), specifying a batch size with the `results` parameter, and filtering by single or multiple ISO nationality codes (such as `US`, `FR`, `GB`, `ES`, `DE`, `CA`).

### Key Features

- **Rich Mock Profiles:** Produces complete persona objects with names, realistic street addresses, coordinates, phone numbers, avatars, and registration metadata.
- **Gender Filtering:** Restrict generated profiles to `male` or `female` for targeted testing scenarios.
- **Multi-Nationality Filtering:** Filter by single country codes (`FR`, `US`, `GB`) or comma-separated lists (`US,FR,ES,GB`) for localized mock datasets.
- **Batch Generation:** Generate single profiles or batches of up to dozens/hundreds of users in a single execution.
- **Synthetic Credentials:** Supplies mock login credentials, including UUIDs, usernames, passwords, salts, and MD5/SHA-1/SHA-256 hashes.
- **Multiple Avatar Sizes:** Provides direct URLs for `large`, `medium`, and `thumbnail` portrait photos.
- **Zero Configuration / No API Key:** Directly queries the public RandomUser API without authentication or API tokens.

### Data Generation Flow

```text
Incoming Trigger / Payload
            ↓
  Configure Parameters
  (results, gender, nat)
            ↓
  Build RandomUser Request URL
  (https://randomuser.me/api/?results=N&gender=G&nat=NAT)
            ↓
    Fetch from API
            ↓
  Parse JSON Response
  (results array + info metadata)
            ↓
  Success Output (Log / Function / DB Seed)
```

### Use Cases

- **UI & Frontend Prototyping:** Populate dashboards, contact lists, user tables, and card grids with realistic placeholder data and portrait photos.
- **Database Seeding & Test Fixtures:** Populate staging databases, test tenants, or CRM environments with hundreds of diverse customer records.
- **Automated Integration Testing:** Simulate high-volume user registration pipelines, form validation checks, and localization tests.
- **Demo Environments & Walkthroughs:** Showcase enterprise SaaS platforms and CRM workflows to prospective clients using clean, realistic persona data without exposing real PII.
- **Load & Performance Testing:** Feed dynamic user profiles into stress-testing scripts for user ingestion APIs.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `results` | `number` | ❌ No | `1` | Number of user profiles to generate in a single request (e.g., `1`, `3`, `5`, `50`). |
| `gender` | `string` | ❌ No | *(all)* | Filter users by gender. Supported values: `female`, `male`. When omitted, returns both genders randomly. |
| `nat` | `string` | ❌ No | *(all)* | Filter users by nationality using standard 2-letter country codes. Supports single or comma-separated codes (e.g. `FR`, `US,FR,ES,GB`). |

---

### Parameter Details

#### `results`
Specifies how many distinct user profiles should be returned in the `results` array.
- **Type:** `number`
- **Default:** `1`
- **Allowed Range:** `1` to `5000`
- **Example:** `1`, `3`, `5`, `25`, `100`

#### `gender`
Limits the returned profiles to a specific gender.
- **Type:** `string`
- **Supported Values:**
  - `female` — Returns only female profiles (including feminine names and portrait photos).
  - `male` — Returns only male profiles (including masculine names and portrait photos).
- **Default:** Omitted (random distribution of male and female profiles).

#### `nat`
Specifies one or more 2-letter ISO 3166-1 alpha-2 nationality codes to filter the demographic origin of the generated profiles.
- **Type:** `string`
- **Supported Codes:**
  - `AU` (Australia), `BR` (Brazil), `CA` (Canada), `CH` (Switzerland), `DE` (Germany), `DK` (Denmark), `ES` (Spain), `FI` (Finland), `FR` (France), `GB` (United Kingdom), `IE` (Ireland), `IN` (India), `IR` (Iran), `MX` (Mexico), `NL` (Netherlands), `NO` (Norway), `NZ` (New Zealand), `RS` (Serbia), `TR` (Turkey), `UA` (Ukraine), `US` (United States).
- **Format:** Single code (`FR`) or comma-separated list (`US,FR,ES,GB`).
- **Example:** `"FR"`, `"US,GB"`, `"US,FR,ES,GB"`

---

### API Endpoint

The node constructs queries to the Random User API endpoint:

```text
https://randomuser.me/api/?results={results}&gender={gender}&nat={nat}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow execution payload. Can supply dynamic parameters via expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted on successful generation. Contains the `results` array of user objects and the `info` metadata block. |
| `error` | `Error` | Emitted when API communication fails or invalid parameter types are passed. |

---

### Output Data Structure

The `success` output returns a structured JSON object matching the standard RandomUser response schema:

```json
{
  "results": [
    {
      "gender": "female",
      "name": {
        "title": "Ms",
        "first": "Camille",
        "last": "Leclerc"
      },
      "location": {
        "street": {
          "number": 4218,
          "name": "Rue Paul-Bert"
        },
        "city": "Versailles",
        "state": "Yvelines",
        "country": "France",
        "postcode": "78000",
        "coordinates": {
          "latitude": "48.8014",
          "longitude": "2.1301"
        },
        "timezone": {
          "offset": "+1:00",
          "description": "Brussels, Copenhagen, Madrid, Paris"
        }
      },
      "email": "camille.leclerc@example.com",
      "login": {
        "uuid": "d3b07384-d113-4a44-96cf-1b8f36c5df92",
        "username": "silverpeacock123",
        "password": "supersecretpassword",
        "salt": "n8s8fhf",
        "md5": "7b0a88b5028cb98fa6a9d7cdfd8b4fb4",
        "sha1": "55c47864817a07775ba0b1f3c39c8eb72322300b",
        "sha256": "4b68e98b049d107a68e8cb554425816d7a46f9d5059632feee8606bf446d3e69"
      },
      "dob": {
        "date": "1991-04-12T07:22:18.123Z",
        "age": 35
      },
      "registered": {
        "date": "2018-09-03T11:42:09.456Z",
        "age": 8
      },
      "phone": "01-44-55-66-77",
      "cell": "06-12-34-56-78",
      "id": {
        "name": "INSEE",
        "value": "2910478000123"
      },
      "picture": {
        "large": "https://randomuser.me/api/portraits/women/44.jpg",
        "medium": "https://randomuser.me/api/portraits/med/women/44.jpg",
        "thumbnail": "https://randomuser.me/api/portraits/thumb/women/44.jpg"
      },
      "nat": "FR"
    }
  ],
  "info": {
    "seed": "abc123xyz",
    "results": 1,
    "page": 1,
    "version": "1.4"
  }
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `results` | `array` | Array of generated user profile objects. |
| `results[].gender` | `string` | User gender (`female` or `male`). |
| `results[].name` | `object` | Full name details containing `title`, `first`, and `last`. |
| `results[].location` | `object` | Complete address object including `street`, `city`, `state`, `country`, `postcode`, `coordinates`, and `timezone`. |
| `results[].email` | `string` | Synthetic email address generated from the user's name. |
| `results[].login` | `object` | Synthetic authentication data containing `uuid`, `username`, `password`, `salt`, `md5`, `sha1`, and `sha256`. |
| `results[].dob` | `object` | Date of birth containing ISO timestamp (`date`) and calculated `age`. |
| `results[].registered` | `object` | Account registration timestamp and age in years. |
| `results[].phone` | `string` | Formatted landline phone number according to country standard. |
| `results[].cell` | `string` | Formatted mobile / cellular phone number. |
| `results[].id` | `object` | National ID type and synthetic identifier value. |
| `results[].picture` | `object` | URLs to portrait avatars in `large` (128x128), `medium` (72x72), and `thumbnail` (48x48) formats. |
| `results[].nat` | `string` | 2-letter nationality code of the profile (e.g., `FR`, `US`). |
| `info` | `object` | Response metadata containing `seed`, `results` count, `page`, and API `version`. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Default Single Random User

Generate a single random user profile with default settings.

**Parameter Configuration:**

```text
Results: 1
Gender: (empty)
Nat: (empty)
```

**Output:**
Returns 1 randomized user profile with random gender and nationality.

---

### Example 2: Female Users Batch

Generate 3 female user profiles for UI testing.

**Parameter Configuration:**

```text
Results: 3
Gender: female
Nat: (empty)
```

**Output:**
Returns an array of 3 female user profiles with feminine names, avatars, and contact details.

---

### Example 3: French Male Users

Generate 2 French male profiles with localized addresses and French phone numbers.

**Parameter Configuration:**

```text
Results: 2
Gender: male
Nat: FR
```

**Output:**
Returns 2 male user profiles based in France with French postal codes, city names, and phone formats.

---

### Example 4: Multi-Nationality Batch Testing

Generate 5 profiles from selected Western & European nationalities.

**Parameter Configuration:**

```text
Results: 5
Gender: (empty)
Nat: US,FR,ES,GB
```

**Output:**
Returns 5 user profiles distributed across United States, France, Spain, and United Kingdom nationalities.

---

### Example 5: Database Seeder Pipeline

Generate 50 users and insert them into a staging database.

**Workflow Configuration:**

```text
Manual Trigger
  → RandomUser (results: 50, nat: "US,CA,GB")
  → Function (map results to database schema)
  → PostgreSQL Action (Bulk Insert into "users" table)
  → Log (print inserted count)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate random user data using the RandomUser API
```

### Common Workflow Patterns

- **Mock Data Pipeline:** `Manual Trigger` → `RandomUser` → `Log` (Quick inspection in console).
- **Automated Database Seeder:** `Cron Trigger` → `RandomUser` (`results: 20`) → `Function` → `Postgres / MySQL / MongoDB Bulk Insert`.
- **Mock User Webhook Ingestion:** `HTTP Webhook Trigger` → `RandomUser` (`results: 1`) → `HTTP Request` (POST payload to staging backend).
- **Demographic Stress Tester:** `Manual Trigger` → `RandomUser` (`results: 100`, `nat: "US,FR,ES,GB"`) → `For Each` → `API Load Tester`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Invalid `gender` Value
- **Symptom:** API ignores gender filter or returns an unexpected error.
- **Cause:** Specifying values other than `female` or `male` (e.g. `test`, `all`, `F`).
- **Solution:** Ensure `gender` is set explicitly to either `"female"` or `"male"`, or leave it blank to include all genders.

#### Invalid or Unsupported Nationality Code
- **Symptom:** The API returns users from default/random nationalities or returns an empty result.
- **Cause:** Typo in nationality code (e.g., using `USA` instead of `US`, or `FRA` instead of `FR`).
- **Solution:** Use standard 2-letter alpha-2 codes (e.g. `US`, `FR`, `GB`, `ES`, `DE`, `CA`, `AU`).

#### Exceeding Batch Limits
- **Symptom:** Request times out or fails with HTTP 414 / 500.
- **Cause:** Setting `results` to an excessively large number in a single request.
- **Solution:** Keep `results` below `5000` per call, or use a loop / batching pattern with **For Each** or **Delay** for multi-thousand record migrations.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Network error` | Failed to reach `randomuser.me` | Check network connectivity and firewall outbound access |
| `HTTP 429 Too Many Requests` | Exceeded public API rate limit | Introduce a delay between batch calls using the **Delay** node |
| `HTTP 500 Internal Server Error` | Upstream RandomUser API temporary glitch | Retry the request or use a seeded fallback |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [JSONPlaceholder](../json-placeholder/en.md) — Mock REST API for posts, comments, and albums
- [Lorem Picsum](../lorem-picsum/en.md) — Placeholder image generator for UI prototypes
- [Barcode/QR Generator](../barcode-generator/en.md) — Generate 1D/2D barcodes and QR codes
- [QuickChart](../quick-chart/en.md) — Generate dynamic charts and visual reports
- [HTTP Request](../http-request/en.md) — Perform custom REST API calls
- [Function](../function/en.md) — Transform and reshape mock user data
- [Log](../log/en.md) — View generated user records in the execution console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial release with support for custom results count, gender filtering, and multi-nationality filtering |

<!-- /SECTION: changelog -->
