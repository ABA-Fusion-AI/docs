---
node_id: "convertkit"

title: "ConvertKit"

description: "Manage subscribers, forms, sequences, tags, and broadcasts in ConvertKit."

category: "Marketing / Email"

version: "1.0.0"

language: "en"

last_updated: "2026-09-17"

author: "Fusion Team"

tags:

- convertkit

- email-marketing

- subscribers

- forms

- sequences

- tags

- broadcasts

related_nodes:

- http-request

- function

- if

---

**# ConvertKit**

> **\*\*Category:\*\*** marketing-nodes | **\*\*Type:\*\*** Action Node

Connect workflows to ConvertKit and manage subscribers, forms, sequences, tags, and broadcasts through its API.

The **\*\*ConvertKit\*\*** node supports listing and managing subscribers, subscribing users to forms and sequences, tagging subscribers, and retrieving ConvertKit resources.

**### Supported Features**

\- List subscribers

\- Retrieve a specific subscriber

\- Create a subscriber

\- Update subscriber information

\- Unsubscribe an email address

\- List forms

\- List sequences

\- Add a subscriber to a form

\- Add a subscriber to a sequence

\- List tags

\- Tag a subscriber

\- List broadcasts

\- Configure the number of subscriber results

\- Return the parsed ConvertKit response directly

**### Use Cases**

\- Manage ConvertKit subscribers inside workflows

\- Subscribe users to forms

\- Add subscribers to email sequences

\- Tag subscribers for segmentation

\- Update subscriber information

\- Unsubscribe email addresses

\- Retrieve forms, sequences, tags, and broadcasts

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `operation` | `enum` | ❌ No | `"listSubscribers"` | Operation to perform. |
| `apiKey` | `string` | ⚠️ Runtime required | — | ConvertKit API key. |
| `apiSecret` | `string` | ⚠️ Runtime required | — | ConvertKit API secret. |
| `subscriberId` | `string` | ❌ No | — | Subscriber ID. |
| `email` | `string` | ❌ No | — | Subscriber email address. |
| `firstName` | `string` | ❌ No | — | Subscriber first name. |
| `formId` | `string` | ❌ No | — | Form ID. |
| `sequenceId` | `string` | ❌ No | — | Sequence ID. |
| `tagId` | `string` | ❌ No | — | Tag ID. |
| `limit` | `number` | ❌ No | `25` | Number of results to return. |

Although `apiKey` and `apiSecret` are optional in the schema, the implementation requires both before executing any operation.

**### Supported Operations**

```text
listSubscribers
getSubscriber
createSubscriber
updateSubscriber
unsubscribe
listForms
listSequences
addSubscriberToForm
addSubscriberToSequence
listTags
tagSubscriber
listBroadcasts
```

Default operation:

```text
listSubscribers
```

**---**

**## Operations**

**### listSubscribers**

Retrieves subscribers.

Request:

```text
GET /subscribers?api_secret=<apiSecret>&count=<limit>
```

Default `limit`:

```text
25
```

---

**### getSubscriber**

Retrieves one subscriber.

Required:

```text
subscriberId
```

Request:

```text
GET /subscribers/<subscriberId>?api_secret=<apiSecret>
```

---

**### createSubscriber**

Creates a subscriber through a form.

Required:

```text
formId
```

Request:

```text
POST /forms/<formId>/subscribe
```

Body:

```json
{
  "api_key": "YOUR_API_KEY",
  "email": "subscriber@example.com",
  "first_name": "Example"
}
```

---

**### updateSubscriber**

Updates an existing subscriber.

Required:

```text
subscriberId
```

Request:

```text
PUT /subscribers/<subscriberId>
```

Body:

```json
{
  "api_secret": "YOUR_API_SECRET",
  "first_name": "Example",
  "email": "subscriber@example.com"
}
```

---

**### unsubscribe**

Unsubscribes an email address.

Required:

```text
email
```

Request:

```text
PUT /unsubscribe
```

Body:

```json
{
  "api_secret": "YOUR_API_SECRET",
  "email": "subscriber@example.com"
}
```

---

**### listForms**

Retrieves forms.

```text
GET /forms?api_key=<apiKey>
```

---

**### listSequences**

Retrieves sequences.

```text
GET /sequences?api_key=<apiKey>
```

---

**### addSubscriberToForm**

Adds a subscriber to a form.

Required:

```text
formId
```

Request:

```text
POST /forms/<formId>/subscribe
```

Body:

```json
{
  "api_key": "YOUR_API_KEY",
  "email": "subscriber@example.com"
}
```

---

**### addSubscriberToSequence**

Adds a subscriber to a sequence.

Required:

```text
sequenceId
```

Request:

```text
POST /sequences/<sequenceId>/subscribe
```

Body:

```json
{
  "api_key": "YOUR_API_KEY",
  "email": "subscriber@example.com"
}
```

---

**### listTags**

Retrieves tags.

```text
GET /tags?api_key=<apiKey>
```

---

**### tagSubscriber**

Tags a subscriber.

Required:

```text
tagId
```

Request:

```text
POST /tags/<tagId>/subscribe
```

Body:

```json
{
  "api_key": "YOUR_API_KEY",
  "email": "subscriber@example.com"
}
```

---

**### listBroadcasts**

Retrieves broadcasts.

```text
GET /broadcasts?api_key=<apiKey>
```

**---**

**## Request Construction**

The base URL is:

```text
https://api.convertkit.com/v3
```

Every request includes:

```text
Content-Type: application/json
```

For `POST` and `PUT`, request bodies are serialized using:

