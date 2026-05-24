# kernel-project.md - Project-Specific Profile v7.0.0
# ABSTRACT LAYER - PROJECT: rules, conventions, and constraints specific to THIS project.
# This file is the SINGLE SOURCE OF TRUTH for project-specific rules.
#
# Separation principle:
#   - kernel-constitution.md -> Universal axioms valid for ANY project
#   - kernel-domains.md      -> Domain framework valid for ANY multi-domain project
#   - kernel-project.md      -> THIS file: project-type + project-instance rules
#
# Derived output: docs/03_PROJECT_RULES.md (generated, not manually edited)
# FOUNDATION: kernel-constitution.md §AXIOMS

<meta_section id="META-PROJECT" version="7.0.0" axiom_refs="phi6,A7,A10">
<purpose>Project-specific profile (PR-1...PR-6). Swappable by design: replacing this file and regenerating `docs/03_PROJECT_RULES.md` retargets the local agent ecosystem without touching universal files.</purpose>
<authority>The Root Admin (ResearchArchitect) edits this file only when onboarding or materially retargeting the project. All other agents consult `docs/03_PROJECT_RULES.md` generated from this file.</authority>
<rules>
- MUST NOT reference project-specific rules from kernel-constitution.md / kernel-domains.md / kernel-ops.md.
- MUST regenerate `docs/03_PROJECT_RULES.md` after any PR-{N} edit.
- PR-IDs are LOCAL to this file; do not clash with A-{N}, C-{N}, P-{N}, Q-{N}, or AU-{N}.
</rules>
<see_also>docs/03_PROJECT_RULES.md (generated), kernel-constitution.md §A, kernel-deploy.md §Stage 2</see_also>

--------------------------------------------------------
# § PROJECT IDENTITY

| Field | Value |
|-------|-------|
| Project type | Web data collection, normalization, and searchable local datastore |
| Product name | WindDB |
| Source focus | Publicly accessible listing information on `cityheaven.net` and related CityHeaven-hosted pages |
| Initial MVP target | `https://www.cityheaven.net/kanagawa/A1401/A140103/moecosu/` |
| Primary method | Polite, provenance-preserving crawling plus parser-tested extraction into purpose-fit databases |
| Core entities | Area, shop, listed woman/profile, profile attributes, introduction text, work schedule, diary post, crawl snapshot, source URL |
| Target output | Searchable local application/API that improves discovery across profile text, diary text, body-size fields, schedule availability, and update history |

--------------------------------------------------------
# § PR - Project-Specific Rules

These rules apply to all agents working within this project. They are not
universal; they derive from the data source, adult-directory context, and the
need to keep crawling lawful, polite, auditable, and update-safe.

## PR-1 - Crawl Boundary and Source Respect

WindDB collects only information that is publicly accessible without login,
payment, captcha bypass, private API abuse, or technical evasion.

| Concern | Rule |
|---------|------|
| Allowed source | Start with `https://www.cityheaven.net/kanagawa/A1401/A140103/moecosu/`; then expand only to same-pattern CityHeaven public listing/profile/diary/schedule pages after parser and policy checks pass |
| Forbidden source | Logged-in member pages, private messages, keep/history state, paid/private content, bypassed anti-bot responses, and pages disallowed by robots or site policy |
| Crawl discovery | For the MVP, use the `moecosu` shop page as the seed, discover the female list/profile links below it, then discover diary/schedule links from those profiles; broader sitemap/index discovery comes after this seed works |
| Crawl behavior | Identify the crawler, rate-limit per host, back off on 403/429/5xx, preserve robots and policy check evidence, and never retry aggressively |
| Parser safety | Treat HTML as untrusted input; sanitize persisted text and never execute scraped scripts |

Before any production crawl, agents MUST record the current robots.txt and
site-policy review in `docs/evidence/` or an equivalent audit artifact. As of
the 2026-05-24 review, public robots data for `cityheaven.net` records a
redirect to `www.cityheaven.net`, lists a sitemap, and disallows multiple
shop-list query variants; the membership terms found on a CityHeaven-hosted
page limit provided data to personal/private use and disallow other secondary
use. These observations are constraints to re-check, not permanent permission.
If the `moecosu` seed returns an anti-bot, age-gate, forbidden, or unstable
response, stop and record the blocker rather than adding bypass behavior.

## PR-2 - Purpose-Fit Database Selection

Database choice is an architectural decision per workload, not a fixed default.
Every storage change MUST document why the selected DB matches query patterns,
update cadence, and operational complexity.

| Workload | Preferred local/default option | Scale-out option | Notes |
|----------|--------------------------------|------------------|-------|
| Canonical entities and relationships | SQLite with migrations for local single-user use | PostgreSQL | Normalize shops, profiles, schedules, diaries, and crawl snapshots; use stable source IDs and URLs |
| Full-text search | SQLite FTS5 for small/local deployments | PostgreSQL `tsvector`/`pg_trgm`, Meilisearch, or OpenSearch | Japanese tokenization/search quality MUST be evaluated with representative diary/profile queries |
| Raw crawl snapshots | Filesystem/object store plus DB metadata | S3-compatible object store | Store HTML/text hashes and fetch metadata for diff/debug; avoid storing unnecessary binary media |
| Change history | Relational history tables | Append-only event table or warehouse | Track first_seen, last_seen, content_hash, source_url, parser_version, and extraction confidence |
| Analytics/export | SQL views/materialized views | Dedicated warehouse only if needed | Derived indexes must be reproducible from canonical data |

