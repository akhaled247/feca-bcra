# FECA ↔ BCRA Reader — Task List

> Plan: [plan.md](./plan.md)  
> **Status:** Decisions locked — ready for implementation (Tasks 1–11 shell, then 12–N all subsection translations).

---

## Task 1: Repository scaffold, README, local serve

**Description:** Create the static-site layout (`docs/` or equivalent), Python tooling entrypoint, `.gitignore`, and README with GovInfo attribution, unofficial-site disclaimer, and `python -m http.server` instructions matching GitHub Pages paths.

**Acceptance criteria:**
- [ ] Directory layout documented in README (sources, build output, assets)
- [ ] Empty or placeholder index loads locally via `python -m http.server`
- [ ] README states site is unofficial and links both GovInfo detail pages

**Verification:**
- [ ] Manual: open index in browser from local server
- [ ] Build: N/A until Task 5

**Dependencies:** None

**Files likely touched:**
- `README.md`
- `.gitignore`
- `docs/index.html` (placeholder)
- `scripts/README.md` (optional one-liner)

**Estimated scope:** Small (1–2 files + docs)

---

## Task 2: GovInfo fetch, cache, and provenance

**Description:** Script to download PLAW-107publ155 and COMPS-985 source files into `data/raw/` with a manifest (`data/manifest.json`) recording URLs, package IDs, and fetch timestamp for display in the site footer.

**Acceptance criteria:**
- [ ] Raw files exist for BCRA and FECA compilation (HTML and/or USLM per plan)
- [ ] Manifest is machine-readable and includes GovInfo detail URLs
- [ ] Re-running fetch is idempotent (overwrite or version by date)

**Verification:**
- [ ] Manual: manifest URLs open to correct GovInfo pages
- [ ] Run: `python scripts/fetch_govinfo.py` (exact name TBD in implementation)

**Dependencies:** Task 1

**Files likely touched:**
- `scripts/fetch_govinfo.py`
- `data/raw/**`
- `data/manifest.json`

**Estimated scope:** Small

---

## Task 3: Parse BCRA (PLAW-107publ155) into section JSON

**Description:** Parse cached BCRA source into `data/parsed/bcra.json`: stable IDs (`sec-101`, etc.), hierarchy (title, section, subsection), verbatim text blocks, and display titles from the Table of Contents.

**Acceptance criteria:**
- [ ] All BCRA sections in the enacted TOC are represented
- [ ] Each section has `id`, `number`, `title`, `verbatim`, `path` (breadcrumb)
- [ ] Parser tested against at least §101 and one Title II section

**Verification:**
- [ ] Run: `python scripts/parse_bcra.py`
- [ ] Manual: spot-check §101 text against GovInfo HTML

**Dependencies:** Task 2

**Files likely touched:**
- `scripts/parse_bcra.py`
- `data/parsed/bcra.json`
- `tests/test_parse_bcra.py` (optional but recommended)

**Estimated scope:** Medium

---

## Task 4: Parse FECA (COMPS-985) into section JSON

**Description:** Parse USLM (or HTML) into `data/parsed/feca.json` with section numbers, titles, and verbatim text suitable for page generation.

**Acceptance criteria:**
- [ ] Major FECA divisions/sections parse with stable IDs (e.g. `sec-323`)
- [ ] Known BCRA-touchstone sections (e.g. 323, 316) retrievable by ID
- [ ] Parser documents known limitations in README or script docstring

**Verification:**
- [ ] Run: `python scripts/parse_feca.py`
- [ ] Manual: compare §323 (soft money) to GovInfo

**Dependencies:** Task 2

**Files likely touched:**
- `scripts/parse_feca.py`
- `data/parsed/feca.json`

**Estimated scope:** Medium

---

## Checkpoint: Foundation

- [ ] `bcra.json` and `feca.json` generate without errors
- [ ] Manifest provenance complete
- [ ] Human confirms COMPS “current law” caveat is acceptable for v1

---

## Task 5: Page generator + Shakescleare section template

**Description:** Build script that renders section pages: two-column layout (verbatim | plain English) **per subsection** (and nested paragraph level from parser), responsive collapse on small screens, USWDS-inspired styles, and shared header/footer with disclaimer.

**Acceptance criteria:**
- [ ] Each subsection block has paired columns with stable anchor IDs (`#sec-101-a`, etc.)
- [ ] Template accepts `plain_en` per subsection (empty → “Translation pending” during dev only)
- [ ] Generated HTML uses relative paths for GitHub Pages
- [ ] Accessibility: semantic headings, skip link, sufficient contrast

**Verification:**
- [ ] Run: `python scripts/build_site.py`
- [ ] Manual: resize viewport; keyboard-navigate header

**Dependencies:** Tasks 3, 4

