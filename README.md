# Qwen3-14B Ascend 910C Decode Optimization

[English](#english) | [中文](#中文)

Driven by [MetaRSI](https://www.cosmosmind.ai/research/metarsi-v1) — an autonomous agent-driven optimization engine that continuously searches, measures, and updates this repository.

---

## English

An agent-discovered decode-path optimization for **Qwen3-14B** on **8 × Ascend 910C NPUs**, built on [PyPTO-Lib](https://www.pypto.ai/pypto-lib/) and validated end-to-end with the open-source [InferenceX / AIPerf](https://github.com/SemiAnalysisAI/InferenceX) harness.

**4.4% lower per-token decode latency, with generated tokens identical to the official baseline.**

### Results

| Metric | Baseline | This work | Delta |
|---|---|---|---|
| TPOT (short-context workload) | reference | **−4.43%** | three independent time windows, same direction |
| ITL (ISL = 8192, OSL = 1024, concurrency 16) | 48.7 ms | **47.0 ms** | −3.6% |
| Output throughput (same shape) | 221.4 tok/s | **227.6 tok/s** | +2.8% |

All measurements on physical Ascend 910C hardware (DP8 / TP1, BF16 official weights). Raw harness output is published under `evidence/agentx/`.

### What the patch changes

`patches/champion.diff` — a unified diff against PyPTO-Lib commit `604d4ead` (`models/qwen3_14b/`), covering scheduling and memory-path behavior:

- Cache-policy bypass on decode-path weight projections (q / k / v / gate / up / down)
- Joint scheduling across the gate and up projections
- Cache bypass on the lm_head weight
- Loop restructuring in the MLP path

Apply on top of that commit with `git apply patches/champion.diff`.

### How it was developed

Produced by an autonomous, agent-driven optimization loop: proposals are generated, debated, implemented, and measured against a moving best-known configuration; above-threshold findings are composed and re-measured before replacing the reference point. The loop is continuously updated — new measurements land here as they are produced.

Measurement discipline: three independent time windows must agree, generated tokens must be identical to the baseline, and aggregate throughput must hold. Crashes and infrastructure faults are recorded separately from candidate outcomes.

See `docs/method.md` for the search method and `docs/layers.md` for how this work maps onto the operator-to-serving stack.

### Repository layout

```
patches/champion.diff   # the optimization, ready to git apply
evidence/protocol.md    # measurement protocol
evidence/results.md     # full results
evidence/reproduce.md   # how to reproduce
evidence/agentx/        # raw InferenceX/AIPerf harness output
docs/method.md          # search method
docs/layers.md          # positioning across the operator→serving layers
```

### License

Apache-2.0

---

## 中文

一个由 agent 自动搜索发现的 **Qwen3-14B** 解码路径优化，面向 **8 × 昇腾 910C NPU**，基于 [PyPTO-Lib](https://www.pypto.ai/pypto-lib/)，并用开源的 [InferenceX / AIPerf](https://github.com/SemiAnalysisAI/InferenceX) 框架做了端到端验证。

**逐 token 解码延迟下降 4.4%，生成 token 与官方基线完全一致。**

### 结果

| 指标 | 基线 | 本方案 | 变化 |
|---|---|---|---|
| TPOT（短上下文工作点） | 基准 | **−4.43%** | 三个独立时间窗同向 |
| ITL（ISL=8192 / OSL=1024 / 并发 16） | 48.7 ms | **47.0 ms** | −3.6% |
| 输出吞吐（同形状） | 221.4 tok/s | **227.6 tok/s** | +2.8% |

全部测量在物理昇腾 910C 硬件上完成（DP8 / TP1，BF16 官方权重）。原始测量数据在 `evidence/agentx/` 下公开。

### 补丁内容

`patches/champion.diff` — 针对 PyPTO-Lib commit `604d4ead`（`models/qwen3_14b/`）的 unified diff，覆盖调度与内存路径行为：

- decode 路径上 q / k / v / gate / up / down 投影的权重 cache 旁路
- gate 与 up 投影的联合调度（依赖合并）
- lm_head 权重 cache 旁路
- MLP 路径的循环结构调整

在该 commit 之上 `git apply patches/champion.diff` 即可使用。

### 开发方式

由 agent 驱动的自动优化循环产出：提案、辩论、实现、测量围绕一个不断移动的最优配置进行；超过阈值的发现先组合堆叠、再复测，超过旧基准才替换基准。循环持续更新——新的测量一经产出就会更新到本仓。

测量纪律：三个独立时间窗必须同向、生成 token 必须与基线完全一致、聚合吞吐必须保持；崩溃与基建故障与候选结果分开记录。

搜索方法见 `docs/method.md`，在算子到 serving 的分层定位见 `docs/layers.md`。

### 仓库结构

```
patches/champion.diff   # 优化补丁，可直接 git apply
evidence/protocol.md    # 测量协议
evidence/results.md     # 完整结果
evidence/reproduce.md   # 复现方法
evidence/agentx/        # InferenceX/AIPerf 原始测量输出
docs/method.md          # 搜索方法
docs/layers.md          # 算子到 serving 的分层定位
```

### 许可证

Apache-2.0
