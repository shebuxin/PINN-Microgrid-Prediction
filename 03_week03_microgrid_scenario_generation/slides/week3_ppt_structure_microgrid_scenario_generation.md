
# Week 3 PPT Structure

**Course:** Physics-informed AI for Three-phase Microgrid Security Screening  
**Week 3 topic:** Microgrid test system construction and scenario generation with `pandapower`  
**Lecture length:** 1.5--2 hours  
**Lab length:** 1.5--2 hours  
**Position in course:** Week 1 introduced balanced static security metrics. Week 2 introduced three-phase unbalanced `runpp_3ph`. Week 3 turns the three-phase solver into a repeatable data-generation engine for Week 4 N-1 labeling and Weeks 5--7 AI modeling.

---

## Learning Objectives

After Week 3, the student should be able to:

1. Explain how a low-voltage feeder can be converted into a microgrid security-assessment test case.
2. Identify the PCC, transformer, LV buses, critical loads, PV buses, and BESS buses in a pandapower network.
3. Use topology and load information to select meaningful device locations.
4. Add phase-specific PV and BESS-like static setpoints using three-phase pandapower element tables.
5. Define deterministic and stochastic operating scenarios for load, PV, BESS, and phase imbalance.
6. Run a three-phase base-case power flow for each scenario.
7. Extract scenario-level features: voltage range, loading, VUF, grid import/export, PV level, and load level.
8. Validate the scenario generator using reproducibility, sign-convention, and engineering-trend checks.
9. Export a scenario dataset that can be used directly by Week 4 N-1 contingency labeling.

---

# Slide-by-slide Structure

## Section 0. Opening and Week 2 Bridge

### Slide 1. Title
**Title:** Week 3: Microgrid Scenario Generation with Three-phase pandapower  
**Subtitle:** From one unbalanced power flow to a reusable security-prediction dataset

Visual idea: a feeder connected to the grid through PCC, with PV, BESS, critical loads, and a scenario sampler.

### Slide 2. Where Week 3 fits in the 8-week project
- Week 1: balanced static security metrics.
- Week 2: three-phase unbalanced power flow.
- **Week 3: microgrid test system + scenario generator.**
- Week 4: N-1 scan and violation labels.
- Weeks 5--7: ML, physics-informed MLP, phase-aware GNN.
- Week 8: paper and presentation.

Key message: Week 3 builds the data engine; later weeks only work if this week is reproducible.

### Slide 3. What we already know from Week 2
- How to represent A/B/C phase loads and DER injections.
- How to run `runpp_3ph`.
- How to read `res_bus_3ph` and `res_line_3ph`.
- How to compute VUF and phase-level loading.
- Why proof checks are necessary.

Transition: now we need many operating points, not one case.

---

## Section 1. Microgrid Test-system Story

### Slide 4. Microgrid security prediction: the operational question
For each operating scenario:

\[
    x_t = \{P_{load,\phi}, Q_{load,\phi}, P_{PV,\phi}, P_{BESS,\phi}, mode, topology\}
\]

we want to know whether the microgrid is close to a static security violation.

Week 3 only generates base-case scenarios. Week 4 will add contingencies.

### Slide 5. From LV feeder to microgrid story
A low-voltage feeder becomes a microgrid test system by identifying:

- PCC / grid connection,
- MV/LV transformer,
- LV radial feeder,
- critical load buses,
- non-critical flexible load buses,
- PV locations,
- BESS locations,
- operating mode and scenario profile.

### Slide 6. Why use `ieee_european_lv_asymmetric`
Teaching advantages:

- already unbalanced,
- many low-voltage buses,
- single-phase loads,
- mostly radial topology,
- large enough to motivate AI screening,
- still small enough for classroom experiments.

Teaching boundary: use it as a feeder-level microgrid proxy, not as a certified real microgrid model.

### Slide 7. Microgrid elements in pandapower tables
| Microgrid concept | pandapower representation |
|---|---|
| PCC / upstream grid | `ext_grid` |
| MV/LV transformer | `trafo` |
| LV feeder | `line` |
| single-phase or unbalanced load | `asymmetric_load` |
| PV / static DER | `asymmetric_sgen` |
| BESS setpoint | phase-specific load/sgen pair or `storage` metadata |
| switch / isolation | `switch` or element `in_service` flag |

### Slide 8. Grid-connected mode first, islanding later
Week 3 MVP:

- keep `ext_grid` in service;
- treat PCC as slack/grid equivalent;
- model DER and BESS as static setpoints;
- generate grid-connected operating scenarios.

