# Chapter 4: Cryptographic Messaging, Provenance, and Replay Protection

**Part of the textbook:** [Agent-to-Agent (A2A) Communications (2024–2026)](README.md)  
**Authoring Team:** Team 5 (Adversarial Crucible)  
**Anchor Target:** `#chapter-4-a2a`

---

## 4.1 Authenticating Agent Identities
In multi-agent networks, an unprivileged worker must never spoof an orchestrator's message. Every agent is bound at spawn time to an immutable `SPAWN_MANIFEST.json` and a role-scoped capability token.

## 4.2 Derived Session HMACs & Replay Protection
To prevent an attacker from replaying a valid message from an earlier epoch:
$$K_{\text{Session}} = \text{HMAC}_{K_{\text{Master}}}(\text{cluster\_id} \parallel \text{epoch})$$
Every `SEED.json` and `FenceToken` is signed with $K_{\text{Session}}$ and bound to a monotonic epoch counter. Stale tokens are rejected immediately with `FENCING_TOKEN_STALE`.

## 4.3 Non-Repudiation: DoneBundles & WORM Envelopes
Inter-agent proposals carry a 5-Hash `DoneBundle` (AST, Git Tree, Test Log, Scars, Provenance) archived to dual WORM lakes (`mdata-envelope/1`), ensuring cryptographic non-repudiation.

## 4.4 Securing Shared Blackboards
*Insider Attacks in Multi-Agent LLM Consensus Systems* (May 2026, [arXiv:2605.04412](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md#ref-insider-attacks)) showed that a single compromised agent can poison a shared blackboard. Defense requires **strict schema validation**, **AST silicon witnesses**, and **least-privilege tool gating**.

---
**Next:** [Chapter 5: High-Throughput Swarm Routing, Backpressure, and Observability](05_CHAPTER_5_ROUTING_BACKPRESSURE_OBSERVABILITY.md) | **Index:** [Bibliography & Index](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md)
