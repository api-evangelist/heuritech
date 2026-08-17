---
name: Harvest the Heuritech product and pricing pages
description: Read Heuritech's platform, pricing, FAQ and legal pages as structured JSON instead of scraping rendered HTML — the correct way to answer what Heuritech sells and what it costs.
api: openapi/heuritech-pages-api-openapi.yml
operations: [getPages, getPagesId, getMediaId]
generated: '2026-08-17'
method: generated
---

# Harvest the Heuritech product and pricing pages

heuritech.com renders through a page builder, so scraping the HTML is fragile. All 63 published pages
are available as JSON from the same host.

**Base URL:** `https://heuritech.com/wp-json/wp/v2`
**Auth:** none.

## Steps

1. **List the pages.** Call `getPages` (`GET /wp/v2/pages`) with `per_page=100` and
   `_fields=id,slug,title,link,parent,menu_order,modified` to get a cheap index of the whole estate in
   one call.

2. **Find the page you want by slug.** The commercially interesting ones are:
   - `heuritech-market-insights` — the Platform & API product page
   - `pricing` — the three published tiers
   - `faq` — data sources, geographies, AI approach
   - `company-about-us`, `luxurynsight-group` — company and parent
   - `terms-conditions`, `privacy-policy` — legal
   You can also filter server-side with `slug=pricing`.

3. **Fetch the page.** Call `getPagesId` (`GET /wp/v2/pages/{id}`). `content.rendered` is an HTML
   string — render or strip it; never present it raw. `modified` tells you how stale the page is.

4. **Follow the hierarchy.** `parent` is a page id (`0` for top level) and `menu_order` gives sibling
   ordering, so you can reconstruct the site tree without crawling navigation.

5. **Pull any embedded asset.** `featured_media` is a media id → `getMediaId`
   (`GET /wp/v2/media/{id}`) → `source_url`.

## Rules

- **Prefer the captured artifacts over re-reading the page.** Pricing is already structured in
  `plans/heuritech-plans-pricing.yml` (Essential from 12k€/year, Business from 35k€/year, Enterprise
  contact-sales, all annual, API access from Business up). Only re-read the page to check for change.
- **Do not infer an API contract from the product page.** It describes the Trend Data API in prose —
  "weekly data points", "6 year historical data and 2 years forecast", "1000+ trends" — and names no
  endpoint, parameter, field, base URL or auth scheme. There is no public specification to derive.
- **The legal pages are stale and should be quoted with dates.** Terms & Conditions last updated
  2024-07-31; the privacy policy last updated 2018-11-25 and still cites French law n° 78-17 rather
  than the GDPR.
- **Errors:** HTTP 404 `rest_post_invalid_id` for an unknown page id; HTTP 400 `rest_invalid_param`
  for a `per_page` outside 1–100.
