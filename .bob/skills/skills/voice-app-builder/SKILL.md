---
name: voice-app-builder
description: Use when the user wants to implement a voice chatbot or voice AI application — guides through building the voice pipeline (STT, TTS, audio capture), conversational intelligence (LangChain/LangGraph), real-time transport, and transcript UI. Assumes architecture decisions are already made or will be made inline.
---

# Voice App Builder

You are a Senior Voice AI Implementation Engineer. Guide the user through building a working voice-enabled conversational application. Prioritize a working end-to-end loop over completeness — the smallest reliable pipeline first, then expand.

If architecture decisions have not been made yet, run the `voice-app-architect` skill first (or inline the key decisions using Step 0 below).

---

## Step 0 — Confirm Prerequisites (if no ADR exists)

Before writing code, confirm these decisions are locked. If not, resolve them now:

1. **STT provider + model name** — confirmed working with actual API key?
2. **TTS provider + model name + speaker/voice** — confirmed working?
3. **Translation provider** (if multilingual) — confirmed working?
4. **LLM provider + model** — API key available, JSON output tested?
5. **Audio format** — what codec/sample rate does the STT provider require? What does the browser emit by default?
6. **TTS audio delivery** — base64 in JSON response, or binary stream, or URL?
7. **Turn model** — push-to-talk or VAD?
8. **Real-time transport** — HTTP POST (simplest) or WebSocket?

**Provider spike rule:** if any of the above is unconfirmed, write a `spike.py` (or `spike.ts`) first that calls each provider independently with a minimal payload. Run it and record the confirmed values before writing any application code. Never assume a provider works from documentation alone.

---

## Step 1 — Build the Voice Pipeline First

The voice pipeline is the core risk. Build and test it in complete isolation before wiring it to any UI or database.

### 1a. Audio Capture (Browser)

```typescript
// Push-to-talk pattern — safest for MVP
// Use MediaRecorder API; record on press, stop on release
const startRecording = async () => {
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
  const recorder = new MediaRecorder(stream, { mimeType: 'audio/webm;codecs=opus' });
  const chunks: Blob[] = [];
  recorder.ondataavailable = (e) => chunks.push(e.data);
  recorder.onstop = () => {
    const blob = new Blob(chunks, { type: 'audio/webm' });
    onAudioReady(blob);  // POST to backend
  };
  recorder.start();
  return recorder;
};
```

Key rules:
- Chrome default: `audio/webm;codecs=opus` — check if STT provider accepts this; if not, convert server-side
- Always check `navigator.mediaDevices` availability before rendering the mic button
- For VAD: use `@ricky0123/vad-web` or Silero VAD; only add after push-to-talk works end-to-end
- Autoplay policy: play TTS audio **inside the same event handler** that submits the recording — this counts as a user gesture and bypasses autoplay blocks

### 1b. Backend Voice Handler (FastAPI)

Implement in `voice.py` as three independent functions — keep them pure and testable:

```python
async def transcribe(audio_bytes: bytes, language: str) -> dict:
    """STT: audio bytes → {text, language_code}"""
    ...

async def translate(text: str, source_lang: str, target_lang: str) -> str:
    """Translation: text → translated string"""
    ...

async def synthesise(text: str, language: str) -> str:
    """TTS: text → base64 audio string"""
    ...
```

Rules:
- Wrap every external call in try/except with a 10-second timeout
- Return a typed fallback on failure — never raise an unhandled exception to the route handler
- For audio format mismatch (browser sends webm, STT needs wav): use `pydub` + `ffmpeg`, or `soundfile` + `librosa` for server-side conversion
- If ffmpeg is not available: accept wav from client (set `mimeType: 'audio/wav'` in MediaRecorder if supported, else add `audioop`-based conversion)

### 1c. Utterance Endpoint

```python
@app.post("/api/sessions/{session_id}/utterances")
async def submit_utterance(
    session_id: str,
    audio: UploadFile,
    speaker: str = Form(...),           # "patient" | "doctor" | "user"
    source_language: str = Form(...),   # "hi-IN", "en-IN", etc.
    target_language: str = Form(...),
):
    audio_bytes = await audio.read()
    original = await transcribe(audio_bytes, source_language)
    translated = await translate(original["text"], source_language, target_language)
    tts_audio = await synthesise(translated, target_language)

    # Persist segment to DB
    segment = save_segment(session_id, speaker, source_language,
                           original["text"], translated, timestamp=utcnow())

    return {
        "segment_id": segment.id,
        "original_text": original["text"],
        "translated_text": translated,
        "tts_audio_b64": tts_audio,
    }
```

**Test this endpoint with `curl` before building any UI.**

---

## Step 2 — Conversational Intelligence (LLM Layer)

Add the LLM layer only after the voice pipeline is working end-to-end.

### Simple Q&A (no state management needed)

For single-turn or simple multi-turn: call the LLM directly in the utterance handler. Pass the last N transcript segments as context.

### Multi-step workflows (use LangGraph)

Use LangGraph when the application needs:
- Multiple processing steps (extract → validate → draft → review)
- Human approval or interruption (`interrupt()`)
- Conditional routing based on LLM output
- Persistent workflow state across HTTP requests

**LangGraph minimal pattern:**

