# Week 7 PPT Structure: Phase-aware GNN and High-recall Screening

**Course module:** Physics-informed AI for three-phase microgrid N-1 security prediction  
**Week 7 theme:** Use graph learning to encode topology, phase-channel information, and contingency masks; evaluate the model as a high-recall screening tool rather than a stand-alone decision maker.

---

## Slide 1 — Title
**Week 7: Phase-aware GNN 与高召回筛查策略**

Subtitle: From feature-vector models to topology-aware N-1 risk ranking.

Visual suggestion: microgrid feeder graph with an outaged line highlighted and a ranked contingency list on the side.

---

## Slide 2 — Where Week 7 Fits
- Week 1: balanced microgrid model and security quantities.
- Week 2: three-phase unbalanced power flow.
- Week 3: scenario generation.
- Week 4: N-1 labeling.
- Week 5: traditional ML screening.
- Week 6: physics-informed / limit-aware MLP.
- **Week 7: phase-aware graph learning and top-k screening.**

Key transition:

\[
\text{feature vector } f(x_t,c) \quad \rightarrow \quad \text{graph } G(x_t,c)
\]

---

## Slide 3 — Learning Objectives
After this week, the student should be able to:

1. Explain why microgrid security prediction is naturally a graph-learning problem.
2. Construct bus-level phase-channel node features.
3. Encode line outage, PV trip, BESS trip, and PCC outage as graph masks or node flags.
4. Implement a small message-passing GNN without relying on PyTorch Geometric.
5. Evaluate random holdout, scenario-OOD, and contingency-OOD performance.
6. Use high-recall thresholding and top-k ranking for AI-assisted N-1 screening.

---

## Slide 4 — Why MLP Is Not Enough
Week 5 and Week 6 models use a flattened feature vector.

This is useful but limited:

- The same numerical features may mean different things at different feeder locations.
- Line outage risk depends on downstream load and topology.
- PV / BESS trip risk depends on where the DER sits in the feeder.
- PCC outage has a system-level meaning.

A graph model can encode:

\[
\text{who is connected to whom}, \quad \text{where the contingency happens}, \quad \text{how power must flow through the feeder}
\]

---

## Slide 5 — Graph Formulation
Define a microgrid graph:

\[
G=(\mathcal{N},\mathcal{E})
\]

- Node \(i \in \mathcal{N}\): bus.
- Edge \((i,j) \in \mathcal{E}\): line segment.
- Node feature \(x_i\): phase load, phase PV, BESS, bus role, contingency flag.
- Edge feature \(e_{ij}\): length, current limit, availability, outage flag.
- Global feature \(g\): scenario settings and base-case security statistics.

Prediction target:

\[
\hat y_{t,c}=f_\theta(G(x_t,c))
\]

---

## Slide 6 — Bus-level Phase-channel GNN MVP
Week 7 MVP uses bus-level nodes with phase channels in the node feature vector.

Example node features:

```text
is_pcc, is_critical, is_load_bus, is_pv_bus, is_bess_bus
P_load_a, P_load_b, P_load_c
P_pv_a, P_pv_b, P_pv_c
P_bess_dis_a, P_bess_dis_b, P_bess_dis_c
P_bess_ch_a, P_bess_ch_b, P_bess_ch_c
contingency flags
base-case voltage / loading / VUF context
```

Why this MVP is good for teaching:

- It keeps the graph small and interpretable.
- It preserves A/B/C phase information.
- It avoids a large phase-node graph implementation in the first version.

---

## Slide 7 — Phase-level GNN as Extension
A more physical future model can split each bus into bus-phase nodes:

\[
(i,a),\ (i,b),\ (i,c)
\]

Advantages:

- Explicit phase topology.
- Better representation of single-phase PV / load.
- Natural encoding of VUF-related effects.

Tradeoff:

- More nodes and edges.
- More difficult to teach in one week.
- Requires more careful three-phase line coupling representation.

Use this as the journal-paper extension.

---

## Slide 8 — Contingency Encoding
Different N-1 events need different graph encodings.

| Contingency | Graph encoding |
|---|---|
| line outage | set edge availability to 0; mark endpoints |
| PV trip | mark PV bus and PV trip flag |
| BESS trip | mark BESS bus and BESS trip flag |
| PCC outage | mark PCC bus and global no-slack flag |

The model sees the same scenario under different contingencies as different graphs.

---

## Slide 9 — Message Passing Equation
For each layer:

\[
m_{ij}^{(k)} = a_{ij}\,\psi_\theta\left(h_j^{(k)}, e_{ij}\right)
\]

\[
h_i^{(k+1)}=\sigma\left(W_s h_i^{(k)} + \sum_{j\in\mathcal{N}(i)} m_{ij}^{(k)}\right)
\]

where \(a_{ij}\in\{0,1\}\) is line availability.

For a line outage:

\[
a_{ij}=0
\]

so the outaged line does not pass messages.

---

## Slide 10 — Graph-level Readout
After message passing, pool node embeddings:

\[
h_G = \mathrm{pool}_{i\in\mathcal{N}}(h_i^{(K)})
\]

Then concatenate global features:

\[
z = [h_G, g]
\]

Predict:

\[
\hat p = \Pr(y=1\mid G), \quad \hat s = \text{severity proxy}
\]

---

## Slide 11 — What Makes It Phase-aware?
Phase-aware does not require phase-level nodes in the MVP.

It means the model is not given only total load / total PV. It receives separate A/B/C channels:

\[
P_{i,a}, P_{i,b}, P_{i,c}
\]

\[
P^{PV}_{i,a}, P^{PV}_{i,b}, P^{PV}_{i,c}
\]

