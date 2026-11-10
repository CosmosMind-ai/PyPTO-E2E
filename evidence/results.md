# Results

## Headline

**+4.43% TPOT reduction** over the official baseline (frozen internal workload, three windows same direction, token-identical output, throughput non-regressing).

Composition of the champion (independently discovered and stacked):

| Component | Mechanism | Contribution |
|---|---|---|
| A | gate/up projection weight cache bypass | +2.68% |
| B | qkv projection weight cache bypass | +0.48% |
| C | down projection weight bypass | +0.69% |
| D | output consumer-cut join | +0.25% |
| A+B+C+D | four-family stack | +3.92% |
| **champion** | stack + MLP unroll tweak + lm_head weight bypass | **+4.43%** |

## InferenceX cross-check (official shape: ISL = 8192, OSL = 1024, concurrency 16)

| Arm | TTFT | ITL | Throughput |
|---|---|---|---|
| Baseline | 23.5 s | 48.7 ms | 221.4 tok/s |
| Champion | 23.4 s | 47.0 ms (−3.6%) | 227.6 tok/s (+2.8%) |

## Method notes

- Crashes are recorded as failures; infrastructure faults are recorded separately and retried — never counted against a candidate.
- Re-measuring identical code on different days drifts ±0.3–0.4pp; every conclusion here rests on three-window consistency plus repeated runs.
