
# Week 2 PPT Structure

**Course:** Physics-informed AI for Three-phase Microgrid Security Screening  
**Week 2 topic:** pandapower 三相不平衡建模与 `runpp_3ph`  
**Lecture length:** 1.5--2 hours  
**Lab length:** 1.5--2 hours  
**Position in course:** Week 1 introduced balanced `pandapower` and static security metrics. Week 2 moves to phase-aware, unbalanced modeling, which becomes the data engine for Week 3 scenario generation and Week 4 N-1 labeling.

---

## Learning Objectives

After Week 2, the student should be able to:

1. Explain why LV microgrids cannot always be represented by a balanced positive-sequence model.
2. Distinguish phase-to-neutral and line-to-line voltage conventions.
3. Compute positive-, negative-, and zero-sequence voltages from abc phasors.
4. Use `create_asymmetric_load` and `create_asymmetric_sgen` for phase-specific loads and PV injections.
5. Run `runpp_3ph` and interpret `res_bus_3ph`, `res_line_3ph`, `res_trafo_3ph`, and phase-level loading.
6. Verify three-phase results using basic physics checks: VUF recomputation, line loading formula, system power balance, and balanced-case cross-validation.

---

# Slide-by-slide Structure

## Section 0. Opening and Week 1 bridge

### Slide 1. Title
**Title:** Week 2: Three-phase Unbalanced Power Flow with pandapower  
**Subtitle:** From balanced microgrid examples to phase-aware security features

Visual idea: a simple radial LV feeder with A/B/C phase channels.

### Slide 2. Where Week 2 fits in the 8-week project
- Week 1: balanced microgrid-like feeder, `runpp`, static security metrics.
- Week 2: phase-aware modeling and `runpp_3ph`.
- Week 3: microgrid scenario generation.
- Week 4: three-phase N-1 scanning and violation labels.
- Weeks 5--7: ML, PI-MLP, phase-aware GNN.

Key message: Week 2 builds the simulation primitive that all later AI labels depend on.

### Slide 3. Recap of Week 1 balanced assumptions
- One voltage magnitude per bus.
- One net P/Q load or injection per bus.
- One loading percentage per line.
- Good first approximation for teaching, but insufficient for LV unbalance.

Question to students: what information was hidden by the balanced model?

---

## Section 1. Why three-phase unbalance matters in microgrids

### Slide 4. Microgrid reality: DERs and loads are not evenly distributed
- Single-phase residential loads.
- Single-phase rooftop PV.
- EV chargers connected to different phases.
- Phase-specific outages or weak feeders.
- BESS and inverter controls may not balance phases automatically.

### Slide 5. What can go wrong if we use only balanced power flow?
- A feeder looks safe in average voltage but one phase is undervoltage.
- Average loading is acceptable but one phase current is overloaded.
- Voltage unbalance can violate power quality limits.
- AI labels trained on balanced results can miss phase-specific risk.

### Slide 6. Static security metrics in a three-phase microgrid
Define the Week 2 metric set:

\[
\min_{i,\phi} |V_{i,\phi}|, \quad
\max_{\ell,\phi} I_{\ell,\phi}, \quad
\max_i \mathrm{VUF}_i, \quad
\text{converged / not converged}
\]

Use these as a bridge to Week 4 N-1 labels.

---

## Section 2. Three-phase electrical quantities

### Slide 7. Phase notation
- Phases: \(a,b,c\)
- Bus-phase voltage: \(V_{i,a}, V_{i,b}, V_{i,c}\)
- Branch-phase current: \(I_{\ell,a}, I_{\ell,b}, I_{\ell,c}\)
- Phase-specific load/generation: \(P_{i,\phi}, Q_{i,\phi}\)

### Slide 8. Phase-to-neutral vs line-to-line voltages
- LV systems often report line-to-line nominal voltage, e.g. 0.4/0.416 kV.
- Phase-to-neutral voltage magnitude is approximately \(V_{LL}/\sqrt{3}\).
- pandapower bus nominal voltage is specified through `vn_kv`.
- Interpret result tables carefully before deriving currents or limits.

### Slide 9. Balanced phasors as a reference case
For an ideal balanced system:

\[
V_a = V\angle 0^\circ, \quad
V_b = V\angle -120^\circ, \quad
V_c = V\angle 120^\circ
\]

Teaching prompt: if all three magnitudes are equal but angles are not 120 degrees apart, is the system balanced?

### Slide 10. Symmetrical components
Introduce Fortescue transform:

