# Method: tree search with crossover

The optimization was not hand-tuned. An autonomous loop searched the space:

1. **Propose and debate.** A proposer LLM suggests a mechanism; a critic LLM tries to kill it. Survivors get implemented.
2. **Cheap screen.** Each candidate runs a single-card (DP1) screen. Most die here, cheaply.
3. **Formal acceptance.** Promoted candidates get the full 8-card, three-window measurement against the frozen baseline.
4. **Rooted tree.** Every candidate hangs off the current best-known stack (the "root"). Breadth lanes explore directions the current composition does not cover.
5. **Crossover to re-root.** When a single-lane win beats the root, it does not replace the root immediately. Above-root findings are composed into a stacked crossover; the crossover becomes the new root only if it measures above the old one.

This "broaden, then stack, then re-root" rhythm is what kept the search from spinning in circles on one mechanism family.

## Failure hygiene

- Candidate crash → FAIL (a real, informative negative).
- Infrastructure fault → INVALID (auto-retried; never counted against the candidate).
- Every measurement carries an evidence fingerprint (execution key, artifact digests, lease binding).

## Scale

22 rounds, up to 96 parallel proposal lanes per round, 723 ledgered measurements, on a single 8×910C machine.