Later extension:

- PCC outage / islanded mode,
- grid-forming inverter or slack-equivalent DER,
- critical-load restoration.

---

## Section 2. Choosing Microgrid Devices

### Slide 9. Selecting critical loads
A simple rule for Week 3:

\[
\text{critical buses} = \text{top-k load buses by } P_a+P_b+P_c
\]

Why this is useful:

- easy to explain,
- deterministic,
- gives meaningful high-impact buses,
- creates a bridge to later violation severity.

### Slide 10. Selecting PV buses
Use topology distance from the LV PCC:

\[
\text{PV buses} = \text{far load buses with available load connection points}
\]

Rationale:

- rooftop PV is naturally distributed;
- far-end PV can create overvoltage and reverse-flow cases;
- phase-specific PV creates unbalance.

### Slide 11. Selecting BESS buses
A practical MVP rule:

- one BESS near a large critical load;
- one BESS near a far feeder end;
- split BESS power equally across phases for the first implementation.

Later extension:

- phase-specific BESS control,
- reactive support,
- SOC time-series update,
- optimized BESS dispatch.

### Slide 12. Why deterministic device selection matters
If the device placement changes every time, model comparison becomes invalid.

Week 3 metadata must record:

```text
critical_load_buses
pv_buses
bess_buses
pv phase assignment
PV rated power
BESS rated power
random seed
```

### Slide 13. Topology-based device placement workflow
1. Load network.
2. Find LV PCC / transformer LV bus.
3. Build network graph.
4. Compute graph distance from LV PCC.
5. Rank load buses by load size and distance.
6. Select critical load, PV, and BESS buses.
7. Save device configuration.

---

## Section 3. Scenario Design

### Slide 14. What is an operating scenario?
A scenario is a structured set of multipliers and setpoints:

```text
scenario_id
load_scale
phase_a_factor
phase_b_factor
phase_c_factor
pv_scale
bess_p_discharge_mw
mode
```

Positive BESS setpoint means discharge/injection in the Week 3 convention.

### Slide 15. Deterministic scenarios for teaching
Recommended Week 3 deterministic set:

| Scenario | Purpose |
|---|---|
| base | reference point |
| high_load | undervoltage / loading stress |
| high_pv_midday | reverse-flow / overvoltage stress |
| evening_peak_bess_discharge | high load with storage support |
| phase_b_heavy | phase unbalance stress |
| stress_unbalance | deliberately severe case |

### Slide 16. Stochastic scenarios for AI data
For AI training, define distributions:

\[
\alpha_L \sim U(0.7,1.6),\quad
\alpha_{PV} \sim U(0,1.8)
\]

\[
\alpha_a,\alpha_b,\alpha_c \sim \text{truncated normal around } 1
\]

\[
P_{BESS} \sim U(-P_{charge}^{max}, P_{dis}^{max})
\]

Use fixed seeds for reproducibility.

### Slide 17. Grid-connected vs islanded scenarios
Week 3 framework uses grid-connected scenarios only.

Why:

- simpler slack/PCC interpretation;
- fewer numerical issues;
- enough to generate base-case features.

Future islanded extension:

- switch off PCC;
- assign a grid-forming DER/slack;
- check whether critical loads are supportable.

### Slide 18. Scenario split for later AI experiments
Do not randomly split every row without thought.

Recommended splits:

- **ID split:** similar load/PV range in train and test.
- **OOD high-PV split:** test PV levels beyond training.
- **OOD high-load split:** test load levels beyond training.
- **phase-unbalance split:** test unseen phase-factor combinations.

---

## Section 4. Running Three-phase Scenario Studies

### Slide 19. Scenario loop
Core workflow:

```python
for scenario in scenario_table:
    reset_to_base_case(net)
    apply_load_scale_and_phase_factors(net, scenario)
    apply_pv_setpoints(net, scenario)
    apply_bess_setpoint(net, scenario)
    runpp_3ph(net)
    extract_security_summary(net)
```

### Slide 20. Why reset before every scenario?
Without reset, scenario multipliers accumulate.

Example bug:

```text
base load -> high load x 1.3 -> high PV x 0.8
```

If reset is missing, the second case is not 0.8 base load but 1.04 base load.

### Slide 21. Base-case security summary
For every scenario, record:

\[
V_{min}=\min_{i,\phi}|V_{i,\phi}|,
\quad
V_{max}=\max_{i,\phi}|V_{i,\phi}|
\]

