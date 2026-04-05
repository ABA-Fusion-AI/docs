---
node_id: "eventhub-admin"
title: "Event Hub Admin"
description: "Manage Azure Event Hubs and Consumer Groups — list, get, create, and delete hubs and consumer groups via the Azure Resource Manager API"
category: "integrations"
subcategory: "messaging"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - azure
  - event-hubs
  - admin
  - management
  - action
related_nodes:
  - eventhub-send
  - eventhub-receive
---

<!-- SECTION: header -->
# Event Hub Admin

> **Category:** Integrations | **Type:** Action Node

Manage the lifecycle of Azure Event Hubs and Consumer Groups within a namespace — list, inspect, create, and delete resources using the Azure Resource Manager (ARM) API with `DefaultAzureCredential`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Event Hub Admin** node uses the `@azure/arm-eventhub` `EventHubManagementClient` to interact with the Azure Resource Manager API. Authentication is handled via `DefaultAzureCredential`, which resolves credentials from the environment automatically (Managed Identity, environment variables, Azure CLI, etc.).

This node is intended for **infrastructure management workflows** — provisioning new Event Hubs before publishing, creating consumer groups for new services, or cleaning up unused hubs as part of a decommission pipeline.

### Key Features

- **8 Operations:** Full CRUD for both Event Hubs and Consumer Groups
- **DefaultAzureCredential:** No explicit secrets needed — works with Managed Identity in Azure environments
- **Consumer Group Management:** Create isolated consumer groups for independent parallel consumers
- **ARM-Level Access:** Works at the namespace level — not limited to a single hub

### Use Cases

- Provision an Event Hub and consumer group as part of a new service onboarding workflow
- List all Event Hubs in a namespace for inventory and compliance reporting
- Delete unused Event Hubs as part of a scheduled resource cleanup job
- Create a dedicated consumer group before starting an [Event Hub Receive](./eventhub-receive.md) workflow

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `subscriptionId` | `string` | ✅ Yes | Azure subscription ID |
| `resourceGroup` | `string` | ✅ Yes | Azure resource group containing the Event Hub namespace |
| `namespaceName` | `string` | ✅ Yes | Event Hub namespace name |

### Operation Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | ✅ Yes | Operation to execute (see below) |
| `eventHubName` | `string` | Conditional | Hub name — required for all operations except `listEventHubs` |
| `consumerGroup` | `string` | Conditional | Consumer group name — required for consumer group operations |
| `options` | `object` | Conditional | Hub or consumer group configuration for `createEventHub` and `createConsumerGroup` |

### `options` Structure

Used by `createEventHub`:

```json
{
  "messageRetentionInDays": 7,
  "partitionCount": 4
}
```

### Available Operations

#### Event Hub Operations

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `listEventHubs` | — | List all Event Hubs in the namespace |
| `getEventHub` | `eventHubName` | Get metadata for a specific Event Hub |
| `createEventHub` | `eventHubName`, `options` | Create or update an Event Hub |
| `deleteEventHub` | `eventHubName` | Delete an Event Hub |

#### Consumer Group Operations

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `listConsumerGroups` | `eventHubName` | List all consumer groups for an Event Hub |
| `getConsumerGroup` | `eventHubName`, `consumerGroup` | Get details of a specific consumer group |
| `createConsumerGroup` | `eventHubName`, `consumerGroup` | Create or update a consumer group |
| `deleteConsumerGroup` | `eventHubName`, `consumerGroup` | Delete a consumer group |

### Authentication

This node uses `DefaultAzureCredential`, which automatically resolves credentials from the following sources (in order):

1. Environment variables (`AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`)
2. Workload Identity (Kubernetes)
3. Managed Identity (Azure VMs, App Service, etc.)
4. Azure CLI (`az login`)
5. Azure PowerShell

