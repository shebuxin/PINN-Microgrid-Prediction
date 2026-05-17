# Final Presentation Outline

## Slide 1. Title
Physics-informed AI for Fast Three-phase N-1 Security Screening in Unbalanced Microgrids

## Slide 2. Problem motivation
Unbalanced microgrids require phase-aware static security checks; exhaustive three-phase N-1 scans are accurate but expensive.

## Slide 3. Research question
Can AI pre-screening maintain high recall while reducing the number of post-contingency `runpp_3ph()` calls?

## Slide 4. Dataset pipeline
Scenario generation -> N-1 contingency catalog -> three-phase power flow -> violation labels.

## Slide 5. Violation definition
Voltage, loading, VUF, service-loss, critical service-loss, and non-convergence labels.

## Slide 6. Screening formulation
Input: base-case features + contingency metadata. Output: risk probability / severity / top-k ranking.

## Slide 7. Baselines
Engineering rule, Logistic Regression, Random Forest, Gradient Boosting, MLP.

## Slide 8. Physics-informed MLP
Multi-head prediction with component margins and consistency losses.

## Slide 9. Phase-aware GNN
Bus-level phase-channel graph with line outage masks and graph-level risk readout.

## Slide 10. Holdout results
Show Table II and explain recall, FNR, PF calls saved, missed violations.

## Slide 11. Screening tradeoff
Show saved PF calls vs missed violations.

## Slide 12. OOD-style evaluation
Scenario-group and contingency-group CV.

## Slide 13. Top-k contingency prioritization
Show top-k violation recall.

## Slide 14. Validation suite
Dataset lineage, no-leakage, metric proof, graph invariance, reproducibility.

## Slide 15. Limitations and future work
More feeders, islanding, grid-forming inverter, phase-level GNN, conformal screening, real data.