**Files likely touched:**
- `scripts/build_site.py`
- `docs/assets/css/site.css`
- `docs/assets/js/site.js` (minimal)
- `templates/section.html` (or equivalent)

**Estimated scope:** Medium

---

## Task 6: BCRA browse (TOC + section pages)

**Description:** Generate BCRA index with Title groupings linking to all section pages; highlight current page in nav.

**Acceptance criteria:**
- [ ] `/bcra/` or equivalent lists all sections
- [ ] Each BCRA section has a dedicated static page
- [ ] Breadcrumb: Home → BCRA → Title → Section

**Verification:**
- [ ] Manual: click through every Title I link
- [ ] Build succeeds after full generate

**Dependencies:** Task 5

**Files likely touched:**
- `templates/bcra-toc.html`
- `docs/bcra/**`

**Estimated scope:** Medium

---

## Task 7: FECA browse (TOC + section pages)

**Description:** Generate **full** COMPS-985 browse UI (entire compilation); section pages use the same subsection Shakescleare template. Nav treats FECA as the primary corpus (home links here first).

**Acceptance criteria:**
- [ ] Full FECA TOC navigable to every section page
- [ ] Section pages show compilation source note (COMPS-985, as-amended-through date from manifest/metadata)
- [ ] BCRA-touched sections show inbound “Amended by BCRA §…” when cross-ref data exists

**Verification:**
- [ ] Manual: open §323 from TOC
- [ ] Build succeeds

**Dependencies:** Task 5

**Files likely touched:**
- `docs/feca/**`
- `templates/feca-toc.html`

**Estimated scope:** Medium

---

## Task 8: BCRA → FECA cross-reference extraction

**Description:** Heuristic parser on BCRA section text for FECA references (`section 323`, `Title III`, USC citations); emit `data/parsed/bcra_crossref.json` and render “Affects FECA” links on BCRA pages.

**Acceptance criteria:**
- [ ] §101 links to FECA §323 (or equivalent target page)
- [ ] Unresolved references listed without breaking build
- [ ] FECA pages can show inbound “Amended by BCRA §…” where data exists

**Verification:**
- [ ] Manual: verify links on 3 sample sections across Titles I and II
- [ ] Run: build + optional unit test on reference regex/parser

**Dependencies:** Tasks 6, 7

**Files likely touched:**
- `scripts/extract_crossref.py`
- `data/parsed/bcra_crossref.json`
- `templates/section.html`

**Estimated scope:** Medium

---

## Checkpoint: Core

- [ ] End-to-end: Home → BCRA §201 → linked FECA section
- [ ] All pages show disclaimer + GovInfo sources
- [ ] Build is reproducible from clean clone + fetch + build

---

## Task 9: Client-side search

**Description:** Build search index JSON at generate time; header search UI with results grouped by document; jump to section anchors.

**Acceptance criteria:**
- [ ] Query “soft money” returns BCRA §101 and FECA §323
- [ ] Search works on GitHub Pages (no server)
- [ ] Index size reasonable for Pages (consider chunking if needed)

**Verification:**
- [ ] Manual: test 5 queries (soft money, electioneering, 316, disclosure, minor)
- [ ] Build includes `docs/assets/search-index.json` (or split shards)

**Dependencies:** Tasks 6, 7

**Files likely touched:**
- `scripts/build_search_index.py`
- `docs/assets/js/search.js`
- `docs/assets/search-index.json`

**Estimated scope:** Medium

---

## Task 10: Hub page and reading guide

**Description:** Home page explaining **FECA as baseline** (current compiled law), **BCRA as the 2002 amendatory layer**, Shakescleare subsection layout, unofficial disclaimer, and reading order: browse FECA → follow BCRA badges / BCRA TOC for “what changed.”

**Acceptance criteria:**
- [ ] Clear statement: not a government website; not legal advice
- [ ] Explains COMPS is current FECA; BCRA shows what was added/changed in 2002; later laws may have amended again
- [ ] Primary CTA to FECA TOC; secondary to BCRA TOC; search prominent

**Verification:**
- [ ] Manual: non-expert read-through for clarity
- [ ] All acceptance criteria visible above the fold or one scroll

**Dependencies:** Tasks 6, 7, 9

**Files likely touched:**
- `docs/index.html`
- `templates/home.html`

**Estimated scope:** Small

---

## Task 11: GitHub Pages deployment

**Description:** Configure repo for Pages from `docs/` on `main`; add `.nojekyll` if needed; verify asset paths.

**Acceptance criteria:**
- [ ] README documents enablement steps
- [ ] Site loads on `*.github.io` with working CSS, search, and internal links

**Verification:**
- [ ] Manual: deploy and click-test 10 links
- [ ] Optional: `htmlproofer` or link checker script

**Dependencies:** Tasks 9, 10

**Files likely touched:**
- `docs/.nojekyll`
- `README.md`
- `.github/workflows/pages.yml` (optional CI build)

