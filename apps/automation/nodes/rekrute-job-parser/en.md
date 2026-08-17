---
node_id: "rekrute-job-parser"
title: "Rekrute Job Parser"
description: "Extracts comprehensive job data including company info, contract details, dates, location, category, job description, profile recherché, personality traits, and technical stack from a single Rekrute job link."
category: "web-search"
subcategory: "Job Boards"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - rekrute
  - morocco
  - jobs
  - parser
  - scraping
  - recruitment
  - web-search
  - emploi
related_nodes:
  - rekrute-scraper
  - emploima
  - linkedin-job-scraper
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# Rekrute Job Parser

> **Category:** Web Search & Information | **Type:** Action Node

Extract deep, granular job specifications, company profiles, required technical skills, personality traits, and contract parameters from any individual [ReKrute.com](https://www.rekrute.com) job listing URL.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Rekrute Job Parser** node provides deep, single-offer HTML extraction for job postings hosted on ReKrute.com. While listing scrapers (such as **Rekrute Scraper**) collect summary cards across multiple pages, the **Rekrute Job Parser** performs an in-depth extraction of a specific job offer page.

The node extracts full structured metadata, including:
- **Company Information:** Enterprise name, logo, corporate presentation, sector, and industry size.
- **Contract & Logistics:** Contract type (CDI, CDD, Freelance, Stage, Intérim), city, region, and remote work policy (`Télétravail`: `Hybride`, `Total`, `Non`).
- **Timestamps:** Publication date, update timestamp, and application closing deadline.
- **Job Content & Scope:** Mission objectives, detailed role responsibilities, and day-to-day duties.
- **Profil Recherché:** Candidate profile, required experience bracket, and minimum education level.
- **Personality Traits & Soft Skills:** Specific behavioural qualities and soft skills tagged by the recruiter (e.g., `Sens de la communication`, `Organisation`, `Autonomie`, `Esprit d'équipe`, `Orientation client`).
- **Technical Stack & Keywords:** Extracted hard skills, tools, programming languages, and certifications.

### Key Features

- **Deep Single-Page Extraction:** Parses every section of a ReKrute job page, transforming unformatted HTML into a rich, structured JSON object.
- **Soft Skills & Trait Recognition:** Extracts structured personality traits and behavioral qualities evaluated by Moroccan recruiters.
- **Technical & Industry Tagging:** Isolates required technical competencies, software proficiencies, and educational thresholds.
- **Dynamic Expression Support:** Bind incoming URLs dynamically from upstream webhooks, database queues, or the **Rekrute Scraper** iteration loop.
- **Zero API Key Requirement:** Parses publicly available job listings directly with built-in HTTP handling.

### Parsing Flow

```text
Incoming Job URL (e.g. https://www.rekrute.com/offre-emploi-...-185572.html)
                                    ↓
                        Fetch Single Job HTML Page
                                    ↓
                         Parse Structured Sections
      ┌─────────────────┬─────────────────┬─────────────────┐
      ↓                 ↓                 ↓                 ↓
Company Profile   Role & Missions   Candidate Profile  Traits & Skills
(Name, Sector)   (Responsibilities) (Studies, Years)   (Soft/Hard Skills)
      └─────────────────┬─────────────────┴─────────────────┘
                                    ↓
                     Emit Complete JSON Job Object
              (For AI Matching / ATS Sync / Notifications)
```

### Use Cases

- **AI-Powered Resume Matching:** Feed the comprehensive job requirements, responsibilities, and soft skills into an LLM node to grade candidate CV suitability.
- **ATS & HRIS Synchronization:** Ingest full job postings from ReKrute directly into internal recruitment software (e.g. Greenhouse, Lever, Odoo, Workday).
- **Enriched Recruitment Alerts:** When a new job URL is detected, parse full missions and requirements before generating detailed Slack or Email digests.
- **Competitor & Labor Market Benchmarking:** Track detailed job specifications, required tech stacks, and compensation criteria across leading Moroccan employers.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `url` | `string` | ✅ Yes | — | The full canonical URL of the ReKrute.com job listing (e.g., `https://www.rekrute.com/offre-emploi-chef-datelier-mecanique-recrutement-bouderka-marrakech-185572.html`). |

---

### Parameter Details

#### `url`
The complete web address pointing to an individual job offer on ReKrute.com.
- **Type:** `string`
- **Required:** Yes
- **Supported Format:** `https://www.rekrute.com/offre-emploi-[slug]-[id].html` or dynamic expression `{{ $json.lien_offre }}`.
- **Example:** `"https://www.rekrute.com/offre-emploi-chef-datelier-mecanique-recrutement-bouderka-marrakech-185572.html"`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming trigger or upstream workflow payload. Can supply the `url` parameter dynamically via expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the job offer is successfully parsed. Contains complete job, company, requirement, and skill metadata. |
| `error` | `Error` | Emitted if the URL is invalid, the job listing has expired/been removed (404), or network timeouts occur. |

---

### Output Data Structure

The `success` output delivers a detailed, structured JSON object:

```json
{
  "success": true,
  "url": "https://www.rekrute.com/offre-emploi-chef-datelier-mecanique-recrutement-bouderka-marrakech-185572.html",
  "id_offre": "185572",
  "titre": "Chef d'Atelier Mécanique Poids Lourds",
  "entreprise": {
    "nom": "Bouderka Recrutement & Conseil",
    "secteur": "Automobile / Matériel de transport",
    "presentation": "Cabinet spécialisé dans le recrutement et le conseil en ressources humaines au Maroc...",
    "logo_url": "https://www.rekrute.com/logos/bouderka_logo.jpg"
  },
  "poste": {
    "ville": "Marrakech",
    "region": "Marrakech-Safi",
    "type_contrat": "CDI",
    "statut": "Cadre",
    "teletravail": "Non",
    "niveau_etudes": "Bac +2 / BTS / DUT en mécanique",
    "niveau_experience": "De 5 à 10 ans",
    "fonction": "Production / Maintenance / Mécanique",
    "date_publication": "17/08/2026",
    "date_limite": "17/09/2026"
  },
  "missions": [
    "Superviser et coordonner les opérations quotidiennes de l'atelier mécanique.",
    "Planifier les opérations de maintenance préventive et corrective sur la flotte poids lourds.",
    "Gérer les plannings des équipes de mécaniciens et techniciens d'atelier.",
    "Assurer la gestion des stocks de pièces de rechange et optimiser les coûts de réparation."
  ],
  "profil_recherche": "De formation technique en mécanique automobile ou poids lourds, vous justifiez d'au moins 5 ans d'expérience réussie en management d'atelier...",
  "traits_personnalite": [
    "Sens de l'organisation",
    "Rigueur et méthode",
    "Leadership et animation d'équipe",
    "Sens de la communication",
    "Réactivité et gestion du stress"
  ],
  "competences_techniques": [
    "Diagnostic mécanique et électronique",
    "Gestion de stock GMAO",
    "Maintenance préventive poids lourds",
    "Réglementation transport et sécurité"
  ],
  "description_complete": "Bouderka Recrutement recherche pour l'un de ses clients majeurs basé à Marrakech un Chef d'Atelier Mécanique Poids Lourds...",
  "lien_postuler": "https://www.rekrute.com/offre-emploi-chef-datelier-mecanique-recrutement-bouderka-marrakech-185572.html"
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates whether parsing succeeded. |
| `url` | `string` | Canonical URL of the parsed job listing. |
| `id_offre` | `string` | Unique alphanumeric listing identifier on ReKrute.com. |
| `titre` | `string` | Full job title. |
| `entreprise.nom` | `string` | Name of the hiring company or recruitment agency. |
| `entreprise.secteur` | `string` | Industry sector (e.g. IT, Banque, Automobile, Industrie). |
| `entreprise.presentation` | `string` | Overview and background of the hiring organization. |
| `entreprise.logo_url` | `string` | Direct link to the company's official logo image. |
| `poste.ville` | `string` | City where the job is located. |
| `poste.region` | `string` | Moroccan administrative region. |
| `poste.type_contrat` | `string` | Contract model (`CDI`, `CDD`, `Freelance`, `Stage`, `Intérim`). |
| `poste.teletravail` | `string` | Remote work framework (`Hybride`, `Total`, `Non`). |
| `poste.niveau_etudes` | `string` | Required academic diploma level (e.g. `Bac +5`, `Bac +2`). |
| `poste.niveau_experience` | `string` | Experience bracket required. |
| `poste.fonction` | `string` | Functional domain classification. |
| `poste.date_publication` | `string` | Publication date string. |
| `poste.date_limite` | `string` | Application deadline date. |
| `missions` | `string[]` | Array of parsed mission statements and core responsibilities. |
| `profil_recherche` | `string` | Full narrative describing the ideal candidate profile. |
| `traits_personnalite` | `string[]` | Extracted behavioral traits and soft skill tags. |
| `competences_techniques` | `string[]` | Extracted technical proficiencies, tools, and certifications. |
| `description_complete` | `string` | Complete raw text content of the offer. |
| `lien_postuler` | `string` | Direct application link. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Parse a Single Engineering Job Offer

Extract full details from a technical role in Marrakech.

**Parameter Configuration:**

```text
Url: https://www.rekrute.com/offre-emploi-chef-datelier-mecanique-recrutement-bouderka-marrakech-185572.html
```

**Result:**
Returns the complete JSON payload with company profile, missions array, required soft skills, and contact links.

---

### Example 2: Dynamic Pipeline with Rekrute Scraper

Combine **Rekrute Scraper** and **Rekrute Job Parser** to crawl recent listings and parse each one deeply.

**Workflow Pipeline:**

```text
Cron Trigger (Daily)
  → Rekrute Scraper (maxPages: 2)
  → For Each (iterate over $json array)
  → Rekrute Job Parser (url: {{ $json.lien_offre }})
  → AI Agent (Match with talent pool)
  → Slack / Discord Notification
```

---

### Example 3: ATS Synchronization Pipeline

Parse a specific Rekrute job link submitted via webhook and insert the job into an internal ATS database.

**Workflow Pipeline:**

```text
Webhook Trigger (Receives: { "job_url": "https://www.rekrute.com/..." })
  → Rekrute Job Parser (url: {{ $json.job_url }})
  → Function (Map to ATS schema)
  → PostgreSQL Action (Insert into "job_openings" table)
  → Log (Print confirmation)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Extracts comprehensive job data from a single Rekrute job link
```

### Common Workflow Patterns

- **Deep Parser Pipeline:** `Manual Trigger` → `Rekrute Job Parser` (`url: "https://www.rekrute.com/..."`) → `Log` console output.
- **Batch Listing Deep-Dive:** `Rekrute Scraper` → `For Each` → `Rekrute Job Parser` → `Vector Store / Pinecone` (Create embeddings for semantic search).
- **Candidate Fit Assessment:** `Webhook Trigger` (Resume + Job URL) → `Rekrute Job Parser` → `AI Chat / LLM` (Generate fit report).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Expired or Removed Job Listing (HTTP 404)
- **Symptom:** Node returns an error stating the page could not be found.
- **Cause:** ReKrute.com removes or unpublishes job offers once the deadline expires or the position is filled.
- **Solution:** Verify the link in your web browser. In automated loops, handle errors with an **If-Else** or error-branching handler.

#### Invalid URL Format
- **Symptom:** Node throws `Invalid Rekrute URL`.
- **Cause:** The provided URL is not a valid ReKrute job page (e.g. pointing to the homepage or search listing page instead of a detail page).
- **Solution:** Ensure the URL matches the pattern `https://www.rekrute.com/offre-emploi-*.html`.

#### Rate Limiting on High-Volume Loops
- **Symptom:** HTTP 429 or connection refused when processing dozens of links in a loop.
- **Cause:** Iterating through hundreds of URLs too quickly without pausing.
- **Solution:** Place a **Delay** node (e.g. 500ms–1s) inside the **For Each** loop before invoking the parser.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `URL parameter is required` | The `url` field was empty or null | Supply a valid ReKrute job URL |
| `Invalid ReKrute URL format` | URL does not match ReKrute job offer pattern | Check that URL points to `/offre-emploi-...html` |
| `Job listing not found (404)` | Offer expired or deleted from ReKrute | Verify the job is still active online |
| `Network error / Timeout` | Target server unreachable | Check outbound internet connection and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Rekrute Scraper](../rekrute/en.md) — Crawl multi-page job listing summaries from ReKrute.com
- [Emploi.ma Scraper](../emploima/en.md) — Scrape job listings from Emploi.ma
- [LinkedIn Job Scraper](../linkedin-job-scraper/en.md) — Extract job listings from LinkedIn
- [Function](../function/en.md) — Transform, sanitize, or filter parsed job records
- [HTTP Request](../http-request/en.md) — Make custom API calls to external ATS or HR platforms
- [Log](../log/en.md) — View extracted job data in the workflow debug console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial release supporting granular parsing of ReKrute job listings including company profiles, missions, traits de personnalité, and technical requirements |

<!-- /SECTION: changelog -->
