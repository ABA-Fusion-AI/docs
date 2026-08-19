---
node_id: "smtp-send-email"
title: "SMTP: Send Email"
description: "Send emails via any custom SMTP server (Gmail, Outlook, Postfix, etc.)"
category: "Communication / Email"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:

- smtp
- email
- nodemailer
- notifications
- gmail
- outlook
- postfix
- tls

related_nodes:
- function
- if

---

# SMTP: Send Email

> **Category:** communication-nodes | **Type:** Action Node

Send an email via **any custom SMTP server** — Gmail, Outlook/Office 365, Postfix, or any other SMTP-compatible provider — using Nodemailer.

The **SMTP: Send Email** node configures a Nodemailer transport from the provided server and authentication settings, **verifies the connection** before sending, and delivers the email with plain-text and/or HTML content.

### Supported Features

- Connect to any SMTP host/port
- Automatic secure-connection detection based on port (falls back to `port === 465` when `secure` is not explicitly set)
- STARTTLS or direct TLS/SSL support
- Optional TLS certificate validation bypass (`ignoreTLS`), for self-signed certificates
- Optional authentication (username/password), toggleable independently of TLS settings
- Multiple recipients via comma-separated `to` field
- HTML and/or plain-text email body
- Pre-send connection verification with **contextual troubleshooting hints** for common port/security mismatches
- Full send result including accepted/rejected recipient lists

### Use Cases

- Send transactional emails (order confirmations, password resets, alerts) from a workflow
- Send notifications through a company's own SMTP relay instead of a third-party email API
- Integrate with Gmail, Outlook/Office 365, or a self-hosted Postfix server
- Send HTML-formatted reports or digests generated earlier in a workflow
- Alert a team or customer when a workflow condition is met

---

## Configuration

### Server Connection

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `host` | `string` | ✅ Yes | — | SMTP server hostname, e.g. `smtp.gmail.com` or `smtp.office365.com`. |
| `port` | `number` | ❌ No | `587` | SMTP port. Common values: `587` (STARTTLS), `465` (SSL/TLS), `25` (unsecured). |

### Security Settings

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `secure` | `boolean` | ❌ No | `false` | If `true`, connects using TLS directly. If `false`, uses STARTTLS when the server supports it. Set `true` for port 465; keep `false` for 587 or 25. |
| `ignoreTLS` | `boolean` | ❌ No | `false` | If `true`, accepts invalid/self-signed certificates. Use with caution. |

### Authentication

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `authEnabled` | `boolean` | ❌ No | `true` | Whether the SMTP server requires login credentials. |
| `user` | `string` | ✅ Yes (if `authEnabled`) | — | SMTP username/email. |
| `pass` | `string` | ✅ Yes (if `authEnabled`) | — | SMTP password or app password. |

