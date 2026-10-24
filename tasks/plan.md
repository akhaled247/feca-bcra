# Implementation Plan: FECA ↔ BCRA Reader

## Overview

Build a **static** site (GitHub Pages + local `python -m http.server`) that presents **verbatim** statutory text from two GovInfo packages:

| Package | Document | Role on this site |
|---------|----------|-------------------|
| [PLAW-107publ155](https://www.govinfo.gov/app/details/PLAW-107publ155) | Bipartisan Campaign Reform Act of 2002 (P.L. 107-155) | **Primary “what changed” source** — each section is mostly amendatory language (`insert`, `strike`, `redesignate`, etc.) |
| [COMPS-985](https://www.govinfo.gov/app/details/COMPS-985) | Federal Election Campaign Act of 1971 (compilation) | **Current FECA text** as maintained by the legislative counsel — shows where BCRA’s changes live in the statute today |

Each navigable **section** uses a **Shakescleare-style** layout: original statutory language on one side, plain-English “translation” on the other (aligned to the same subsection boundaries), inspired by [LitCharts Shakescleare](https://www.litcharts.com/shakescleare/shakespeare-translations/hamlet/act-1-scene-1).

The UI should feel **government-adjacent** (clear hierarchy, accessible typography, sober palette) while remaining obviously **unofficial**: persistent disclaimer, no seals masquerading as federal sites, and explicit sourcing to GovInfo URLs and retrieval dates.

## Product decisions (locked)

| Topic | Decision |
|-------|----------|
| **Plain-English scope** | **All** BCRA sections and **all** FECA compilation sections — every subsection pair filled before calling the site “launch complete” |
| **FECA browse** | **Full** COMPS-985 hierarchy (entire compilation), not BCRA-touched-only |
| **Shakescleare unit** | **Subsection** (and nested `(1)`, `(2)`, …) — not whole-section blobs |
| **Historical FECA** | **No** pre-BCRA snapshot. **FECA (COMPS) is the baseline** for reading current law; **BCRA** is the amendatory “what it added/changed” layer with cross-links |

## Architecture Decisions

### Static site + build step (recommended)

- **Runtime:** pure HTML/CSS/JS — no backend on GitHub Pages.
- **Authoring:** Python scripts in `scripts/` fetch GovInfo sources, normalize structure to JSON, and **generate** pages under `docs/` (GitHub Pages root) or site root with `.nojekyll`.
- **Rationale:** BCRA (~124 KB text) and FECA compilation (much larger) are unwieldy to hand-edit; regeneration keeps verbatim text aligned with GovInfo and makes search indexing reproducible.

### Source formats from GovInfo

| Document | Preferred source | Fallback |
|----------|------------------|----------|
| P.L. 107-155 | [HTML package](https://www.govinfo.gov/content/pkg/PLAW-107publ155/html/PLAW-107publ155.htm) or [USLM XML](https://www.govinfo.gov/content/pkg/PLAW-107publ155/uslm/PLAW-107publ155.xml) (beta) | Plain text ZIP from details page |
| COMPS-985 | [USLM XML](https://www.govinfo.gov/content/pkg/COMPS-985/uslm/COMPS-985.xml) | [HTML](https://www.govinfo.gov/content/pkg/COMPS-985/html/COMPS-985.htm) or PDF |

USLM gives the cleanest hierarchy (titles, sections, subsections); HTML is acceptable for an MVP parser if USLM parsing slips.

### Shakescleare content model

- **Unit of pairing (required):** one row per **subsection** with stable IDs — e.g. `sec-101-a`, `sec-101-a-1`, mirroring `(a)`, `(b)`, `(1)`, `(2)` in the source. Section headers may have a short overview `plain_en_summary` only when useful.
- **Verbatim column:** extracted from GovInfo output with minimal cleanup (preserve wording; normalize only whitespace/entities for display).
- **Plain-English column:** stored per subsection in `data/plain_en/` (YAML or JSON), merged at build time. **Not** generated at runtime on GitHub Pages.
- **Workflow:** build pipeline may ship early with “translation pending” during development; **launch** requires 100% subsection coverage for both corpora. Quality bar: explain *legal effect*; on BCRA pages, tie language to *FECA location* where the section amends FECA.

### “What BCRA added to FECA” — information design

**COMPS-985 is current compiled FECA** (as amended through P.L. 118-26 per GovInfo), including changes after BCRA. There is **no** pre-2002 FECA snapshot on this site.

Site strategy:

1. **FECA-first path (default):** Home and nav emphasize **FECA (baseline)** — full COMPS browse with subsection Shakescleare pairs for the statute as it exists today.
2. **BCRA path:** Full P.L. 107-155 browse — amendatory verbatim text + subsection plain English + **cross-links** to FECA sections cited in each block (e.g. “section 323”, “Title III”).
3. **BCRA badges on FECA:** Sections created or materially touched by BCRA show an inbound link (“Amended by BCRA §…”) from parsed cross-refs.
4. **Hub page:** FECA = how the law reads now; BCRA = what Congress passed in 2002 to change FECA; later statutes may have amended the same provisions again.

### UI / visual language (official-adjacent, not impersonation)

- Base on **[U.S. Web Design System (USWDS)](https://designsystem.digital.gov/)** or a **minimal subset** (grid, typography, header/footer, focus states) to match familiar federal readability without copying agency branding.
- **Required elements:** site title clearly unofficial (e.g. “FECA & BCRA — Private Study Site”); footer disclaimer; prominent **Source:** links to both GovInfo detail pages; “last fetched” date from build metadata.
- **Avoid:** `.gov` domain implications, HHS/DOJ-style logos, “official” badges, or language implying FEC/government endorsement.

### Search

- **Client-side** full-text search (e.g. FlexSearch or Lunr) over a build-generated index JSON.
- Index fields: `document` (bcra|feca), `section_id`, `title`, `verbatim`, `plain_en`, `bcra_touched` flag.
- Global search in header; optional scoped search on BCRA vs FECA browse pages.

### GitHub Pages

- Publish from `/docs` on `main` **or** root with `docs/` output — configure in repo settings.
- Add `.nojekyll` if needed; use relative asset paths for local server parity.

### Licensing and attribution

- GovInfo / public domain U.S. government works — still attribute source URLs in UI and `README`.
- USWDS: follow its license/attribution if bundling assets.
- Plain-English translations: your original content; note they are **not legal advice**.

## Task List

### Phase 1: Foundation
- [ ] Task 1: Repository scaffold, README, local serve instructions
- [ ] Task 2: GovInfo fetch + cache + provenance metadata
- [ ] Task 3: Parse BCRA into section JSON
- [ ] Task 4: Parse FECA (COMPS-985) into section JSON

**Checkpoint: Foundation** — JSON validates; section counts sane; provenance recorded.

### Phase 2: Core reading experience
- [ ] Task 5: Static page generator + Shakescleare section template
- [ ] Task 6: BCRA browse (TOC, title grouping, section pages)
- [ ] Task 7: FECA browse (TOC + section pages)
- [ ] Task 8: BCRA → FECA cross-reference extraction and link UI

**Checkpoint: Core** — Navigate BCRA and FECA end-to-end locally; cross-links work for sampled sections (e.g. 101, 201).

### Phase 3: Discovery and polish
- [ ] Task 9: Client-side search index + UI
- [ ] Task 10: Hub page + disclaimers + “how to read this site” (FECA baseline narrative)
- [ ] Task 11: GitHub Pages deployment config
- [ ] Task 12: Plain-English merge tooling + subsection coverage validator

**Checkpoint: Site shell** — Search works; Pages deploys; build reports missing `plain_en` counts (may be >0 until Phase 4 done).

### Phase 4: Plain-English (required for launch — parallelizable)
- [ ] Task 13: BCRA Title I (§§101–103)
- [ ] Task 14: BCRA Title II (§§201–214)
- [ ] Task 15: BCRA Title III (§§301–319)
- [ ] Task 16: BCRA Titles IV–V (§§401–403, 501–504)
- [ ] Task 17–N: FECA — **all** COMPS sections, split by top-level division after Task 4 (one task per FECA title/part; count fixed when parser lands)

**Checkpoint: Launch complete** — Zero missing subsection translations for BCRA and FECA; validator passes; hub + disclaimers reviewed.

## Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| COMPS-985 is **current** FECA, not pre-BCRA | Med | Hub copy: FECA = baseline today; BCRA = 2002 changes; note later P.L.s |
| USLM schema / parsing complexity | Med | USLM for both corpora; subsection IDs from parser are source of truth |
| Volume of plain-English work (**full FECA + BCRA, per subsection**) | **Very high** | Subsection validator; split FECA tasks by title; parallel sessions; optional draft pass then accuracy review |
| Site looks like a federal impersonation | High | USWDS without agency identity; explicit unofficial disclaimer; no seals |
| GitHub Pages path / asset breakage | Med | Test `python -m http.server` and Pages with same `base` path strategy |
| Later amendments obscure “BCRA-only” meaning | Med | Cross-ref page cites BCRA section; optional footnote when FECA text diverges from 2002 intent |

## Definition of Done (tailored for this static project)

- Acceptance criteria for the task met
- `python -m http.server` serves the built site; links and search work without console errors
- Regenerating from GovInfo reproduces the same structure (documented command in README)
- Disclaimers and GovInfo source links visible on every page template
- No secrets or API keys in repo
- Every subsection has `plain_en` before **launch complete** (Task 12 validator)
- Human review before public launch

## Parallelization Opportunities

- **After Task 4:** BCRA page gen (Task 6) vs FECA page gen (Task 7) in parallel
- **After Task 5:** Search index (Task 9) vs cross-ref parser (Task 8) in parallel
- **Phase 4:** BCRA Titles I–V and FECA divisions in parallel once Task 12 merge/validator exists
