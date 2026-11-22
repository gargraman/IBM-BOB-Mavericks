# Ask Mode Context (Non-Obvious Only)

- The only source file is `index.html` — it is both the UI and the entire application logic.
- `docs/architecture.md` contains the ADRs and stack rationale. The mock API contract in that doc is **incomplete** — the real payload accepted by both mock APIs includes `problemType`, `duration`, `landmark`, and `geo` in addition to what the doc shows.
- The `Intent` file (no extension, project root) is the original hackathon brief — useful for understanding scope decisions.
- Web Speech API (`webkitSpeechRecognition`) is Chrome/Edge only. This is a known, intentional constraint (ADR-001). Do not suggest fixing it without noting the V2 migration path to Sarvam Saaras.
- Complaint history is session-only (no localStorage, no backend). This is deliberate per ADR-003.
- The IBM Plex font family is used throughout — this is an IBM brand constraint, not a stylistic choice.