### Email Content

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `from` | `string` | ✅ Yes | — | Sender name/email, e.g. `"Support <support@example.com>"`. |
| `to` | `string` | ✅ Yes | — | Recipient email(s), comma-separated. |
| `subject` | `string` | ✅ Yes | — | Email subject line. |
| `html` | `string` | ❌ No | — | HTML body content. |
| `text` | `string` | ❌ No | — | Plain-text body content (fallback for clients that don't render HTML). |

At least one of `html` or `text` should typically be provided, though the schema does not enforce this — an email with neither will be sent with an empty body.

---

## How It Works

1. **Determine the `secure` setting** — if `secure` is left unset, it is auto-detected as `true` only when `port === 465`; otherwise `false`.
2. **Build the Nodemailer transport config** — `host`, `port`, `secure`, and a `tls.rejectUnauthorized` flag (inverted from `ignoreTLS`).
3. **Add authentication** — if `authEnabled` is `true`, `user` and `pass` are required and attached to the transport config; the node throws if either is missing.
4. **Create the transporter** via `nodemailer.createTransport(...)`.
5. **Verify the connection** with `transporter.verify()` before attempting to send — this catches connection/auth/TLS issues early, with additional hints for common `secure`/port mismatches (see [Connection Verification Hints](#connection-verification-hints)).
6. **Send the email** with `transporter.sendMail(...)`, splitting `to` on commas and trimming whitespace to support multiple recipients.
7. **Return** the send result, including message ID and accepted/rejected recipient lists.

---

## Connection Verification Hints

If `transporter.verify()` fails with an error message containing `"wrong version number"` or `"SSL routines"` (classic TLS/SSL mismatch signatures), the node appends a contextual hint to the thrown error:

| Condition | Hint |
| --------- | ---- |
| `port === 587` and `secure` resolved to `true` | Suggests setting `secure` to `false` (587 typically uses STARTTLS). |
| `port === 465` and `secure` resolved to `false` | Suggests setting `secure` to `true` (465 typically uses SSL/TLS). |
| Any other port/secure combination | Generic reminder: port 465 → `secure: true`, port 587 → `secure: false`. |

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration — including email content — is provided through the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` if the send completed without throwing. |
| `messageId` | `string` | The email's message ID, as assigned by the sending process. |
| `response` | `string` | Raw SMTP server response string for the send operation. |
| `accepted` | `string[]` | List of recipient addresses the server accepted. |
| `rejected` | `string[]` | List of recipient addresses the server rejected. |

Note: a non-empty `rejected` array does **not** cause the node to throw — partial delivery failures are reported in the output, not as an error.

---

## Output Example

```json
{
  "success": true,
  "messageId": "<1a2b3c4d5e6f@smtp.gmail.com>",
  "response": "250 2.0.0 OK  1755590400 a1b2-c3d4e5f6g7h8-1 - gsmtp",
  "accepted": ["customer@example.com"],
  "rejected": []
}
```

### Partial Rejection

```json
{
  "success": true,
  "messageId": "<1a2b3c4d5e6f@smtp.gmail.com>",
  "response": "250 2.0.0 OK",
  "accepted": ["valid@example.com"],
  "rejected": ["invalid-address"]
}
```

---

## Configuration Examples

### Gmail (App Password, STARTTLS on 587)

```json
{
  "host": "smtp.gmail.com",
  "port": 587,
  "secure": false,
  "authEnabled": true,
  "user": "you@gmail.com",
  "pass": "your-app-password",
  "from": "Support <you@gmail.com>",
  "to": "customer@example.com",
  "subject": "Your order has shipped",
  "html": "<p>Your order is on its way!</p>"
}
```

### Office 365 (STARTTLS on 587)

```json
{
  "host": "smtp.office365.com",
  "port": 587,
  "secure": false,
  "authEnabled": true,
  "user": "you@company.com",
  "pass": "your-password",
  "from": "Alerts <you@company.com>",
  "to": "team@company.com",
  "subject": "Workflow alert",
  "text": "The scheduled job completed successfully."
}
```

### Direct SSL/TLS on Port 465

```json
{
  "host": "smtp.example.com",
  "port": 465,
  "secure": true,
  "authEnabled": true,
  "user": "noreply@example.com",
  "pass": "your-password",
  "from": "Example <noreply@example.com>",
  "to": "user1@example.com, user2@example.com",
  "subject": "Weekly digest",
  "html": "<h1>This week's highlights</h1>"
}
```

### Internal Relay, No Authentication

```json
{
  "host": "mail.internal.local",
  "port": 25,
  "secure": false,
  "authEnabled": false,
  "from": "noreply@internal.local",
  "to": "ops@internal.local",
  "subject": "Nightly job report",
  "text": "Nightly job completed with 0 errors."
}
```

### Self-Signed Certificate (Internal Postfix)

```json
{
  "host": "mail.internal.local",
  "port": 587,
  "secure": false,
  "ignoreTLS": true,
  "authEnabled": true,
  "user": "relay-user",
  "pass": "relay-password",
  "from": "noreply@internal.local",
  "to": "ops@internal.local",
  "subject": "Alert",
  "text": "Something needs attention."
}
```

---

## Workflow Integration

### Sample Workflow: Function (build content) → SMTP

```json
{
  "nodes": [
    {
      "id": "build-email-content",
      "type": "function"
    },
    {
      "id": "send-email",
      "type": "smtp-send-email",
      "config": {
        "host": "smtp.gmail.com",
        "port": 587,
        "authEnabled": true,
        "user": "you@gmail.com",
        "pass": "your-app-password",
        "from": "Support <you@gmail.com>",
        "to": "customer@example.com",
        "subject": "Your requested report"
      }
    }
  ]
}
```

### Sample Workflow: If → SMTP (conditional alerting)

```json
{
  "nodes": [
    {
      "id": "check-condition",
      "type": "if"
    },
    {
      "id": "send-alert",
      "type": "smtp-send-email",
      "config": {
        "host": "smtp.office365.com",
        "port": 587,
        "authEnabled": true,
        "user": "alerts@company.com",
        "pass": "your-password",
        "from": "Alerts <alerts@company.com>",
        "to": "oncall@company.com",
        "subject": "Threshold exceeded"
      }
    }
  ]
}
```

### Sample Workflow: NewsAPI → LLM → SMTP (automated digest)

```json
{
  "nodes": [
    {
      "id": "fetch-news",
      "type": "newsapi"
    },
    {
      "id": "summarize",
      "type": "llm"
    },
    {
      "id": "send-digest",
      "type": "smtp-send-email",
      "config": {
        "host": "smtp.gmail.com",
        "port": 587,
        "authEnabled": true,
        "user": "digest@company.com",
        "pass": "your-app-password",
        "from": "Daily Digest <digest@company.com>",
        "to": "team@company.com",
        "subject": "Daily news digest"
      }
    }
  ]
}
```

### Common Patterns

- If (condition met) → SMTP — conditional alerting
- Function (render HTML report) → SMTP — automated report delivery
- Schedule → Database/API fetch → Function → SMTP — recurring digest emails

---

## Error Handling

### Missing Credentials

```text
SMTP Error: Username and Password are required when Authentication is enabled.
```

Raised when `authEnabled` is `true` but `user` or `pass` is missing.

### Connection/Verification Failure

```text
SMTP Connection Failed: <underlying error message>
```

Raised when `transporter.verify()` fails — invalid host, wrong port, authentication rejected, or a TLS/SSL mismatch. May include an appended hint (see [Connection Verification Hints](#connection-verification-hints)) for `secure`/port mismatches.

### Send Failure

Errors thrown by `transporter.sendMail(...)` itself (e.g. malformed addresses, server-side rejection at send time) propagate as their native Nodemailer error, without additional wrapping.

---

## Troubleshooting

### "SMTP Error: Username and Password are required when Authentication is enabled."

**Cause**

`authEnabled` is `true` but `user` and/or `pass` is empty.

**Solution**

Provide both `user` and `pass`, or set `authEnabled` to `false` if the server genuinely requires no authentication (e.g. an internal relay).

---

### "SMTP Connection Failed: ... wrong version number" or "... SSL routines"

**Cause**

A mismatch between the configured `port` and `secure` setting — most commonly, `secure: true` on port 587 (which expects STARTTLS, not direct TLS), or `secure: false` on port 465 (which expects direct TLS).

**Solution**

Follow the hint appended to the error message: use `secure: false` for port 587, `secure: true` for port 465. See [Connection Verification Hints](#connection-verification-hints).

---

### "SMTP Connection Failed: Invalid login" or Authentication Error

**Cause**

Incorrect `user`/`pass`, or the provider requires an app-specific password rather than the account password (common with Gmail when 2FA is enabled).

**Solution**

For Gmail, generate an App Password instead of using the account password. For Office 365, verify SMTP AUTH is enabled for the account/tenant.

---

### "SMTP Connection Failed: self signed certificate" or Similar TLS Errors

**Cause**

The SMTP server presents a self-signed or otherwise untrusted certificate, and `ignoreTLS` is `false` (the default), so the connection is rejected.

**Solution**

If connecting to a trusted internal server with a self-signed certificate, set `ignoreTLS: true`. Avoid this for servers reached over the public internet, as it disables certificate validation.

---

### Email "Sends" Successfully but Never Arrives

**Cause**

The recipient address may have been added to the `rejected` array in the output despite `success: true` — this node does not throw on partial or full rejection, only on connection/auth failure.

**Solution**

Always check the `rejected` array in the node's output, not just `success`, to confirm actual delivery per recipient.

---

### Some Recipients in a Multi-Address `to` Field Don't Receive the Email

**Cause**

`to` is split on commas and trimmed, but a malformed address in the list (typo, missing `@`) will typically end up in `rejected` rather than blocking the whole send.

**Solution**

Validate email addresses before constructing the `to` string, and check `rejected` after sending.

---

## Security

The node connects to the configured SMTP `host`/`port` using the provided credentials (`user`/`pass`), which are passed directly to Nodemailer's transport configuration.

Setting `ignoreTLS: true` disables certificate validation (`rejectUnauthorized: false`), which makes the connection vulnerable to man-in-the-middle attacks if used over an untrusted network — only use it for known internal servers with self-signed certificates.

Credentials (`pass`) should be treated as sensitive and stored using the workflow platform's secret-handling mechanism where available, rather than as plain node configuration, if supported.

---

## Notes

The node performs a **connection verification step** (`transporter.verify()`) before every send — this adds a small amount of latency per call but surfaces configuration issues clearly, rather than failing silently or with a cryptic error only at send time.

The node does not:

- Support file attachments
- Support CC/BCC fields
- Support email templates or variable substitution (the caller must render `html`/`text` beforehand, e.g. via a `Function` node)
- Retry failed sends
- Validate email address syntax before sending (validation, if any, happens server-side and shows up in `rejected`)
- Persist or log sent messages

It is intended to provide direct, provider-agnostic SMTP sending for downstream notification and transactional-email workflows.

---



## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-19 | Initial release |