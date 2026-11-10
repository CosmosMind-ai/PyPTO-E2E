# Results

## Headline

Champion **e000698**: **+4.43% TPOT reduction** over the official baseline (internal frozen workload, three windows same direction, token-identical output, S3 throughput non-regressing).

Composition of the champion (independently discovered and stacked):

| Family | Mechanism | Contribution |
|---|---|---|
| e000301 | gate/up projection weight cache bypass | +2.68% |
| e000166 | qkv projection weight cache bypass | +0.48% |
| e000434 | down projection weight bypass | +0.69% |
| e000037 | output consumer-cut join | +0.25% |
| — | four-family stack (e000566) | +3.92% |
| xover613_612 | stack + MLP unroll tweak + lm_head weight bypass (e000698) | **+4.43%** |

## InferenceX cross-check (official shape: ISL=8192, OSL=1024, concurrency 16)

| Arm | TTFT | ITL | Throughput |
|---|---|---|---|
| B0 (official baseline) | 23.5 s | 48.7 ms | 221.4 tok/s |
| e000698 (champion) | 23.4 s | 47.0 ms (−3.6%) | 227.6 tok/s (+2.8%) |

## Method notes

- Candidates that crashed are FAIL; infrastructure faults are INVALID. 723 ledgered measurements across 22 rounds.
- Same code re-measured on different days drifts ±0.3–0.4pp; every conclusion here rests on three-window consistency plus repeated runs.
