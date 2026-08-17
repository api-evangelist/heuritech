---
name: Read the Heuritech trend editorial
description: Pull Heuritech's published fashion-trend analysis by category, date window or search term, with author attribution and imagery, from the public WordPress content API.
api: openapi/heuritech-posts-api-openapi.yml
operations: [getCategories, getPosts, getPostsId, getUsersId, getMediaId]
generated: '2026-08-17'
method: generated
---

# Read the Heuritech trend editorial

Heuritech publishes 169 analysis posts on fashion trend forecasting, consumer insight, fashion tech
and company analysis. They are readable as JSON with no credentials.

**Base URL:** `https://heuritech.com/wp-json/wp/v2`
**Auth:** none for reads. Do not send an Authorization header.

## Before you start

This is the **content** API. It does not contain Heuritech trend data — no attribute market shares, no
forecast series, no brand tracking. Those live in the commercial Trend Data API, which is sales-gated
and has no public contract. If a user asks for trend *numbers*, say so and point them at
<https://heuritech.com/get-a-demo/>; do not attempt to synthesise the numbers from blog prose.

## Steps

1. **Pick a category.** Call `getCategories` (`GET /wp/v2/categories`). Heuritech uses five:
   `articles` (169), `industry-insights` (103), `consumer-insights` (66), `fashion-tech` (47),
   `company-analysis` (30). Keep the numeric `id` — the posts endpoint filters by id, not slug.

2. **List posts.** Call `getPosts` (`GET /wp/v2/posts`) with the filters you need:
   - `categories=<id>` — one or more category ids
   - `search=<term>` — full-text; pair with `search_columns` to narrow to `post_title`
   - `after=` / `before=` — ISO 8601 date window on publication
   - `orderby=date&order=desc` — newest first (the default)
   - `per_page=` — **1 to 100 inclusive**. Anything outside that range returns HTTP 400
     `rest_invalid_param`, not a clamped result.

3. **Page through the collection.** Read `X-WP-Total` and `X-WP-TotalPages` from the response headers
   and increment `page`. Prefer the `Link: …; rel="next"` header over constructing the next URL
   yourself. Do not walk past `X-WP-TotalPages` — the next page returns a 400.

4. **Fetch one post in full.** Call `getPostsId` (`GET /wp/v2/posts/{id}`). `title.rendered`,
   `excerpt.rendered` and `content.rendered` are HTML strings; strip or render them, never treat them
   as plain text. An unknown id returns HTTP 404 `rest_post_invalid_id`.

5. **Attribute the author.** The post's `author` field is a user id. Call `getUsersId`
   (`GET /wp/v2/users/{id}`) for the name, biography and avatar. Anonymous callers get name, slug,
   description, link and avatar_urls only — email and role require authentication.

6. **Resolve the image.** The post's `featured_media` is a media id. Call `getMediaId`
   (`GET /wp/v2/media/{id}`) and use `source_url` for the original, or pick a rendition from
   `media_details.sizes`. `featured_media: 0` means no image — do not call the endpoint with id 0.

## Rules

- **Never send `context=edit`.** It requires authentication and returns HTTP 401 `rest_forbidden`.
- **Errors are not RFC 9457.** Every failure is `{"code": …, "message": …, "data": {"status": …}}`.
  Branch on `code`, not on the message text. See `errors/heuritech-problem-types.yml`.
- **There is no rate limit signal.** No `RateLimit-*`, `X-RateLimit-*` or `Retry-After` header is
  returned, and no limit is published. Be conservative: this is a marketing site's CMS, not a metered
  product API. Batch with `per_page=100` rather than issuing many single-item calls.
- **There is no idempotency contract.** Reads are safe to retry by HTTP method; nothing more is
  promised.
- **Verify identity before trusting a response.** Responses are served through a shared CDN. If a
  route index or payload looks wrong, re-read `https://heuritech.com/wp-json/` and confirm
  `name == "Heuritech"` and `home == "https://heuritech.com"` before using it.
