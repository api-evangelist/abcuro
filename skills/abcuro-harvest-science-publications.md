---
name: Harvest Abcuro scientific publications and presented data
description: >-
  Pull Abcuro's publication records and the PDF/media assets behind them (MUSCLE topline data,
  GCOM 2026 presentations, the ulviprubart infographic) from the abcuro.com WordPress REST API.
api: openapi/abcuro-content-openapi.yml
operations:
  - listPublications
  - getPublication
  - listMediaItems
  - getMediaItem
  - search
generated: '2026-08-02'
method: generated
source: openapi/abcuro-content-openapi.yml + data-model/abcuro-data-model.yml
---

# Harvest Abcuro scientific publications and presented data

Abcuro registers a custom post type `abcuro_pubs` for the peer-reviewed literature behind its KLRG1
programme, and its media library holds the presented clinical decks and PDFs. Both are readable
anonymously.

## Before you start

- **Base URL:** `https://abcuro.com/wp-json`
- **Auth:** none.
- **Know the limitation up front:** publication records carry **no structured DOI, journal, PubMed
  id or author list**. The `acf` field returned `[]` on every record sampled. Citation detail lives
  inside `content.rendered` HTML and must be parsed. Plan for extraction, not for querying.

## Steps

### 1. List every publication — `listPublications`

```
GET /wp/v2/abcuro_pubs?per_page=100&orderby=date&order=desc&_fields=id,date,slug,link,title
```

13 records as of 2026-08-02 (`X-WP-Total: 13`), so one page at `per_page=100` covers the set.
Titles are substantive, e.g. *"Impact of anti-cytosolic nucleotidase 1A antibody status on disease
phenotype and therapeutic response to ulviprubart in inclusion body myositis"*.

### 2. Read one publication — `getPublication`

```
GET /wp/v2/abcuro_pubs/{id}
```

Parse `content.rendered` for the citation. `featured_media` is a media id (`0` when unset) — pass
it to `getMediaItem` if you need the associated figure.

A bad id returns **404 `rest_post_invalid_id`**.

### 3. Find the presented clinical data in the media library — `listMediaItems`

```
GET /wp/v2/media?per_page=100&_fields=id,slug,title,mime_type,source_url,date
```

141 assets. Filter client-side on `mime_type == "application/pdf"` to isolate documents — the
media collection has no server-side mime filter you can rely on for this. Real assets present
include the MUSCLE topline plain-language summary and the GCOM 2026 oral presentation, plus
`ulviprubart_infographic.jpg`.

`source_url` is the direct, unauthenticated file URL. `media_details` carries dimensions and
generated image sizes.

### 4. Retrieve one asset's metadata — `getMediaItem`

```
GET /wp/v2/media/{id}
```

Use `alt_text` and `caption.rendered` for accessible labelling, and `post` to see which record the
file was uploaded against (`0` when unattached).

### 5. Search across science content — `search`

```
GET /wp/v2/search?search=KLRG1&subtype=abcuro_pubs&per_page=20
GET /wp/v2/search?search=myositis&subtype=any&per_page=20
```

`subtype=any` reaches publications, pages and press releases at once — useful when you do not know
whether a topic was published as a paper or announced as a release.

### 6. Do not forget the narrative pages — `listPages`

The clinical programme itself (ulviprubart, IBM, T-LGLL, T and NK cell lymphomas, additional
indications) is **not modelled as data anywhere on this API**. It is prose inside
`Page.content.rendered` at `/science/`, `/pipeline/` and `/clinical-trials/`. There is no
programme, indication or trial-phase entity to query — see `data-model/abcuro-data-model.yml`.

## Error handling

| Code | Status | What to do |
|---|---|---|
| `rest_post_invalid_id` | 404 | Bad publication or media id. Re-list. |
| `rest_invalid_param` | 400 | `per_page` max is 100. Read `data.params`. |
| `rest_forbidden_context` | 401 | Drop `context=edit`; use the default `view`. |

## Cautions

- Always send `_fields`. Full records embed large `yoast_head` SEO blobs.
- Records are unversioned. `abcuro_pubs` is a site-defined custom post type — its `rest_base`, its
  fields, and its existence can change with any site redesign, with no changelog and no deprecation
  notice. Re-read `https://abcuro.com/wp-json/` and reconcile before each run.
- These are the company's own selections of the literature. Treat them as a curated reading list,
  not as a complete bibliography, and resolve each paper to its journal of record before citing.