```ts
JSON.stringify(body)
```

Credentials are supplied through query parameters or JSON request bodies depending on the operation.

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

Successful responses are parsed using:

```ts
await response.json()
```

and returned directly.

The node does not add a custom output wrapper.

**### Output Example**

The exact output depends on the selected operation and ConvertKit response.

```json
{
  "...": "raw ConvertKit API response"
}
```

**---**

**## Configuration Examples**

**### List Subscribers**

```json
{
  "operation": "listSubscribers",
  "apiKey": "YOUR_API_KEY",
  "apiSecret": "YOUR_API_SECRET",
  "limit": 25
}
```

**### Get Subscriber**

```json
{
  "operation": "getSubscriber",
  "apiKey": "YOUR_API_KEY",
  "apiSecret": "YOUR_API_SECRET",
  "subscriberId": "SUBSCRIBER_ID"
}
```

**### Create Subscriber**

```json
{
  "operation": "createSubscriber",
  "apiKey": "YOUR_API_KEY",
  "apiSecret": "YOUR_API_SECRET",
  "formId": "FORM_ID",
  "email": "subscriber@example.com",
  "firstName": "Example"
}
```

**### Update Subscriber**

```json
{
  "operation": "updateSubscriber",
  "apiKey": "YOUR_API_KEY",
  "apiSecret": "YOUR_API_SECRET",
  "subscriberId": "SUBSCRIBER_ID",
  "email": "subscriber@example.com",
  "firstName": "Updated"
}
```

**### Unsubscribe**

```json
{
  "operation": "unsubscribe",
  "apiKey": "YOUR_API_KEY",
  "apiSecret": "YOUR_API_SECRET",
  "email": "subscriber@example.com"
}
```

**### Add Subscriber to Form**

```json
{
  "operation": "addSubscriberToForm",
  "apiKey": "YOUR_API_KEY",
  "apiSecret": "YOUR_API_SECRET",
  "formId": "FORM_ID",
  "email": "subscriber@example.com"
}
```

**### Add Subscriber to Sequence**

```json
{
  "operation": "addSubscriberToSequence",
  "apiKey": "YOUR_API_KEY",
  "apiSecret": "YOUR_API_SECRET",
  "sequenceId": "SEQUENCE_ID",
  "email": "subscriber@example.com"
}
```

**### Tag Subscriber**

```json
{
  "operation": "tagSubscriber",
  "apiKey": "YOUR_API_KEY",
  "apiSecret": "YOUR_API_SECRET",
  "tagId": "TAG_ID",
  "email": "subscriber@example.com"
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → ConvertKit (`createSubscriber`)

\- ConvertKit (`listSubscribers`) → Function

\- Form Submission → ConvertKit (`addSubscriberToForm`)

\- ConvertKit (`addSubscriberToSequence`) → Notification

\- ConvertKit (`tagSubscriber`) → Data Processing

\- ConvertKit (`listBroadcasts`) → Function

**---**

**## Error Handling**

The node validates both credentials before executing an operation.

Missing API key:

```text
apiKey is required
```

Missing API secret:

```text
apiSecret is required
```

For unsuccessful API responses:

```text
ConvertKit API Error: <status> <statusText> - <error body>
```

Operation-specific validation errors include:

```text
subscriberId is required for getSubscriber
formId is required for createSubscriber
subscriberId is required for updateSubscriber
email is required for unsubscribe
formId is required for addSubscriberToForm
sequenceId is required for addSubscriberToSequence
tagId is required for tagSubscriber
Unknown operation: <operation>
```

**### Missing Email**

The implementation explicitly requires `email` only for `unsubscribe`.

The following operations send `email` but do not explicitly validate it:

```text
createSubscriber
addSubscriberToForm
addSubscriberToSequence
tagSubscriber
```

**---**

**## Troubleshooting**

**### ConvertKit Authentication Error**

Verify:

```text
apiKey
apiSecret
```

Both values must be configured because the implementation checks them before executing the selected operation.

---

**### Subscriber Is Not Added**

Verify the corresponding `formId`, `sequenceId`, or `tagId`.

Also verify that `email` is configured when the API operation requires it.

---

**### Incoming Data Is Ignored**

`_incomingData` is accepted by `handleTick()` but is not used.

Configure values through node parameters.

**---**

**## Security**

The node handles both a ConvertKit API key and API secret.

For production workflows:

\- Store credentials securely

\- Never commit real credentials to Git

\- Do not include real credentials in workflow examples

\- Avoid logging URLs containing `api_key` or `api_secret`

\- Avoid logging request bodies containing credentials

\- Rotate exposed credentials

**---**

**## Notes**

Metadata label:

```text
ConvertKit
```

Metadata description:

```text
Manage subscribers, forms, sequences, and broadcasts in ConvertKit.
```

ConvertKit API base:

```text
https://api.convertkit.com/v3
```

Default operation:

```text
listSubscribers
```

Default limit:

```text
25
```

Supported operations:

```text
listSubscribers
getSubscriber
createSubscriber
updateSubscriber
unsubscribe
listForms
listSequences
addSubscriberToForm
addSubscriberToSequence
listTags
tagSubscriber
listBroadcasts
```

The node does not:

\- Use incoming workflow data

\- Implement retries

\- Cache responses

\- Transform successful responses

\- Implement DELETE requests

\- Explicitly URL-encode path IDs

\- Explicitly validate `email` for every subscription operation

Both `apiKey` and `apiSecret` are required globally by the runtime implementation.

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| --- | --- | --- |
| `1.0.0` | `2026-09-17` | Initial release |
