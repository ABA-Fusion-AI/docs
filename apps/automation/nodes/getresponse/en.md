---
node_id: "getresponse"
title: "GetResponse"
description: "Manage contacts, campaigns, newsletters, and account information in GetResponse"
category: "ungrouped"
subcategory: "marketing"
version: "1.0.1"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags:
  - integration
  - getresponse
  - email-marketing
  - contacts
  - campaigns
  - newsletters
related_nodes:
  - webhook
  - function
  - http-request
---

<!-- SECTION: header -->
# GetResponse

> **Category:** Integrations | **Type:** Action Node

Manage GetResponse contacts, campaigns, newsletters, and account information from a Fusion workflow.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **GetResponse** node connects Fusion workflows to the GetResponse API.

### Supported Operations

- `getContacts`
- `createContact`
- `deleteContact`
- `getCampaigns`
- `getNewsletters`
- `getAccountInfo`

### Common Use Cases

- Add a lead to a GetResponse campaign
- Retrieve contacts from a campaign
- Delete a contact
- List available campaigns
- Retrieve newsletters
- Verify account and API-key information

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `apiKey` | `string` | ✅ Yes | GetResponse API key |
| `operation` | `enum` | ✅ Yes | Operation to execute |
| `campaignId` | `string` | Conditional | Campaign identifier |
| `contactId` | `string` | Conditional | Contact identifier |
| `name` | `string` | Conditional | Contact name |
| `email` | `string` | Conditional | Contact email address |

### Operation Parameters

| Operation | Required Parameters | Optional Parameters |
|-----------|---------------------|---------------------|
| `getContacts` | `apiKey` | `campaignId` |
| `createContact` | `apiKey`, `campaignId`, `email` | `name` |
| `deleteContact` | `apiKey`, `contactId` | — |
| `getCampaigns` | `apiKey` | — |
| `getNewsletters` | `apiKey` | `campaignId` |
| `getAccountInfo` | `apiKey` | — |

### Authentication

Generate an API key in:

```text
GetResponse → Tools → Integrations and API → API
```

Paste only the API key value into `apiKey`.

The node should internally send:

```http
X-Auth-Token: api-key YOUR_API_KEY
```

