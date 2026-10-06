---
node_id: "pushcut"
title: "Pushcut"
description: "Send smart push notifications to iOS devices and execute Apple Shortcuts remotely via the Pushcut API."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-06"
author: "Fusion Team"
tags:
  - pushcut
  - notifications
  - ios
  - apple
  - shortcuts
  - mobile
  - automation
related_nodes:
  - webhook-trigger
  - log
  - filter
---

<!-- SECTION: overview -->
# Pushcut

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Send smart, actionable push notifications to Apple devices (iPhone, iPad, Mac) and trigger Apple Shortcuts remotely through Pushcut.

### Use Cases

- **System Alerts & Incidents:** Instantly notify on-call engineers on their iPhones when a server crashes or an API health check fails.
- **E-Commerce & Orders:** Send high-priority notifications with deep links directly to orders when a customer completes a purchase.
- **Remote iOS Automation:** Remotely execute Apple Shortcuts (e.g. toggle HomeKit lights, trigger backup routines, run local scripts) from a cloud workflow.
- **Device Management:** Discover registered devices and inspect defined notification templates dynamically.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|---------|-------------|
| `apiKey` | `string` | ✅ Yes | — | Secret API Key generated from the Pushcut app (**Account > INTEGRATIONS > Add API Key**). |
| `operation` | `enum` | ✅ Yes | `sendNotification` | Action to execute: `sendNotification`, `executeAction`, `getDevices`, or `getNotifications`. |

---

### Parameters by Operation

#### 1. `sendNotification`
Delivers an actionable notification to registered devices.

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|---------|-------------|
| `notificationName` | `string` | ✅ Yes | — | Name of the notification defined in Pushcut (e.g. `My First Notification`). |
| `title` | `string` | ❌ No | — | Custom subject line overriding the default title. |
| `text` | `string` | ❌ No | — | Message content displayed in the notification body. |
| `target` | `string` | ❌ No | — | Target device name or comma-separated list of devices (e.g. `iPhone` or `iPhone, iPad`). |
| `devices` | `string` | ❌ No | — | Alias for `target`. Used if `target` is not set. |
| `priority` | `enum` | ❌ No | — | Delivery urgency: `low`, `normal`, or `high` (time-sensitive alert). |
| `sound` | `string` | ❌ No | — | Sound alert name (`system`, `loud`, `jobDone`, `lasers`, `alarm`), or set to `false`/`silent`/`mute` for silent vibration. |
| `url` | `string` | ❌ No | — | Action URL opened in browser when tapping the notification banner or the action button. |
| `input` | `string` | ❌ No | — | Custom data payload forwarded to actions or shortcuts triggered by the notification. |

#### 2. `executeAction`
Triggers an Apple Shortcut on an iOS device running the Pushcut Automation Server.

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|---------|-------------|
| `actionName` | `string` | ✅ Yes | — | Exact name of the Apple Shortcut in iOS Shortcuts app to execute. |
| `actionInput` | `string` | ❌ No | — | Input argument passed to the Apple Shortcut (accessed via `Shortcut Input`). |

#### 3. `getDevices`
Fetches all registered Apple devices linked to the Pushcut account.
* Requires only `apiKey`. No additional parameters needed.

#### 4. `getNotifications`
Fetches all pre-configured notification definitions in the Pushcut app.
* Requires only `apiKey`. No additional parameters needed.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Trigger data or previous node payload passed into the execution. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the Pushcut API request succeeds. |
| `error` | `Error` | Emitted when validation fails or Pushcut returns an error code (400, 401, 404, 500). |

---

### Output Schemas

#### `sendNotification` Output (`success`)
Returns a standardized delivery receipt:

```json
{
  "status": "success",
  "notificationId": "SPgEnS1COVRf7pdxP6mBE",
  "timestamp": "2026-10-06T16:12:51.326Z",
  "deviceCount": 1,
  "message": "Success!",
  "id": "SPgEnS1COVRf7pdxP6mBE"
}
```

#### `executeAction` Output (`success`)
Emitted when the shortcut is acknowledged and executed:

```json
{
  "success": true
}
```

#### `getDevices` Output (`success`)
```json
[
  {
    "id": "iPhone",
    "name": "iPhone"
  }
]
```

