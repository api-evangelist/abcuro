# Abcuro

Abcuro is a clinical-stage biotechnology company in Newton, Massachusetts, developing first-in-class
immunotherapies that precisely modulate highly cytotoxic T cells for autoimmune disease and cancer.
Its lead candidate, **ulviprubart (ABC008)**, is a KLRG1-targeting monoclonal antibody evaluated in
the registrational Phase 2/3 MUSCLE study in inclusion body myositis (IBM), and in T cell large
granular lymphocytic leukemia (T-LGLL) and mature T and NK cell lymphomas.

- Website: https://abcuro.com/
- Secondary-market profile: https://forgeglobal.com/abcuro_stock/

## API posture

Abcuro publishes **no developer platform**. There is no API key, signup, pricing, SDK, CLI,
sandbox, changelog, status page, MCP server, agent card or webhook surface, and no `/.well-known/`
document of any kind.

The one machine-readable surface the company exposes is the **public WordPress REST API** behind
`abcuro.com`, at `https://abcuro.com/wp-json/`. It answers anonymously and serves the corporate
content graph, including four Abcuro-specific custom post types. Counts observed 2026-08-02:

| Collection | Records |
|---|---|
| `/wp/v2/posts` — press releases | 28 |
| `/wp/v2/pages` — site pages | 17 |
| `/wp/v2/media` — assets incl. MUSCLE topline PDFs | 141 |
| `/wp/v2/abcuro_people` — leadership, team, board | 26 |
| `/wp/v2/abcuro_investors` | 15 |
| `/wp/v2/abcuro_pubs` — scientific publications | 13 |
| `/wp/v2/abcuro_jobs` | 0 |

This is a **website content API, not a product API** — no SLA, no versioning policy, no deprecation
notice. It is catalogued here because it is real, public, and the only contract Abcuro exposes.

## Artifacts

| Artifact | File | Method |
|---|---|---|
| OpenAPI 3.1 — 30 operations, 14 schemas | `openapi/abcuro-content-openapi.yml` | derived |
| Route index, verbatim | `openapi/_original/abcuro-wp-json-routes.json` | searched |
| Authentication | `authentication/abcuro-authentication.yml` | derived |
| Conventions | `conventions/abcuro-conventions.yml` | derived |
| Error catalog — 8 of 9 codes observed live | `errors/abcuro-problem-types.yml` | probed |
| Data model — 13 entities, 16 relationships | `data-model/abcuro-data-model.yml` | derived |
| Lifecycle | `lifecycle/abcuro-lifecycle.yml` | probed |
| Conformance + security findings | `conformance/abcuro-conformance.yml` | derived |
| Well-known probes (all 404) | `well-known/abcuro-well-known.yml` | probed |
| Domain security | `security/abcuro-domain-security.yml` | probed |
| Agentic access — 30 ops | `agentic-access/abcuro-agentic-access.yml` | generated |
| MCP candidate — 16 tools | `mcp/abcuro-mcp.yml` | derived |
| Tool crosswalk | `mcp/abcuro-tool-crosswalk.yml` | derived |
| Overlay | `overlays/abcuro-content-overlay.yaml` | generated |
| Agent skills — 3 | `skills/` | generated |
| llms.txt | `llms/abcuro-llms.txt` | generated |

The OpenAPI was **derived by API Evangelist** from the live route discovery document; it is not
published or endorsed by Abcuro.

## Not present, verified

No A2A agent card (`/.well-known/agent-card.json` and the legacy `/.well-known/agent.json` both
404 on every host), no AsyncAPI or event surface, no GraphQL, no gRPC, no published MCP server, no
first-party packages on npm/PyPI/RubyGems/NuGet/crates.io, no GitHub organization, no security.txt,
no vulnerability disclosure programme, no trust center, no idempotency contract, and no documented
rate limits.
