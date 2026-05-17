# Week 1 PPT Structure  
## Microgrid Security Prediction: Story, Toolchain, and First pandapower Model

**Course project:** Physics-informed AI for three-phase unbalanced N-1 security screening in microgrids  
**Week 1 goal:** Build the research motivation, introduce the minimum power-system vocabulary, and run a first pandapower microgrid-like power-flow example.

---

## Suggested class format

| Segment | Time | Purpose |
|---|---:|---|
| Research story and motivation | 15 min | Why this project matters |
| Microgrid static security basics | 25 min | What needs to be predicted |
| pandapower data model | 25 min | How the simulation environment works |
| Notebook walkthrough | 35–45 min | Build and solve a small system |
| Discussion and assignment | 10 min | Prepare for Week 2 |

---

## Slide-by-slide structure

### Slide 1 — Title
**Title:** AI for Static Security Prediction in Unbalanced Microgrids  
**Subtitle:** Week 1 — Storyline, Toolchain, and First pandapower Model  
**Visual:** Microgrid schematic: utility grid, PCC, transformer, PV, BESS, loads, critical load.  
**Speaker note:** Frame the project as “AI-assisted fast screening,” not “AI replacing power-flow solvers.”

### Slide 2 — What the student will build in eight weeks
**Key message:** The final product is a complete research prototype.  
**Content:** scenario generation; three-phase unbalanced power flow; N-1 scanning; violation labels; ML/PI-MLP/GNN screeners; high-recall evaluation.  
**Visual:** End-to-end pipeline diagram.

### Slide 3 — Why microgrids?
**Key message:** Microgrids are controllable local systems with DERs and critical loads.  
**Content:** electrical boundary; DERs; critical/non-critical loads; grid-connected and islanded operation; resilience and fast decision support.  
**Visual:** Boundary box around a microgrid connected to the main grid through PCC.

### Slide 4 — Why static security prediction?
**Key message:** Operators need to know whether the current state can survive plausible outages.  
**Content:** N-0 base case; N-1 outage; line/transformer/PCC/PV/BESS outages; voltage/current/VUF/non-convergence violations.  
**Visual:** Before/after outage diagram.

### Slide 5 — Why three-phase unbalanced analysis?
**Key message:** Low-voltage microgrids are often not captured well by balanced positive-sequence models.  
**Content:** single-phase loads; uneven rooftop PV; EV charging; phase imbalance; phase-wise checks.  
**Visual:** A/B/C voltage bars at one bus.

### Slide 6 — The computational bottleneck
**Key message:** Scenario × contingency × three-phase power-flow scanning can be expensive.  
**Content:** many time points; stochastic load/PV/BESS scenarios; many candidate outages; three-phase solver cost.  
**Visual:** Nested loop pseudocode.

### Slide 7 — Role of AI in this project
**Key message:** AI is a fast pre-screener for detailed power-flow verification.  
**Content:** input = base-case state + topology + contingency; output = violation probability/severity/ranking; objective = high recall and low false-negative rate.  
**Visual:** AI block before detailed pandapower validation.

### Slide 8 — Key objects in a microgrid model
**Content:** bus, line/cable, transformer, external grid, load, static generator, storage, switch/PCC.  
**Visual:** Physical object → pandapower table mapping.

### Slide 9 — Power-flow variables
**Content:** voltage magnitude, voltage angle, active/reactive power, branch current/loading, transformer loading.  
**Visual:** Feeder with P/Q arrows and bus voltages.

### Slide 10 — What is a violation?
**Content:** undervoltage, overvoltage, branch/transformer overloading, voltage unbalance, non-convergence.  
**Visual:** Voltage profile with acceptable band.

### Slide 11 — From violation detection to prediction
**Content:** detection = simulate and check limits; prediction = infer risk before exhaustive scanning; labels come from simulations.  
**Visual:** Detection branch versus prediction branch.

### Slide 12 — Why pandapower?
**Content:** Python-based; DataFrame-oriented; supports power flow, contingency analysis, time series, OPF, and three-phase/asymmetric power flow; fits Jupyter teaching and reproducible research.  
**Visual:** Python + pandas + pandapower + PyTorch workflow.

### Slide 13 — pandapower data model
**Key message:** A network is input tables plus result tables.  
**Content:** `net.bus`, `net.line`, `net.load`, `net.sgen`, `net.storage`, `net.trafo`, `net.ext_grid`; result tables such as `net.res_bus`, `net.res_line`.  
**Visual:** Table screenshots.

### Slide 14 — Minimal modeling workflow
**Content:** create empty network → create buses/elements → run `pp.runpp(net)` → inspect result tables → check limits.  
**Visual:** Code-to-result flowchart.

### Slide 15 — Week 1 example network
**Content:** 20 kV utility grid; 0.4 kV LV microgrid; PCC; critical load; flexible/EV load; PV; BESS.  
**Visual:** Simple radial feeder.

### Slide 16 — Sign conventions to remember
**Content:** load positive P = consumption; sgen positive P = injection; storage positive P = charging and negative P = discharging.  
**Visual:** Load arrow into bus, PV arrow out of bus, BESS bidirectional arrow.

### Slide 17 — Balanced today; three-phase next week
**Content:** Week 1 uses `pp.runpp()` to learn the workflow; Week 2 uses `runpp_3ph()` and asymmetric elements.  
**Visual:** Balanced model → three-phase unbalanced model.

### Slide 18 — Notebook 1 learning objectives
**Content:** create network; run base-case power flow; read bus/line result tables; plot voltage and loading; generate simple operating scenarios; output a security summary.

### Slide 19 — Notebook cell map
**Content:** imports; build network; inspect tables; run base case; plot results; check simple violations; run scenarios; save outputs.

### Slide 20 — Base-case interpretation questions
**Content:** lowest-voltage bus; highest-loading line; PV effect on grid import; BESS charging/discharging effect.  
**Visual:** Voltage bar chart and line-loading bar chart.

### Slide 21 — First security summary table
**Content:** scenario, total load, PV generation, BESS setpoint, minimum voltage, maximum line loading, transformer loading, violation flag.  
**Visual:** Example table.

### Slide 22 — Common modeling mistakes
**Content:** wrong voltage levels; unavailable line type; sign convention errors; not checking `net.converged`; checking average instead of worst-case voltage; treating balanced result as three-phase security result.

### Slide 23 — How Week 1 connects to the paper
**Content:** Week 1 simulation literacy; Week 2 three-phase physics; Week 3 scenarios; Week 4 N-1 labels; Weeks 5–7 AI; Week 8 mini-paper.  
**Visual:** Eight-week roadmap.

### Slide 24 — In-class discussion prompts
**Content:** most risky asset; high PV vs high load vs BESS charging; why false negatives are dangerous; what variables should become AI features.

### Slide 25 — Homework
**Content:** run Notebook 1; vary PV and load; find the lowest-voltage scenario; write 5–8 sentences explaining it; prepare one question about three-phase unbalance.

### Slide 26 — Exit checklist
**Content:** explain project storyline; identify microgrid components; create/inspect pandapower tables; run one power flow; interpret voltage/loading; understand why Week 2 moves to `runpp_3ph`.

---

## Instructor notes

### Key teaching emphasis
Week 1 should be conceptually light but operationally concrete. The student must leave with a working notebook and confidence manipulating a grid model.

### Avoid in Week 1
Full AC Jacobian derivation; deep GNN/PINN details; full N-1 contingency loops; detailed sequence-component derivation.

### Prepare for Week 2
Preview `asymmetric_load`, `asymmetric_sgen`, per-phase results, voltage unbalance factor, and `runpp_3ph`.