# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Project

**Mavericks** — single-file browser app (`index.html`). No build step, no package manager, no bundler. Open the file directly in Chrome/Edge; no server required.

## Running / Testing

- **Run:** Open `index.html` directly in Chrome or Edge (no `npm`, no server).
- **Voice features require Chrome or Edge** — `webkitSpeechRecognition` is not available in Firefox or Safari.
- **No automated test suite.** Manual testing only.

## Architecture (all in `index.html`)

Four `<script>` blocks in order:
1. `MockBWSSB` IIFE — in-memory water complaint store
2. `MockBESCOM` IIFE — in-memory electricity complaint store
3. `IntentDetector` IIFE — keyword regex routing (EN/HI/KN)
4. Main app controller — `state` object + DOM wiring

**Critical constraint:** All complaint history is DOM-only (page session). There is no localStorage, no backend, no persistence across page reloads.

## Non-Obvious Patterns

### Mock API interface
Both mock APIs accept an expanded `payload` object (not just `description, language` as the architecture doc states). The actual signature is:
```js
raiseComplaint({ description, language, problemType, duration, landmark, geo })
```
`geo` is `{ lat, lng, accuracy, label }` or `null`.

### Intent detection tie-breaking
When a transcript matches both WATER and ELECTRICITY patterns, `IntentDetector.detect()` counts hits across all patterns and picks the higher score — not first-match.

### Form ↔ mic mutual exclusion
`showDetailForm()` disables `micBtn`; `hideDetailForm()` re-enables it. The mic is also re-enabled in the `finally` block of `submitComplaint()` only if the detail form is not visible — do not break this logic.

### escapeHtml is mandatory for DOM injection
All user-supplied text (transcript, landmark, referenceId) inserted via `.innerHTML` must pass through `escapeHtml()`. The existing code does this — maintain the pattern.

## Design Tokens (CSS custom properties in `:root`)

| Token | Usage |
|---|---|
| `--accent-water` | BWSSB / water UI elements |
| `--accent-electric` | BESCOM / electricity UI elements |
| `--status-error/success/warning/info` | Status states |
| `--font-sans/serif/mono` | IBM Plex font stack |

Spacing uses `--space-1` through `--space-12` (4px increments). Do not use raw `px` values for spacing.

## Languages / Locale

STT locale codes: `en-IN`, `hi-IN`, `kn-IN`. These are passed directly to `webkitSpeechRecognition.lang`. Do not use generic codes like `en`, `hi`, `kn`.

Date/time formatting uses `en-IN` locale throughout: `toLocaleString('en-IN', { dateStyle: 'short', timeStyle: 'short' })`.

## V2 Integration Points

Real agency endpoints (not yet live):
- BWSSB: `POST https://api.bwssb.gov.in/complaints`
- BESCOM: `POST https://api.bescom.karnataka.gov.in/complaints`

V2 upgrade path: swap `MockBWSSB`/`MockBESCOM` IIFEs for fetch-based modules; `IntentDetector` can be replaced with IBM watsonx Granite for ambiguous utterances.
