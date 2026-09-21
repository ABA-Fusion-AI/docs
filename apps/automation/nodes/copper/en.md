---
node_id: "copper"

title: "Copper CRM"

description: "Manage people, companies, opportunities, and activities in Copper CRM."

category: "CRM / Sales"

version: "1.0.0"

language: "en"

last_updated: "2026-09-17"

author: "Fusion Team"

tags:

- copper

- crm

- sales

- people

- companies

- opportunities

- activities

related_nodes:

- http-request

- function

- if

---

**# Copper CRM**

> **\*\*Category:\*\*** crm-nodes | **\*\*Type:\*\*** Action Node

Connect workflows to Copper CRM and manage people, companies, opportunities, and activities through its Developer API.

The **\*\*Copper CRM\*\*** node supports retrieving, creating, updating, and deleting people, managing companies and opportunities, and listing activities using a configured API key and Copper account email.

**### Supported Features**

\- List people

\- Retrieve a specific person

\- Create a person

\- Update a person

\- Delete a person

\- List companies

\- Retrieve a specific company

\- Create a company

\- List opportunities

\- Retrieve a specific opportunity

\- Create an opportunity

\- List activities

\- Configure the number of results returned

\- Authenticate using Copper Developer API headers

\- Return parsed Copper API responses directly

**### Use Cases**

\- Manage CRM contacts inside workflows

\- Create and update Copper people

\- Retrieve company information

\- Create companies

\- Manage sales opportunities

\- Retrieve CRM activities

\- Connect Copper CRM to automated sales workflows

