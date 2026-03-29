# Security Policy

## Credentials & Secrets

All credentials are managed as n8n environment variables — never hardcoded in workflow nodes:

- **Supabase service key** — stored as n8n environment variable
- **Firecrawl API key** — stored as n8n header auth credential
- **OpenAI API key** — stored as n8n header auth credential

## Security Controls

- **Input validation** — URL format validated with regex before any downstream calls; userId required
- **Rate limiting** — 10 scrapes per hour per user enforced via Supabase; HTTP 429 on breach
- **Response sanitization** — success response exposes only scrapeId, url, title, summary, and stats — no internal system details
- **Content caps** — headings capped at 50, links at 100, raw markdown at 5,000 chars

## Reporting a Vulnerability

1. **Do not** open a public GitHub issue
2. Email: lamontesmithpmp@gmail.com with subject line `[SECURITY] website-content-scraper`
3. Expected response time: 72 hours

## Disclaimer

This workflow was built for academic demonstration purposes as part of the Outskill Generative AI Mastermind program. Before deploying to production, conduct a full security review appropriate to your use case.
