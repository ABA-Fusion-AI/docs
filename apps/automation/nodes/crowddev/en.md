---
node_id: "crowddev"
title: "crowd.dev"
description: "Automate developer community workflows: manage members, organizations, tasks, notes, and activities via the crowd.dev API."
category: "CRM & Marketing"
subcategory: "integrations"
version: "2.0.0"
language: "en"
last_updated: "2026-09-28"
author: "Fusion Team"
tags:
  - crowddev
  - community
  - crm
  - developer-relations
  - devrel
  - member-management
  - activities
related_nodes:
  - http-request
  - webhook-trigger
  - if
  - function
---

<!-- SECTION: overview -->
# crowd.dev

> **Category:** CRM & Marketing&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **crowd.dev** node connects Fusion workflows to the crowd.dev community intelligence platform. It enables Developer Relations (DevRel), community managers, and growth teams to automate member engagement, synchronize cross-platform identities, manage organizational accounts, and track activities across GitHub, Discord, Slack, Discourse, LinkedIn, and custom channels without writing custom API code.

### Supported Resources & Operations

- **Members:**
  - `list`: Query community members with pagination, filtering, and custom payload sorting.
  - `find`: Retrieve a member's complete profile by their unique `memberId`.
  - `findByEmail`: Look up a member by their primary email address.
  - `findByUsername`: Look up a member by platform handle and platform type (e.g., GitHub, Discord).
  - `create`: Add a new member with identity handles, email, display name, and custom attributes.
  - `update`: Modify an existing member's profile attributes.
  - `merge`: Merge duplicate member identities into a single primary profile.
  - `addTag`: Attach badges or labels (e.g., `VIP`, `Contributor`, `Champion`) to a member.
  - `delete`: Remove a member record from the tenant.
- **Organizations:**
  - `list`: Query organizations with pagination and custom filters.
  - `find`: Fetch organization details by `organizationId`.
  - `create`: Register a new organization with name, website, and description.
  - `update`: Update organization profile details.
  - `delete`: Delete an organization record.
- **Tasks:**
  - `list`: Query community tasks and to-do items.
  - `find`: Retrieve a specific task by `taskId`.
  - `create`: Create actionable tasks with title, body, status, and type.
  - `update`: Update task status (e.g., `in_progress`, `done`) or description.
  - `delete`: Remove a task from the tenant.
- **Notes:**
  - `list`: Query notes and team annotations.
  - `find`: Retrieve a specific note by `noteId`.
  - `create`: Create internal notes linked to specific community members.
  - `update`: Modify note contents.
  - `delete`: Remove a note.
- **Activities:**
  - `list`: Query activity feeds and engagement logs.
  - `createWithMember`: Ingest custom community touchpoints (stars, messages, forum posts, pull requests) while automatically creating or linking the corresponding member profile.
- **Legacy Compatibility:**
  - Supports legacy operation selectors (`getMembers`, `getActivities`, `getOrganizations`, `getConversations`) for seamless backwards compatibility.

### Use Cases

- **Community Onboarding:** Automatically create or enrich member profiles when users star a GitHub repository, join a Discord server, or register on your forum.
- **Identity Unification:** Deduplicate and merge member accounts across multiple platforms using shared emails or usernames.
- **VIP & Contributor Tagging:** Automatically tag active contributors as `Champion` or `Core Contributor` when their activity threshold is reached.
- **DevRel Task Automation:** Create follow-up tasks for your Developer Relations team whenever an enterprise user or high-intent developer engages with the community.
- **Internal Member Notes:** Append support interaction summaries or event attendance notes to member records.
- **CRM & Data Sync:** Ingest verified community interactions and members into your central data warehouse or CRM.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection & Authentication

| Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `baseUrl` | `string` | ❌ No | `"https://app.crowd.dev/api"` | crowd.dev API base endpoint. Customize this field when connecting to a self-hosted instance. Trailing slashes are stripped automatically. |
| `apiKey` | `string` | ✅ Yes | — | crowd.dev API Auth Token (Bearer token). Required for all requests. Supports expression syntax (`{{secrets.token}}`). |
| `tenantId` | `string` | ✅ Yes | — | crowd.dev Tenant ID representing your workspace. Required for all requests. |

