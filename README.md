# EKOS — Eco-Skills Knowledge Operating System

A local-first, single-file **Scientific Knowledge Operating System** for the Eco-Skills body of knowledge (eco-counselling by Prof. B. D. Tripathi, BHU). Provenance-first, AI-optional, fully offline-capable.

**Version:** 4.0 · build 8 · IndexedDB schema v5
**Architecture:** v2.0 (FROZEN)

---

## What EKOS is

A governed knowledge platform that turns documents into traceable, citable knowledge. Every claim in EKOS is traceable: **Topic → Knowledge Block → Principle(s) → Evidence → Source (+ locator)**. AI extraction is *optional*: with no API key, EKOS works fully offline (search, tree, notes, threads, snapshots). With an AI provider connected, ingestion produces evidence-rich proposals that you review before they enter the Knowledge Core.

EKOS is delivered as **one self-contained HTML file**. Double-click `EKOS.html` — that's it. No install, no server, no internet required.

---

## Features

### Knowledge Core
- Topics, Knowledge Blocks, Principles, Evidence, Sources — stable typed UUIDs (`kb_`, `prin_`, `src_`, …).
- Immutable references: blocks cite `{sourceId, locator}`; renaming a source updates every citation.
- Blob file storage in IndexedDB. Citation chips open the original document (PDFs jump to page).
- Bilingual UI (English / हिंदी), document translation, custom backgrounds.

### Knowledge Intelligence (AI optional, proposal-only)
- 5 providers: Anthropic Claude, OpenAI, Google Gemini, Ollama (local), Offline heuristic.
- Pipeline: Document → Source → Evidence → Knowledge Blocks → **Proposal** → Review → Knowledge Core.
- Nothing enters the Core without an explicit human Accept. Confidence levels, reasoning logs, duplicate + contradiction detection on every proposal.
- Transient-error retry (30s → 60s → 120s) for 429/503/timeout/network; "Resume Failed Imports" to continue exactly where the queue stopped.

### Discovery & Research
- Hybrid + semantic search with facets (type, verified, year). Result counts per facet. Keyboard navigation.
- Timeline Explorer — chronological view of events + governance actions.
- Research Studio — research questions, sessions, evidence boards, citation graph, report generation.
- Knowledge Health Dashboard — coverage, pending review, conflicts, broken refs.

### Relationship Intelligence
- Typed/categorized relationships (structural / scientific / educational / organizational).
- Relationship Inspector, shortest path, trust scores, lineage, dependencies, cascading-update impact.

### Governance & Integrity
- Audit log, Version History + Diff Viewer, Conflict Center, Duplicate Center, Auto-Fix engine (with undo).
- **Snapshots** — git-style freeze / restore / publish / archive / duplicate / compare. Version-stamped (appVersion, build, dbVersion).
- **Certification** — Draft → Reviewed → Verified → Scientific → Gold. Humans only.
- **Readiness breakdowns** — Publication readiness + Research readiness, per topic.

---

## System Requirements

- Any modern browser (Chrome, Edge, Firefox, Safari) — desktop preferred.
- No internet required for core functionality.
- AI features need an internet connection to the chosen provider (unless you use Ollama locally).
- Persistent storage uses IndexedDB (the browser may ask you to allow persistent storage).

---

## How to start

1. Locate `EKOS.html` in the `RELEASE/` folder.
2. Double-click it. It opens in your default browser.
3. The first run seeds the Eco-Skills knowledge tree (37 topics + 14 relationships) automatically.
4. Bookmark the file (or pin its tab) for easy access.

---

## How to import data

EKOS ships with two ready-made knowledge bundles:

- `EKOS-Core-v2.0-gemini.json` — **production milestone** (51 evidence-rich knowledge blocks extracted via Gemini, 26 principles, 65 evidence quotes, 3 sources, 14 relationships).
- `EKOS-ecoskills-population.json` — same as v2.0 (kept under a generic name for tooling).

To load a bundle into a fresh EKOS instance:

1. Open EKOS.
2. Click the **⚙ Settings** icon in the top bar.
3. Scroll to **Backup & Restore** → click **Import**.
4. Select one of the JSON bundles above.
5. Confirm the merge. EKOS will load all topics / knowledge / sources / principles / relationships.

To ingest a new document:

1. Click **✨ Add Knowledge** in the top bar (or use the in-app dropzone).
2. Drop a PDF, DOCX, PPTX, TXT, Markdown, or image (OCR).
3. EKOS extracts atomic knowledge blocks → writes **Proposals** (never to the Core).
4. Open **🗂 Review Center** → Accept / Reject / Edit each proposal.

---

## How to connect an AI provider

All keys are stored **locally only** in your browser's IndexedDB. They never leave your device except to call the provider you configured.

