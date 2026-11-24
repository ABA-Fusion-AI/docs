---
node_id: "gotowebinar"
title: "GoToWebinar"
description: "List GoToWebinar webinars, read their registrants and attendees, and register new participants through the GoToWebinar REST API v2."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-11-24"
author: "Fusion Team"
tags:
  - gotowebinar
  - goto
  - webinar
  - registration
  - marketing
related_nodes:
  - demio
---

<!-- SECTION: overview -->
# GoToWebinar

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Work with **GoToWebinar** from a workflow through the **GoToWebinar REST API v2** (`https://api.getgo.com/G2W/rest/v2`): list the webinars of an organizer, read their **registrants** and **attendees**, and **register** new participants. The node offers **4 operations**:

| Operation | Description |
|-----------|-------------|
| `getWebinars` | Webinars of the organizer within a time range. |
| `getRegistrants` | People registered to a webinar, with their join URL. |
| `createRegistrant` | Register a person to a webinar. |
| `getAttendees` | People who attended a webinar (after it took place). |

### Use Cases

- **Form to webinar:** Register leads from a form or a CRM automatically.
- **Follow-up:** After a webinar, read the attendees and send them a recording or a survey.
- **Reporting:** Export registrants and attendees to a spreadsheet or a dashboard.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `apiKey` | `string` | Yes | **GoTo OAuth access token** (not a static API key), sent as `Authorization: Bearer <token>`. It expires after **1 hour**. |
| `organizerKey` | `string` | Yes | Organizer key of the GoToWebinar account (numeric, e.g. `6031694749179231984`). |

#### Get an access token

1. On **developer.goto.com → OAuth Clients**, create a client with a redirect URI (e.g. `https://localhost`) and the GoToWebinar scopes. Note the **Client ID** and **Client Secret**.
2. Open `https://authentication.logmeininc.com/oauth/authorize?client_id=<CLIENT_ID>&response_type=code&redirect_uri=https://localhost`, sign in, allow, and copy the `code` from the redirect URL.
3. Exchange the code for a token:

```bash
curl -X POST "https://authentication.logmeininc.com/oauth/token" \
  -H "Authorization: Basic <base64(clientId:clientSecret)>" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code&code=<CODE>&redirect_uri=https://localhost"
```

4. Put `access_token` in `apiKey`. The **organizer key** is the account user key: the `sub` claim of the access token (also the `organizerKey` field returned by `getWebinars`).

> The response also contains a `refresh_token` (valid for weeks) that can get new access tokens with `grant_type=refresh_token`.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | Yes | `getWebinars` (default), `getRegistrants`, `createRegistrant` or `getAttendees`. |
| `webinarKey` | `string` | For all except `getWebinars` | Webinar key (19 digits, e.g. `9085574001675911936`), from `getWebinars`. |
| `firstName` | `string` | For `createRegistrant` | Registrant first name. |
| `lastName` | `string` | For `createRegistrant` | Registrant last name. |
| `email` | `string` | For `createRegistrant` | Registrant email. |
| `fromTime` | `string` | No | `getWebinars`: start of the range (ISO 8601). Default: 1 year ago. |
| `toTime` | `string` | No | `getWebinars`: end of the range (ISO 8601). Default: 1 year from now. |

> All parameters are shown for every operation; only the ones listed above are used.

> GoTo keys (`webinarKey`, `registrantKey`, `organizerKey`) are 19-digit numbers. The node keeps them as **strings** so they are not rounded.

### Operations