### Resource & Action Selectors

| Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `resource` | `enum` | ❌ No | `"member"` | The community resource entity: `member`, `organization`, `task`, `note`, or `activity`. |
| `action` | `enum` | ❌ No | `"list"` | The action to execute: `create`, `update`, `find`, `list`, `delete`, `createWithMember`, `findByEmail`, `findByUsername`, `merge`, or `addTag`. |
| `operation` | `string` | ❌ No | — | Optional legacy operation selector (`getMembers`, `getActivities`, `getOrganizations`, `getConversations`) maintained for backwards compatibility. |

### Dynamic Parameters & Field Dependencies

The node displays input fields dynamically depending on the selected `resource` and `action`:

| Parameter | Type | Required Condition | Depends On | Description |
| :--- | :--- | :--- | :--- | :--- |
| `memberId` | `string` | Required for `member` (`find`, `update`, `delete`, `merge`, `addTag`) | `resource` in `["member", "activity", "note"]` | Target member ID (e.g., `mem_12345678`). Also associates notes to members or identifies the primary member in merge operations. |
| `secondaryMemberId` | `string` | Required for `merge` | `action == "merge"` | The secondary member profile ID that will be merged into the primary `memberId`. |
| `tag` | `string` | Required for `addTag` | `action == "addTag"` | Tag or badge label to assign to the member (e.g., `VIP`, `Contributor`, `Ambassador`). |
| `displayName` | `string` | ❌ No | `resource == "member"` | Member's full or public display name (e.g., `Jane Developer`). |
| `email` | `string` | Required for `findByEmail` | `resource == "member"` | Member's primary email address (e.g., `developer@example.com`). |
| `username` | `string` | Required for `findByUsername` | `resource == "member"` | Platform username or handle (e.g., `janedev`). |
| `platform` | `string` | ❌ No | `resource` in `["member", "activity"]` | Identity platform identifier (e.g., `github`, `discord`, `slack`, `discourse`, `linkedin`, `twitter`). Defaults to `"github"` for username lookups and `"custom"` for activities. |
| `organizationId` | `string` | Required for `organization` (`find`, `update`, `delete`) | `resource == "organization"` | Unique organization ID (e.g., `org_12345678`). |
| `name` | `string` | ❌ No | `resource == "organization"` | Organization legal or trade name (e.g., `Acme Innovations`). |
| `website` | `string` | ❌ No | `resource == "organization"` | Organization website URL (e.g., `https://acme.example.com`). |
| `description` | `string` | ❌ No | `resource == "organization"` | Brief summary or overview of the organization. |
| `taskId` | `string` | Required for `task` (`find`, `update`, `delete`) | `resource == "task"` | Target task ID (e.g., `tsk_12345678`). |
| `taskTitle` | `string` | ❌ No | `resource == "task"` | Title of the task. Defaults to `"Untitled Task"` on create. |
| `taskBody` | `string` | ❌ No | `resource == "task"` | Detailed instructions or notes for the task. |
| `taskType` | `string` | ❌ No | `resource == "task"` | Task categorization (e.g., `follow_up`, `review`, `triage`). Defaults to `"task"`. |
| `taskStatus` | `string` | ❌ No | `resource == "task"` | Execution status (e.g., `in_progress`, `done`, `archived`). Defaults to `"in_progress"`. |
| `noteId` | `string` | Required for `note` (`find`, `update`, `delete`) | `resource == "note"` | Target note ID (e.g., `not_12345678`). |
| `noteBody` | `string` | ❌ No | `resource == "note"` | Note content or markdown body. |
| `activityType` | `string` | ❌ No | `resource == "activity"` | Activity event type (e.g., `star`, `fork`, `pull_request`, `message`). Defaults to `"custom"`. |
| `activityTimestamp` | `string` | ❌ No | `resource == "activity"` | ISO 8601 timestamp string. Defaults to the current UTC execution time if omitted. |

### Search, Query & Pagination

| Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `filter` | `string` | ❌ No | — | JSON object string containing query filters for `list` operations (e.g., `{"platform": "github"}`). |
| `customData` | `string` | ❌ No | — | Optional custom JSON object merged into the request payload. Allows passing custom attributes, nested metadata, or tags directly. |
| `limit` | `string` | ❌ No | `"20"` | Maximum number of records to return in list operations (default: `20`, maximum: `200`). |
| `offset` | `string` | ❌ No | `"0"` | Zero-based pagination offset indicating how many items to skip. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input Ports

| Input | Type | Description |
| :--- | :--- | :--- |
| `input` | `any` | Trigger or upstream node payload. Incoming values can be referenced dynamically using expressions (e.g., `{{input.email}}`, `{{input.githubHandle}}`). |

### Output Ports

| Output | Type | Description |
| :--- | :--- | :--- |
| `success` | `object` | Emitted when the crowd.dev API returns an HTTP 2xx response. Contains data payload and execution metadata. |
| `error` | `Error` | Emitted if authentication fails, parameters are invalid, or the remote server returns an error. |

### Standard Response Envelope

Every successful invocation outputs a standardized JSON envelope:

```json
{
  "success": true,
  "status": "success",
  "data": { ... },
  "metadata": {
    "resource": "member",
    "action": "create",
    "timestamp": "2026-09-28T12:00:00.000Z",
    "resourceId": "mem_12345678",
    "count": 1
  }
}
```

> **Note:** For `delete` actions, `status` returns `"confirmed"`. When `findByEmail` or `findByUsername` is executed, the node unwraps the first match into `data` (or `null` if no record matches).

---

### Payload & Response Examples

#### 1. Create Community Member

**Configuration:**
- `resource`: `"member"`
- `action`: `"create"`
- `displayName`: `"Alex Rivera"`
- `email`: `"alex.rivera@example.com"`
- `username`: `"arivera"`
- `platform`: `"github"`
- `customData`: `'{"attributes": {"isSponsor": true}}'`

**Output (`success`):**
```json
{
  "success": true,
  "status": "success",
  "data": {
    "id": "mem_98765432-1111-2222-3333-abcdef012345",
    "displayName": "Alex Rivera",
    "emails": ["alex.rivera@example.com"],
    "username": {
      "github": "arivera"
    },
    "attributes": {
      "isSponsor": true
    },
    "score": 10,
    "createdAt": "2026-09-28T14:32:00.000Z"
  },
  "metadata": {
    "resource": "member",
    "action": "create",
    "timestamp": "2026-09-28T14:32:01.120Z",
    "resourceId": "mem_98765432-1111-2222-3333-abcdef012345",
    "count": 1
  }
}
```

---

#### 2. Log Activity with Member (`createWithMember`)

Logs an interaction and automatically creates or links the member profile in a single atomic request.

**Configuration:**
- `resource`: `"activity"`
- `action`: `"createWithMember"`
- `activityType`: `"star"`
- `platform`: `"github"`
- `username`: `"octocat"`
- `displayName`: `"Mona Lisa Octocat"`

**Output (`success`):**
```json
{
  "success": true,
  "status": "success",
  "data": {
    "id": "act_45678912-aaaa-bbbb-cccc-1234567890ab",
    "type": "star",
    "platform": "github",
    "timestamp": "2026-09-28T14:35:00.000Z",
    "member": {
      "id": "mem_11112222-3333-4444-5555-666677778888",
      "username": {
        "github": "octocat"
      },
      "displayName": "Mona Lisa Octocat"
    }
  },
  "metadata": {
    "resource": "activity",
    "action": "createWithMember",
    "timestamp": "2026-09-28T14:35:01.000Z",
    "resourceId": "act_45678912-aaaa-bbbb-cccc-1234567890ab",
    "count": 1
  }
}
```

---

#### 3. Merge Duplicate Profiles (`merge`)

Consolidates two profiles when a user is identified across platforms with different IDs.

**Configuration:**
- `resource`: `"member"`
- `action`: `"merge"`
- `memberId`: `"mem_primary123"`
- `secondaryMemberId`: `"mem_secondary456"`