No explicit credentials are needed in the node configuration if running inside an Azure environment with the appropriate RBAC role (`Azure Event Hubs Data Owner` or `Contributor`).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data — `eventHubName` and `consumerGroup` can be resolved via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` or `array` | Result of the operation |
| `error` | `Error` | Emitted if the operation fails |

### Output Schemas by Operation

**`listEventHubs`**
```json
[
  {
    "id": "/subscriptions/.../resourceGroups/.../providers/Microsoft.EventHub/namespaces/my-ns/eventhubs/telemetry",
    "name": "telemetry",
    "partitionCount": 4,
    "messageRetentionInDays": 7,
    "status": "Active",
    "partitionIds": ["0", "1", "2", "3"]
  }
]
```

**`getEventHub`**
```json
{
  "name": "telemetry",
  "partitionCount": 4,
  "messageRetentionInDays": 7,
  "status": "Active",
  "createdAt": "2026-01-01T00:00:00.000Z",
  "updatedAt": "2026-03-01T00:00:00.000Z"
}
```

**`createEventHub`** — returns the full ARM resource object for the created/updated hub.

**`deleteEventHub`**
```json
{
  "status": "deleted",
  "eventHubName": "telemetry"
}
```

**`listConsumerGroups`**
```json
[
  { "name": "$Default" },
  { "name": "analytics-consumer" },
  { "name": "backfill-consumer" }
]
```

**`deleteConsumerGroup`**
```json
{
  "status": "deleted",
  "consumerGroup": "backfill-consumer"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Create an Event Hub with Retention

**Configuration:**
```json
{
  "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "resourceGroup": "my-rg",
  "namespaceName": "my-eventhub-ns",
  "operation": "createEventHub",
  "eventHubName": "device-telemetry",
  "options": {
    "messageRetentionInDays": 7,
    "partitionCount": 4
  }
}
```

---

### Example: Create a Consumer Group for a New Service

**Configuration:**
```json
{
  "subscriptionId": "...",
  "resourceGroup": "my-rg",
  "namespaceName": "my-eventhub-ns",
  "operation": "createConsumerGroup",
  "eventHubName": "device-telemetry",
  "consumerGroup": "analytics-service"
}
```

---

### Example: List All Hubs in a Namespace

**Configuration:**
```json
{
  "subscriptionId": "...",
  "resourceGroup": "my-rg",
  "namespaceName": "my-eventhub-ns",
  "operation": "listEventHubs"
}
```

---

### Example: Delete a Consumer Group After Decommission

**Configuration:**
```json
{
  "subscriptionId": "...",
  "resourceGroup": "my-rg",
  "namespaceName": "my-eventhub-ns",
  "operation": "deleteConsumerGroup",
  "eventHubName": "device-telemetry",
  "consumerGroup": "legacy-consumer"
}
```

**Output:**
```json
{ "status": "deleted", "consumerGroup": "legacy-consumer" }
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Service Onboarding — Provision Event Hub Infrastructure

When a new microservice is registered in the platform, automatically provision its Event Hub and consumer group.

```json
{
  "nodes": [
    {
      "id": "service-registered",
      "type": "webhook-trigger"
    },
    {
      "id": "create-hub",
      "type": "eventhub-admin",
      "config": {
        "subscriptionId": "...",
        "resourceGroup": "platform-rg",
        "namespaceName": "platform-events",
        "operation": "createEventHub",
        "eventHubName": "{{input.serviceName}}-events",
        "options": { "messageRetentionInDays": 3, "partitionCount": 2 }
      }
    },
    {
      "id": "create-consumer-group",
      "type": "eventhub-admin",
      "config": {
        "subscriptionId": "...",
        "resourceGroup": "platform-rg",
        "namespaceName": "platform-events",
        "operation": "createConsumerGroup",
        "eventHubName": "{{input.serviceName}}-events",
        "consumerGroup": "{{input.serviceName}}-processor"
      }
    },
    {
      "id": "notify",
      "type": "slack-action",
      "config": {
        "operation": "message",
        "messageOperation": "send",
        "text": "✅ Event Hub provisioned for *{{input.serviceName}}*"
      }
    }
  ]
}
```

### Common Patterns

- **Onboarding Pipeline:** Webhook trigger → `createEventHub` → `createConsumerGroup` → notify
- **Compliance Audit:** Cron → `listEventHubs` → Function (check retention) → Slack report
- **Cleanup Job:** Cron → `listConsumerGroups` → Function (find stale) → `deleteConsumerGroup`

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `EventHubManagementClient not initialized. Did setup() run?`

**Cause:** `handleTick()` was invoked before `setup()` completed.

**Solution:** This is framework-managed. If it occurs, check for errors during `setup()` — usually a missing `subscriptionId` or credential resolution failure.

#### `DefaultAzureCredential: No credential found`

**Cause:** The Fusion runtime has no Azure credentials in scope.

**Solution:** Set `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, and `AZURE_TENANT_ID` environment variables, or run inside an Azure environment with Managed Identity enabled.

#### `eventHubName is required for getEventHub`

**Cause:** An operation that requires a hub name was called without it.

**Solution:** Provide `eventHubName` in the configuration or resolve it from an upstream expression.

#### `eventHubName and consumerGroup are required for createConsumerGroup`

**Cause:** Either field is missing or empty.

**Solution:** Ensure both fields are set. Use `listEventHubs` first to discover available hub names if needed.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `ResourceNotFound` | Hub, namespace, or resource group does not exist | Verify names in the Azure portal |
| `AuthorizationFailed` | Insufficient RBAC permissions | Assign `Azure Event Hubs Data Owner` or `Contributor` role |
| `SubscriptionNotFound` | `subscriptionId` is incorrect | Double-check the subscription ID |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Event Hub Send](./eventhub-send.md) - Publish events to a provisioned Event Hub
- [Event Hub Receive](./eventhub-receive.md) - Consume events from an Event Hub

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
