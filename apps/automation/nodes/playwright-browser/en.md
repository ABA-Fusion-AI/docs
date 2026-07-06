---
  node_id: "playwright-browser"
  title: "Playwright Browser"
  description: "Run a user-supplied Playwright script against a fresh headless Chromium browser and capture the script's result plus a screenshot."
  category: "actions"
  subcategory: "scraping"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-07-06"
  author: "Fusion Team"
  tags:
    - scraping
    - browser-automation
    - playwright
    - chromium
    - screenshot
    - no-code
  related_nodes:
    - http-request
    - function
---

  # Playwright Browser

  > **Category:** Actions&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

  Run a Playwright/Chromium script and capture the script's result plus a screenshot. A fresh browser is launched for every execution, so concurrent runs on the same node instance never share browser or page state.

### Use Cases

- Scrape data from pages that require JavaScript rendering
- Automate form submission or login flows as part of a workflow
- Capture a visual screenshot of a page for auditing or reporting
- Drive multi-step browser interactions (click, type, wait) before extracting data

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `code` | `string` (expression, textarea) | ✅ Yes | — | Playwright script body, run against a fresh Chromium browser for this execution. `page`, `context`, `browser`, and `data` (the upstream input) are available inside the script. Supports top-level `await`. End with an explicit `return` to produce this node's result. |
| `timeoutMs` | `number` | ❌ No | `30000` | Hard time limit in milliseconds for the whole script run. The browser is force-closed if this is exceeded. |
| `viewportWidth` | `number` | ❌ No | `1280` | Browser viewport width in pixels. |
| `viewportHeight` | `number` | ❌ No | `720` | Browser viewport height in pixels. |
| `headless` | `boolean` | ❌ No | `false` | Run Chromium headless. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream data, exposed inside the script as `data`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | `{ result, screenshot }` — the script's return value and a base64-encoded PNG screenshot taken at the end of the run. |
| `error` | `Error` | Emitted if the script throws, or if it exceeds `timeoutMs`. |

### Key Features

- **Isolated Execution:** A new Chromium browser is launched per execution, so concurrent runs never race on shared browser/page state.
- **Full Playwright API:** The script body runs with `page`, `context`, `browser`, and `data` in scope, and supports top-level `await`.
- **Automatic Screenshot:** A screenshot of the final page state is captured and returned alongside the script's result on every successful run.
- **Hard Timeout:** The browser is force-closed if the script exceeds `timeoutMs`.
- **Configurable Viewport:** Set `viewportWidth`/`viewportHeight` to match the target site's expected layout.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Scrape a Page Title

**Configuration:**
```json
{
  "code": "await page.goto(data.url);\nconst title = await page.title();\nreturn { title };",
  "timeoutMs": 15000,
  "headless": true
}
```

**Input:** `{ "url": "https://example.com" }`

**Output (success):**
```json
{
  "result": { "title": "Example Domain" },
  "screenshot": "<base64 PNG>"
}
```

### Example 2: Fill and Submit a Login Form

**Configuration:**
```json
{
  "code": "await page.goto(data.url);\nawait page.fill('#username', data.username);\nawait page.fill('#password', data.password);\nawait page.click('button[type=\"submit\"]');\nawait page.waitForNavigation();\nreturn { url: page.url() };",
  "timeoutMs": 20000,
  "viewportWidth": 1440,
  "viewportHeight": 900
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Screenshot a URL from a Webhook

```json
{
  "nodes": [
    {
      "id": "webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "browse",
      "type": "playwright-browser",
      "config": {
        "code": "await page.goto(data.url, { waitUntil: 'networkidle' });\nreturn { title: await page.title() };",
        "headless": true,
        "timeoutMs": 20000
      }
    }
  ]
}
```

**How it flows:**
1. Webhook emits `{ "url": "https://example.com" }`.
2. The Playwright Browser node navigates to `data.url`, waits for the network to go idle, and returns the page title.
3. `success` carries `{ result: { title }, screenshot }` for downstream nodes (e.g. store the screenshot, or branch on the title).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Script exceeded <N>ms timeout`

**Cause:** The script (including page navigation and waits) took longer than `timeoutMs`.

**Solution:** Increase `timeoutMs`, or make the script's waits more targeted (e.g. wait for a specific selector instead of full network idle).

#### Script throws inside `page.goto` / selector not found

**Cause:** The target page didn't load as expected, or a selector used in the script doesn't exist on the page.

**Solution:** Verify the URL and selectors against the real page; add explicit waits (`page.waitForSelector`) before interacting with dynamic content.

#### No screenshot in output

**Cause:** The script threw before completion — the screenshot is only captured after the script resolves successfully.

**Solution:** Check the `error` output for the underlying script failure and fix the script logic.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) – Lighter-weight option when JavaScript rendering isn't required
- [Function](./function.md) – Post-process the script's result with custom JavaScript

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-07-06 | Initial release |

<!-- /SECTION: changelog -->
