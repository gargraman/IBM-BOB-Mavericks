# Plan Mode Architecture Rules (Non-Obvious Only)

- **Single-file constraint is intentional (ADR-003).** Any V2 planning must account for migrating away from this — the architecture doc recommends React/Vue + FastAPI.
- **No backend = no persistence.** Complaint history dies on page reload. Any feature requiring history (re-open, analytics, notifications) requires a backend as a prerequisite.
- **Mock APIs have a 5% random failure rate** built in. Any retry logic or UX must be designed around this.
- **Intent detection is tie-broken by pattern hit count**, not confidence score. Mixed-intent utterances (e.g., "water and electricity both not working") will be routed to whichever has more regex matches — this is a known limitation.
- **STT is push-to-talk only** (`continuous: false`). VAD (Voice Activity Detection) is an open V2 item — planning for it requires replacing the entire recording model.
- **Geolocation is non-blocking.** The app works without location; `state.geo` can be `null`. Location is attached to the payload as optional metadata.
- **Real agency APIs do not exist publicly** (as of ADR writing). Any V2 integration plan must include a government API portal access step, not just an endpoint swap.
- **IBM Plex font is loaded from Google Fonts CDN.** Offline or intranet deployments will fall back to system fonts — plan accordingly for enterprise contexts.
