---
  node_id: "write-file"
  title: "Write file"
  description: "Write content to a file and pass the file on to the next steps, attached beside the data."
  category: "storage"
  subcategory: "files"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-10-22"
  author: "Fusion Team"
  tags:
    - file
    - write
    - export
    - storage
  related_nodes:
    - read-file
    - webhook
    - to-csv
---

<!-- SECTION: overview -->
  # Write file

  > **Category:** Storage & Files&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

  Writes content (text, base64 or a value as JSON) to a file and passes the file on. The file travels beside the step's data, so later steps can read it, attach it to an email, or upload it, and you can download it from the step's output.

### Use Cases

- Turn a CSV a **To CSV** step built into a file to send on
- Save an API response as a JSON file
- Decode a base64 attachment into the file it carries

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `name` | `string` | ✅ Yes | `file.txt` | The file's name, with its extension. Accepts expressions. |
| `mimeType` | `string` | ❌ No | from the name | The file's type, e.g. `text/csv`. Left empty, it follows the name's extension (`report.xlsx` is an Excel file). |
| `content` | `any` | ❌ No | — | What to write, usually an earlier step's output. Bytes (an Excel or PDF a step made) are written as they are, whatever `encoding` says. |
| `encoding` | `enum` | ✅ Yes | `text` | `text`: written as UTF-8 (a value that is not text is written as JSON).<br>`base64`: decoded to the bytes it carries.<br>`json`: the value as formatted JSON. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

**Input:** any data; use it in `content` through an expression.

**Success output:**

```json
{
  "$files": {
    "file": { "id": "…", "name": "report.csv", "mimeType": "text/csv", "size": 1024 }
  }
}
```

The output is the file. The editor shows it as a file card you can download; `$files` belongs to the platform and cannot be edited. A later step picks the file as **Write file › file**, and reads its details as `$files.file.name`, `.mimeType` and `.size`.

**Error output:** the file is larger than the platform's file limit (25 MB unless your administrator changed it), or the worker has no database to store files in.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Example

**Webhook → To CSV → Write file**: a webhook receives rows, **To CSV** turns them into text, and **Write file** (`name: rows.csv`, `mimeType: text/csv`, `content: {{ To CSV }}`) makes the file. Open the run's output to download it.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

- **"File … is larger than the … limit."** The content is over the file limit. Write less, or split it into several files.
- **"This worker cannot store files"**: the worker has no database connection for files. Ask your administrator.
- **The file is gone from an old run.** Files a run writes are kept as long as that run's output, then removed.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- **Read file**: read a file's content into the data
- **Webhook**: receives uploaded files the same way, under `$files`

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

- **1.0.0** (2026-10-22): first version.

<!-- /SECTION: changelog -->
