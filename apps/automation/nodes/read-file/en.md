---
  node_id: "read-file"
  title: "Read file"
  description: "Read a file's content into the data: a file an earlier step attached, or one uploaded in the node's settings."
  category: "storage"
  subcategory: "files"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-10-22"
  author: "Fusion Team"
  tags:
    - file
    - read
    - import
    - storage
  related_nodes:
    - write-file
    - webhook
    - parse-csv
---

<!-- SECTION: overview -->
  # Read file

  > **Category:** Storage & Files&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

  Reads a file into the data as text, parsed JSON, or base64. The file is one an earlier step attached (a webhook upload, a **Write file** output) or one you upload in the node's settings.

### Use Cases

- Read a CSV someone posted to a webhook, then parse it with **Parse CSV**
- Load a JSON configuration file uploaded with the workflow
- Turn a file into base64 for an API that wants it that way

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `file` | `file` | ✅ Yes | — | The file to read. **Choose file** opens a dialog: **From a step** picks a file an earlier step attached (e.g. *Webhook › invoice*); **Upload** uploads one into this workflow. |
| `as` | `enum` | ✅ Yes | `text` | `text`: UTF-8 text. `json`: parsed JSON. `base64`: the bytes as base64 text. |
| `maxBytes` | `number` | ✅ Yes | `10485760` (10 MB) | Largest file this step reads. A larger file fails the step without being read. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

**Input:** any data; it starts the read.

**Success output:**

```json
{ "name": "rates.csv", "mimeType": "text/csv", "size": 42, "content": "currency,rate\nEUR,1.08\n" }
```

**Error output:** the step fails on its error output, and the workflow keeps running, when:

- the file is not available to this workflow: deleted, or another workflow's;
- the file is larger than `maxBytes`;
- `as` is `json` and the file is not valid JSON.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Example

**Webhook → Read file → Parse CSV**: a form posts a CSV as the field `rates`. In **Read file**, choose **From a step › Webhook › rates**, `as: text`. **Parse CSV** turns `{{ Read file.content }}` into rows.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

- **"File … is not available to this workflow"**: the file was deleted, removed with an old run, or belongs to another workflow. Pick it again or upload it again.
- **"File … is larger than the … limit for reading."** Raise `maxBytes`, up to what the data can sensibly hold.
- **"Choose a file to read."** The `file` setting is empty.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- **Write file**: write content to a file
- **Parse CSV** / **Parse JSON**: turn the content into data

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

- **1.0.0** (2026-10-22): first version.

<!-- /SECTION: changelog -->
