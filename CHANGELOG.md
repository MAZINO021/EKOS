# EKOS Changelog

Version 4.0 · build 8 · IndexedDB schema v5.
The journey from v1 (monolithic prototype) through the frozen v2.0 architecture.

---

## v1.0 (2026-06-24) — Foundation prototype

- Single self-contained `EKOS.html`, vanilla JS + IndexedDB, no install, no server.
- Three layers: Home search-first, Tree split-screen + Topic tabs (Overview / Documents / Images / Videos / Notes), full-screen hand-written canvas force-graph Thread Explorer.
- Seeded with the real Eco-Skills framework (~37 topics, 14 seed threads).
- Liquid-glass UI (66% glass + 22px blur), custom background uploader.
- Bilingual EN/हिं via `I18N` + `tr()`; document translation via Claude API or free MyMemory fallback.

---

## Phase 2 (2026-06-24) — Modular Refactor (no behavior change)

- Split monolithic `EKOS.html` into `src/index.html` + `src/styles/app.css` + 18 numbered `src/js/` modules concatenated by a **zero-dependency `build.mjs`** into one shippable HTML.
- Workflow: edit `src/`, run `cd ekos && node build.mjs`. Output is byte-identical to v1 baseline.
- Bug fixes: window-listener leak in resizer (audit #8); double render in `switchTab` (audit #9).

## Phase 3 — Performance

- Secondary in-memory indexes (`childrenByParent`, `notesByTopic`, `filesByTopic`, `threadsByNode`) maintained via observable Maps — zero call-site changes.
- Search corpus cached with `_searchDirty` flag; debounced (110ms); regexes precompiled per query.
- Removed redundant `renderTree()` from tab switches.
- Web Worker harness (`EKOSWorker.spawn`) for background ingestion.
- Verified at 5,000 topics: search 10ms cold / 3ms warm; no console errors.

## Phase 4 — Knowledge Core (v2.2)

- DB v1 → v2. Six new stores: `sources`, `knowledge`, `principles`, `proposals`, `conflicts`, `events`.
- Typed stable UUIDs (`nid(prefix)` → `kb_`, `prin_`, `src_`, `ev_`, …) using `crypto.randomUUID()`.
- Immutable references: blocks cite `{sourceId, locator}`; names resolved at render time.
- **Knowledge Block** as the primary unit of knowledge; **Principle** as a first-class cross-cutting entity.
- Files migrated from base64 to **Blob** storage in IndexedDB (auto migration v1→v2).
- Per-repository schema versions + `migrations.js`.
- Overview now renders knowledge blocks + citation chips + principle chips additively (existing structured content untouched).

## Phase 5 — Knowledge Intelligence Layer

- Provider adapter (`AI.extract`, `AI.semanticQuery`) for Anthropic Claude, OpenAI, Google Gemini, Ollama, Offline.
- Ingestion pipeline: Document → Source → Evidence → Knowledge Blocks → **Proposals** (KI-2: never silent writes to the Core).
- Per-topic **Review Center** with multi-dimensional confidence bars, reasoning logs, duplicate + contradiction warnings.
- Web Worker queue for offline extraction; cancelable progress modal.
- Settings UI for 5-provider selection, per-provider keys, model, temperature, chunk size, feature toggles.
- Translation now routes through the unified provider (was Anthropic-only).

## Phase 6a — Discovery & Navigation

- DB v2 → v3. Five new stores: `searches`, `collections`, `research`, `people`, `orgs`.
- **Query Builder** search: hybrid keyword+fuzzy over the cached index across every entity type, optional semantic NL→terms via the adapter (AI-optional, keyword fallback), facets (type, verified-only, after-year, AI-Semantic).
- **Timeline Explorer** — full-screen chronology of events + dated sources + (later) governance actions.
- **Knowledge Health Dashboard** on the homepage (coverage %, pending review, conflicts, missing sources, duplicates, broken refs).
- **Discover-more navigation rail** on every topic / principle page.
- **Saved Searches** (re-run live).

## Phase 6b — Research Studio

- DB v3 → v4 (+`questions` store; first-class **Research Question** entity).
- Reproducible **research sessions** (question + documents + collected evidence + notes + proposals + findings + saved queries + snapshots).
- **Evidence Boards**, **Citation Graph** (columnar Topic → Knowledge → Source provenance map).
- **Knowledge Gap Detection** (topics without evidence, open questions, blocks without sources, open conflicts).
- **Research Report generator** (Markdown, copy / download / print).
- Knowledge Analytics + Research Analytics + Governance stat tiles.
- Invariant: the Studio reads and assembles only; new knowledge still flows through proposals.

## Phase 7a — Relationship Studio

- Relationship taxonomy: 4 categories (`structural`, `scientific`, `educational`, `organizational`); upgraded legacy `{a,b,type,label}` threads via `normalizeThreads()`.
- Edge coloring by 4-level confidence (Verified / AI-High / AI-Med / AI-Low).
- Filters (category + confidence). **Relationship Inspector** panel. **Shortest path** mode (BFS). Node search.
- **Trust Score** per topic (sources + evidence + principles + verified + structured content).

## Phase 7b — Lineage & Dependency Intelligence

- `traceKnowledge(blockId)` — full provenance modal (Topic → Block → Evidence → Source → Document → Page → Original text).
- `knowledgeEvolution(blockId)` — git-like lifecycle timeline from versions + events.
- `computeDependents(topicId)`, `computeInfluence(topicId)`, `openLineage(topicId)`.
- **Delete guard** — `deleteTopic` shows full dependency-impact modal before destructive action.
- `sourceImpact(sourceId)` + cascading-update notice on source rename.
- Explorer Modes (Relationships / Scientific / Dependencies / Provenance). Strength + Last reviewed in the Inspector.

## Phase 8a — Governance & Integrity foundation

- DB v4 → v5. Four new stores: `snapshots`, `rules`, `certifications`, `audit`.
- `meta.audit.log` migrated to the `audit` store.
- 6 seeded **integrity rules** (rules-as-data; addable without code).
- `sanitizeHTML()` (strips script/on*/javascript:/data:text/html) applied to imported/ingested HTML (fixes audit #14).
- **Capability registry** `EKOS.capabilities` / `CAP`.
- **Permission model** `ROLES` + `can()` (constant only — no auth yet; seeds Phase 11).
- Governance Center shell + Dashboard + Queue + Audit Viewer + Rules tab.
- `stability` field added to Knowledge Block model (belief vs. how-unlikely-to-change).

## Phase 8b — Versioning & Resolution

- `recordVersion(targetType, obj, reason, changeType)` — unified version store wired into `saveBlock`, `saveSource`, `savePrinciple`.
- **Diff Viewer** with side-by-side comparison + Restore.
- **Conflict Center** (resolve: keep A / keep B / keep both / dismiss → audit).
- **Duplicate Center** with provenance-preserving merge (sources, evidence, principles, children all migrate).
- **Cascading-update** section in Monitor.
- **Dependency & Broken-link Monitor** + **Auto-Fix engine** (drop dangling refs / reassign orphan / clear bad file link) with **Undo** + audit.

## Phase 8c — Snapshots, Certification, Readiness

- **Knowledge Snapshots** — freeze / restore / publish / archive / duplicate / compare. Version-stamped (appVersion, build, dbVersion, schemaVersion, migrationVersion). Lineage (parentId / childIds / branch / status).
- **Certification** (KG-4 human-only): Draft → Reviewed → Verified → Scientific → Gold. Each certification recorded as audit + timeline event.
- **Publication Readiness** + **Research Readiness** — per-category breakdowns (knowledge / evidence / sources / governance / certification / overall · conflicts resolved / independent validation / evidence gaps / open questions).
- Timeline now surfaces governance from the audit store (single source of truth, no duplication).

---

## Architecture FROZEN at v2.0 (2026-06-27)

After Phase 8c, the EKOS foundation is declared complete and the architecture frozen. Future work builds *products on top of* the platform (Phase 9 Publishing, Phase 10 Eco-Skills Platform, Phase 11 Enterprise) — never new foundational layers.

---

## Phase 8.5 — Knowledge Population & Validation

- Imported 5 real documents through the governance pipeline (Eco-Skills Book, Manual, ESS, 3-Month Syllabus, Website).
- Stress test (synthetic): 337 topics / 3,044 knowledge blocks / 5,000 evidence / 154 sources / 100 principles / 10,014 relationships — 0 broken / 0 orphan / 0 cycle / 5ms cold search / 1ms warm / 0 console errors.
- 5/6 subsystems pass; the 6th (Evidence/Principles) is gated by AI provider availability — proven structurally identical.
- **Milestone:** `EKOS-Core-v1.0-alpha.json` (44 blocks, governed).

## KE pass — v1.1

- 6 research questions added (ph-resp, ph-light, mh-stress, ec-attn, ph-nutri, hp-char).
- 5 topic certifications (Physical Health, Mental Health = Verified; Self Awareness, Eco-Centralization, Holistic Personality = Reviewed).
- Snapshot frozen.
- **Milestone:** `EKOS-Core-v1.1.json`.

## KE pass — v1.2 (offline)

- 5 first-class **Principles** created from cross-block frequency (Ecological Principles, Oxygenated Blood Circulation, Five Natural Channels, Eco-Centralization & Energy Flow, Mental Health & Stress).
- 32 principle ↔ block links across 24 blocks.
- 5 sources received complete metadata (author = Prof. B. D. Tripathi, publisher = Ganga Research Centre BHU, language en/hi, tags). Unknown values left blank.
- 21-Day Program "fallback bucket" cleared — 8 blocks re-placed.
- 4 gap-derived research questions added.
- **Milestone:** `EKOS-Core-v1.2.json` (declared AI-Ready, overall score 89/100).

## Gemini integration + v2.0 (2026-06-28)

- Provider Gemini wired and verified.
- Gemini request now sends `responseMimeType: "application/json"` (structured output), `maxOutputTokens: 16384`, and `thinkingConfig: { thinkingBudget: 0 }` — thinking ON consumed the output budget and truncated JSON, causing silent offline fallback; OFF = reliable extraction.
- Transient-error classifier `isTransientErr(e)`: HTTP 429/500/502/503/504, network, timeout, aborted → retry. 400/401/403 (auth/bad request) → never retry.
- Retry schedule: **30s → 60s → 120s**. UI shows `Running → Retrying (1/3) → … → Completed | Retry Failed`.
- Failed documents preserved in an in-memory queue (no new store); modal shows **↻ Resume Failed Imports (N)** so the user resumes exactly where the pipeline stopped. Source / file / orphan version records cleaned on failure → no duplicates on resume.
- **Bug fix (critical):** `rvBulk('accept')` / `rvBulkConf('verified')` used to fall back to "all filtered / all pending" with an empty selection — a single stray click on "Accept" mass-accepted the entire queue (observed: 88 blocks in 67ms). Stack-traced via wrapped `acceptProposal` / `saveBlock`. Fix: bulk actions now operate only on an explicit `RV.sel` selection; bulk buttons are `disabled` when selection is empty.
- **Bug fix:** added 90s request timeout (`AbortController`) around the LLM `fetch` — a stalled provider call no longer hangs the pipeline forever; abort → transient → retry/resume.
- Real Gemini ingestion: Book + Manual → 60 proposals → 9 duplicates rejected → **51 evidence-rich blocks accepted**, 65 evidence quotes, 26 principles auto-created.
- **Milestone:** `EKOS-Core-v2.0-gemini.json`.

## UX Sprint 1–3 (verified, frozen)

- **Sprint 1:** Review Center bulk actions (select + accept/reject + Verified bulk), confidence/severity icon+label (✓ ◆ ◑ ○ — color-blind safe), tree keyboard nav (↑↓←→/Enter/f) + roving tabindex + persistent star + gold active bar, Health Dashboard cards clickable into Review/Governance, focus-visible rings, `prefers-reduced-motion`.
- **Sprint 2:** Search ↑↓/Home/End/Enter/Esc nav, result count header, facet counts + disable-empty, sticky topic tabs, scroll-position preserved per tab, Overview section jump-links, aria-labels on every icon button, 44px touch targets on the topbar.
- **Sprint 3:** Knowledge card Compact/Expanded toggle, Pin block, Copy block / Copy citation; topic actions menu (`<details>`); Quick-Add Knowledge primary button; Review Center progress bar, Next button, Accept-Verified-selected.

## Production-ready release (this build)

- Full subsystem verification: 0 console errors, 0 integrity defects.
- Single-folder release packaged.
