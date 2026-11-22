# Fix: `meter` Keyword Misroutes Electricity Complaints to BWSSB

## Overview

**Goal:** Remove `meter` from `WATER_PATTERNS` so that voice complaints about "electricity meter", "meter issue", "meter reading" are correctly routed to BESCOM instead of BWSSB.

**Scope:** Single change in `IntentDetector` IIFE inside `index.html`. No UI, no routing logic, no mock API changes needed.

**Root cause:** The English regex in `WATER_PATTERNS[0]` includes `meter` as a bare keyword. Since `IntentDetector.detect()` runs WATER_PATTERNS before ELECTRICITY_PATTERNS, any transcript containing the word "meter" (regardless of context — electricity meter, power meter, etc.) matches water first and routes to BWSSB.

**Correct behavior:**
- "water meter issue", "meter leaking" → BWSSB (water meter is a BWSSB concern)
- "electricity meter issue", "meter not working", "meter reading problem" → BESCOM

The fix must preserve routing of "water meter" to BWSSB while letting "electricity meter" fall through to BESCOM. Since `meter` alone is ambiguous, the solution is to remove the bare `meter` keyword from WATER_PATTERNS. Users who say "water meter" will still match on `water`, `tap`, `supply`, etc., which are unambiguous water keywords in the same pattern.

---

## Sub-Tasks

### Sub-Task 1 — Remove `meter` from WATER_PATTERNS

**Status:** `[ ] pending`

**Intent:** Remove the bare `meter` token from `WATER_PATTERNS[0]`. The word is ambiguous and causes electricity meter complaints to be incorrectly routed to BWSSB. Users reporting water meter problems will still route correctly via other water keywords (`water`, `supply`, `pipe`, `tap`, `bwssb`).

**Expected Outcomes:**
- Saying "my electricity meter is not working" routes to BESCOM
- Saying "water meter leaking" still routes to BWSSB (matched by `water`)
- Saying "meter issue" alone with no other context routes to BESCOM (since `मीटर` in ELECTRICITY_PATTERNS Hindi pattern is the only remaining meter signal)
- No other patterns, routing logic, or UI are changed

**Todo List:**
1. Open `index.html` and locate `WATER_PATTERNS` at line ~972
2. In the first regex of `WATER_PATTERNS`, remove `|meter` from the alternation group
3. Verify the remaining pattern is syntactically valid
4. Manually test: voice/type "electricity meter not working" → should route to BESCOM; "water meter leaking" → should route to BWSSB

**Relevant Context:**
- File: `index.html`
- Symbol: `IntentDetector` IIFE, `WATER_PATTERNS[0]`, line 972
- Current value:
  ```
  /\b(water|pipe|pipeline|leak|leakage|sewage|drainage|drain|tap|supply|pressure|flood|overflow|contamina|bwssb|bore|tank|meter)\b/i
  ```
- Target value (remove `|meter`):
  ```
  /\b(water|pipe|pipeline|leak|leakage|sewage|drainage|drain|tap|supply|pressure|flood|overflow|contamina|bwssb|bore|tank)\b/i
  ```
- `मीटर` (Hindi for meter) already exists in `ELECTRICITY_PATTERNS[4]` at line 984 — no change needed there
- `meter` does NOT appear in any ELECTRICITY_PATTERNS English regex — it is only in WATER_PATTERNS. After removal, "meter" in English will not match either agency, so it falls to `'unknown'` unless paired with other keywords. This is acceptable; users who say just "meter" typically say more context.
