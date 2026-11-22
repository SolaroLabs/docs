# Chapter 2: Semantic Handoffs, Schema Compression, and Intent Graphs

**Part of the textbook:** [Agent-to-Agent (A2A) Communications (2024–2026)](README.md)  
**Authoring Team:** Team 2 (Concurrency & State)  
**Anchor Target:** `#chapter-2-a2a`

---

## 2.1 Lossless vs. Lossy State Handoffs
Passing a 150,000-token conversation history to a successor agent is economically ruinous and triggers attention dilution. Instead, A2A systems perform **lossy-syntax, lossless-intent compression**: stripping raw compiler noise while preserving `goal`, `decisions`, `files_touched`, and `open_questions`.

## 2.2 Structured Handoff Payloads (`HandoffState`, `SEED.json`)
In COSMOS Clusters, inter-agent handoffs are standardized into an ~11 KB `SEED.json` payload:
- `seed_version`: `"cosmos-seed/2"`
- `done_bundle`: 5-Hash cryptographic proof of AST, git tree, test log, scars, and provenance.
- `handoff`: Minimal structured state for the successor to boot clean at ~20k tokens.

## 2.3 Semantic Intent Graphs (Acharya, 2026)
Acharya ([arXiv:2603.01182](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md#ref-acharya)) demonstrated that two agents can pass text-merge checks while holding contradictory business logic. **Intent Graphs** model each agent's planned mutations as a directed graph of preconditions and postconditions, detecting semantic conflicts before execution.

## 2.4 Cross-Model Translation
Heterogeneous swarms bridge **Gemini 4 Carbon** (Orchestrators), **Grok 4.7** (Builders), and **Cursor Composer** (Integrators) by normalizing all inter-agent payloads to strict JSON schemas, eliminating model-specific XML/Markdown dialect drift.

---
**Next:** [Chapter 3: Consensus, Negotiation, and Sycophancy Mitigation](03_CHAPTER_3_CONSENSUS_AND_NEGOTIATION.md) | **Index:** [Bibliography & Index](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md)