\[
\begin{bmatrix}
V_0\\V_1\\V_2
\end{bmatrix}
=\frac{1}{3}
\begin{bmatrix}
1&1&1\\
1&a&a^2\\
1&a^2&a
\end{bmatrix}
\begin{bmatrix}
V_a\\V_b\\V_c
\end{bmatrix},\quad
 a=e^{j2\pi/3}
\]

### Slide 11. Voltage unbalance factor
Define:

\[
\mathrm{VUF}=100\times \frac{|V_2|}{|V_1|}
\]

Teaching point: VUF is a phase-coupled quantity; it cannot be recovered from phase magnitudes alone.

---

## Section 3. pandapower three-phase modeling

### Slide 12. `runpp` vs `runpp_3ph`
- `runpp`: balanced AC power flow.
- `runpp_3ph`: asymmetric/unbalanced three-phase load flow.
- `runpp_3ph` solves in the sequence frame.
- Positive-sequence network uses Newton-Raphson; zero/negative sequence networks use current injection.

### Slide 13. Three-phase input tables
Core input tables for Week 2:

| Element | Table | Purpose |
|---|---|---|
| Bus | `net.bus` | Network nodes and nominal voltage |
| Line | `net.line` | Feeder sections and impedance data |
| Transformer | `net.trafo` | MV/LV transformer and vector group |
| Phase load | `net.asymmetric_load` | A/B/C phase-specific load |
| Phase sgen | `net.asymmetric_sgen` | A/B/C phase-specific PV/DER injection |
| External grid | `net.ext_grid` | Grid/PCC equivalent |

### Slide 14. `asymmetric_load`: consumer convention
- Use positive `p_a_mw`, `p_b_mw`, `p_c_mw` for consumption.
- Specify `q_a_mvar`, `q_b_mvar`, `q_c_mvar` by phase.
- Choose connection type: `wye` or `delta`.
- For constant generation, use `asymmetric_sgen` instead of a negative load.

### Slide 15. `asymmetric_sgen`: generator convention
- Use positive `p_a_mw`, `p_b_mw`, `p_c_mw` for generation.
- Good for single-phase PV or phase-specific DER injection.
- In `res_bus_3ph`, generator injection may appear as negative net demand at the bus.

### Slide 16. Wye and delta connection choice
- `wye`: phase-to-neutral modeling.
- `delta`: input powers are interpreted in line-line terms by the algorithm.
- Week 2 MVP: use `wye` first; introduce `delta` as an extension.

### Slide 17. Zero-sequence parameters are not optional for serious 3ph work
For lines:
- `r0_ohm_per_km`
- `x0_ohm_per_km`
- `c0_nf_per_km`

For transformers:
- vector group, e.g. `Dyn`
- `vk0_percent`, `vkr0_percent`, `mag0_percent`, `mag0_rx`, `si0_hv_partial`

Teaching point: a three-phase solver needs more than positive-sequence impedance.

### Slide 18. Transformer caveat for `runpp_3ph`
Recommended teaching boundary:
- Start with grounded transformer vector groups known to work well, such as `Dyn` or `Yyn`.
- Avoid exotic transformer configurations in Week 2.
- Make numerical robustness part of the model-building workflow.

---

## Section 4. Week 2 teaching network

### Slide 19. Minimal three-phase microgrid-like feeder
Topology:

```text
MV grid / PCC -- Dyn transformer -- LV PCC -- line 1 -- bus 1 -- line 2 -- bus 2
```

Devices:
- unbalanced household block at bus 1
- heavier phase-B load at bus 2
- single-phase PV on phase C at bus 2

### Slide 20. Why this minimal system is useful
It is small enough to inspect by hand but rich enough to show:
- unequal phase voltages,
- unequal line currents,
- VUF,
- generator/load sign conventions,
- balanced vs unbalanced cross-check.

### Slide 21. Lab workflow
1. Build standard line and transformer types.
2. Create bus, line, transformer, external grid.
3. Add asymmetric loads and PV.
4. Run `runpp_3ph`.
5. Extract phase results.
6. Run proof checks.
7. Compare against a balanced equivalent case.

---

## Section 5. Result interpretation

### Slide 22. `res_bus_3ph`
Important columns:
- `vm_a_pu`, `vm_b_pu`, `vm_c_pu`
- `va_a_degree`, `va_b_degree`, `va_c_degree`
- `p_a_mw`, `p_b_mw`, `p_c_mw`
- `q_a_mvar`, `q_b_mvar`, `q_c_mvar`
- `unbalance_percent`