\[
L_{max}=\max_{\ell,\phi}\text{loading}_{\ell,\phi},
\quad
U_{max}=\max_i\mathrm{VUF}_i
\]

Also record:

```text
p_grid_mw
p_load_mw
p_pv_mw
q_grid_mvar
converged
```

### Slide 22. What becomes AI input later?
Week 3 exported features can include:

- load scale and phase factors,
- PV scale and PV penetration,
- BESS setpoint,
- min/max voltage,
- max line loading,
- max transformer loading,
- max VUF,
- grid import/export.

Week 4 will add contingency ID and post-contingency labels.

### Slide 23. What should not become a label yet?
Week 3 is not N-1 assessment.

Do not label a scenario safe/unsafe only from base case unless the research question is base-case security.

For this course:

```text
Week 3 = base-case operating points
Week 4 = operating point × contingency labels
```

---

## Section 5. Validation and Proof Checks

### Slide 24. Proof 1: input integrity
Check before every scenario study:

- exactly one external grid in the grid-connected MVP;
- one MV/LV transformer;
- all element bus references exist;
- line zero-sequence parameters are present;
- phase-specific load and PV tables have A/B/C columns.

### Slide 25. Proof 2: reset/apply idempotence
For the same scenario:

\[
\text{signature}(apply(s)) = \text{signature}(reset; apply(s))
\]

This catches accumulated scaling bugs.

### Slide 26. Proof 3: sign-convention checks
Expected behavior:

- increasing load increases total load;
- increasing PV decreases grid import;
- BESS charging increases grid import;
- BESS discharging decreases grid import.

These are not formal power-flow proofs, but they catch common sign mistakes.

### Slide 27. Proof 4: engineering trend checks
Expected behavior:

- high load lowers minimum voltage;
- high load increases line/transformer loading;
- phase-heavy load increases VUF;
- high PV can raise maximum voltage;
- stress scenario should be more severe than base.

### Slide 28. Cross-validation: two independent nets
Run the same scenario on two freshly built networks:

\[
summary_1(s) \approx summary_2(s)
\]

If the two differ, hidden state or non-deterministic scenario application is likely present.

### Slide 29. Dataset schema validation
Before saving to CSV, verify required columns:

```text
scenario_id
load_scale
pv_scale
phase_a_factor
phase_b_factor
phase_c_factor
bess_p_discharge_mw
converged
min_vm_pu
max_vm_pu
max_vuf_percent
max_line_loading_percent
p_grid_mw
p_load_mw
p_pv_mw
```

---

## Section 6. Lab Plan and Outputs

### Slide 30. Notebook 03 lab workflow
1. Load feeder.
2. Inspect network tables.
3. Select critical/PV/BESS buses.
4. Add phase-specific PV and BESS-like elements.
5. Define deterministic scenarios.
6. Run `runpp_3ph` for each scenario.
7. Extract scenario summary.
8. Run validation suite.
9. Export CSV files.

### Slide 31. Files created by Week 3
Expected outputs:

```text
week3_scenario_table.csv
week3_device_config_pv.csv
week3_device_config_bess.csv
week3_scenario_security_summary.csv
week3_basecase_ai_features.csv
week3_validation_summary.csv
```

These become inputs to Week 4 N-1 scanning.

### Slide 32. Student checkpoint questions
- Which bus is the LV PCC?
- Which buses were chosen as PV locations, and why?
- Which scenario has the lowest voltage?
- Which scenario has the highest VUF?
- Does high PV reduce grid import?
- Does BESS charging/discharging use the correct sign?

### Slide 33. Common mistakes
- Applying load multipliers cumulatively.
- Treating generation as positive load.
- Forgetting zero-sequence parameters.
- Comparing phase-to-neutral and line-to-line voltages incorrectly.
- Using random scenarios without saving the seed.
- Exporting features before confirming convergence.

### Slide 34. Homework
1. Add two extra deterministic scenarios.
2. Create a 20-scenario random sample with fixed seed.
3. Identify the top 5 most stressed scenarios by max VUF.
4. Identify the top 5 most stressed scenarios by minimum voltage.
5. Write one paragraph explaining how Week 3 data will be used for Week 4 N-1 labels.

### Slide 35. Bridge to Week 4
Week 3 produced operating states:

\[
    x_t
\]

Week 4 will produce labels by applying contingencies:

\[
    (x_t, c) \rightarrow y_{t,c},\ s_{t,c}
\]

The Week 4 question:

> For each scenario and each N-1 outage, does the post-contingency three-phase power flow violate voltage, loading, VUF, or convergence limits?
