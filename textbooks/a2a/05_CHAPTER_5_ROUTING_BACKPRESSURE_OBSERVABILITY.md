# Chapter 5: High-Throughput Swarm Routing, Backpressure, and Observability

**Part of the textbook:** [Agent-to-Agent (A2A) Communications (2024–2026)](README.md)  
**Authoring Team:** Team 4 (Telemetry & Replay)  
**Anchor Target:** `#chapter-5-a2a`

---

## 5.1 Fan-Out Limits ($7 \pm 1$) and $O(\log N)$ Scaling
Unbounded flat swarms suffer $O(N^2)$ communication overhead. By capping each orchestrator's direct reports to $7 \pm 1$ workers and nesting tiers hierarchically, 40+ agents coordinate with $O(\log N)$ message depth.

## 5.2 Backpressure & Door Capacity Throttling
When downstream model providers (Vertex AI, xAI Grok, Cursor) approach rate limits, `doorcap.py` applies backpressure by queuing non-critical inter-agent dispatches in memory rather than triggering HTTP 429 cascades.

## 5.3 Tracing Inter-Agent Lineage & Divergence Analysis
Using `csm_lmdb_schema.py` and `SessionReplayEngine`, every inter-agent message is indexed in sub-microsecond LMDB storage, enabling deterministic replay and cosine-similarity divergence detection.

## 5.4 Synthesis & 2027+ Horizons
Future A2A communications will adopt **Zero-Knowledge Agent Proofs (ZK-STARKs)**—allowing agents to prove task verification without exposing proprietary prompts—and **native OS-level MCP IPC**.

---
**Index:** [Bibliography & Hyperlinked Master Index](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md)
