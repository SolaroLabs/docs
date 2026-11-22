# Chapter 2: Concurrency Control, Sandbox Isolation, and State Convergence

**Part of the textbook:** [Frontier Multi-Agent Systems (2024–2026)](README.md)  
**Research & Writing:** Team 2 (Worktrees, Isolation & Single-Writer PIN Authority)  
**Lead Author:** Gemini 4 Carbon (Orc2)  
**Contributors:** Grok 4.7xHigh Builders (T2-B1..B4), Gemini 4 Carbon Reviewers (T2-R1..R2), Cursor Composer (T2-C1)  
**Anchor Target:** `#chapter-2`

---

## Chapter Overview
When multiple autonomous agents operate simultaneously on a shared codebase or environment, naive concurrent writes inevitably cause state corruption, race conditions, and livelock thrashing. In this chapter, we formalize the concurrency challenges unique to stochastic multi-agent systems, dissect sandbox isolation architectures (with particular emphasis on ephemeral git worktrees), and introduce the mathematical and cryptographic foundations of **Monotonic Epoch Fencing** (`FenceToken`) and **Upfront Dispatch Path Locking** (`DispatchLockManager`).

---

## 2.1 Concurrency Challenges in Stochastic Multi-Agent Environments

In classical distributed database systems, concurrency control algorithms (e.g., Two-Phase Locking, Multi-Version Concurrency Control, Paxos, Raft) assume deterministic transactional processes executing predictable read/write operations. 

In contrast, Large Language Model agents introduce unprecedented non-determinism:
1. **Unpredictable Turn Latency:** An agent turn may take 800 milliseconds for a cached read or 45 seconds for a complex code generation pass. Lock holding times cannot be bounded by tight millisecond timeouts without causing premature lease expiration.
2. **Stochastic Modification Footprints:** An agent asked to "refactor the authentication handler" might modify only `auth.py`, or it might unexpectedly rewrite `config.py`, `middleware.py`, and `database.py`.
3. **Semantic Intent Divergence:** Two agents operating under seemingly disjoint prompts may introduce semantically contradictory changes that pass git text-merge conflict checks cleanly (e.g., Agent A renames a function signature in module X, while Agent B adds a caller in module Y expecting the old signature).

Without rigorous isolation and concurrency guarantees, multi-agent execution collapses into race conditions, dirty reads, and catastrophic repository corruption.

---

## 2.2 The Split-Brain Dilemma and Dual-Writer Repository Corruption

A primary failure mode observed in multi-agent deployments between 2024 and 2025 was the **Split-Brain Dual-Writer Crisis** (documented in COSMOS operational scar logs as the *BTS Scar*). 

Consider the failure trajectory in un-fenced multi-agent swarms:

```
Timeline:
t0: Agent A acquires write lock on repository live tree.
t1: Agent A begins generating large refactoring patch (latency spikes).
t2: Central lock lease expires due to timeout.
t3: Agent B acquires write lock and commits changes to live tree.
t4: Agent A finishes generation, unaware its lease expired, and commits its patch.
===> RESULT: Live tree is corrupted with interleaved, irrecoverable state.
```

To eliminate dual-writer corruption, modern multi-agent systems enforce the **Constitutional Single-Writer Invariant**:
$$\forall t \in \text{Time}, \quad |\text{Writers}(\text{LiveTree}, t)| \le 1$$
At any given moment $t$, at most one entity—the designated Orchestrator holding an active cryptographic lease—is permitted to mutate the master live tree. All other agents must execute strictly within detached sandboxes and emit non-destructive proposals.

---

## 2.3 Sandboxing Paradigms: Ephemeral Git Worktrees and Isolated Sandboxes

To enforce the Single-Writer Invariant while maintaining high parallel throughput, systems separate the *execution environment* from the *target repository*.