This allows the model to learn patterns associated with phase imbalance and VUF-related violations.

---

## Slide 12 — Dataset Interface
Week 7 consumes files generated in earlier weeks:

```text
week5_ml_input_dataset.csv
week4_contingency_table.csv
```

Each row is still:

\[
(x_t,c,y_{t,c},s_{t,c})
\]

but Week 7 converts each row into:

```text
node feature matrix X
edge index
edge feature matrix E
global feature vector g
label y
severity s
```

---

## Slide 13 — No-leakage Principle
The GNN input can use:

- scenario settings;
- base-case security summaries;
- static topology and device placement;
- contingency identity and location.

It must not use:

- post-contingency voltages;
- post-contingency line loadings;
- post-contingency VUF;
- violation flags;
- final labels;
- severity scores.

---

## Slide 14 — Validation Proofs in Notebook
Notebook should include:

1. Topology proof: connected and radial base graph.
2. Feature proof: finite node / edge / global tensors.
3. Contingency mask proof: line outages remove exactly one edge.
4. No-leakage proof.
5. Forward-pass shape proof.
6. Finite loss / gradient proof.
7. Permutation-invariance proof.
8. Split-disjoint proof for random, scenario, and contingency tests.

---

## Slide 15 — Random Holdout Is Not Enough
Random split answers:

> Can the model interpolate among similar scenarios and contingencies?

It is useful but optimistic.

For deployment, also test:

- unseen operating scenarios;
- unseen contingencies;
- high-stress conditions;
- low-frequency violation modes.

---

## Slide 16 — Scenario-group CV
Scenario-OOD split:

\[
\mathcal{S}_{train}\cap\mathcal{S}_{test}=\emptyset
\]

Purpose:

> Can the model generalize to a new operating condition, such as high PV or stress imbalance?

This is closer to operating-day generalization.

---

## Slide 17 — Contingency-group CV
Contingency-OOD split:

\[
\mathcal{C}_{train}\cap\mathcal{C}_{test}=\emptyset
\]

Purpose:

> Can the model generalize to a new device outage?

This is harder because the model must infer risk from topology and device placement, not memorize a contingency ID.

---

## Slide 18 — High-recall Thresholding
The classifier probability is not the final decision.

Select threshold on validation data:

\[
\max_\tau \ \mathrm{PFCallsSaved}_{val}(\tau)
\]

subject to:

\[
\mathrm{Recall}_{val}(\tau) \geq 0.95
\]

The test set is used only after threshold selection.

---

## Slide 19 — Screening Interpretation
If \(\hat p_{t,c}\geq \tau\):

```text
send contingency to three-phase power flow verification
```

If \(\hat p_{t,c}<\tau\):

```text
screen out as low risk
```

Critical safety quantity:

\[
\text{missed violations}=FN
\]

---

## Slide 20 — Top-k Ranking
For each scenario, rank contingencies:

\[
c_1,c_2,\ldots,c_K = \operatorname{argsort}_c \hat p_{t,c}
\]

Then run exact three-phase power flow only for top \(K\) events.

Metric:

\[
\mathrm{ViolationRecall@K}
=\frac{\#\text{unsafe contingencies in top K}}{\#\text{unsafe contingencies}}
\]

---

## Slide 21 — Why Top-k Matters in Practice
Operators may have a limited verification budget:

```text
Run exact 3ph PF for only top 3, top 5, or top 10 contingencies
```

The AI model is useful if:

- dangerous events appear near the top;
- safe events are pushed down;
- the model is conservative under uncertainty.

---

## Slide 22 — Baselines for Week 7
Compare:

1. Flattened MLP: ignores graph topology.
2. Phase-aware GNN: uses topology, masks, and device locations.
3. Optional: Week 6 PI-MLP scores as reference.

The purpose is not to prove GNN always wins, but to teach what graph information adds.

---

## Slide 23 — Expected Results Tables
Table I: graph dataset statistics.

Table II: random holdout metrics.

Table III: scenario-group CV and contingency-group CV.

Table IV: top-k recall.

Table V: validation checks.

---

## Slide 24 — Expected Figures
Suggested figures:

1. Microgrid graph with bus roles.
2. Training curves.
3. Precision-recall curve.
4. PF calls saved vs missed violations.
5. Top-k recall curve.
6. OOD CV summary bar chart.

---

## Slide 25 — Lab Workflow
Notebook sequence:

```text
1. Load Week 5/4 datasets
2. Define teaching feeder graph
3. Convert each row to graph tensors
4. Validate graph tensors and no-leakage
5. Train flattened MLP and GNN
6. Select high-recall threshold
7. Evaluate random holdout
8. Run scenario/contingency group CV
9. Compute top-k ranking metrics
10. Save tables and figures
```

---

## Slide 26 — Student Deliverables
By the end of Week 7, students should submit:

1. Executed notebook.
2. Graph dataset summary table.
3. Holdout comparison table.
4. Scenario-OOD and contingency-OOD table.
5. Top-k ranking figure.
6. A paragraph explaining whether graph structure helped and why.

---

## Slide 27 — Connection to Week 8
Week 8 will turn the project into a mini-paper.

Week 7 provides the main AI method section:

```text
Phase-aware GNN for topology-changing N-1 risk screening
```

and the main screening result:

```text
high recall + fewer exact three-phase power-flow calls
```

---

## Slide 28 — Instructor Notes
Key teaching points:

- Make clear that GNN is not magic; the feature design and validation protocol matter.
- Emphasize that random split can be misleading.
- Focus on conservative screening, not pure accuracy.
- Keep the first GNN small and readable.
- Use permutation-invariance and mask checks to teach graph-model correctness.
