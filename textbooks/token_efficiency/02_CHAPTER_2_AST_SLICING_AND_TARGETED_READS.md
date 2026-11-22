# Chapter 2: AST Slicing, Symbol-Targeted Reads, and Zero-Waste Navigation

**Part of the textbook:** [Ultra-Efficient Token Use in AI Multi-Agent Coding (2024–2026)](README.md)  
**Authoring Team:** Team 1 (Harness & AST Navigation)  
**Anchor Target:** `#chapter-2-tokens`

---

## 2.1 Locate Before Reading
Reading entire 2,000-line files wastes 15,000+ tokens per call. Efficient agents use `rg -n` or `ast_nav.py` to locate exact symbol line ranges, then read only `[StartLine, EndLine]` (<150 lines).

## 2.2 Zero-Body Directory `mtime` Caching
For session scanners (`cmsessions.py`), reading directory `mtime` via `os.scandir` skips historical logs outside the 7-day hot set with **zero tokens** and $O(1)$ memory.

## 2.3 ACON & ContextEvolve (2025–2026)
*ACON* ([arXiv:2510.xxxxx](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md#ref-acon)) demonstrated 26–54% token reduction by dynamically pruning stale tool observations while preserving failure-analysis guidelines.

---
**Next:** [Chapter 3: Prompt Caching and KV-Cache Optimization](03_CHAPTER_3_PROMPT_CACHING_AND_KV_OPTIMIZATION.md) | **Index:** [Bibliography & Index](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md)