Do not add a new datastore because it is fashionable. Add it only when a named
query or operational requirement cannot be met cleanly by the current stack.

## PR-3 - Profile, Schedule, and Diary Extraction

Extraction starts from the `moecosu` female list page, follows each listed profile, and
stores structured facts with source provenance. Extractors MUST be fixture-tested
against saved HTML samples before production runs.

| Source area | Required extraction |
|-------------|---------------------|
| Female list page | Shop, area, listing URL, profile URL, display name, listing order, listing status, thumbnail URL/hash when needed |
| Profile page | Display name, shop, public profile attributes, body-size fields when present, introduction/self-introduction text, tags/categories, source URL |
| Schedule page/block | Work date, start/end time, status labels, shop, profile, source timestamp, normalization timezone |
| Diary list/detail | Diary title, body text, posted_at, updated_at when visible, related profile/shop, media URL/hash metadata when needed, source URL |

Extraction MUST NOT infer real-world identity, contact details, health status,
or sensitive facts that are not explicitly published in the source. Images and
videos are not a primary target; store URLs, hashes, dimensions, and provenance
only when necessary for deduplication or UI display.

The first acceptance fixture is the `moecosu` seed: one crawl run should produce
shop metadata, discovered profile URLs, profile records, schedule records when
present, diary records when present, and a search index update without duplicate
records on the second run. Other shops/attached sources are out of scope until
that fixture passes.

## PR-4 - Search Semantics and Index Quality

The product goal is better searchability across profile, diary, body-size, and
schedule information without losing provenance or freshness.

| Search need | Required behavior |
|-------------|-------------------|
| Text search | Search introduction text, diary title/body, tags, shop, and area; support Japanese text normalization and synonym handling where useful |
| Attribute search | Query height/body-size fields and other structured profile attributes with numeric/range filters when parse confidence is sufficient |
| Schedule search | Query date/time availability, current/upcoming shifts, and shop/area filters; expired schedules must not appear as current |
| Freshness search | Expose last_crawled_at, last_changed_at, and source URL for each result |
| Explainability | Search results must show which field matched and link back to the source/provenance record |

Any ranking change MUST be tested with a small evaluation set of realistic
queries. Full-text improvements should be measured by recall/precision notes or
side-by-side result examples, not by intuition alone.

## PR-5 - Update, Diff, and Idempotency Policy

Crawls are incremental and idempotent. Re-running the same crawl should update
changed records, preserve history, and avoid duplicating profiles or diaries.

| Case | Required handling |
|------|-------------------|
| New profile/diary/schedule | Insert canonical record and initial snapshot |
| Changed source content | Update current record, write history row, retain old content hash and parser version |
| Removed or missing content | Mark as not_seen/tombstoned only after a configured confirmation window; do not hard-delete by default |
| Parser change | Record parser_version and allow re-extraction from raw snapshots |
| Partial fetch failure | Preserve previous good data, record fetch error, and retry only under crawl budget/backoff rules |

Every mutable table needs stable uniqueness constraints such as source_url,
source_site_id, shop/profile IDs when available, and normalized timestamps for
diary/schedule records. Updates MUST be transaction-safe.

## PR-6 - Compliance, Privacy, and Operational Safety

This project handles adult-directory listing data and must be conservative by
default. Technical success is invalid if collection violates source policy,
privacy expectations, or operational safety.

| Risk | Project rule |
|------|--------------|
| Site policy drift | Re-check robots.txt, sitemap, relevant terms, and anti-bot responses before production crawling and after sustained failures |
| Secondary use and copyright | Do not republish scraped text/images wholesale; keep data for local search/indexing unless explicit permission and legal review exist |
| Personal data | Minimize stored fields, encrypt/limit access where appropriate, provide deletion/suppression workflow, and do not enrich with external identity data |
| Adult-content context | Keep age-gate assumptions explicit; never collect or surface data suggesting minors; stop and escalate on any underage/high-school indication |
| Load and reliability | Use crawl budgets, per-host concurrency limits, randomized polite delays, conditional requests when supported, and circuit breakers |
| Security | Secrets stay out of git; raw HTML is untrusted; admin/search endpoints must avoid leaking full raw snapshots unnecessarily |
| Auditability | Keep source URL, fetch time, parser version, hash, and robots/policy evidence for data that drives search results |

Similar systems should be treated as cautionary references: search quality,
deduplication, update detection, opt-out handling, and anti-bot respect matter
as much as extraction coverage. Do not optimize for maximum scrape volume before
the provenance, deletion, and freshness model is correct.

--------------------------------------------------------
# § PORTABILITY NOTES

To adapt this system for a different data-collection project:

1. Replace this file with project-appropriate PR rules.
2. Regenerate `docs/03_PROJECT_RULES.md` and agent prompts.
3. Re-evaluate robots, terms, privacy, datastore, and search-quality rules for the new source.
4. Keep universal files (`kernel-constitution.md`, `kernel-domains.md`, `kernel-ops.md`) unchanged unless the reusable kernel itself needs a general rule.
5. Confirm `grep -c "^## PR-" docs/03_PROJECT_RULES.md` equals the six PR rules defined here.

The PR-{N} numbering is local to this file. Universal rules use A-{N}, C-{N},
P-{N}, Q-{N}, and AU-{N}.
</meta_section>
