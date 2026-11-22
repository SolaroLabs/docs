# Proof the floor is built

Measured on this machine on 2026-11-13. The commands below were run against the checkouts named here. The pass counts are the program's own output.

The product repositories are private. This file is the public record of what ran. A green sentence in a paper is not this record.

## Grok close

Command: `py -3.14 cosmos/cosmos_resession.py --selftest`

Result: `ok  31/31`

Lines from that run that fix the ceiling:

- `OK    Grok 130000 tokens warns (65 percent of 200k)`
- `OK    90 percent with tokens below 170000 does not close`
- `OK    170000 is review_stop and does not fire`
- `OK    180000 tokens fires the close`
- `OK    180001 raises and does not fire`
- `OK    a non-grok rail does not use 170 or 180`

170,000 input tokens is the Grok review stop. It does not close the session. 180,000 is the hard close, down from 190,000. A context percent does not close a Grok session. Other models are not on this pair.

That behavior is in the COSMOS body, branch `channel/engine-room`, commit `2f8532bf`.

## Coding harness

Command: `py -3.14 -m pytest cosmos_harness/tests/test_plane.py -q --tb=no`

Result: `17 passed in 0.84s`

The same modules ship inside cmCode. cmCode `main` is `6452932`. The harness is not a separate install.

## cmCode

Command: `py -3.14 -m pytest cmCode/tests -q --tb=no`

Result: `34 passed in 5.02s`

Those tests import `cosmos_harness.jail.Jail` and `cosmos_harness.bundle.DoneBundle` from the tree inside cmCode. Code's own copies of those classes are gone.

## Cluster smoke

Command: `py -3.14 -m pytest cmClusters/stage/cosmos_clusters_v0.2.0/tests/test_three_plane_smoke.py -q --tb=no`

Result: `2 passed in 0.05s`

The stage fork's old writers refuse with `CONTROL_PLANE_RETIRED` before they write a ledger. The live supervisor is cmClusters `main` `d97e483`, which includes that retired stage tree under `stage/` and the swarm modules (`swarm.py`, `a2a.py`, and the rest). A lead defaults to 6 workers, may have 7, and the 8th worker is refused.

## What else is a repository, and what was not in this run

| Product | Public? | Tip used as the source of these pages | This run |
|---|---|---|---|
| COSMOS body | Private (scrubbed SolaroLabs export pending), engine-room branch | `2f8532bf` | 31/31 self-test |
| cmCode | Private | `6452932` | 34 passed, plus the 17 harness tests |
| cmClusters | Private | `d97e483` | 2 passed on the retired stage smoke |
| cmSessions | Private | `2bf6368` | Not re-run here. The git tree is dated 2026-11-13 and is larger than the 2026-11-10 `sessions-page` tree. `replay_engine.py` and `scale_indexer.py` match that older tree. `desk.py` and `ui/index.html` do not. |
| cmMobile | Private | `308514a` | Not re-run here. The tree is the cdm history plus `cvm-phone`. |
| VoiceDrop | Private | `b348d10` | Not re-run here. The tree is the voice package plus `voice_duplex`. |
| cDeck | Private | `01ccd6d` | Not a test run. `ui/index.html` and `ui/deck_front.js` are on `main`. |
| TokenCenter | Private | `de0664c` | Not re-run here. The pay modules and `site/` are on `main`. The public door is https://tokenctr.com |

## What this file does not claim

No bake-off between model families. The motif paper says the same thing. No claim that the Windows cmSessions zip from 2026-11-10 is the 2026-11-13 tree. No claim that a phone app is in a store. No claim that SpiderCaster's canvas harness is built. That harness is later work.