```mermaid
graph TD
    subgraph Host Repository
        LT[(Live Master Tree: <cosmos-root>)]
        PL[Single-Writer PIN Authority / FenceToken]
    end

    subgraph Ephemeral Sandboxes (Read-Only Live Tree Clone)
        WT1[Git Worktree Task 01: sandbox_task_01]
        WT2[Git Worktree Task 02: sandbox_task_02]
        WT3[Git Worktree Task 03: sandbox_task_03]
    end

    LT -.->|Detached Worktree Add| WT1
    LT -.->|Detached Worktree Add| WT2
    LT -.->|Detached Worktree Add| WT3

    WT1 -->|Emits Verified Proposal| PB1[ProposalBundle 1]
    WT2 -->|Emits Verified Proposal| PB2[ProposalBundle 2]
    WT3 -->|Emits Verified Proposal| PB3[ProposalBundle 3]

    PB1 --> PL
    PB2 --> PL
    PB3 --> PL
    PL -->|Sequential PIN Landing| LT
```

### 2.3.1 Ephemeral Git Worktrees (`git worktree`)
Rather than copying tens of gigabytes of repository data for each agent, systems leverage detached git worktrees:
- **Instantaneous Provisioning:** `git worktree add -b cluster-task-<id> <sandbox_dir> HEAD` creates an isolated checkout sharing the underlying object store (`.git/objects`).
- **Complete Filesystem Isolation:** Modifications, temporary build artifacts, and test scratchpads exist exclusively in the sandbox directory.
- **Fail-Safe Abort:** If an agent hallucinates, enters an infinite loop, or fails tests, the orchestrator simply runs `git worktree remove --force <sandbox_dir>`, leaving the live tree pristine.

### 2.3.2 Sandbox Boundary Validation (`sandbox_runner.py`)
To prevent "sandbox escapes" where an agent uses relative path traversal (`../../`) or absolute paths to write outside its boundary, the sandbox runner ([`sandbox_runner.py`](clusters/sandbox_runner.py)) enforces static and runtime verification:
- Inspects every modified file before packaging.
- Validates that every file modified by the worker was explicitly declared in its `SPAWN_MANIFEST.json`.
- Raises `UNDECLARED_WRITE_VIOLATION` if any rogue write is detected, refusing to package the proposal.

---

## 2.4 Monotonic Epoch Fencing: The `FenceToken` Standard

To solve the split-brain timeout race condition, frontier architectures implement **Monotonic Epoch Fencing** ([`pin_engine.py`](clusters/pin_engine.py)).

### 2.4.1 Cryptographic Fence Token Structure
A `FenceToken` is an immutable, signed attestation issued by the PinEngine:

$$\text{FenceToken} = \langle \text{epoch}, \text{lease\_id}, \text{holder}, \text{target\_tree}, \text{issued\_at}, \text{expires\_at}, \sigma \rangle$$

Where the cryptographic signature $\sigma$ is computed using HMAC-SHA256 across the token tuple:
$$\sigma = \text{HMAC}_{K_{\text{PIN}}}(\text{epoch} \parallel \text{lease\_id} \parallel \text{holder} \parallel \text{expires\_at})$$

### 2.4.2 Verification Invariant
When any commit is attempted against the live tree, the verification engine checks three mandatory predicates:
$$\text{Verify}(T) = \big( \text{time.now}() \le T.\text{expires\_at} \big) \land \big( T.\text{epoch} = \text{Engine}.\text{current\_epoch} \big) \land \big( \text{VerifyHMAC}(T, K_{\text{PIN}}) \big)$$

If an agent whose lease timed out at epoch 101 attempts to land changes after epoch has incremented to 102, the verification engine instantly rejects the operation with `FENCING_TOKEN_STALE`:

```python
# From clusters.pin_engine
def verify_pin(self, token: FenceToken) -> bool:
    if token.is_expired():
        raise Refuse("FENCING_TOKEN_EXPIRED", f"Fence token {token.lease_id} expired")
    if token.epoch != self.current_epoch:
        raise Refuse(
            "FENCING_TOKEN_STALE",
            f"Token epoch {token.epoch} superseded by current epoch {self.current_epoch}"
        )
    expected_sig = self._sign(token.epoch, token.lease_id, token.holder, token.expires_at)
    if not hmac.compare_digest(token.signature, expected_sig):
        raise Refuse("FENCING_TOKEN_TAMPERED", "Invalid token signature")
    return True
```

---

## 2.5 Mitigating CAS Livelock Thrashing: Upfront File Write Path Claiming

