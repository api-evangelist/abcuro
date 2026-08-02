---
name: Map the Abcuro company graph — people, roles and investors
description: >-
  Build a structured view of Abcuro's leadership, team, board of directors and investors from the
  abcuro.com WordPress REST API, using the person_role taxonomy to partition people correctly.
api: openapi/abcuro-content-openapi.yml
operations:
  - listPersonRoles
  - listPeople
  - getPerson
  - listInvestors
  - getInvestor
  - listPages
generated: '2026-08-02'
method: generated
source: openapi/abcuro-content-openapi.yml + data-model/abcuro-data-model.yml
---

# Map the Abcuro company graph — people, roles and investors

Abcuro registers two custom post types for its company graph — `abcuro_people` and
`abcuro_investors` — plus a `person_role` taxonomy that drives the three About pages.

## Before you start

- **Base URL:** `https://abcuro.com/wp-json`
- **Auth:** none.
- **This is public corporate biographical content** — named executives and board members that
  Abcuro publishes on its own About pages. Handle it as such: it is not a contact database. Do not
  join it against personal data from other sources, and do not use it to build outreach lists.

## Steps

### 1. Get the roles first — `listPersonRoles`

```
GET /wp/v2/person_role?per_page=100&_fields=id,slug,name,count
```

Observed terms (2026-08-02):

| id | slug | name | count |
|---|---|---|---|
| 3 | `team` | Team | 17 |
| 4 | `leadership` | Leadership | 3 |
| 5 | `board` | Board | 10 |

**The counts sum to 30 against 26 people.** A person can hold more than one role — leadership
members also sit on the board. Do not assume the partitions are disjoint; deduplicate by person
`id` when you union them.

### 2. List people, optionally filtered by role — `listPeople`

```
GET /wp/v2/abcuro_people?per_page=100&_fields=id,slug,title,person_role,link
GET /wp/v2/abcuro_people?person_role=5&per_page=100&_fields=id,slug,title,person_role
```

26 records total. `person_role` on each record is an **array** of term ids — read it, don't assume
one value.

### 3. Read one person — `getPerson`

```
GET /wp/v2/abcuro_people/{id}
```

`title.rendered` is the name; the bio is HTML in `content.rendered`. There is **no structured job
title, bio or LinkedIn field** — `acf` returned `[]` on every record sampled, so the title has to
be extracted from the rendered HTML. `featured_media` is a media id but was observed as `0` on the
sampled record, so headshots are not consistently resolvable through the API even where they render
on the site.

### 4. List investors — `listInvestors`

```
GET /wp/v2/abcuro_investors?per_page=100&_fields=id,slug,title
```

15 records (e.g. `abrdn`, and logos in the media library for NEA and Samsara BioCapital). Records
carry **only `id`, `slug` and `title`** — no URL, no logo reference, no round, no investment date.
`getInvestor` returns nothing more in the `view` context. If you need firm URLs or round detail,
this API cannot supply it; go to the press releases (`listPosts`) where financings are announced.

### 5. Fall back to the pages for anything structured is missing — `listPages`

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,parent,title,link
```

The About hierarchy is real page structure: `/about/` is the parent of `/about/leadership/`,
`/about/team/`, `/about/board-of-directors/` and `/about/investors/`. `Page.parent` gives you the
tree. The rendered pages carry presentation the raw records lack.

## Error handling

| Code | Status | What to do |
|---|---|---|
| `rest_term_invalid` | 404 | Bad `person_role` term id. Re-list the taxonomy. |
| `rest_post_invalid_id` | 404 | Bad person or investor id. |
| `rest_invalid_param` | 400 | `per_page` max is 100. |

## Cautions

- **Do not use `/wp/v2/users` for this.** That collection returns 4 WordPress *author accounts*,
  not company people, and it enumerates login slugs. It is a distinct entity and a known
  low-severity exposure — see `conformance/abcuro-conformance.yml`.
- `abcuro_people`, `abcuro_investors` and `person_role` are site-defined and unversioned. Their
  names and shapes can change with any redesign, with no announcement. Re-read
  `https://abcuro.com/wp-json/` and reconcile before each run.
- Read-only. There is no write path available to third parties.
