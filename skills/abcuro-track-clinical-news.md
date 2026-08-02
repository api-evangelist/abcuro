---
name: Track Abcuro clinical and corporate news
description: >-
  Pull Abcuro's press releases from the abcuro.com WordPress REST API, filter to a date window, and
  read the full announcement text — the reliable way to follow ulviprubart / MUSCLE trial news
  without scraping HTML pages.
api: openapi/abcuro-content-openapi.yml
operations:
  - listPosts
  - getPost
  - listCategories
  - search
generated: '2026-08-02'
method: generated
source: openapi/abcuro-content-openapi.yml + conventions/abcuro-conventions.yml
---

# Track Abcuro clinical and corporate news

Abcuro is a clinical-stage biotech developing **ulviprubart (ABC008)**, a KLRG1-targeting antibody
for inclusion body myositis (IBM), T-LGLL and mature T and NK cell lymphomas. Its News page is
backed by the WordPress core `post` type, so every press release is available as JSON.

## Before you start

- **Base URL:** `https://abcuro.com/wp-json`
- **Auth:** none. Every operation below answers anonymously over TLS. Do not send credentials.
- **There is no rate-limit header and no documented limit.** Back off on 429/503 and keep polling
  conservative — press releases appear a few times a quarter, not hourly.

## Steps

### 1. List recent press releases — `listPosts`

```
GET /wp/v2/posts?per_page=20&orderby=date&order=desc&_fields=id,date,slug,link,title,excerpt
```

Always send `_fields`. Without it every record ships `yoast_head` and `yoast_head_json` SEO blobs
that dwarf the actual content and carry nothing you need.

Read pagination from the response headers, never by paging until empty:

- `X-WP-Total` — total press releases (28 as of 2026-08-02)
- `X-WP-TotalPages` — stop here

Paging past the last page returns **400 `rest_post_invalid_page_number`**, not an empty array.

### 2. Narrow to a date window — `listPosts`

```
GET /wp/v2/posts?after=2026-01-01T00:00:00&orderby=date&order=asc&_fields=id,date,title,link
```

`after`, `before`, `modified_after` and `modified_before` all take ISO 8601 date-times. Use
`modified_after` when you want to catch corrections to already-published releases, which is
material for clinical results announcements.

### 3. Read one release in full — `getPost`

```
GET /wp/v2/posts/{id}
```

`title.rendered`, `excerpt.rendered` and `content.rendered` are **HTML strings, not plain text**,
and they carry HTML entities (`&amp;`, `&#8217;`). Unescape before feeding them to a model or an
index.

A missing or unpublished id returns **404 `rest_post_invalid_id`**.

### 4. Search rather than crawl when you have a term — `search`

```
GET /wp/v2/search?search=ulviprubart&subtype=post&per_page=20
```

`search` spans every content type. Constrain with `subtype` (`post`, `page`, `abcuro_pubs`,
`abcuro_people`, `abcuro_investors`, `abcuro_jobs`, `category`, `post_tag`, `person_role`, `any`).
Results are thin — `{id, title, url, type, subtype}` — so follow `_links.self` or call `getPost`
for the body.

### 5. Categories are not a useful filter here — `listCategories`

```
GET /wp/v2/categories
```

Exactly one term exists: slug `uncategorized`, display name **"Press Release"**, count 28. Every
post carries it. Do not build filtering logic on this taxonomy; use `search` and the date window
instead.

## Error handling

| Code | Status | What to do |
|---|---|---|
| `rest_post_invalid_id` | 404 | The id does not resolve. Re-list; ids are not stable across rebuilds. |
| `rest_invalid_param` | 400 | Read `data.params` for the field-level reason. `per_page` max is 100. |
| `rest_post_invalid_page_number` | 400 | You paged past `X-WP-TotalPages`. |
| `rest_forbidden_context` | 401 | You sent `context=edit`. Use the default `view`. |

Errors are `application/json` with `{code, message, data.status}` — **not** RFC 9457
`problem+json`. Branch on the `code` string. Full catalogue in `errors/abcuro-problem-types.yml`.

## Cautions

- This is a corporate marketing site's content API, not a product API. There is no SLA, no status
  page, no changelog and no deprecation policy — see `lifecycle/abcuro-lifecycle.yml`.
- No writes. Every write route requires a WordPress application password that is not obtainable by
  third parties.
- Press releases are **corporate communications, not regulatory filings**. For trial registration
  and results of record, go to ClinicalTrials.gov, not this API.
