# Changelog

All notable changes to mcp-forge will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.2.3] — 2026-07-20

### ✨ Added

- **Comparison table in README / PyPI.** New "How it compares" section benchmarks
  mcp-forge against FastMCP, Smithery, and openapi-mcp-generator across 14 features
  with an honest positioning conclusion.
- **Troubleshooting page** at `/docs/troubleshooting/no-tools`. Covers the four
  most common causes of `NO_TOOLS_EXTRACTED`: wrong OpenAPI URL, GraphQL
  introspection disabled, JavaScript SPA requiring Playwright, and codebase
  with no public functions. Linked from every `NO_TOOLS_EXTRACTED` error response.

### 🐛 Fixed

- **`hint_url` in error responses** now points to the live troubleshooting page
  (`https://mcp-forge.vulcai.io/docs/troubleshooting/no-tools`) instead of
  the dead `docs.mcp-forge.io` URL.

---

## [0.2.2] — 2026-07-20

Builds on 0.2.1 with SaaS platform hardening: admin panel, beta-mode licence
provisioning, multi-LLM eval/enrichment fallback chain, and dashboard isolation fixes.

### ✨ Added

- **Admin panel** (`/admin`). Secured with `is_admin` flag on user accounts
  (replaces `FORGE_ADMIN_KEY` env var). Exposes licence management, user table
  with search/sort, and a "create & link licence" action per user.
  `scripts/create_admin.py` CLI to promote/demote admins from the terminal.
- **Beta-mode licence provisioning.** When `FORGE_BETA_MODE=true`, a
  `mfg_beta_*` licence is automatically created and linked on user registration.
  The "create & link" admin action also uses the `mfg_beta_` prefix in beta mode.
- **Multi-LLM fallback chain for eval & enrichment.** Eval now runs Groq first
  (fast/cheap), falls back to Anthropic on rate-limit, and supports Mistral as
  an additional provider. Enrichment mirrors the same Groq-first / Anthropic-
  fallback strategy.

### 🔄 Changed

- **Dashboard generation isolation.** Generations are now bound to
  `user.tenant_id` instead of the licence, removing ambiguity when a user
  switches plan.
- **`POST /dashboard/link-license` removed.** Licence linking is now handled
  exclusively through the admin panel; the dashboard route is no longer exposed.
- **CLI output directory** now derived deterministically from the URL hostname,
  preventing collisions when generating from multiple endpoints in the same
  session.

### 🐛 Fixed

- **White Label plan** now uses the correct slug `white_label` (was `enterprise`).
- **`TemplateResponse` argument order** corrected for the Starlette modern API
  (request first, then template name).
- **Generation FK constraint** on `generation_jobs` is nullified before old
  generation rows are purged by the retention reaper.

### 🛠 Internal

- `api/scripts/create_admin.py`: interactive CLI for admin account management.
- `docs/ops/runbook.md`: personal runbook covering Azure env vars, admin setup,
  beta mode, and log access.

---

## [0.2.1] — 2026-07-20

Builds on 0.2.0 with Phase 2 improvements: enriched API observability, legacy
parser test coverage, CLI language flag, template dependency alignment, and a
README comparison table for PyPI discoverability.

### ✨ Added

- **Legacy parser test suite.** New `tests/parsers/` with 85 tests covering
  COBOL, Fortran, Java, and C++ parsers (paragraph extraction, type mapping,
  slug formatting, metadata, edge cases). Total suite: **363 tests passing**.
- **`/v1/health` enriched.** Response now includes `version` (read from
  `importlib.metadata`), `region` (from `FORGE_REGION` env var), and
  `degraded_features` (e.g. `["llm_enrichment"]` when no LLM key is
  configured). `status` switches to `"degraded"` when the list is non-empty.
- **`--lang` flag for generated code comments.** `mcp-forge run --lang fr`
  generates server comments in French; default is English (`--lang en`).
  Header strings and response fallbacks are now driven by an `i18n` dict
  injected into all Jinja2 templates.
- **Comparison table in README / PyPI.** New "How it compares" section
  benchmarks mcp-forge against FastMCP, Smithery, and openapi-mcp-generator
  across 14 features with an honest positioning conclusion.
- **Quality evaluation section on home page.** The six scoring criteria
  (structure, coverage, auth, docs, fidelity, robustness) are now surfaced
  on the landing page and in the PyPI README.
- **Forgot-password flow.** Full email-based password reset: JWT token (1 h),
  Resend SMTP integration (`smtp.resend.com:465`), anti-enumeration POST
  (always returns 200), `/forgot-password` and `/reset-password` routes,
  branded HTML templates.

### 🔄 Changed

