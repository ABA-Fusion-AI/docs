---
node_id: "linkedin-job-scraper"

title: "LinkedIn Job Scraper"

description: "Scrape job postings from LinkedIn search results with pagination, retries, configurable delays, and HTML parsing."

category: "Data Extraction / LinkedIn"

version: "1.0.0"

language: "en"

last_updated: "2026-09-16"

author: "Fusion Team"

tags:

- linkedin
- jobs
- scraper
- cheerio
- recruitment
- data-extraction

related_nodes:

- http-request
- function
- if

---

**# LinkedIn Job Scraper**

> **\*\*Category:\*\*** data-extraction-nodes | **\*\*Type:\*\*** Action Node

Scrape LinkedIn job search result pages and convert detected job cards into structured job records.

The node fetches paginated HTML search results, retries failed requests, applies configurable delays, parses job cards with Cheerio, and returns job metadata.

**### Supported Features**

\- Scrape LinkedIn job search HTML

\- Pagination using the `start` query parameter

\- Configurable maximum job count

\- Retry failed HTTP requests

\- Configurable request timeout and random delays

\- Browser-like request headers

\- Multiple fallback selectors for LinkedIn job cards

\- Extract title, company, location and posting date

\- Extract description, seniority and employment type

\- Extract industries and job URL

\- Return structured job records

**### Use Cases**

\- Collect job listings from a LinkedIn search URL

\- Build job-market datasets

\- Feed job listings into filtering or analysis workflows

\- Store structured recruitment data

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `searchUrl` | `string` | ✅ Yes | — | Search URL to scrape. |
| `maxJobs` | `number` | ❌ No | `100` | Maximum jobs to collect. Minimum `1`. |
| `maxRetries` | `number` | ❌ No | `3` | Maximum fetch attempts per page. Minimum `1`. |
| `requestTimeout` | `number` | ❌ No | `20000` | Request timeout in milliseconds. Minimum `5000`. |
| `delayMinMs` | `number` | ❌ No | `500` | Minimum random delay in milliseconds. |
| `delayMaxMs` | `number` | ❌ No | `1500` | Maximum random delay in milliseconds. |
| `useProxy` | `boolean` | ❌ No | `false` | Passes `proxyUrl` into the scraper when enabled. |
| `proxyUrl` | `string` | ❌ No | — | Proxy configuration value. Current fetch implementation does not apply a proxy agent. |

**---**

**## Operations**

The node exposes one fixed operation: scrape LinkedIn job search results.

There is no `operation` parameter. Every execution calls `scrapeSearch()` with the configured search URL and scraping settings.

**---**

**## Pagination**

Pagination starts at:

```text
start = 0
```

The internal page size is:

```text
25
```

After each parsed page:

```ts
start += 25;
```

The node adds or replaces the `start` query parameter in the configured URL.

Pagination stops when the requested maximum is reached, no jobs are found on a page, or a later page fails after jobs have already been collected.

**---**

**## HTTP Request Construction**

The node sends browser-like headers including:

