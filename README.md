# MegaMicroParser — price scraper and catalog builder for MicroCatalog

A Python service that collects prices and stock of electronic components from Ukrainian online shops, matches shop
listings to a catalog of parts, and grows that catalog from what the shops actually sell. It feeds
[MicroCatalog](https://github.com/San4ost/microcatalog-showcase), a price-comparison website for the Ukrainian market
(Ukrainian shops vs. AliExpress, landed price including delivery). The source code is in a private repository; this page
describes the project.

## What it does

- **13 shops**, including the Prom.ua marketplace, without per-site code. A shop is described by a data profile (CSS
  selectors with a JSON-LD fallback). Onboarding a new shop is one command: an LLM proposes selectors and the scraper
  validates them against canary products.
- **Polite crawling**: honors robots.txt, rate limits with jitter, retries on 429/5xx, stops after repeated 403. It never
  bypasses protection (no CAPTCHA solving, proxy rotation or disguise); a shop that objects is switched off.
- **Product matching** ("seller title → catalog part"): layered rules (part numbers, packages, modules, kits, analogs,
  lots) plus an LLM for ambiguous cases. A golden set of 349 labeled titles measures it; wrong matches to a different
  part: 0.
- **Self-checks**: profile health check on every run (canary product, share of parsed prices), automatic retraction of
  matches that current rules no longer agree with, and a second opinion from a stronger model on the fast model's
  decisions.
- **Catalog growth**: unrecognized part numbers collect in a local market index, become candidates with LLM drafts
  (title, package, category), then are approved by a human or auto-approved under strict conditions with rollback and an
  automatic stop. The catalog grew from 141 to 484 parts.
- **Enrichment**: part photos (WebP, uploaded to Supabase Storage over S3), specifications, kits as a separate feed, and
  datasheet discovery from manufacturer sites (PDF text verification, link only, no copies published).
- **New shop discovery** through the Tavily API with automatic domain checks.
- **LLM fault tolerance**: fallback model, circuit breaker for the primary, response cache, client for any
  OpenAI-compatible API.
- **Result**: about 8,800 offers live on the site.

## Stack

Python 3.13, httpx, selectolax, pydantic, psycopg 3, PostgreSQL (Supabase), SQLite, Gemini API (Gemma 4, Gemini),
Tavily API, boto3 (S3), Pillow, pypdf, pytest (350+ tests, integration tests on a disposable Postgres), uv.