- **`validate_license` / `get_quota` fully separated.** `GET /v1/license/validate`
  now returns only identity fields (`valid`, `reason`, `plan`, `tenant_id`,
  `valid_until`, `license_mode`) and always responds HTTP 200 (never 401).
  Quota fields (`sources_used`, `sources_limit`) are exclusive to
  `GET /v1/account/quota`. New `reason` field explains invalid keys:
  `"no key provided"` · `"unknown key"` · `"license revoked"` · `"license expired"`.
- **Template dependency pinning relaxed.** `requirements.txt.j2` and
  `requirements_website.txt.j2` now use `~=` (compatible-release) instead
  of `==`, allowing generated servers to receive patch-level updates without
  manual intervention.
- **MCP server descriptions enriched.** `get_health`, `validate_license`,
  `get_quota`, and all `generate_from_*` tools now carry structured
  "When to call / Prerequisites / Returns JSON" descriptions for agent guidance.
- **BOM removed from 7 templates.** `server_cli`, `server_codebase*`,
  `server_graphql`, and `server_website` templates were UTF-8-BOM-prefixed,
  causing silent corruption on some editors. All now saved as clean UTF-8.

### 🛠 Internal

- `api/config.py`: 5 SMTP settings added (`SMTP_HOST/PORT/USER/PASSWORD/FROM`)
  and `FORGE_REGION` env var for health endpoint.
- `api/services/auth.py`: `create_reset_token()` / `decode_reset_token()`
  (JWT, `type=reset` claim, 1 h expiry).
- `api/services/email.py`: new transactional email service with SSL/STARTTLS
  auto-detection and silent fallback to log when SMTP is not configured.
- `deploy-api.yml`: SMTP secrets wired through `az containerapp secret set`
  and `FORGE_REGION` passed as env var.

---

## [0.2.0] — 2026-05-29

This release is the result of an internal Phase 1 hardening sprint that
restructures the agent-facing surface of mcp-forge ahead of the public beta.

**This is a breaking release.** The CLI `0.1.x` is **not compatible** with the
`0.2.0` backend — please upgrade with `pip install -U vulcai-mcp-forge-cli`.

### ✨ Added

- **High-level MCP tools for agents.** Agents can now generate an MCP server
  by passing only a public URL — no pre-parsing required. New tools exposed by
  the public MCP server at `mcp.mcp-forge.vulcai.io/sse`:
  - `generate_from_openapi(spec_url, name?)` — server-side OpenAPI/Swagger parsing
  - `generate_from_graphql(endpoint_url, name?, headers?)` — server-side introspection
  - `generate_from_website(url, name?)` — server-side HTML/JS pattern analysis
- **Async job pattern.** Long-running generations no longer time out on the MCP
  client side. New endpoints:
  - `POST /v1/jobs` — submits a job, returns `{job_id, status: "pending"}` in <100 ms
  - `GET /v1/jobs/{job_id}` — current job status and metadata
  - `GET /v1/jobs/{job_id}/result` — full result when ready (200) or 404/410
  - New MCP tools `generate_async` (advanced) and `get_job_status` for agent-driven polling
- **Bundle download endpoint.** New authenticated endpoint
  `GET /v1/generations/{generation_id}/download` returns the generated server
  as a zip archive. Multi-tenant isolation enforced; 404 on wrong tenant
  (no leak of generation IDs).
- **Public Trust & Security page** at `/trust`, covering encryption, hosting,
  data retention, LLM access, compliance, and subprocessors. Linked from the
  global footer on every page.
- **Quality scoring is now agent-visible.** The `eval_score` (0–10) and a
  breakdown across six criteria (structure, coverage, auth, docs, fidelity,
  robustness) are now returned in every generation response.
- **Startup reaper** for stale background jobs. Any job stuck in `pending` or
  `running` for more than 10 minutes is automatically marked `failed` with
  `error.code=SERVER_RESTART` on the next API startup.
- **Founder pricing framing.** Beta users see a clear "Free during public
  beta · 50 % off for 12 months when billing starts" notice in place of the
  previous "Stripe integration coming soon" message.

### 🔄 Changed (breaking)

- **`POST /v1/generate` response schema is redesigned.** The endpoint no longer
  inlines file contents in the JSON response. Instead it returns:
  - `bundle_url` — authenticated download URL for the generated zip
  - `preview_tools` — first 5 generated tools (name + description) for quick
    agent feedback
  - `eval_score`, `eval_report` — quality scoring (see above)
  - `success: false` (with HTTP 200) when zero tools are extracted, together
    with a structured `errors` array containing actionable hints. The previous
    behaviour of returning `success: true` with `tools_count: 0` was a silent
    failure and is no longer possible.
