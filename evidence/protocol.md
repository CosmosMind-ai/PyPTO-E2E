# Measurement protocol

All headline numbers come from a discipline designed to survive scrutiny:

- **Three time windows, same direction.** A candidate only counts if it wins in three independent measurement windows and in the merged aggregate.
- **Token identity.** Generated output tokens must be identical to the official baseline's, every window, no exceptions.
- **Throughput non-regression.** Aggregate tok/s must not regress.
- **FAIL vs INVALID.** A candidate that crashes is FAIL. An infrastructure fault is INVALID. Never one disguised as the other.
- **Lease-serialized hardware.** All NPU work holds a lease from a shared store; measurements never share a device.
- **Noise discipline.** Repeated measurements of the same code drift ±0.3–0.4pp day to day. Conclusions require consistency across windows, and cross-checks across days.

Two metric families are reported:

- **Internal frozen workload** (short context, ISL=256/OSL=128, temperature 0, DP8/TP1 on 8×910C): our fast iteration loop. TPOT is measured as native HTTP sum/count over the window.
- **InferenceX official shape** (ISL=8192/OSL=1024, concurrency 16): the cross-check, measured with the open-source InferenceX/AIPerf harness against a serving instance of the patched model.