#### `getNotifications` Output (`success`)
```json
[
  {
    "id": "My First Notification",
    "title": "Automate Away 🚀"
  }
]
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Send Pushcut Notification on Event
```

### Step-by-Step Walkthrough

```
[ Manual Trigger ] ──▶ [ Pushcut Action ] ──▶ [ Log Result ]
```

1. **Trigger:** The workflow is initiated manually or via an incoming webhook/cron event.
2. **Pushcut Integration:**
   * **ApiKey:** `{{credentials.pushcutKey}}`
   * **Operation:** `sendNotification`
   * **NotificationName:** `My First Notification`
   * **Title:** `Deployment Success`
   * **Text:** `Version 2.4.0 deployed to production.`
   * **Priority:** `high`
   * **URL:** `https://my-dashboard.example.com`
3. **Log Result:** Receives the delivery confirmation and records `notificationId` and `timestamp`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `notificationName required`
* **Cause:** The `sendNotification` operation was triggered without specifying `notificationName`.
* **Solution:** Enter the exact name of a notification defined in your Pushcut iOS app (e.g. `My First Notification`).

#### `actionName required`
* **Cause:** The `executeAction` operation was triggered without providing the shortcut name.
* **Solution:** Provide the exact name of the Apple Shortcut in the `actionName` field.

#### `Invalid API-Key provided.` / `401 Unauthorized`
* **Cause:** The API Key is incorrect, expired, or deleted in Pushcut.
* **Solution:** Open the Pushcut app on iOS, go to **Account > INTEGRATIONS**, generate a new API Key, and update your node configuration.

#### `Automation Server is currently not running on any iOS device`
* **Cause:** You attempted to execute a shortcut (`executeAction`), but no device has the Automation Server active.
* **Solution:** Open Pushcut on your iOS device, go to the **Server** tab, tap **Start Server On This Device**, and keep the app in the foreground.

#### Notification does not arrive on iOS
* **Cause:** Pushcut notification permissions are disabled, or Do Not Disturb / Focus Mode is active.
* **Solution:** 
  1. Go to iOS **Settings > Notifications > Pushcut** and verify **Allow Notifications** is enabled.
  2. Disable **Do Not Disturb** or whitelist Pushcut in Focus Mode.
  3. In Pushcut app, tap **Test Notification** once to ensure the device token is registered with Apple APNs.

#### Sound not playing
* **Cause:** Device is in Silent Mode (orange physical switch on iPhone) or `sound` was configured with an unsupported string.
* **Solution:** 
  * Turn off Silent Mode on your device.
  * Use supported sound identifiers: `system`, `loud`, `jobDone`, `lasers`, or `alarm`.
  * To explicitly silence notifications, set `sound` to `false` or `silent`.

---

### Error Reference

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `notificationName required` | Missing required parameter for `sendNotification` | Supply `notificationName` |
| `actionName required` | Missing required parameter for `executeAction` | Supply `actionName` |
| `Invalid API-Key provided.` | Authentication rejected by Pushcut API | Re-verify your API key in Pushcut |
| `Notification not found.` | Notification name does not exist in account | Check notification name in Pushcut app |
| `Automation Server is currently not running...` | iOS device server is offline | Start Server in Pushcut app on device |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: security -->
## Security

- **Credential Storage:** Always store your Pushcut API key in Fusion's credential management system. Avoid hardcoding plaintext API keys directly in exported workflow JSON.
- **Sanitization:** All notification and shortcut names are automatically URI-encoded to prevent URL injection attacks.
- **Minimal Privilege:** Pushcut API keys provide full access to account triggers. Rotate keys immediately if compromised via the Pushcut app.

<!-- /SECTION: security -->

---

<!-- SECTION: related -->
## Related Nodes

- [Webhook Trigger](../webhook-trigger/en.md) – Trigger push notifications from external third-party events
- [Log](../log/en.md) – Inspect delivery receipts and debug error responses
- [Filter](../filter/en.md) – Conditionally trigger urgent push alerts based on severity

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Full production release: added `target`/`devices`, `priority`, `sound`, `url` deep-linking, standardized output receipts, and resilient empty-body handling. |

<!-- /SECTION: changelog -->