- **`/v1/license/validate` and `/v1/account/quota` responses** now return the
  string `"unlimited"` (and `is_unlimited: true`) instead of `null` for
  unbounded plans, removing ambiguity for client agents.
- **License model renamed `sites` → `sources`** end-to-end:
  - Database table `sites` → `sources`
  - Column `licenses.sites_limit` → `licenses.sources_limit`
  - API fields `sites_used` / `sites_limit` → `sources_used` / `sources_limit`
  - Service function `register_site()` → `register_source()`
  - Dashboard and billing labels updated accordingly
  - Existing databases are automatically migrated on first startup
    (`_rename_legacy_tables` runs before `init_db`)
- **`POST /v1/generate` parameters relaxed.** `title`, `description`,
  `forge_version`, `tools`, and `auth_schemes` are now all optional with
  sensible defaults. The CLI continues to populate them; agents that go through
  the new high-level tools never have to set them.
- **`create_generation` MCP tool deprecated for agents.** The tool is still
  registered and remains in the MCP server (for CLI compatibility), but it has
  been removed from the `server_card.tools` listing and now carries a
  `[DEPRECATED for agents — use generate_from_openapi / generate_async instead]`
  marker in its docstring.

### 🐛 Fixed

- **Security: open-redirect vulnerability in generated OAuth servers.**
  The `oauth_authorize` template now validates `redirect_uri` against a
  whitelist driven by the new `OAUTH_ALLOWED_REDIRECT_HOSTS` environment
  variable. In dev mode (variable unset), only `localhost` and `127.0.0.1`
  are accepted; in production, the variable must be set explicitly or the
  request is rejected with HTTP 400. This addresses the
  `"No input validation for redirect_uri"` finding flagged by the built-in
  evaluator.
- **Silent zero-tools failure** in `/v1/generate` is now explicit
  (`success: false`, structured `errors`, actionable hint URL).
- **MCP tool descriptions for high-level tools** no longer use the
  `"""…""".format(...)` pattern that silently dropped `__doc__` to `None`.
  Agents now see the full descriptions, including limitations (e.g. GraphQL
  introspection being disabled in production, SPA-only websites needing the
  CLI) and the pointer to the local CLI for non-public sources. A regression
  test ensures these docstrings remain populated.

### 🛠 Internal

- New service module `api/services/zip_bundle.py` factors the zip-bundle
  construction shared between the dashboard download route and the new
  authenticated `/v1/generations/{id}/download` endpoint.
- New service module `api/services/discovery.py` wraps the existing
  `mcp_forge.discovery.*` modules for server-side parsing from the API layer.
- New service module `api/services/jobs.py` houses `create_job`,
  `run_job_sync`, and `reap_stale_jobs`.
- New ORM model `GenerationJob` (table `generation_jobs`) tracks asynchronous
  generation jobs with full multi-tenant isolation.
- New `ForgeConfig.extra_headers` field allows callers to forward custom HTTP
  headers (e.g. GraphQL authentication) into the discovery layer.
- Database migrations are now applied via `_rename_legacy_tables()` in the
  FastAPI lifespan, ahead of `init_db()`, to avoid an
  `Base.metadata.create_all()` ↔ rename collision.
- Test suite grew from 222 to **277 tests passing**, including three new
  migration tests (fresh install, first migration, idempotence) for any
  future DB rename.

### 📝 Known follow-ups (Phase 2)

These items are intentionally not addressed in this release and are tracked
in `TODO.md`:

- **Quota consumption on `/v1/generate` (sync) failure path.** The legacy sync
  endpoint still consumes a `Source` slot before generation runs; aligning it
  on the async pattern (consume on success) is planned.
- **Runtime OAuth tests on a fully generated server.** Current tests cover
  the rendered template and the handler logic in isolation. End-to-end tests
  running against a generated `server.py` subprocess will be added.

---

## [0.1.17] — 2026-05-06

Last public release of the `0.1.x` line. Incremental improvements to parsers and
template generation. Versions 0.1.18–0.1.27 were internal milestone bumps on the
`saas-platform` branch and were not separately published. See git history for details.

[0.2.3]: https://github.com/vulcai-io/MCP-FORGE-CORE/releases/tag/v0.2.3
[0.2.2]: https://github.com/vulcai-io/MCP-FORGE-CORE/releases/tag/v0.2.2
[0.2.1]: https://github.com/vulcai-io/MCP-FORGE-CORE/releases/tag/v0.2.1
[0.2.0]: https://github.com/vulcai-io/MCP-FORGE-CORE/releases/tag/v0.2.0
[0.1.17]: https://github.com/vulcai-io/MCP-FORGE-CORE/releases/tag/v0.1.17