**---**
<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Copper CRM in a workflow
```
<!-- /SECTION: examples -->

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `operation` | `enum` | ❌ No | `"listPeople"` | Operation to perform. |
| `apiKey` | `string` | ⚠️ Runtime required | — | Copper CRM API key. |
| `userEmail` | `string` | ⚠️ Runtime required | — | Copper account user email. |
| `personId` | `string` | ❌ No | — | Person ID. |
| `companyId` | `string` | ❌ No | — | Company ID. |
| `opportunityId` | `string` | ❌ No | — | Opportunity ID. |
| `name` | `string` | ❌ No | — | Name of the person, company, or opportunity. |
| `email` | `string` | ❌ No | — | Email address. |
| `phoneNumber` | `string` | ❌ No | — | Phone number. |
| `website` | `string` | ❌ No | — | Company website URL. |
| `contactId` | `string` | ❌ No | — | Primary contact ID for the opportunity. |
| `pipelineId` | `string` | ❌ No | — | Pipeline ID for the opportunity. |
| `stageId` | `string` | ❌ No | — | Pipeline stage ID for the opportunity. |
| `value` | `number` | ❌ No | — | Monetary value of the opportunity. |
| `limit` | `number` | ❌ No | `25` | Number of results to return. |

Although `apiKey` and `userEmail` are optional in the schema, the implementation requires both before executing any operation.

**### Supported Operations**

```text
listPeople
getPerson
createPerson
updatePerson
deletePerson
listCompanies
getCompany
createCompany
listOpportunities
getOpportunity
createOpportunity
listActivities
```

Default operation:

```text
listPeople
```

**---**

**## Operations**

**### listPeople**

Retrieves people using the Copper search endpoint.

Request:

```text
POST /people/search
```

Body:

```json
{
  "page_size": 25
}
```

---

**### getPerson**

Retrieves one person.

Required:

```text
personId
```

Request:

```text
GET /people/<personId>
```

---

**### createPerson**

Creates a person.

Request:

```text
POST /people
```

Body:

```json
{
  "name": "Example Person",
  "emails": [
    {
      "email": "person@example.com",
      "category": "work"
    }
  ],
  "phone_numbers": [
    {
      "number": "+10000000000",
      "category": "work"
    }
  ]
}
```

If `email` or `phoneNumber` is not configured, the corresponding array is sent empty.

---

**### updatePerson**

Updates an existing person.

Required:

```text
personId
```

Request:

```text
PATCH /people/<personId>
```

Body:

```json
{
  "name": "Updated Person",
  "emails": [
    {
      "email": "person@example.com",
      "category": "work"
    }
  ]
}
```

The implementation does not include `phoneNumber` in the update request.

---

**### deletePerson**

Deletes a person.

Required:

```text
personId
```

Request:

```text
DELETE /people/<personId>
```

Successful output:

```json
{
  "success": true
}
```

---

**### listCompanies**

Request:

```text
POST /companies/search
```

Body:

```json
{
  "page_size": 25
}
```

---

**### getCompany**

Required:

```text
companyId
```

Request:

```text
GET /companies/<companyId>
```

---

**### createCompany**

Request:

```text
POST /companies
```

Body:

```json
{
  "name": "Example Company",
  "website": "https://example.com",
  "phone_numbers": [
    {
      "number": "+10000000000",
      "category": "work"
    }
  ]
}
```

---

**### listOpportunities**

Request:

```text
POST /opportunities/search
```

Body:

```json
{
  "page_size": 25
}
```

---

**### getOpportunity**

Required:

```text
opportunityId
```

Request:

```text
GET /opportunities/<opportunityId>
```

---

**### createOpportunity**

Request:

```text
POST /opportunities
```

Body:

```json
{
  "name": "Example Opportunity",
  "primary_contact_id": "CONTACT_ID",
  "pipeline_id": "PIPELINE_ID",
  "pipeline_stage_id": "STAGE_ID",
  "monetary_value": 1000
}
```

The implementation does not explicitly validate these creation fields before sending the request.

---

**### listActivities**

Request:

```text
POST /activities/search
```

Body:

```json
{
  "page_size": 25
}
```

**---**

**## Request Construction**

The base URL is:

```text
https://api.copper.com/developer_api/v1
```

Authentication uses:

```text
X-PW-AccessToken: <apiKey>
X-PW-Application: developer_api
X-PW-UserEmail: <userEmail>
Content-Type: application/json
```

The node supports:

```text
GET
POST
PATCH
DELETE
```

For non-GET and non-DELETE requests, the body is serialized using:

```ts
JSON.stringify(body)
```

**---**

**## Inputs & Outputs**

**### Inputs**

`handleTick()` receives:

```text
_incomingData
```

The implementation does not use incoming workflow data.

All request values come from node configuration.

**### Outputs**

Successful non-DELETE responses are parsed using:

```ts
await response.json()
```

and returned directly.

Successful DELETE requests return:

```json
{
  "success": true
}
```

**### Output Example**

The exact output depends on the selected operation and Copper API response.

```json
{
  "...": "raw Copper API response"
}
```

**---**

**## Configuration Examples**

**### List People**

```json
{
  "operation": "listPeople",
  "apiKey": "YOUR_COPPER_API_KEY",
  "userEmail": "user@example.com",
  "limit": 25
}
```

**### Get Person**

```json
{
  "operation": "getPerson",
  "apiKey": "YOUR_COPPER_API_KEY",
  "userEmail": "user@example.com",
  "personId": "PERSON_ID"
}
```

**### Create Person**

```json
{
  "operation": "createPerson",
  "apiKey": "YOUR_COPPER_API_KEY",
  "userEmail": "user@example.com",
  "name": "Example Person",
  "email": "person@example.com",
  "phoneNumber": "+10000000000"
}
```

**### Create Company**

```json
{
  "operation": "createCompany",
  "apiKey": "YOUR_COPPER_API_KEY",
  "userEmail": "user@example.com",
  "name": "Example Company",
  "website": "https://example.com",
  "phoneNumber": "+10000000000"
}
```

**### Create Opportunity**

```json
{
  "operation": "createOpportunity",
  "apiKey": "YOUR_COPPER_API_KEY",
  "userEmail": "user@example.com",
  "name": "Example Opportunity",
  "contactId": "CONTACT_ID",
  "pipelineId": "PIPELINE_ID",
  "stageId": "STAGE_ID",
  "value": 1000
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → Copper CRM (`createPerson`)

\- Copper CRM (`listPeople`) → Function

\- Form Submission → Copper CRM (`createPerson`)

\- Copper CRM (`createCompany`) → Copper CRM (`createOpportunity`)

\- Copper CRM (`listOpportunities`) → Data Processing

\- Copper CRM (`listActivities`) → Notification

**---**

**## Error Handling**

The node validates both authentication values before executing an operation.

Missing API key:

```text
apiKey is required
```

Missing user email:

```text
userEmail is required
```

For unsuccessful API responses:

```text
Copper API Error: <status> <statusText> - <error body>
```

Operation-specific validation errors include:

```text
personId is required for getPerson
personId is required for updatePerson
personId is required for deletePerson
companyId is required for getCompany
opportunityId is required for getOpportunity
Unknown operation: <operation>
```

**---**

**## Troubleshooting**

**### Copper Authentication Error**

Verify:

```text
apiKey
userEmail
```

The request sends:

```text
X-PW-AccessToken: <apiKey>
X-PW-Application: developer_api
X-PW-UserEmail: <userEmail>
```

---

**### Person Cannot Be Retrieved**

Verify `personId` for `getPerson`, `updatePerson`, and `deletePerson`.

---

**### Company Cannot Be Retrieved**

Verify `companyId` for `getCompany`.

---

**### Opportunity Cannot Be Retrieved**

Verify `opportunityId` for `getOpportunity`.

---

**### Wrong Number of Results**

The list operations send:

```text
page_size: <limit>
```

Default:

```text
25
```

This applies to `listPeople`, `listCompanies`, `listOpportunities`, and `listActivities`.

---

**### Incoming Data Is Ignored**

`_incomingData` is accepted by `handleTick()` but is not referenced by the implementation.

Configure values through node parameters.

**---**

**## Security**

The node authenticates using the Copper API key and Copper account email.

For production workflows:

\- Store the API key securely

\- Never commit real Copper credentials to Git

\- Do not include real credentials in workflow examples

\- Avoid logging authentication headers

\- Rotate exposed API keys

**---**

**## Notes**

Metadata label:

```text
Copper CRM
```

Metadata description:

```text
Manage people, companies, opportunities, and activities in Copper CRM.
```

Copper API base:

```text
https://api.copper.com/developer_api/v1
```

Default operation:

```text
listPeople
```

Default limit:

```text
25
```

Supported operations:

```text
listPeople
getPerson
createPerson
updatePerson
deletePerson
listCompanies
getCompany
createCompany
listOpportunities
getOpportunity
createOpportunity
listActivities
```

The node does not:

\- Use incoming workflow data

\- Implement retries

\- Cache responses

\- Transform successful non-DELETE responses

\- Implement pagination beyond sending `page_size`

\- Explicitly URL-encode IDs inserted into endpoint paths

\- Explicitly validate all create-operation fields

The `updatePerson` operation does not send `phoneNumber`.

Successful DELETE requests return `{ "success": true }` without parsing the response body.

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| --- | --- | --- |
| `1.0.0` | `2026-09-17` | Initial release |
