# Where this sits (four layers)

```
L1  Operators        Individual kernels. Where the changes live (PyPTO DSL scheduling directives).
L2  Operator packs   Hot combos shipped as libraries (the KDA/DSA/MLA pattern). Our four-family stack is a small one.
L3  Serving link     Fused operator sets inside a real inference link. Our candidates run inside pypto-serving.
L4  End to end       Whole-model TPOT/TTFT under a real load. Where the evidence lives.
```

We change things at L1 and prove them at L4. The repository leads with L4 evidence.
