# Chapter 3: Prompt Caching, Prefix Stability, and KV-Cache Hit Optimization

**Part of the textbook:** [Ultra-Efficient Token Use in AI Multi-Agent Coding (2024–2026)](README.md)  
**Authoring Team:** Team 2 (State & Caching)  
**Anchor Target:** `#chapter-3-tokens`

---

## 3.1 Prefix Stability Invariant
Cloud KV-caches (Vertex AI Context Caching, Anthropic Prompt Caching, xAI Prefix Cache) discount cached input tokens by **75–90%**. However, mutating a single character in the system prompt or tool definitions invalidates the entire prefix cache.

## 3.2 Append-Only Turn Discipline
- Keep system instructions, `SPAWN_MANIFEST.json`, and tool schemas **100% static** at the top of the prompt.
- Never inject dynamic timestamps (`time.now()`) into the system prompt; place dynamic state only in the latest user/tool turn.

---
**Next:** [Chapter 4: Zero-Token Silicon Witnesses](04_CHAPTER_4_ZERO_TOKEN_SILICON_WITNESSES.md) | **Index:** [Bibliography & Index](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md)