### Google Gemini (recommended)
1. Get an API key at [aistudio.google.com](https://aistudio.google.com/).
2. Open EKOS → **⚙ Settings**.
3. **Provider:** `Gemini`.
4. **Model:** `gemini-2.5-flash` (default fallback: `gemini-1.5-flash`).
5. Paste your API key into the **Gemini API key** field.
6. **Temperature:** `0.1` (recommended for extraction).
7. **Chunk size:** `10000` (max per single-chunk extraction).
8. Click **Save**.

Gemini ingestion produces evidence quotes + principles + ai-high confidence on most blocks. See KNOWN_LIMITATIONS for free-tier rate limits.

### Anthropic Claude
1. Get an API key at [console.anthropic.com](https://console.anthropic.com/).
2. **Provider:** `Claude` · **Model:** `claude-sonnet-4-6` (or `claude-opus-4-8` for highest quality).
3. Paste your API key into the **Claude API key** field. Save.

### OpenAI
1. Get an API key at [platform.openai.com](https://platform.openai.com/).
2. **Provider:** `OpenAI` · **Model:** `gpt-4o-mini` (or `gpt-4o`).
3. Paste key. Save.

### Ollama (local, fully offline)
1. Install [Ollama](https://ollama.ai) and pull a model: `ollama pull llama3.1`.
2. **Provider:** `Ollama` · **Model:** `llama3.1` (or any local model name).
3. **Ollama URL:** `http://localhost:11434` (default). Save.

### Offline mode (no AI)
- **Provider:** `Offline`. EKOS uses a built-in heuristic extractor. Lower quality (no evidence quotes, no principle extraction) — but 100% private and works without internet.

---

## Keyboard shortcuts

Available when focus is **not** on a text input or contenteditable field.

| Key | Action |
|---|---|
| `/` | Focus the top search bar |
| `Esc` | Close search / modal / overlay / threads / timeline |
| `↑ ↓` | Navigate search results / tree |
| `← →` | Collapse / expand tree node |
| `Home` / `End` | Jump to first / last search result |
| `Enter` | Open selected result / activate |
| `f` or `*` | Toggle favorite on focused tree node |
| **Review Center** | |
| `j` / `k` | Next / previous proposal |
| `a` | Accept proposal under cursor |
| `r` | Reject proposal under cursor |
| `x` | Toggle selection on proposal under cursor |

---

## Backup & Restore

### Export
**⚙ Settings → Export** → downloads `ekos-backup-<date>.json` containing every topic, knowledge block, source, principle, relationship, version, audit entry, snapshot — and embedded file blobs (base64).

### Import
**⚙ Settings → Import** → choose a previously exported `.json` bundle or one of the milestone bundles. EKOS merges into the current knowledge base (you'll be asked to confirm).

### Where data lives
All data is in your browser's IndexedDB under database `ekos-db` (schema v5). Clearing browser data will erase your knowledge base — back up regularly.

---

## Snapshots (governed releases)

Snapshots are version-stamped, restorable freezes of your Knowledge Core.

1. **⚖ Governance** → tab **Snapshots** → **📦 Freeze current state**.
2. Label your snapshot (e.g. `Eco-Skills Core v1.0`).
3. Each snapshot records the app version, build, schema version, plus full Core data + audit + certifications.
4. From the same tab you can: **Publish**, **Duplicate** (branch), **Compare** (to any other snapshot or to current), **Restore** (replaces current Core — audited), **Archive**.

Lineage (parentId / childIds) is recorded so you can branch and diff like git.

---

## Research workflow

1. **🔬 Research Studio** → **New Question** → state your research question.
2. Within the question's session: collect evidence (drag blocks/sources into the Evidence Board), keep field notes, save search queries.
3. **Reproduce Session** — re-runs every saved search to detect drift in the Core.
4. **Gap Detection** — surfaces topics with no evidence, open conflicts, blocks without sources.
5. **Citation Graph** — visual Topic → Knowledge → Source provenance map.
6. **Generate Report** — produces a Markdown report ready to copy / download / print.

The Studio reads and assembles the Core but **never writes to it** (any new knowledge still flows through the proposal pipeline).

---

## Governance workflow

1. **⚖ Governance Center** in the top bar.
2. **Dashboard** — Integrity Score, Trust, Coverage, Pending review counts.
3. **Queue** — proposals waiting for human review.
4. **Versions** — every change to every entity, with **Diff** + **Restore**.
5. **Conflicts** — contradictions detected by the engine; resolve (keep A / keep B / keep both / dismiss).
6. **Duplicates** — near-duplicate Topics / Blocks / Sources / Principles; provenance-preserving merge.
7. **Monitor** — broken references, orphan blocks, missing files, circular dependencies + **Auto-Fix** (with undo).
8. **Snapshots / Certify / Readiness / Audit / Rules** — see above.

Every governance action is recorded in the audit store and surfaces in the Timeline.

---

## Known limitations

See `KNOWN_LIMITATIONS.md`.

---

## Credits

Knowledge framework: **Prof. B. D. Tripathi**, UGC Emeritus Professor & Chairman, Mahamana Malaviya Ganga Research Centre, Banaras Hindu University.
Software: built phase-by-phase from a frozen v2.0 architecture (see `CHANGELOG.md`).

License: see `LICENSE.md`.
