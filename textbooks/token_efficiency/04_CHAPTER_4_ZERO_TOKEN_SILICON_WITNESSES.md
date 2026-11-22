# Chapter 4: Zero-Token Silicon Witnesses & Deterministic Verification

**Part of the textbook:** [Ultra-Efficient Token Use in AI Multi-Agent Coding (2024–2026)](README.md)  
**Authoring Team:** Team 5 (Crucible & Silicon Witness)  
**Anchor Target:** `#chapter-4-tokens`

---

## 4.1 Eliminating "LLM-as-a-Judge" Token Waste
Using a frontier model to review syntax or guess whether tests pass burns thousands of output tokens and still hallucinates.

## 4.2 Crucible Amendment A3: Deterministic Silicon Witness
In `tidyup.py` and `lsp_guard.py`, syntax (`ast.parse`), type checking (`pyright`), and test suites (`pytest -q`) run locally on CPU:
- **Token Cost:** **0 LLM tokens**.
- **Latency:** <50ms for AST parse.
- **Accuracy:** 100% deterministic ground truth.

---
**Next:** [Chapter 5: Output Compression, Diff-Only Edits, and Multi-Model Tiering](05_CHAPTER_5_DIFF_EDITS_AND_MODEL_TIERING.md) | **Index:** [Bibliography & Index](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md)
