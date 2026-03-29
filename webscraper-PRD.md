# Product Requirements Document
## Website Content Scraper

| Field | Detail |
|-------|--------|
| Project Name | Website Content Scraper |
| Document Type | Product Requirements Document (PRD) |
| Version | 1.0 |
| Date | January 2026 |
| Program | Generative AI Mastermind |
| Institution | Outskill |
| Session | Session 3 — Custom GPTs, AI Bots & Agentic AI |
| Stack | n8n · Firecrawl · OpenAI GPT-4o-mini · Supabase |
| Status | Production — Deployed |

---

## 1. Executive Summary

The Website Content Scraper is a production-quality n8n API that accepts a URL via POST request, scrapes it using Firecrawl, extracts structured content (title, headings, links, word count), generates an AI summary using GPT-4o-mini, and stores the results in Supabase — with per-user rate limiting and full input validation enforced on every request.

The system implements a clean separation between the primary scrape record (URL, title, AI summary, metadata) and individual content items (headings, links) stored in separate Supabase tables, enabling queryable, structured reuse of scraped data.

---

## 2. Problem Statement

Web content extraction is a high-friction task requiring multiple tools: a scraper for raw content, a parser for structure, an AI model for summarization, and a database for persistence. Assembling these manually creates inconsistency, no rate governance, and unstructured output that cannot be queried or reused.

The Website Content Scraper solves this as a single API call — input a URL, receive structured content, AI summary, and database-stored results ready for downstream use.

---

## 3. Objectives

- Accept a POST request with URL and userId; validate both before any processing
- Enforce per-user rate limiting — max 10 scrapes per hour — with HTTP 429 on breach
- Scrape target URL using Firecrawl, extracting markdown, HTML, headings, and links
- Parse and structure content: title, description, headings (up to 50), links (up to 100, classified internal/external), word count
- Generate a 3-4 sentence AI summary using GPT-4o-mini
- Persist scrape record and individual content items to Supabase in normalized tables
- Return clean success response with scrape ID, stats, and summary

---

## 4. Request Pipeline (17 Nodes)

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | Webhook | Webhook (POST) | Entry point — /scrape-website |
| 2 | Validate Input | Code | URL format validation, userId required check |
| 3 | Prepare Rate Check | Code | Pass userId and URL to rate check node |
| 4 | Check Rate Limit | HTTP Request | Query Supabase for scrape count in last hour |
| 5 | Evaluate Limit | Code | Compare count to max (10); set underLimit flag |
| 6 | Under Limit? | IF | Route: proceed to scrape OR return 429 |
| 7 | Firecrawl Scrape | HTTP Request | POST to Firecrawl API; extract markdown + HTML |
| 8 | Parse Content | Code | Extract title, headings, links, word count from markdown |
| 9 | OpenAI Summary | HTTP Request | POST to GPT-4o-mini; generate 3-4 sentence summary |
| 10 | Combine Data | Code | Merge parsed content + AI summary |
| 11 | Save Scrape | HTTP Request | POST scrape record to Supabase scrapes table |
| 12 | Prepare Content Items | Code | Build batch of heading + link records with scrape_id |
| 13 | Save Content Items | HTTP Request | POST content items to Supabase scraped_content table |
| 14 | Success Response | Code | Format clean success response with stats |
| 15 | Respond Success | Respond to Webhook | HTTP 200 with JSON response |
| 16 | Rate Limit Error | Code | Format 429 response with retry-after message |
| 17 | Respond Rate Limit | Respond to Webhook | HTTP 429 with JSON error |

---

## 5. Content Extraction Detail

### 5.1 Firecrawl Output Processing
- Formats requested: markdown and HTML
- onlyMainContent: true — strips navigation, footers, ads
- Included HTML tags: h1–h6, a, title, meta
- 30-second timeout

### 5.2 Parsed Content Fields
- **Title** — page title from metadata
- **Description** — meta description if present
- **Headings** — extracted via regex from markdown; includes level (h1–h6); capped at 50
- **Links** — all hrefs extracted; classified as internal or external; capped at 100
- **Word count** — derived from markdown with formatting stripped

### 5.3 Database Schema
- **scrapes table:** user_id, url, title, scraped_at, ai_summary, metadata (JSON)
- **scraped_content table:** scrape_id (FK), content_type (title/heading/link), content, context (JSON)

---

## 6. Security & Rate Limiting

| Control | Implementation |
|---------|---------------|
| Input validation | URL format validated with regex; userId required; both checked before any downstream calls |
| Per-user rate limiting | 10 scrapes per hour per userId; enforced via Supabase query; HTTP 429 on breach |
| API key management | All credentials stored as n8n environment variables — never hardcoded |
| Response sanitization | Success response exposes only: scrapeId, url, title, summary, stats — no internal details |
| Content caps | Headings capped at 50; links at 100; raw markdown at 5,000 chars |

---

## 7. API Usage

**Request:**
```bash
curl -X POST https://<your-n8n-instance>/webhook/scrape-website \
  -H "Content-Type: application/json" \
  -d '{ "url": "https://example.com", "userId": "user-123" }'
```

**Success Response (HTTP 200):**
```json
{
  "success": true,
  "data": {
    "scrapeId": "uuid",
    "url": "https://example.com",
    "title": "Page Title",
    "summary": "AI-generated summary...",
    "stats": { "wordCount": 850, "headingCount": 12, "linkCount": 34 },
    "scrapedAt": "2026-01-15T10:30:00Z"
  }
}
```

**Rate Limit Response (HTTP 429):**
```json
{
  "success": false,
  "error": "Rate limit exceeded",
  "retryAfter": "1 hour",
  "maxAllowed": 10
}
```

---

## 8. Academic Context

| Field | Detail |
|-------|--------|
| Program | Generative AI Mastermind |
| Institution | Outskill |
| Session | Session 3 — Custom GPTs, AI Bots & Agentic AI |
| Completed | January 2026 |
| Author | Lamonte Smith |
| GitHub | github.com/LSmithPMP/website-content-scraper |

---

*Lamonte Smith · Outskill — Generative AI Mastermind · January 2026 · Confidential*
