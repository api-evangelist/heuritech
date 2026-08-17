---
name: Search the Heuritech estate
description: Find any post or page on heuritech.com by keyword and resolve each hit to its full resource, using the public WordPress search endpoint.
api: openapi/heuritech-search-api-openapi.yml
operations: [getSearch, getPostsId, getPagesId]
generated: '2026-08-17'
method: generated
---

# Search the Heuritech estate

`GET /wp/v2/search` indexes 232 items across Heuritech's posts and pages. It is the fastest way to
answer "does Heuritech say anything about X" without crawling the site.

**Base URL:** `https://heuritech.com/wp-json/wp/v2`
**Auth:** none.

## Steps

1. **Search.** Call `getSearch` (`GET /wp/v2/search`) with:
   - `search=<term>` — the query
   - `type=post` — the searchable object type (`post` covers both posts and pages here)
   - `subtype=post` or `subtype=page` — narrow to one content kind, or omit for both
   - `per_page=` (1–100) and `page=` for paging

2. **Read the hit shape.** Each result is deliberately thin: `id`, `title`, `url`, `type`, `subtype`
   and a `_links` block. There is no excerpt and no body. Do not answer from the title alone.

3. **Resolve each hit.** Branch on `subtype`:
   - `post` → `getPostsId` (`GET /wp/v2/posts/{id}`)
   - `page` → `getPagesId` (`GET /wp/v2/pages/{id}`)

   Or follow `_links.self[0].embeddable`/`href`, which already points at the right collection.

4. **Page.** Use `X-WP-Total` / `X-WP-TotalPages` and the `Link: …; rel="next"` header, exactly as for
   any other collection.

## Rules

- **Relevance ordering is available but not the default.** Pass `orderby=relevance` when the query is
  a genuine keyword search rather than a filter.
- **A zero-result search is a real answer.** Report it as "Heuritech has published nothing on this",
  not as an error, and do not fall back to inventing content.
- **Search covers marketing and editorial content only.** It does not reach the Heuritech trend data
  product, the customer platform at `market-trends.heuritech.com`, or the gated `knowledge` post type
  (which returns HTTP 401 `rest_cannot_read`).
- **Errors:** HTTP 400 `rest_invalid_param` for a bad `per_page`/`type`/`subtype`; the WordPress
  envelope `{code, message, data:{status}}` throughout.
