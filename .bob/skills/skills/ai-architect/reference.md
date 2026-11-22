# AI Architect Skill — Reference Material

This file is loaded alongside `SKILL.md` and contains the full reference detail for technology options, design principles, communication style, and phase guidance. Refer to it during the engagement as needed.

---

## Technology Stack Options

Select only what the requirements justify. This is a menu, not a mandatory list.

| Area | Options |
|---|---|
| Backend language | Python (default) |
| API framework | FastAPI |
| Data validation | Pydantic |
| LLM integration | LangChain |
| Agent orchestration | LangGraph |
| Relational storage | PostgreSQL, SQLite (MVP/hackathon) |
| Graph database | Neo4j (only when graph relationships provide real value) |
| Vector search | pgvector, OpenSearch |
| Cache | Redis (only when caching is a stated requirement) |
| Search | OpenSearch, Elasticsearch |
| Event streaming | Kafka, Redpanda (only for genuine async/event-driven needs) |
| Observability | OpenTelemetry, Jaeger, Prometheus |
| Containers | Docker, Docker Compose |
| Deployment | Kubernetes, cloud platforms |
| Frontend | React |

---

## Architecture Style Selection Guide

| Style | Use when |
|---|---|
| Modular monolith | Early-stage product, small team, unclear boundaries — **default choice** |
| Microservices | Independently deployable components with separate scaling needs and clear domain boundaries |
| Event-driven | Async workflows, decoupled producers/consumers, audit trails |
| Agent-based | Dynamic tool selection, multi-step reasoning, non-deterministic workflows |
| Hybrid | Monolith core with event-driven or agent extensions |

---

## AI Component Selection Guide

Work down this list. Use the simplest option that satisfies the requirement.

| Need | Use |
|---|---|
| Single LLM call | Direct API call |
| Structured extraction | LLM + Pydantic output parser |
| Document Q&A | RAG pipeline |
| Tool use with fixed flow | LangChain tool-calling chain |
| Stateful multi-step workflow | LangGraph |
| Multi-agent collaboration | LangGraph multi-agent (only when single agent is insufficient) |

---

## RAG Pipeline Checklist

When retrieval is required, evaluate each stage:

1. **Ingestion** — source format, update frequency, ownership
2. **Chunking** — strategy (fixed, semantic, sentence), chunk size, overlap
3. **Metadata extraction** — source, timestamp, entity tags
4. **Embedding** — model selection, dimensionality, update strategy
5. **Vector storage** — pgvector vs OpenSearch depending on scale
6. **Retrieval** — semantic only, lexical only, or hybrid
7. **Reranking** — cross-encoder or score fusion
8. **Citation tracking** — which chunks informed the answer

---

## Neo4j Decision Criteria

Introduce Neo4j when **all** of these hold:
- The domain has meaningful relationships between entities (not just foreign keys)
- Those relationships need to be traversed (multi-hop queries)
- The traversal logic would be significantly harder in SQL

Default to PostgreSQL for:
- Simple entity relationships
- Permission lists / allowlists
- Metadata storage
- Anything expressible cleanly with JOINs

---

## LangGraph Decision Criteria

Use LangGraph when the agent needs **at least one** of:
- Persistent state across steps
- Conditional branching based on intermediate results
- Checkpointing or resumability
- Human-in-the-loop approval gates
- Multi-agent coordination

Do not use LangGraph for:
- Single-turn LLM calls
- Simple tool chains with no branching
- Workflows that are fully deterministic

---

## NFR Evaluation Questions

For each NFR, ask whether it needs implementation in the MVP or can be deferred:

| NFR | MVP consideration |
|---|---|
| Scalability | What is the expected concurrent user/agent count? |
| Performance | What latency is acceptable? |
| Availability | Is downtime tolerable? What SLA? |
| Security | What data is sensitive? Who are the threat actors? |
| Privacy | Is PII involved? Any regulatory requirements? |
| Observability | What do operators need to see? |
| Cost | Is token or infrastructure cost a constraint? |

---

## Communication Style Guidelines

### Lead with plain language

State what a component does and why it exists before introducing technical detail.

### Use analogies where they help

| Component | Analogy |
|---|---|
| API Gateway | Reception desk — routes visitors to the right department |
| Message Queue | Waiting room — work waits its turn |
| Redis | Short-term memory — fast, temporary |
| PostgreSQL | Structured filing cabinet — organized, queryable |
| Neo4j | Relationship map — who knows whom, what depends on what |
| LangGraph | Workflow controller — decides what step comes next |
| Vector database | Meaning-based library — finds documents by concept, not keyword |

Use analogies only when they clarify. Skip them for experts.

### Output format preference

- Short explanations over long prose
- Tables for comparisons and trade-offs
- Mermaid diagrams for flows and architecture
- Bullet points for lists
- Small code snippets only when they clarify the design
- Never large code blocks just to look thorough

### Decision explanations

Do not say: "We will use Neo4j."

Say: "We need to traverse agent → tool → permission relationships across multiple hops. A graph database makes this natural. If we only need a simple permission list, PostgreSQL with a junction table is sufficient and simpler."

---

## Architecture Decision Record (ADR) Template

```
## ADR: [Short title]

**Context:** What situation or requirement is driving this decision?

**Options considered:**
1. Option A — [brief description]
2. Option B — [brief description]

**Decision:** Option [X]

**Rationale:** Why this option over the alternatives.

**Trade-offs:** What are we accepting or giving up?

**Consequences:** What does this constrain or enable going forward?
```

---

## Mermaid Diagram Types to Consider

| Diagram | When to use |
|---|---|
| `graph TD` / `LR` | Component relationships, system context |
| `sequenceDiagram` | Request flows, agent execution, failure paths |
| `flowchart` | Decision logic, workflow branching |
| `erDiagram` | Data models |
| `C4Context` | System context (if C4 is preferred) |

Keep diagrams readable. Remove nodes that don't add meaning.

---

## Final Review Checklist (Phase 8)

| Question | Pass / Fail |
|---|---|
| Solving the actual business problem? | |
| Unnecessary components removed? | |
| Component boundaries clear? | |
| Data ownership clear? | |
| Failure scenarios covered? | |
| AI component genuinely necessary? | |
| Security and privacy addressed? | |
| MVP achievable with available resources? | |
| System can evolve without full redesign? | |

State the three highest risks and their mitigations before finalizing.
