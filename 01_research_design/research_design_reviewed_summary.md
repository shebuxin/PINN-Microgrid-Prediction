# Optimized Research Design Summary

## Working title

**Physics-informed Phase-aware Learning for Fast Static N-1 Security Screening in Unbalanced Microgrids**

## Core problem

For each microgrid operating scenario `x_t` and contingency `c`, predict whether three-phase static security violations occur after the contingency. Violations include phase voltage limits, phase current / loading limits, voltage unbalance, service loss, and non-convergence / no-slack cases.

## Screening role

The AI model is designed as a pre-screening module:

```text
base-case state + scenario features + contingency descriptor
        -> AI risk score
        -> select high-risk contingencies for detailed runpp_3ph validation
```

It should not be framed as a replacement for three-phase power flow.

## Recommended paper experiments

1. Teaching MVP: compact 9-bus radial feeder, 7 scenarios, 11 contingencies.
2. Research extension: larger LV / microgrid feeders, hundreds or thousands of stochastic scenarios.
3. Report results for all-contingency screening and PF-solvable-contingency screening separately.
4. Use scenario-group, contingency-group, and preferably feeder-group splits.
5. Keep high-recall metrics central: recall, FNR, missed violations, PF calls saved, and top-k violation recall.

## Strongest current contribution

The strongest current contribution is not the small-dataset performance number; it is the reproducible, leakage-aware, proof-heavy workflow connecting three-phase pandapower simulation to AI screening and paper-ready evaluation.