```text
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

Requests use `fetch()` with an `AbortController` timeout.

**---**

**## Retry Logic**

Each page can be attempted up to `maxRetries` times.

When an attempt fails and retries remain, the node waits for a random delay between `delayMinMs` and `delayMaxMs`.

After all attempts fail:

```text
Fetch failed after retries
```

is used only if no captured error is available; otherwise the last captured error is thrown.

**---**

**## Proxy Configuration**

The schema accepts `useProxy` and `proxyUrl`, but proxy transport is not implemented.

The code contains only a placeholder for adding an HTTP proxy agent. Therefore enabling `useProxy` does not currently route `fetch()` through the configured proxy.

**---**

**## HTML Parsing**

The node uses Cheerio and tries these job-card selectors in order:

```text
li.jobs-search-results__list-item
div.base-card
div.job-search-card
li.jobs-search__results-list
```

A later selector is tried only when the previous selector returns zero elements.

Each detected card is parsed individually. Cards without a detectable title are skipped.

**---**

**## Field Extraction**

**### Job Title**

Fallback selectors include:

```text
a.job-card-list__title
a.base-card__full-link
h3.base-search-card__title
a.disabled.ember-view.job-card-container__link.job-card-list__title
h3
a[href*='/jobs/view/']
```

**### Company Name**

```text
a.job-card-container__company-name
a.base-search-card__subtitle
span.job-card-container__primary-description
h4.base-search-card__subtitle
a[href*='/company/']
```

**### Location**

```text
span.job-card-container__metadata-item
span.job-search-card__location
div.base-search-card__metadata
span.job-card-container__metadata-wrapper
```

**### Date Posted**

```text
time
div.job-card-container__listed-time
div.job-search-card__listdate
time.job-search-card__listdate
```

**### Description**

```text
div.job-card-list__insight
```

**### Seniority Level**

```text
span.job-card-container__metadata-item--seniority
span[class*='seniority']
```

**### Employment Type**

```text
span.job-card-container__metadata-item--employment-type
span[class*='employment']
```

**### Industries**

Industries are collected from `li` elements inside:

```text
ul.job-card-container__industry-list
```

**### Job URL**

The node searches for:

```text
a[href*='/jobs/view/']
```

Absolute URLs are retained. Relative URLs are prefixed with:

```text
https://www.linkedin.com
```

**---**

**## Inputs & Outputs**

**### Inputs**

`handleTick()` receives incoming `data`, but does not use it. Scraping behavior is controlled by node configuration.

**### Job Structure**

```json
{
  "job_title": "Software Engineer",
  "company_name": "Example Company",
  "location": "Example Location",
  "date_posted": "1 day ago",
  "job_description": "",
  "seniority_level": "",
  "employment_type": "",
  "industries": [],
  "job_url": "https://www.linkedin.com/jobs/view/..."
}
```

**### Node Output**

```json
{
  "status": "success",
  "total_jobs": 1,
  "jobs": [],
  "scraped_at": "2026-09-16T00:00:00.000Z",
  "search_url": "https://www.linkedin.com/jobs/search/..."
}
```

| Output | Description |
| ------ | ----------- |
| `status` | Fixed value `"success"`. |
| `total_jobs` | Number of collected jobs. |
| `jobs` | Array of parsed job records. |
| `scraped_at` | ISO timestamp generated after scraping. |
| `search_url` | Original configured search URL. |

**---**

**## Configuration Examples**

**### Basic Search**

```json
{
  "searchUrl": "https://www.linkedin.com/jobs/search/?keywords=software%20engineer",
  "maxJobs": 100
}
```

**### Custom Retry Settings**

```json
{
  "searchUrl": "https://www.linkedin.com/jobs/search/?keywords=developer",
  "maxJobs": 50,
  "maxRetries": 5,
  "requestTimeout": 30000
}
```

**### Custom Delays**

```json
{
  "searchUrl": "https://www.linkedin.com/jobs/search/?keywords=developer",
  "maxJobs": 50,
  "delayMinMs": 1000,
  "delayMaxMs": 3000
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Manual Trigger → LinkedIn Job Scraper

\- LinkedIn Job Scraper → Function

\- LinkedIn Job Scraper → Database

\- LinkedIn Job Scraper → AI Node

\- LinkedIn Job Scraper → If

**---**

**## Error Handling**

Top-level errors are wrapped as:

```text
LinkedIn scraping failed: <error>
```

Non-success HTTP responses throw:

```text
HTTP <status>: <statusText>
```

If the first page fails after all retries, execution fails.

If a later page fails after jobs have already been collected, pagination stops and the existing jobs are returned.

**---**

**## Troubleshooting**

**### No Jobs Returned**

The returned HTML may not contain any of the configured job-card selectors, or detected cards may not contain a title matching the fallback selectors.

When a page returns zero parsed jobs, pagination stops.

---

**### HTTP Error**

Every non-success response is treated as an error and retried according to `maxRetries`.

---

**### Proxy Does Not Work**

`proxyUrl` is currently only passed through the scraper. It is not attached to `fetch()` through a proxy agent.

---

**### Existing `start` Parameter**

When the URL can be parsed, the node uses:

```ts
url.searchParams.set("start", start.toString());
```

so an existing `start` value is replaced.

If URL parsing fails, the node appends `?start=` or `&start=` manually.

**---**

**## Security**

The implementation does not accept or send LinkedIn login credentials, cookies, or session tokens.

For workflow usage:

\- Avoid placing sensitive credentials in the search URL

\- Treat credentials embedded in a proxy URL as secrets

\- Avoid logging sensitive URL parameters

\- Review applicable access requirements before automated collection

**---**

**## Notes**

Metadata label:

```text
LinkedIn Job Scraper
```

Metadata description:

```text
Scrape job postings from LinkedIn search results
```

Defaults:

```text
maxJobs = 100
maxRetries = 3
requestTimeout = 20000
delayMinMs = 500
delayMaxMs = 1500
useProxy = false
```

Internal page size:

```text
25
```

The node does not:

\- Use incoming workflow data

\- Authenticate to LinkedIn

\- Send LinkedIn cookies

\- Use a logged-in LinkedIn session

\- Execute page JavaScript

\- Open a browser

\- Use Playwright or Puppeteer

\- Fetch individual job-detail pages

\- Deduplicate jobs

\- Implement a functional proxy agent

\- Cache responses

The parser depends on LinkedIn HTML selectors and may stop finding jobs if the returned markup changes.

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-16 | Initial release |
