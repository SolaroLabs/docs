# Chapter 5: Output Compression, Diff-Only Edits, and Multi-Model Tiering

**Part of the textbook:** [Ultra-Efficient Token Use in AI Multi-Agent Coding (2024–2026)](README.md)  
**Authoring Team:** Team 4 (Telemetry & Routing)  
**Anchor Target:** `#chapter-5-tokens`

---

## 5.1 Diff-Only Edits vs. Full-File Rewrites
Output tokens cost $3\times$–$5\times$ more than input tokens. Replacing `write_file` (rewriting 800 lines) with targeted `edit_file` (replacing 10 lines) cuts output token spend by **95%**.

## 5.2 Heterogeneous Model Tiering (`hero_router.py`)
- **Orchestration & Architecture:** Gemini 4 Carbon / Argon (Vertex AI \$300 tier).
- **Bulk Code Generation:** Grok 4.7xHigh (SuperGrokHeavy quota, <180k ceiling).
- **Trivial / High-Volume Lookups:** Flash / Flash-Lite models.

---
**Index:** [Bibliography & Hyperlinked Master Index](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md)
