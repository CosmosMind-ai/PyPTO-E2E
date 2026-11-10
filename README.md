# qwen3-14b-910c-decode-opt

An AI-agent-discovered optimization of Qwen3-14B decode on 8×Ascend 910C NPUs (PyPTO-Lib), validated end-to-end.

**4.4% lower per-token decode latency (TPOT/ITL), with output tokens identical to the official baseline.**

| Metric | Baseline | This work | Delta |
|---|---|---|---|
| TPOT (frozen internal workload) | reference | **−4.43%** | three time windows, token-identical output |
| ITL (InferenceX official shape, ISL=8192/OSL=1024) | 48.7 ms | **47.0 ms** | −3.6% |
| Output throughput (same shape) | 221.4 tok/s | **227.6 tok/s** | +2.8% |

## What the patch does

`patches/champion.diff` — a unified diff against PyPTO-Lib commit `604d4ead` (models/qwen3_14b). Scheduling/memory-path changes only, no arithmetic changes:

- Cache-policy bypass on decode-path weight projections (q/k/v/gate/up/down)
- Dependency merge across the gate/up projections (joint scheduling)
- Cache bypass on the lm_head weight
- Minor MLP loop restructuring

Apply with `git apply patches/champion.diff` on top of that commit.

## How it was found

Not hand-tuned. Produced by a Harness-RSI loop — an autonomous search in which LLM agents propose, debate, implement, and measure candidates against a moving SOTA root:

```
propose → debate → implement → cheap single-card screen
→ 8-card three-window acceptance → promote to new root
→ breadth search on uncovered directions → crossover → re-root
```

- Tree search: every candidate hangs off the current root; a stacked crossover must beat the root to replace it.
- Two-level funnel: cheap DP1 screen filters most candidates; only promoted ones get the formal 8-card three-window measurement.
- Anti-gaming discipline: candidate crashes are recorded FAIL, infra issues INVALID — never disguised; every measurement carries evidence fingerprints.

Scale: 22 rounds, 723 ledgered measurements.

## Evidence

- `evidence/protocol.md` — measurement protocol (three windows, token identity, noise discipline, lease-serialized hardware).
- `evidence/results.md` — full numbers.
- `evidence/agentx/` — raw JSON from the InferenceX/AIPerf harness.

## Roadmap

- [x] Champion at +4.43% (internal frozen workload)
- [x] Cross-validation with the InferenceX harness (small shape)
- [ ] Top-5 × 5-rep table at the official InferenceX shape (8192/1024)
- [ ] Sealed-holdout confirmation
- [ ] Upstream PR to PyPTO-Lib
- [ ] Same method applied to DeepSeek V4-Flash

## Disclaimer

Unofficial results. The patch targets PyPTO-Lib commit `604d4ead`; official InferenceX results live at [SemiAnalysisAI/InferenceX](https://github.com/SemiAnalysisAI/InferenceX).