### Slide 23. `res_line_3ph`
Important columns:
- phase powers at from/to ends,
- phase losses,
- `i_a_ka`, `i_b_ka`, `i_c_ka`,
- `i_n_ka`,
- `loading_a_percent`, `loading_b_percent`, `loading_c_percent`,
- `loading_percent`.

### Slide 24. `res_trafo_3ph` and `res_ext_grid_3ph`
Use these to check:
- per-phase grid import/export,
- transformer loading by phase,
- active/reactive losses,
- whether the PCC is importing power on all phases or exporting on one phase.

---

## Section 6. Proof and cross-validation

### Slide 25. Proof check 1: input sanity
Before trusting results:
- all bus references exist,
- all phase loads are nonnegative,
- line and transformer zero-sequence fields exist,
- exactly one external grid in the small teaching network,
- no critical element is out of service accidentally.

### Slide 26. Proof check 2: recompute VUF
Use result phasors and Fortescue transform to recompute:

\[
100\times\frac{|V_2|}{|V_1|}
\]

Then compare with `net.res_bus_3ph.unbalance_percent`.

### Slide 27. Proof check 3: recompute line loading
For each phase:

\[
\mathrm{loading}_{\ell,\phi}=100\times\frac{I_{\ell,\phi}}{I^{max}_{\ell}}
\]

Then:

\[
\mathrm{loading}_\ell=\max_\phi \mathrm{loading}_{\ell,\phi}
\]

### Slide 28. Proof check 4: system active-power balance
For total three-phase active power:

\[
P_{grid} + P_{sgen} \approx P_{load} + P_{line\ loss} + P_{trafo\ loss}
\]

This is a global check, not a substitute for node-level power-flow residuals.

### Slide 29. Cross-validation: balanced 3ph vs balanced `runpp`
Create a balanced version of the same feeder:
- equal phase loads,
- equal phase PV,
- same topology and parameters.

Then compare:
- mean phase voltage from `runpp_3ph`,
- balanced voltage from `runpp`,
- phase loading equality,
- line loading agreement.

### Slide 30. Unbalanced stress test
Increase one phase load or single-phase PV:
- Which phase voltage changes most?
- Does VUF increase?
- Which line phase current becomes limiting?

---

## Section 7. Link to research project

### Slide 31. What becomes AI features later?
From Week 2 outputs:
- phase voltage magnitudes and angles,
- phase line currents and loading,
- VUF,
- per-phase load and PV injection,
- transformer and feeder phase loading,
- mode indicators and topology features.

### Slide 32. What becomes labels later?
Week 4 labels will use:

\[
y(x,c)=1 \quad \text{if any post-contingency phase voltage, phase current, VUF, or convergence limit fails.}
\]

Week 2 teaches how to measure these quantities before adding contingencies.

### Slide 33. Student deliverables for Week 2
- A minimal three-phase pandapower network.
- A table of phase voltages, line currents, and VUF.
- Proof cells for VUF, line loading, and total active-power balance.
- A balanced-case cross-check against `runpp`.
- A short paragraph explaining why balanced results can hide phase-specific risk.

### Slide 34. Exit ticket questions
1. Why does `runpp_3ph` require zero-sequence parameters?
2. What does positive `p_a_mw` mean for `asymmetric_load`? What about `asymmetric_sgen`?
3. Why is VUF not recoverable from voltage magnitudes alone?
4. In line result tables, why should we check phase-level loading rather than only total loading?
5. What result quantities will become features for the AI model?

---

# Suggested lab timing

| Time | Activity |
|---:|---|
| 0--10 min | Recap Week 1 and introduce phase-aware motivation |
| 10--25 min | Three-phase notation and symmetrical components |
| 25--40 min | pandapower input/output tables |
| 40--60 min | Build and run minimal 3ph feeder |
| 60--80 min | Proof checks: VUF, loading, power balance |
| 80--100 min | Balanced-vs-unbalanced cross-validation |
| 100--120 min | Student exercises and discussion |

---

# References to cite in final slides

- pandapower `runpp_3ph` documentation: sequence-frame solution, positive-sequence NR, zero/negative-sequence current injection, and transformer caveats.
- pandapower `asymmetric_load` documentation: phase-wise load parameters, positive load convention, and wye/delta type.
- pandapower `asymmetric_sgen` documentation: phase-wise generator injection and positive generation convention.
- pandapower IEEE European LV asymmetric network documentation: optional follow-up dataset for Week 3.
