---
node_id: "mailgun"
title: "Mailgun"
description: "Mailgun email service - Send emails, manage events, routes, domains, and validate emails."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-08"
author: "Fusion Team"
tags:
  - mailgun
  - email
  - transactional-email
  - integration
  - peer-only
related_nodes:
  - http-request
  - function
  - cron
---

<!-- SECTION: overview -->
# Mailgun

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Send e-mails and read sending data with [Mailgun](https://www.mailgun.com), the e-mail API service: send a message from one of your Mailgun domains, list your domains and inbound routes, read the events log (deliveries, failures, opens…), and validate an e-mail address. The node calls the Mailgun REST API with an API key.

### Use Cases

- **Transactional e-mails:** Send a notification, receipt, or alert when a workflow runs.
- **Delivery monitoring:** Read the events log to check which messages were delivered, failed, or bounced.
- **Account overview:** List the sending domains and inbound routes of the account.
- **Address checks:** Validate an e-mail address before using it (paid Mailgun plans only).

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | No | `sendEmail` | `sendEmail`, `getEvents`, `getRoutes`, `getDomains`, or `validateEmail`. |
| `apiKey` | `string` | Yes | — | Mailgun API key. Supports expressions. |
| `domain` | `string` | Yes | — | Mailgun sending domain, e.g. `mg.example.com` or a sandbox domain `sandboxXXXX.mailgun.org`. Used by `sendEmail` and `getEvents`. Supports expressions. |
| `from` | `string` | For `sendEmail` | — | Sender, e.g. `Support <support@mg.example.com>`. Must be an address of `domain`. Supports expressions. |
| `to` | `string` | For `sendEmail` | — | Recipient address(es), comma-separated. Supports expressions. |
| `subject` | `string` | For `sendEmail` | — | E-mail subject. Supports expressions. |
| `text` | `string` | No | — | Plain-text body. Supports expressions. |
| `html` | `string` | No | — | HTML body. Supports expressions. |
| `emailToValidate` | `string` | For `validateEmail` | — | Address to validate. Supports expressions. |
| `limit` | `number` | No | — | Maximum number of events returned by `getEvents` (Mailgun default: 100). |

### Available Operations

| Operation | Description | Mailgun request |
|-----------|-------------|-----------------|
| `sendEmail` | Send an e-mail from `domain`. | `POST https://api.mailgun.net/v3/<domain>/messages` (form data: `from`, `to`, `subject`, `text`, `html`) |
| `getEvents` | Read the events log of `domain`, newest first. | `GET https://api.mailgun.net/v3/<domain>/events?limit=<limit>` |
| `getRoutes` | List the inbound routes of the account. | `GET https://api.mailgun.net/v3/routes` |
| `getDomains` | List the domains of the account. | `GET https://api.mailgun.net/v3/domains` |
| `validateEmail` | Validate an e-mail address. | `GET https://api.mailgun.net/v4/address/validate?address=<emailToValidate>` |

> **US region only:** the node always calls `https://api.mailgun.net` (Mailgun's US region). Domains created in Mailgun's **EU region** (`api.eu.mailgun.net`) are not supported.

All parameters are shown for every operation. Provide at least `text` or `html` for `sendEmail`. The node does not check `from`, `to`, and `subject` before sending: if one is missing, Mailgun rejects the request with a `400` error.

### Authentication

Every request uses HTTP Basic authentication with the user `api` and the API key as password (`Authorization: Basic base64("api:<apiKey>")`).

### Getting an API Key and a Domain

1. Sign up at [mailgun.com](https://signup.mailgun.com) and choose the **US** region. The Free plan needs no credit card.
2. Activate the account if Mailgun asks for it (confirmation e-mail or phone verification). Until then, sending is refused with `Account disabled`.
3. Open **Send → Domains**. A **sandbox domain** (`sandboxXXXX.mailgun.org`) is created automatically; you can also add your own domain (DNS records required).
4. With a sandbox domain, add each recipient under **Authorized Recipients** and confirm the invitation e-mail. Sandbox domains can only send to authorized recipients.
5. Open **API Keys** (account settings), create a key, and copy it (it is shown only once).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. The incoming data is not used directly; use expressions in the parameters to pass upstream values. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The Mailgun API response (see examples). |
| `error` | `Error` | `Mailgun API Error: <status> <statusText> <response>`, e.g. `Mailgun API Error: 403 Forbidden {"message":"Email Validations are only available for paid accounts."}`. |

### Output Examples

#### `sendEmail`

```json
{
  "id": "<20261008160000.abc123@sandboxXXXX.mailgun.org>",
  "message": "Queued. Thank you."
}
```

#### `getDomains`

```json
{
  "total_count": 1,
  "items": [
    {
      "name": "sandboxa4d18ecc8f5442e2afcae5016092f9a9.mailgun.org",
      "created_at": "Thu, 08 Oct 2026 15:54:31 GMT",
      "state": "active",
      "type": "sandbox",
      "is_disabled": false,
      "require_tls": false,
      "spam_action": "disabled"
    }
  ]
}
```

#### `getRoutes`

```json
{
  "total_count": 0,
  "items": null
}
```

#### `getEvents`

```json
{
  "items": [],
  "paging": {
    "previous": "https://api.mailgun.net/v3/<domain>/events/…",
    "next": "https://api.mailgun.net/v3/<domain>/events/…"
  }
}
```

Each item of `items` describes one event (`event`: `accepted`, `delivered`, `failed`, `opened`, …) with its `timestamp`, `recipient`, and `message` headers. Events are kept for a limited time depending on the plan (1 day on the Free plan).

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Mailgun in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Mailgun Node:** Runs the configured operation (e.g. `sendEmail`).
3. **Log Node:** Displays the Mailgun response.

### Example Configurations

#### Send an e-mail

```json
{
  "operation": "sendEmail",
  "apiKey": "{{ secrets.MAILGUN_API_KEY }}",
  "domain": "mg.example.com",
  "from": "Alerts <alerts@mg.example.com>",
  "to": "{{ input.email }}",
  "subject": "Your order {{ input.orderId }} is confirmed",
  "html": "<p>Thank you for your order.</p>"
}
```

#### Read the latest events

```json
{
  "operation": "getEvents",
  "apiKey": "{{ secrets.MAILGUN_API_KEY }}",
  "domain": "mg.example.com",
  "limit": 20
}
```

### Common Patterns

- **Webhook → Mailgun (`sendEmail`):** Send a confirmation e-mail when a form is submitted.
- **Cron → Mailgun (`getEvents`) → Function → Notification:** Report failed deliveries once a day.
- **Function → Mailgun (`sendEmail`):** Build the HTML body from upstream data, then send it.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `apiKey` in Fusion's **Secrets** system. Never paste it directly into workflow parameters or commit it to version control.

- The API key gives full access to the Mailgun account (sending, domains, logs); create a dedicated key for Fusion and delete it when it is no longer used.
- Only send to recipients who expect your e-mails, and use a verified domain in production to protect your sender reputation.
- E-mail content and event logs can contain personal data; avoid logging full responses in shared workflows.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Mailgun API Error: 401 Unauthorized`
- **Cause:** Wrong or deleted API key, or an account/domain in Mailgun's **EU region** (the node only supports the US region, `api.mailgun.net`).
- **Solution:** Create a new key under **API Keys** and update `apiKey`; use a US-region Mailgun domain.

#### `403 Forbidden … is not allowed to send: Account disabled`
- **Cause:** The Mailgun account is not activated yet, or was disabled by Mailgun.
- **Solution:** Complete the activation steps in the Mailgun dashboard (confirmation e-mail, phone verification) or contact Mailgun support.

#### `403 Forbidden … Email Validations are only available for paid accounts.`
- **Cause:** `validateEmail` uses Mailgun's validation service, which is not included in the Free plan.
- **Solution:** Upgrade the Mailgun plan, or skip address validation.

#### Sandbox: recipient not authorized
- **Cause:** Sandbox domains only send to **Authorized Recipients**.
- **Solution:** Add and confirm the recipient on the sandbox domain page, or use a verified custom domain.

#### `400 Bad Request` on `sendEmail`
- **Cause:** `from`, `to`, or `subject` is missing (sent as `undefined`), `from` is not an address of `domain`, or neither `text` nor `html` is set.
- **Solution:** Fill in all sending fields and use a `from` address on your Mailgun domain.

#### `404 Not Found` on `sendEmail` or `getEvents`
- **Cause:** `domain` is wrong or does not belong to the account.
- **Solution:** Copy the exact domain name from `getDomains` or **Send → Domains**.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-08 | Full documentation: parameters, operations, authentication, account and domain setup, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->