**Estimated scope:** Small

---

## Task 12: Plain-English merge tooling + coverage validator

**Description:** Merge `data/plain_en/**/*.yaml` into parsed JSON at build time; CLI reports missing subsection IDs for BCRA and FECA; optional `--strict` fails build when any `plain_en` is missing (used for launch).

**Acceptance criteria:**
- [ ] Every parser subsection ID can have a matching `plain_en` key
- [ ] `python scripts/validate_plain_en.py` prints counts and missing IDs
- [ ] Build documents how to add translations incrementally

**Verification:**
- [ ] Run validator on empty plain_en → lists all IDs
- [ ] Run merge on sample YAML → subsection appears in generated HTML

**Dependencies:** Tasks 3, 4, 5

**Files likely touched:**
- `scripts/merge_plain_en.py`
- `scripts/validate_plain_en.py`
- `data/plain_en/README.md`

**Estimated scope:** Small

---

## Checkpoint: Site shell

- [ ] Tasks 1–11 complete; validator runs
- [ ] Pages may show “Translation pending” until Phase 4 done
- [ ] Human OK to begin bulk plain-English work

---

## Task 13: Plain-English — BCRA Title I

**Description:** Author subsection `plain_en` for §§101–103; store in `data/plain_en/bcra/title_i.yaml` (paths TBD).

**Acceptance criteria:**
- [ ] Validator reports **zero** missing IDs for Title I subsections
- [ ] Translations describe amendatory effect and name FECA targets where applicable

**Verification:**
- [ ] Manual: §101 subsection sample vs verbatim
- [ ] `validate_plain_en.py --corpus bcra --title I` passes

**Dependencies:** Task 12

**Files likely touched:**
- `data/plain_en/bcra/title_i.yaml`

**Estimated scope:** Large

---

## Task 14: Plain-English — BCRA Title II

**Description:** Translations for §§201–214 (electioneering communications, coordinated/independent expenditures).

**Acceptance criteria:**
- [ ] All Title II sections have `plain_en` content
- [ ] Cross-links align with Task 8 targets

**Verification:**
- [ ] Manual: spot-check §203 corporate/labor ban explanation

**Dependencies:** Task 12

**Files likely touched:**
- `data/plain_en/bcra/title_ii.yaml`

**Estimated scope:** Large

---

## Task 15: Plain-English — BCRA Title III

**Description:** Translations for miscellaneous amendments §§301–319.

**Acceptance criteria:**
- [ ] All Title III sections translated

**Verification:**
- [ ] Manual: sample §311 (stand-by-your-ad clarity)

**Dependencies:** Task 12 (may run parallel with 13–14)

**Files likely touched:**
- `data/plain_en/bcra/title_iii.yaml`

**Estimated scope:** Large

---

## Task 16: Plain-English — BCRA Titles IV–V

**Description:** Translations for severability/effective date/judicial review and additional disclosure provisions.

**Acceptance criteria:**
- [ ] §§401–403, 501–504 translated

**Verification:**
- [ ] Manual: §402 effective dates readable by non-lawyer

**Dependencies:** Task 12

**Files likely touched:**
- `data/plain_en/bcra/title_iv_v.yaml`

**Estimated scope:** Medium

---

## Task 17: Plain-English — FECA (division 1 of N)

**Description:** After Task 4, split **all** COMPS-985 subsections into N parallel content tasks (one per FECA title / major division — exact split recorded in `tasks/plan.md` when parser output is known). Task 17 is the first division.

**Acceptance criteria:**
- [ ] Validator reports zero missing subsection IDs for this division
- [ ] Plain English explains operative rules (not amendatory meta unless the block is purely definitional)

**Verification:**
- [ ] `validate_plain_en.py --corpus feca --division <id>` passes
- [ ] Manual spot-check 3 sections including one BCRA-badged section

**Dependencies:** Tasks 7, 12

**Files likely touched:**
- `data/plain_en/feca/<division_id>.yaml`

**Estimated scope:** Large (repeat per division)

---

## Tasks 18–N: Plain-English — FECA (remaining divisions)

**Description:** Same acceptance criteria as Task 17 for each remaining top-level FECA division until the full compilation is covered.

**Acceptance criteria:**
- [ ] All FECA subsection IDs have `plain_en`
- [ ] Launch validator `--strict` passes for entire site

**Verification:**
- [ ] `validate_plain_en.py --strict` exits 0
- [ ] Search index includes plain_en text for FECA and BCRA

**Dependencies:** Task 17 pattern; Task 4 defines N

**Estimated scope:** Large (site-wide content effort)

---

## Checkpoint: Launch complete

- [ ] All BCRA and all FECA subsections translated
- [ ] `build_site.py` + `--strict` in CI or documented release step
- [ ] Hub page and disclaimers final read-through
- [ ] GitHub Pages live