```python
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from langgraph.types import interrupt, Command
from typing import TypedDict, Optional

class WorkflowState(TypedDict):
    session_id: str
    transcript: list[dict]
    extracted_data: Optional[dict]
    draft: Optional[dict]
    human_decision: Optional[dict]
    status: str

def build_graph():
    graph = StateGraph(WorkflowState)
    graph.add_node("extract", extract_node)
    graph.add_node("draft", draft_node)
    graph.add_node("human_review", human_review_node)   # uses interrupt()
    graph.add_node("finalize", finalize_node)

    graph.set_entry_point("extract")
    graph.add_edge("extract", "draft")
    graph.add_edge("draft", "human_review")
    graph.add_conditional_edges("human_review", route_after_review,
                                {"approved": "finalize", "rejected": END})
    graph.add_edge("finalize", END)

    return graph.compile(checkpointer=MemorySaver())

# Human review node — pauses graph
def human_review_node(state: WorkflowState):
    decision = interrupt({"draft": state["draft"], "message": "Awaiting doctor approval"})
    return {"human_decision": decision}

# FastAPI: trigger graph
@app.post("/api/sessions/{id}/complete")
async def complete_session(id: str, background_tasks: BackgroundTasks):
    state = load_initial_state(id)
    background_tasks.add_task(run_graph, id, state)
    return {"status": "processing"}

# FastAPI: resume graph after human decision
@app.post("/api/sessions/{id}/decision")
async def submit_decision(id: str, body: DecisionRequest):
    config = {"configurable": {"thread_id": id}}
    await graph.ainvoke(Command(resume=body.dict()), config=config)
    return {"status": "resumed"}
```

Rules:
- Use `MemorySaver` for demos/hackathons — sufficient if the backend does not restart
- Use `AsyncSqliteSaver` or `AsyncPostgresSaver` for production
- Always use `consultation_id` or `session_id` as the `thread_id`
- Run `graph.ainvoke` as a FastAPI `BackgroundTasks` task so the HTTP response is not blocked
- Frontend polls `GET /status` to detect state changes

### Structured LLM Output

For JSON extraction from transcripts:

```python
prompt = f"""
Extract information from this transcript. Return ONLY valid JSON matching this schema:
{json.dumps(OUTPUT_SCHEMA)}

Rules:
- Only use facts explicitly present in the transcript
- Use null for missing fields
- Do not invent or infer information

Transcript:
{transcript_text}

JSON only:"""

response = model.generate_text(prompt)
# Extract JSON from response
start, end = response.find("{"), response.rfind("}") + 1
parsed = json.loads(response[start:end])
validated = OutputSchema(**parsed)   # Pydantic validation
```

---

## Step 3 — Transcript UI

Build the transcript display after the backend is working. Keep it simple.

### Transcript Bubble Component

Each segment shows:
- Speaker label (patient / doctor / user / assistant)
- Original text — smaller, muted colour
- Translated text — larger, prominent
- Language badge
- Timestamp

Colour convention: use different bubble alignment/colour per speaker (e.g. user right/blue, assistant left/grey). Never mix original and translated text in the same visual weight.

### Polling Pattern (simplest)

```typescript
const useTranscript = (sessionId: string) => {
  const [segments, setSegments] = useState([]);
  useEffect(() => {
    const interval = setInterval(async () => {
      const data = await fetch(`/api/sessions/${sessionId}/transcript`).then(r => r.json());
      setSegments(data);
    }, 2000);
    return () => clearInterval(interval);
  }, [sessionId]);
  return segments;
};
```

Use polling for MVPs. Upgrade to WebSocket only when polling latency is visibly unacceptable in the demo.

---

## Step 4 — Approval and Action UI (if applicable)

If the application includes human-in-the-loop approval (doctor review, manager sign-off, etc.):

- Show draft content clearly labelled as "AI-drafted" until approved
- Approve / Reject / Edit-then-approve pattern
- Wire Approve → `POST /decision {action: "approve"}`
- Wire Reject → `POST /decision {action: "reject"}` → disable downstream actions
- Make idempotent: check `status !== "pending"` before processing; return current status if already decided
- Show audit trail: who approved, when, what version

---

## Step 5 — Error Handling and Fallbacks

Apply these before demo day:

| Failure | Frontend behaviour | Backend behaviour |
|---|---|---|
| Mic permission denied | Show "Microphone access required" message | N/A |
| STT fails | Show "Could not transcribe — try again" | Return `{error: "stt_failed", original_text: ""}` with HTTP 200 |
| Translation fails | Show original text only | Return `{translated_text: original_text, error: "translation_failed"}` |
| TTS fails | Show "▶ Play" button disabled + translated text visible | Return `{tts_audio_b64: null}` |
| LLM fails | Show "Processing failed — using sample output" with SAMPLE badge | Return deterministic fixture data, visibly labelled |
| LLM invents data | Pydantic validation catches schema violations; flag and reject | Constrain prompt + validate output schema |

**Never silently present fallback data as live AI output.** Always show a visible label.

---

## Step 6 — Demo Hardening Checklist

Before any live demo, verify:

- [ ] Full journey runs twice without manual intervention
- [ ] Audio autoplay works in both browser tabs (or fallback button is visible)
- [ ] Rejection path: reject → downstream actions disabled
- [ ] Duplicate request: click approve twice → one record only
- [ ] Reset endpoint: `GET /api/demo/reset` → clean state in <30s
- [ ] Backup recording of full demo saved
- [ ] All simulated actions labelled "DEMO — not sent"
- [ ] No `console.error` or unhandled promise rejections in browser
- [ ] Backend request timeout (10s) on all external API calls

---

## Reminders

- Build the smallest working loop first. Voice pipeline → transcript → LLM → approval UI. In that order.
- Do not add WebSocket, VAD, or streaming until the basic loop is demo-stable.
- Every external API call needs a timeout and a fallback return value.
- LangGraph interrupt/resume is powerful but tricky — always build a minimal standalone spike before integrating into the full app.