**Output (`success`):**
```json
{
  "success": true,
  "status": "success",
  "data": {
    "id": "mem_primary123",
    "displayName": "Alex Rivera",
    "emails": ["alex.rivera@example.com", "alex@github.com"],
    "username": {
      "github": "arivera",
      "discord": "arivera#1024"
    }
  },
  "metadata": {
    "resource": "member",
    "action": "merge",
    "timestamp": "2026-09-28T14:40:00.000Z",
    "resourceId": "mem_primary123",
    "count": 1
  }
}
```

---

#### 4. Query Members (`list`) with Filters

**Configuration:**
- `resource`: `"member"`
- `action`: `"list"`
- `filter`: `'{"score": {"gte": 50}}'`
- `limit`: `"10"`
- `offset`: `"0"`

**Output (`success`):**
```json
{
  "success": true,
  "status": "success",
  "data": {
    "rows": [
      {
        "id": "mem_001",
        "displayName": "Jordan Smith",
        "score": 85,
        "joinedAt": "2026-01-10T09:00:00.000Z"
      },
      {
        "id": "mem_002",
        "displayName": "Sam Taylor",
        "score": 62,
        "joinedAt": "2026-02-14T11:20:00.000Z"
      }
    ],
    "count": 2,
    "limit": 10,
    "offset": 0
  },
  "metadata": {
    "resource": "member",
    "action": "list",
    "timestamp": "2026-09-28T14:45:00.000Z",
    "resourceId": null,
    "count": 2
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Sync and Track Community Member
```

### Sample Scenario: Ingest Webhook Event and Tag Member

When an external webhook delivers a community event (such as a GitHub sponsorship or Discord milestone), the workflow logs the activity into crowd.dev and applies a status tag:

```json
{
  "nodes": [
    {
      "id": "webhook-trigger",
      "type": "webhook-trigger"
    },
    {
      "id": "crowddev-log-activity",
      "type": "crowddev",
      "config": {
        "baseUrl": "https://app.crowd.dev/api",
        "apiKey": "{{secrets.crowdDevApiKey}}",
        "tenantId": "{{secrets.crowdDevTenantId}}",
        "resource": "activity",
        "action": "createWithMember",
        "activityType": "sponsor",
        "platform": "github",
        "username": "{{input.sender.login}}",
        "displayName": "{{input.sender.name}}",
        "email": "{{input.sender.email}}"
      }
    },
    {
      "id": "crowddev-tag-vip",
      "type": "crowddev",
      "config": {
        "apiKey": "{{secrets.crowdDevApiKey}}",
        "tenantId": "{{secrets.crowdDevTenantId}}",
        "resource": "member",
        "action": "addTag",
        "memberId": "{{crowddev-log-activity.metadata.resourceId}}",
        "tag": "Sponsor"
      }
    }
  ]
}
```

### Execution Flow:
1. **Webhook Trigger:** Receives the inbound event payload containing GitHub username, name, and email.
2. **crowd.dev Action (Activity):** Executes `createWithMember`, ingesting the `"sponsor"` activity and linking or creating the member.
3. **crowd.dev Action (Tag):** Uses the returned `memberId` to add the `"Sponsor"` tag to the contributor's profile.

### Common Workflow Patterns

- **Webhook → Find by Username → Condition (Exists?) → Create or Update:** Prevent duplicates by verifying existing member records before creation.
- **Activity Ingestion → Score Threshold → Add Tag / Assign Task:** Monitor high-value community interactions and automatically assign follow-up tasks to community managers.
- **Form Submission → Find by Email → Add Internal Note:** Attach survey feedback or conference conversation summaries directly to existing member records.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues & Solutions

#### `crowd.dev Error: apiKey and tenantId are required for all operations`
- **Cause:** One or both required authentication credentials (`apiKey` or `tenantId`) are empty or undefined in the node configuration.
- **Solution:** Verify that your `apiKey` and `tenantId` are set, or ensure environment variable/secret expressions (e.g., `{{secrets.crowdDevKey}}`) resolve to valid strings.