| Operation | Request | Returns |
|-----------|---------|---------|
| `getWebinars` | `GET /organizers/{organizerKey}/webinars?fromTime=&toTime=` | `{ _embedded: { webinars: [...] }, page }` |
| `getRegistrants` | `GET /organizers/{organizerKey}/webinars/{webinarKey}/registrants` | Array of registrants. |
| `createRegistrant` | `POST /organizers/{organizerKey}/webinars/{webinarKey}/registrants` with `{ firstName, lastName, email }` | `{ registrantKey, joinUrl, status }` |
| `getAttendees` | `GET /organizers/{organizerKey}/webinars/{webinarKey}/attendees` | Paged list of attendees. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values (e.g. a name and email from a form) in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` / `array` | The GoToWebinar API response. |
| `error` | `Error` | `GoToWebinar error (<status>): <response>` or `Invalid date: ...`. |

### Output Examples

#### `getWebinars`

```json
{
  "_embedded": {
    "webinars": [
      {
        "webinarKey": "9085574001675911936",
        "webinarId": "315794939",
        "subject": "Sample Webinar",
        "organizerKey": "6031694749179231984",
        "status": "NEW",
        "times": [{ "startTime": "2027-04-07T15:52:35Z", "endTime": "2027-04-07T16:52:35Z" }],
        "timeZone": "America/Los_Angeles",
        "registrationUrl": "https://attendee.gotowebinar.com/register/..."
      }
    ]
  },
  "page": { "size": 20, "totalElements": 1, "totalPages": 1, "number": 0 }
}
```

#### `getRegistrants`

```json
[
  {
    "registrantKey": "1957588552890607192",
    "firstName": "Fusion",
    "lastName": "Test",
    "email": "academy@example.com",
    "status": "APPROVED",
    "registrationDate": "2026-10-08T10:03:18Z",
    "joinUrl": "https://global.gotowebinar.com/join/9085574001675911936/218587917",
    "timeZone": "America/Los_Angeles"
  }
]
```

#### `getAttendees` (webinar not held yet)

```json
{ "page": { "size": 20, "totalElements": 0, "totalPages": 0, "number": 0 } }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use GoToWebinar in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **GoToWebinar Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### List the webinars of the next 3 months

```json
{
  "apiKey": "{{ secrets.GOTO_ACCESS_TOKEN }}",
  "organizerKey": "6031694749179231984",
  "operation": "getWebinars",
  "fromTime": "2026-11-24T00:00:00Z",
  "toTime": "2027-02-24T00:00:00Z"
}
```

#### Register a participant from a form

```json
{
  "apiKey": "{{ secrets.GOTO_ACCESS_TOKEN }}",
  "organizerKey": "6031694749179231984",
  "operation": "createRegistrant",
  "webinarKey": "9085574001675911936",
  "firstName": "{{ input.firstName }}",
  "lastName": "{{ input.lastName }}",
  "email": "{{ input.email }}"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the access token (and any refresh token or client secret) in Fusion's **Secrets** vault. Do not commit them into version control or paste them in chats.

- `createRegistrant` sends a real confirmation email to the registrant: validate the input and protect it from repeated triggers.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `GoToWebinar error (403): {"int_err_code":"InvalidToken","msg":"Invalid token passed"}`
- **Cause:** The access token expired (1 hour) or is not a GoTo OAuth token.
- **Solution:** Get a new access token (or use the refresh token) and update `apiKey`.

#### `GoToWebinar error (403): {"errorCode":"RegistrationApiAccessDenied", ...}`
- **Cause:** The GoToWebinar account is not allowed to register participants through the API (plan or account setting).
- **Solution:** Use an account/plan with API registration enabled. Reading webinars and registrants still works.

#### `GoToWebinar error (404)` / empty results
- **Cause:** Wrong `organizerKey` or `webinarKey`, or no webinar in the time range.
- **Solution:** Use the `sub` of the token as organizer key, and widen `fromTime` / `toTime`.

#### `getAttendees` returns `totalElements: 0`
- **Cause:** The webinar has not taken place yet.
- **Solution:** Expected; attendees exist only after a session.

#### `Invalid date: ...`
- **Cause:** `fromTime` / `toTime` is not a valid date.
- **Solution:** Use ISO 8601, e.g. `2026-11-24T00:00:00Z`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-24 | Full documentation: 4 operations, OAuth access token and organizer key, output examples, troubleshooting |

<!-- /SECTION: changelog -->