Prior to late 2025, systems relying purely on optimistic concurrency control (Compare-And-Swap / 3-way merge at commit time) suffered severe **CAS Livelock Thrashing**. In high-concurrency clusters (e.g. 20–40 builders executing in parallel), multiple workers would concurrently modify the same critical files (e.g., `api.py` or `models.py`). While all workers completed their tasks in isolation, only the first worker could commit; all other workers failed at merge time and had to re-run, causing exponential latency inflation (*MAFBench*, arXiv:2602.03128).

### 2.5.1 Crucible Amendment A2: Upfront Dispatch Locking
To resolve CAS thrashing, Team 5 proposed and the consortium ratified **Amendment A2 (File Ownership at Dispatch)**, implemented in [`dispatch_lock.py`](clusters/dispatch_lock.py):

```mermaid
sequenceDiagram
    participant Orch as Orchestrator / Dispatcher
    participant DL as DispatchLockManager
    participant W1 as Builder 1 (Task A)
    participant W2 as Builder 2 (Task B)

    Orch->>DL: claim_paths(Task A, paths=['clusters/api.py'])
    DL-->>Orch: Claim Granted (Task A owns api.py)
    Orch->>W1: Spawn Task A in Sandbox

    Orch->>DL: claim_paths(Task B, paths=['clusters/api.py'])
    DL-->>Orch: REFUSE: DISPATCH_PATH_COLLISION (held by Task A)
    Note over Orch,W2: Task B is queued sequentially; never run concurrently
```

By enforcing that no two concurrent tasks can hold write claims on overlapping normalized paths, CAS merge conflicts are reduced to **zero**. Tasks with overlapping write footprints execute sequentially; tasks with disjoint write footprints execute in parallel with 100% commit success.

---

## 2.6 State Convergence Models: 3-Way CAS vs. Semantic Intent Merging

When isolated sandboxes complete execution, their changes must converge into the shared repository:

| Convergence Paradigm | Mechanism | Strengths | Failure Modes |
|---|---|---|---|
| **Pessimistic Sequential Merge** | Single-writer PIN lands proposals strictly one at a time. | Absolute safety; zero merge conflicts; deterministic test verification. | Lower parallel landing throughput if tasks are heavily serialized. |
| **Optimistic 3-Way Git CAS** | Merges sandbox branch into base branch using git 3-way merge (`base`, `ours`, `theirs`). | High parallel throughput on non-overlapping lines of code. | Silent semantic merge conflicts (syntactically valid code that fails runtime logic). |
| **Semantic Consensus Merging** | Acharya (Mar 2026, [arXiv:2603.xxxxx](06_BIBLIOGRAPHY_AND_HYPERLINKED_INDEX.md#ref-acharya)) intent-graph reconciliation. | Resolves conflicting business logic across autonomous agents. | Requires high token overhead to deliberate semantic intent graphs. |

In the COSMOS Clusters architecture, the **Pessimistic Sequential Proposal Landing** model is adopted for all Ring 0 production updates:
1. Builders produce a `ProposalBundle` in their sandbox.
2. The Main Orchestrator verifies test receipts and acquiring the single-writer PIN lease.
3. The Orchestrator applies the patch directly to the live tree, executes the local test suite (`pytest`), and signs the commit.
4. If tests fail, the Orchestrator rolls back the live tree immediately.

---

## Section Summary & Cross References
- **Lifecycle Guarantees:** How workers are bounded before entering sandboxes, see [Chapter 1: Foundations and Evolutionary Architecture](01_CHAPTER_1_FOUNDATIONS_AND_LIFECYCLES.md).
- **Silicon-Witnessed Durability:** How sandbox changes are collapsed into verifiable SEEDs and committed to WORM lakes, see [Chapter 3: Agent Durability, Memory Compression, and Cryptographic Attestation](03_CHAPTER_3_DURABILITY_AND_VERIFICATION.md).
- **Crucible Chaos Testing:** Empirical tests verifying stale token rejection and path collision prevention, see [Chapter 5: Adversarial Resilience, Security Governance, and Red-Teaming](05_CHAPTER_5_ADVERSARIAL_RESILIENCE_AND_CRUCIBLE.md).
