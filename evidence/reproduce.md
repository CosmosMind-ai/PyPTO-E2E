# Reproduce

Everything below runs on a single 8 × Ascend 910C machine with PyPTO-Serving installed.

## 1. Apply the patch

```bash
git clone https://github.com/hw-native-sys/pypto-lib.git
cd pypto-lib
git checkout 604d4eade07f6cfffe1bf7821692d5cea1f3061e
git apply /path/to/patches/champion.diff
```

## 2. Serve the model

Serve the patched tree with PyPTO-Serving (Qwen3-14B, BF16 official weights, DP8 / TP1):

```bash
pypto-serving \
  --model /path/to/Qwen3-14B --served-model-name qwen3-14b \
  --backend npu --platform a2a3 --devices 0,1,2,3,4,5,6,7 --dp 8 --tp 1 \
  --block-size 128 --max-model-len 512 --max-num-seqs 16 \
  --max-num-batched-tokens 512 --long-prefill-token-threshold 512 \
  --ring-dep-pool 262144 --ring-task-window 262144 --ring-heap 1073741824 \
  --no-enable-prefix-caching --host 127.0.0.1 --port 8325 \
  --generate-config '{"ignore_eos":true,"temperature":0}'
```

## 3. Measure

With the open-source [InferenceX / AIPerf](https://github.com/SemiAnalysisAI/InferenceX) harness (`agentx-harness`):

```bash
agentx profile \
  --url http://127.0.0.1:8325 --endpoint /v1/chat/completions --endpoint-type chat \
  --model qwen3-14b --tokenizer /path/to/Qwen3-14B \
  --streaming --concurrency 16 --request-count 64 --num-warmup-requests 8 \
  --isl 256 --osl 128 --osl-stddev 0 \
  --extra-inputs ignore_eos:true --extra-inputs max_tokens:128 \
  --ui none --output-artifact-dir ./out
```

For the InferenceX official shape, use `--isl 8192 --osl 1024` and raise `--max-model-len` / `--max-num-batched-tokens` accordingly (the serving tree must be built with a matching kernel MAX_SEQ).

## 4. Acceptance discipline

- Three independent time windows must agree in direction, and the merged aggregate must hold.
- Generated tokens must be identical to the unpatched baseline.
- Aggregate throughput must not regress.
- Run baseline and candidate on the same machine, same day; identical code re-measured across days drifts ±0.3–0.4pp.
