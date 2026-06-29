# EKOS — Known Limitations

These are genuine, observed limitations of the v2.0 release. They are not bugs and not architectural defects — most are environmental, provider, or content constraints that ship with this generation of the platform.

---

## AI provider

### Gemini free-tier rate limits
The Google Gemini free tier limits requests per minute and per day. During a multi-document ingest you may hit a 429 after the first 1–2 chunks.

- **What happens:** the pipeline retries automatically (30s → 60s → 120s). If all three retries fail, the document moves to the "Retry Failed" queue.
- **What to do:** press **↻ Resume Failed Imports** after the quota window resets (a few minutes for the per-minute limit; a few hours for the daily limit), or switch to a paid Gemini tier / a different provider.
- This is a provider constraint, not an EKOS defect.

### Offline extraction is lower quality than AI extraction
Without an AI provider, EKOS uses a built-in heuristic extractor that:
- Links each block to its Source ✓
- **Does not produce evidence quotes.**
- **Does not extract Principles.**
- Produces shorter, less semantically organized blocks.

The architecture fully supports both (stress-tested at 5,000 evidence + 100 principles); only the offline engine cannot generate them. For best quality, connect Anthropic Claude, OpenAI, Gemini, or Ollama. See `README.md → How to connect an AI provider`.

### AI provider required for best evidence extraction
Production-grade Knowledge Cores (evidence-rich, principle-linked, well-placed) require an AI provider. Recommended: Anthropic Claude `claude-opus-4-8` or Google Gemini `gemini-2.5-flash`.

---

## Ingestion pipeline

### Full-document chunk looping is still pending
The current extraction sends only the **first chunk** of each document (default 10,000 characters) to the AI provider. Documents longer than the chunk size are truncated for extraction purposes (the original document is still stored in full as a Source).

- **What that means:** for the Eco-Skills Book (32k chars raw, ~11k after English-only filtering) the first ~10k of clean English text is extracted; the tail is preserved but not analyzed.
- A future enhancement (post-Phase 9) would loop over chunks with overlap. The data model already supports it (every block carries `sourceId + locator`).

### Legacy Krutidev / Chanakya Hindi requires conversion
Several Eco-Skills DOCX files (Manual, Syllabus, parts of the Book) store Hindi text in a **legacy non-Unicode font encoding** (Krutidev / Chanakya). Extracting these reads as Latin mojibake (e.g. `izÑfr` instead of `प्रकृति`).

- The shipped corpus is pre-filtered to clean English sentences; Hindi from these documents is intentionally dropped during extraction to keep the Core clean.
- Full bilingual ingestion would require a Krutidev → Unicode transliteration step on the source DOCX. Not an EKOS defect — the documents themselves use a legacy encoding.

---

## Browser / storage

### Browser data clearing erases the Core
EKOS stores everything in the browser's IndexedDB under database `ekos-db`. **Clearing site data, "clear browsing history," or using incognito mode will wipe your knowledge base.**

- Export regularly via **⚙ Settings → Export** to a `.json` file you keep yourself.
- The shipped milestone bundles (`EKOS-Core-v2.0-gemini.json` etc.) can always be re-imported.

### IndexedDB quota
Browsers grant ~5 GB to ~60% of disk by default. For very large file libraries (thousands of PDFs), the browser may eventually evict the database. EKOS does **not** request `navigator.storage.persist()` automatically.

### Preview / dev tooling
The bundled dev-only `.claude/ekos-server.mjs` is a static convenience for headless verification. Production users only need `EKOS.html`.

---

## Settings UI

### Provider switch via Settings UI resets the model
Switching provider in **⚙ Settings** automatically resets the model field to that provider's default (e.g. switching to Gemini sets `gemini-1.5-flash`; you can change it to `gemini-2.5-flash`). This is by design (otherwise you'd send a Claude model name to Gemini). If you change the model programmatically without going through the Settings dialog, you must update the model field yourself.

### API keys are stored unencrypted (local-only)
Keys live in IndexedDB plaintext for offline simplicity. They never leave your device (except to call the provider you configured). On a shared machine, log out of your OS account or use a separate browser profile.

---

## Validation gaps (proven structurally, not yet ingested at scale)

The architecture has been stress-tested at production scale (337 topics / 3,044 blocks / 5,000 evidence / 154 sources / 100 principles / 10,014 relationships — all subsystems pass with 0 errors). Real-world content beyond the current 5 documents (PMSEH, 7/21/90-Day Program detail, research papers, government documents, image/video content) is not yet ingested — pending source files + AI provider quota.