Do not expose the API key in logs or outputs.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Data from the previous node, usable in expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object \| object[]` | GetResponse API response |
| `error` | `Error` | Authentication, validation, network, or API error |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Get Account Information

```json
{
  "apiKey": "YOUR_API_KEY",
  "operation": "getAccountInfo"
}
```

---

### Get Campaigns

```json
{
  "apiKey": "YOUR_API_KEY",
  "operation": "getCampaigns"
}
```

Example output:

```json
[
  {
    "campaignId": "p86zQ",
    "name": "Fusion Test"
  }
]
```

---

### Create a Contact

```json
{
  "apiKey": "YOUR_API_KEY",
  "operation": "createContact",
  "campaignId": "p86zQ",
  "name": "Fusion Test",
  "email": "fusion-test@example.com"
}
```

Use a unique email address for repeated tests.

---

### Get Contacts

```json
{
  "apiKey": "YOUR_API_KEY",
  "operation": "getContacts",
  "campaignId": "p86zQ"
}
```

`campaignId` may be left empty to retrieve all contacts if supported by the implementation.

---

### Delete a Contact

```json
{
  "apiKey": "YOUR_API_KEY",
  "operation": "deleteContact",
  "contactId": "abc123"
}
```

---

### Get Newsletters

```json
{
  "apiKey": "YOUR_API_KEY",
  "operation": "getNewsletters",
  "campaignId": "p86zQ"
}
```

`campaignId` is optional.

<!-- /SECTION: examples -->

---

<!-- SECTION: testing -->
## Testing in Fusion Platform

### Recommended Test Order

1. `getAccountInfo`
2. `getCampaigns`
3. `createContact`
4. `getContacts`
5. `getNewsletters`
6. `deleteContact`

### Test 1: Validate the API Key

Configure:

```text
ApiKey: your GetResponse API key
Operation: getAccountInfo
```

Expected result: account information is returned through `success`.

### Test 2: Retrieve Campaign IDs

Configure:

```text
Operation: getCampaigns
```

Copy a returned `campaignId`.

### Test 3: Create a Test Contact

Configure:

```text
Operation: createContact
CampaignId: campaign ID from the previous test
Name: Fusion Test
Email: a unique test email
```

Then confirm that the contact appears in GetResponse.

### Test 4: Retrieve Contacts

Configure:

```text
Operation: getContacts
CampaignId: the test campaign ID
```

Copy the created contact's `contactId`.

### Test 5: Retrieve Newsletters

Configure:

```text
Operation: getNewsletters
CampaignId: optional campaign ID
```

### Test 6: Delete the Test Contact

Configure:

```text
Operation: deleteContact
ContactId: contact ID from getContacts
```

### Negative Tests

| Test | Expected Result |
|------|-----------------|
| Invalid API key | Authentication error |
| Empty email in `createContact` | Validation error |
| Invalid email format | Validation error |
| Invalid campaign ID | Campaign not found error |
| Invalid contact ID | Contact not found error |
| Duplicate email | Duplicate contact error |
| Excessive requests | Rate-limit error |

### Export Validation

The node export currently contains:

```json
"parameters": {}
```

After saving the node, it should contain values similar to:

```json
{
  "apiKey": "********",
  "operation": "getContacts",
  "campaignId": "p86zQ"
}
```

For security, the exported workflow should preferably store a credential reference instead of the raw API key.

<!-- /SECTION: testing -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: getresponse-contact.workflow.json
title: Add a webhook contact to GetResponse
```

```json
{
  "nodes": [
    {
      "id": "receive-contact",
      "type": "webhook",
      "config": {
        "method": "POST",
        "path": "getresponse-contact"
      }
    },
    {
      "id": "prepare-contact",
      "type": "function",
      "config": {
        "code": "return { name: input.name, email: input.email.toLowerCase().trim(), campaignId: input.campaignId };"
      }
    },
    {
      "id": "create-contact",
      "type": "getresponse",
      "config": {
        "operation": "createContact",
        "name": "{{input.name}}",
        "email": "{{input.email}}",
        "campaignId": "{{input.campaignId}}"
      }
    }
  ]
}
```

### Common Patterns

- **Lead Capture:** Webhook → Function → GetResponse `createContact`
- **Campaign Lookup:** Trigger → GetResponse `getCampaigns`
- **Contact Cleanup:** GetResponse `getContacts` → Filter → `deleteContact`
- **Newsletter Reporting:** GetResponse `getNewsletters` → Function → Store

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Authentication Error

**Cause:** Missing, invalid, or expired API key.

**Solution:** Generate a new GetResponse API key and paste only the key value.

### Campaign ID Rejected

**Cause:** An incorrect campaign identifier was used.

**Solution:** Run `getCampaigns` and use the returned `campaignId`.

### Contact Not Created

Possible causes:

- Invalid email address
- Duplicate email
- Invalid campaign ID
- Missing required field
- Double opt-in configuration

### Contact Not Deleted

**Cause:** Invalid or outdated `contactId`.

**Solution:** Run `getContacts` again and use the latest returned identifier.

### Empty Parameters in Export

**Cause:** The configuration form did not persist its values.

**Solution:** Save the node, export the workflow again, and verify that `parameters` is no longer empty.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Webhook](./webhook.md) - Receive contact or form data
- [Function](./function.md) - Validate and transform contact fields
- [HTTP Request](./http-request.md) - Call unsupported GetResponse operations

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.1 | 2026-08-05 | Updated supported operations and testing steps |
| 1.0.0 | 2026-08-05 | Initial release |

<!-- /SECTION: changelog -->