#### `crowd.dev Authentication Error [401]: Invalid or expired API Key`
- **Cause:** The provided Bearer token is invalid, revoked, or has expired.
- **Solution:** Generate a new API token in your crowd.dev workspace settings under **Settings → API Keys** and update your configuration.

#### `crowd.dev Not Found Error [404]: The requested <resource> or tenant was not found`
- **Cause:** The specified `tenantId` does not exist, or the target `memberId`, `organizationId`, `taskId`, or `noteId` was not found.
- **Solution:** Confirm your `tenantId` is spelled correctly. For update, find, or delete actions, verify that the resource ID exists in the tenant before invoking the operation.

#### `crowd.dev Permission Error [403]: Access forbidden for this resource`
- **Cause:** The API key lacks permissions to access or modify the selected resource.
- **Solution:** Verify that your API key has appropriate admin or read/write role permissions within your crowd.dev tenant.

#### `crowd.dev Network Error: Unable to reach endpoint`
- **Cause:** The node failed to establish a network connection to `baseUrl`.
- **Solution:** Check your internet connectivity. If using a self-hosted instance, verify that the host is accessible from your Fusion runner, the URL protocol (`http://` vs `https://`) is correct, and port configurations are open.

#### `memberId (primary) and secondaryMemberId are required for merge operation`
- **Cause:** The `merge` action was selected without supplying both member IDs.
- **Solution:** Ensure both `memberId` (the surviving profile) and `secondaryMemberId` (the profile being merged) are populated.

### Error Reference Table

| Status Code / Error | Category | Likely Cause | Resolution |
| :--- | :--- | :--- | :--- |
| `400 Bad Request` | Client Error | Malformed JSON in `filter` or `customData`, or missing required payload fields. | Validate JSON syntax in `filter` and `customData`. Check API schema requirements. |
| `401 Unauthorized` | Auth Error | Bearer token invalid or expired. | Refresh and update the crowd.dev API Key in Credential Manager. |
| `403 Forbidden` | Permission Error | Insufficient permissions for the requested resource. | Upgrade token privileges in crowd.dev workspace settings. |
| `404 Not Found` | Resource Error | Target resource ID or Tenant ID does not exist. | Double check ID strings and verify tenant ownership. |
| `FetchError / Network` | Network | DNS resolution failure or network firewall block. | Check `baseUrl` and network egress rules. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: security -->
## Security & Best Practices

- **Credential Storage:** Store your `apiKey` and `tenantId` in Fusion's Credential Manager or secret store. Never hardcode credentials into workflow parameter fields.
- **Data Sanitization:** Avoid committing raw workflow JSON exports containing unencrypted secrets or tenant IDs. The platform strips sensitive credentials in compliant exports, but always review files before committing to public repositories.
- **PII Compliance:** When logging activities and managing member emails or usernames, ensure compliance with applicable data privacy regulations (GDPR, CCPA).
- **Self-Hosted Endpoints:** If utilizing a custom self-hosted crowd.dev instance, ensure HTTPS encryption with valid TLS certificates to safeguard tokens in transit.

<!-- /SECTION: security -->

---

<!-- SECTION: related -->
## Related Nodes

- [Webhook Trigger](../webhook-trigger/en.md) – Ingest real-time event payloads from GitHub, Discord, or external services.
- [HTTP Request](../http-request/en.md) – Execute direct HTTP requests to complementary third-party services.
- [If](../if/en.md) – Branch workflows based on member scores, tags, or activity types.
- [Function](../function/en.md) – Format, transform, or filter member data before sending it to crowd.dev.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
| :--- | :--- | :--- |
| `2.0.0` | 2026-09-28 | Major upgrade: added dynamic schemas for all 5 resources (members, organizations, tasks, notes, activities), cross-platform activity ingestion (`createWithMember`), profile merging (`merge`), tag assignment (`addTag`), query filters, and backward compatibility for legacy operations. |
| `1.0.0` | 2026-08-12 | Initial integration release supporting basic member and activity query operations. |

<!-- /SECTION: changelog -->
