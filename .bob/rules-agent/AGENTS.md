# Agent Coding Rules (Non-Obvious Only)

- **Single file, no modules.** All code lives in `index.html`. IIFEs share state through globals (`MockBWSSB`, `MockBESCOM`, `IntentDetector`, `state`). Do not introduce ES modules or import/export — the file runs without a server.
- **DOM injection requires `escapeHtml()`** for any user-controlled string. Failure to do so is an XSS vector in the complaint card HTML.
- **Mock API payload is wider than the architecture doc.** Pass `{ description, language, problemType, duration, landmark, geo }` — the doc only mentions `description, language` but the actual IIFE destructures all six fields.
- **Mic ↔ form mutual exclusion is state-managed.** `micBtn.disabled` is toggled by `showDetailForm()` / `hideDetailForm()`. The `finally` block in `submitComplaint()` only re-enables the mic if `.detail-form-wrap` is not visible — preserve this guard.
- **CSS state classes, not JS style.** UI states (recording, processing, success, error) are applied by toggling CSS class names on elements, not `element.style`. Follow this pattern.
- **No `let`/`const` in IIFE internals for mock APIs** — they use `var`-equivalent function scope. Main app controller uses `const`/`let` throughout.
- **Complaint cards are prepended** (`panel.insertBefore(card, panel.firstChild)`) — newest first. Do not use `appendChild`.
- **Tab counts read the DOM, not state.** `updateTabCounts()` queries `.complaint-card` elements in the panel; BWSSB/BESCOM counts come from `MockBWSSB.getAll().length`. Do not maintain a separate counter variable.
