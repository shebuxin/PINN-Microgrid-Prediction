# Physics-informed Explainable Graph Learning for Fast Contingency Screening of Unbalanced Microgrids

**Prepared date:** 2026-07-09
**Working scope:** three-phase unbalanced microgrids, N-1 contingency screening, physics-informed graph learning, explainable AI, conservative deployment.

---

## Executive Summary

This project can be positioned as a **physics-informed, explainable, risk-aware graph learning framework** for fast contingency screening of unbalanced inverter-rich microgrids.

The key idea is not simply to train a GNN that classifies N-1 contingencies as safe or unsafe. A stronger journal-level story is:

> Build a fast screening model that predicts whether a three-phase unbalanced microgrid contingency is risky, explains **where** and **why** the risk appears at the bus-phase / branch-phase / device / sequence-component level, and uses conservative calibration so that only high-confidence safe contingencies are screened out while uncertain or risky cases are sent to exact three-phase power-flow verification.

The upgraded framework should combine five technical pillars:

1. **Signed phase-aware security margin** instead of only a binary violation label or non-negative severity.
2. **Contingency-conditioned phase-sequence heterogeneous graph** instead of a simple bus-level graph with three phase channels.
3. **Post-contingency physics-informed residual learning** instead of only soft limit penalties.
4. **Physically faithful explainable AI** that explains the contingency result at bus-phase, branch-phase, device, sequence, and event-pathway levels.
5. **Risk-controlled conservative screening** using conformal or selective calibration to reduce missed dangerous contingencies.

A concise target contribution statement is:

> We propose a physics-informed explainable graph learning framework for fast N-1 contingency screening of unbalanced microgrids. The framework learns signed three-phase security margins, localizes violations at phase-resolved network elements, generates physically validated explanations of unsafe or uncertain contingencies, and conservatively screens only calibrated safe cases to reduce exact three-phase power-flow calls.

---

## 1. Reframed Research Story

### 1.1 Background

Microgrids are not simply small balanced transmission systems. They usually contain photovoltaic generation, BESS, diesel generation, grid-forming or grid-following inverters, critical and non-critical loads, and a PCC that allows the microgrid to operate in either grid-connected or islanded mode. The U.S. DOE defines a microgrid as a group of interconnected loads and distributed energy resources within clear electrical boundaries that can operate as a controllable entity and can run in grid-connected or islanded mode. See DOE Microgrid Overview: [https://www.energy.gov/sites/default/files/2024-02/46060_DOE_GDO_Microgrid_Overview_Fact_Sheet_RELEASE_508.pdf](https://www.energy.gov/sites/default/files/2024-02/46060_DOE_GDO_Microgrid_Overview_Fact_Sheet_RELEASE_508.pdf).

For unbalanced microgrids, a contingency can trigger:

- phase-specific undervoltage or overvoltage;
- phase-specific branch or transformer overload;
- voltage unbalance factor (VUF) violation;
- negative-sequence or zero-sequence abnormality;
- DER or BESS reserve insufficiency;
- islanding infeasibility after PCC outage;
- power-flow non-convergence.

Therefore, fast screening must be **three-phase, phase-aware, topology-aware, device-aware, and explainable**.

### 1.2 Why Fast Screening Is Needed

A full N-1 scan requires running three-phase power flow for every operating point and every contingency:

$$
(x_t, c) \rightarrow \text{three-phase PF} \rightarrow \{V^c_{i,\phi}, I^c_{\ell,\phi}, VUF^c_i, \eta^c\}
$$

This is accurate but expensive when the scenario set includes time-varying PV, EV charging, load uncertainty, BESS dispatch, topology switching, DER trips, PCC outage, and islanded operating conditions.

The proposed AI model should therefore be framed as a **conservative pre-screening module**:

- It does **not** replace exact power flow.
- It screens out only cases that are confidently safe.
- It sends unsafe or uncertain cases to exact three-phase power-flow verification.
- It explains the physical reason for unsafe or uncertain decisions.

### 1.3 Upgraded Storyline

Original simplified storyline:

> Physics-informed GNN for fast N-1 security classification.

Upgraded journal-level storyline:

> Physics-informed explainable graph learning for fast, conservative, and physically interpretable contingency screening of unbalanced microgrids.

The upgraded model should output:

$$
\{\hat y, \hat g, \hat g^V_{i,\phi}, \hat g^I_{\ell,\phi}, \hat g^U_i, \hat g^{inv}, E(x,c), U(x,c)\}
$$

where:

- $\hat y$: predicted unsafe probability;
- $\hat g$: predicted global signed security margin;
- $\hat g^V_{i,\phi}$: bus-phase voltage margin;
- $\hat g^I_{\ell,\phi}$: branch-phase current/loading margin;
- $\hat g^U_i$: VUF margin;
- $\hat g^{inv}$: inverter/island feasibility margin;
- $E(x,c)$: explanation object;
- $U(x,c)$: calibrated conservative upper bound for screening.

---

## 2. State-of-the-Art Review

### 2.1 Traditional Static Security Assessment and Contingency Screening

Traditional static security assessment evaluates whether the post-contingency system violates operating limits after the outage of a component. Conventional N-1 screening often uses performance indices, linear sensitivities, or reduced models to prioritize severe contingencies before detailed AC power-flow analysis.

Representative review and background:

- Hailu et al. provide a broad review of static security assessment and improvement methods in modern power systems: [https://www.sciencedirect.com/science/article/pii/S2405844023017310](https://www.sciencedirect.com/science/article/pii/S2405844023017310).
- Classical and ML-based security assessment methods have long been studied, including decision trees, neural networks, and rule-based methods. A historical overview is Wehenkel's machine-learning framework for security assessment: [https://orbi.uliege.be/bitstream/2268/79719/2/LW-Machine_learning_approaches_to_power-system_security_assessment.pdf](https://orbi.uliege.be/bitstream/2268/79719/2/LW-Machine_learning_approaches_to_power-system_security_assessment.pdf).

**Gap for this project:** Most traditional screening methods are not designed for three-phase unbalanced microgrids with single-phase DERs, VUF constraints, PCC outage, and inverter capability constraints.

---

### 2.2 Deep Learning for N-1 Static Security Assessment

Deep learning has already been applied to N-1 static security assessment. For example, Qian et al. proposed a DCNN-based N-1 static security assessment method for renewable-rich power grids and used D2GAN to generate wind and solar scenarios: [https://www.sciencedirect.com/science/article/abs/pii/S0378779622004096](https://www.sciencedirect.com/science/article/abs/pii/S0378779622004096).

**Implication:** Simply saying "we use AI to accelerate N-1 security assessment" is not novel enough.

**Gap for this project:** Existing deep-learning N-1 work often focuses on balanced or transmission-level systems and does not provide phase-specific explanations or conservative risk-controlled screening for unbalanced microgrids.

---

### 2.3 GNNs for Power-System Security and Contingency Analysis

GNNs are naturally suitable for power systems because grids are graphs. Existing work has used GNNs for security assessment and contingency analysis.

Representative work:

- Dezvarei et al. formulate voltage static security assessment using a GNN framework that incorporates network topology and centrality measures: [https://arxiv.org/abs/2301.12988](https://arxiv.org/abs/2301.12988).
- Nakiganda et al. combine physics-informed neural networks with topology-aware architectures such as Guided Dropout and edge-varying GNNs for fast contingency analysis, including N-1, N-2, and N-3 cases: [https://arxiv.org/abs/2310.04213](https://arxiv.org/abs/2310.04213).
- GNN-based security assessment with interpretable feature attribution has recently been explored using gradient-based attribution to highlight critical buses, lines, and power-flow features: [https://www.energy-proceedings.org/graph-neural-networkaebased-security-assessment-for-power-grids-with-interpretable-feature-attribution/](https://www.energy-proceedings.org/graph-neural-networkaebased-security-assessment-for-power-grids-with-interpretable-feature-attribution/).

**Implication:** "GNN for contingency screening" is already a known direction.

**Gap for this project:** Existing GNN security-screening work is mostly positive-sequence, transmission-grid, or general topology-aware learning. It rarely addresses:

- unbalanced three-phase microgrid operation;
- phase-specific contingencies and violations;
- VUF / sequence-component risk;
- PCC outage and island feasibility;
- physically faithful explanations of the contingency mechanism.

---

### 2.4 Three-Phase Unbalanced Power Flow and GNNs

Unbalanced three-phase modeling is essential for distribution networks and microgrids.

Relevant resources:

- pandapower `runpp_3ph` performs unbalanced/asymmetric/three-phase load flow and uses a sequence-frame formulation: [pandapower 3.3.2 documentation](https://pandapower.readthedocs.io/en/v3.3.2/powerflow/ac_3ph.html).
- pandapower includes an IEEE European LV asymmetric network: a 0.416 kV network supplied by one 0.8 MVA MV/LV transformer, with 906 LV buses and 55 single-phase loads: [pandapower 3.3.2 network documentation](https://pandapower.readthedocs.io/en/v3.3.2/networks/3phase_grids.html).
- PowerFlowMultiNet proposes a multigraph GNN framework for unbalanced three-phase distribution systems, modeling phases separately to capture asymmetry: [https://arxiv.org/html/2403.00892v2](https://arxiv.org/html/2403.00892v2).
- Gardan et al. emphasize three-phase N-1 contingency analysis and VUF as a power-quality/security indicator: [https://www.mdpi.com/1996-1073/17/17/4429](https://www.mdpi.com/1996-1073/17/17/4429).

**Implication:** Three-phase GNNs and three-phase contingency analysis both exist, but they are not yet fully integrated into explainable, conservative, microgrid-specific N-1 screening.

**Gap for this project:** Existing unbalanced three-phase GNN work is closer to power-flow approximation than contingency screening and explanation. Existing three-phase contingency work is mostly simulation-based rather than AI-accelerated and explainable.

---

### 2.5 XAI in Energy and Power Systems

Explainable AI is increasingly important in power systems because operators need to trust AI outputs and understand why a model makes a decision.

Representative work:

- Machlev et al. review XAI techniques for energy and power systems and discuss challenges and opportunities: [https://www.sciencedirect.com/science/article/pii/S2666546822000246](https://www.sciencedirect.com/science/article/pii/S2666546822000246).
- PNNL's AI/ML report for power-system applications emphasizes the importance of transparency, trustworthiness, human interaction, and safe deployment of AI/ML in grid applications: [https://www.pnnl.gov/publications/artificial-intelligencemachine-learning-technology-power-system-applications-0](https://www.pnnl.gov/publications/artificial-intelligencemachine-learning-technology-power-system-applications-0).
- Recent power-grid GNN security assessment work has started to use gradient-based attribution to identify critical buses and transmission lines: [https://www.energy-proceedings.org/wp-content/uploads/icae2025/1770198481.pdf](https://www.energy-proceedings.org/wp-content/uploads/icae2025/1770198481.pdf).

**Implication:** XAI is a recognized requirement for power-system AI.

**Gap for this project:** Generic XAI methods usually explain features or graph nodes, but unbalanced microgrid contingency screening requires explanations at the level of:

- bus-phase;
- branch-phase;
- DER/BESS/GFM/PCC device;
- sequence component;
- contingency event;
- physical risk propagation path.

---

### 2.6 General GNN Explainability Methods

General graph explainability provides useful baselines.

Representative methods:

- **GNNExplainer:** identifies a compact subgraph and important node features by maximizing mutual information between the prediction and explanatory subgraph: [https://arxiv.org/abs/1903.03894](https://arxiv.org/abs/1903.03894).
- **PGExplainer:** learns a parameterized explainer that can generalize across multiple graph instances and inductive settings: [https://proceedings.neurips.cc/paper/2020/hash/e37b08dd3015330dcbb5d6663667b8b8-Abstract.html](https://proceedings.neurips.cc/paper/2020/hash/e37b08dd3015330dcbb5d6663667b8b8-Abstract.html).
- **Integrated Gradients:** provides an axiomatic feature-attribution method satisfying sensitivity and implementation invariance: [https://arxiv.org/abs/1703.01365](https://arxiv.org/abs/1703.01365).
- **Attention caution:** Jain and Wallace show that attention weights should not automatically be treated as faithful explanations: [https://aclanthology.org/N19-1357/](https://aclanthology.org/N19-1357/).

**Implication:** The proposed paper should not rely only on attention heatmaps. It should evaluate explanation fidelity quantitatively.

**Gap for this project:** General GNN explainers are domain-agnostic. They do not enforce three-phase power-flow physics, KCL residuals, branch-current consistency, sequence consistency, or inverter feasibility.

---

### 2.7 Trustworthy and Risk-Controlled Screening

For security screening, high recall and low false-negative rate are more important than ordinary classification accuracy. A model that misses dangerous contingencies is unacceptable.

Relevant trend:

- Conformal prediction and trustworthiness layers are being explored for power-system contingency screening to reduce missed detections and provide statistically calibrated confidence bounds: [https://arxiv.org/abs/2602.07995](https://arxiv.org/abs/2602.07995).

**Implication:** A journal-level screening paper should avoid relying only on heuristic threshold tuning.

**Gap for this project:** Existing conformal or trustworthy screening studies are not focused on unbalanced three-phase microgrids or physically explainable local phase-level risk.

---

## 3. Summary of Research Gaps

| Gap                                              | Current SOTA                                                              | Opportunity for This Paper                                                                                            |
| ------------------------------------------------ | ------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| G1: Balanced/transmission bias                   | Many ML/GNN contingency studies focus on balanced or transmission systems | Target three-phase unbalanced inverter-rich microgrids                                                                |
| G2: Three-phase GNN ≠ contingency screening     | Three-phase GNNs mainly approximate power flow                            | Use phase-sequence graph learning for N-1 screening                                                                   |
| G3: Limited microgrid modeling                   | PCC outage, islanding, GFM/BESS constraints often ignored                 | Include grid-connected/islanded mode, DER trip, PCC outage, inverter reserve                                          |
| G4: Binary labels lose margin information        | Many models predict safe/unsafe only                                      | Learn signed security margin and local margin maps                                                                    |
| G5: Physics-informed loss is often shallow       | Limit penalties or simple power balance are common                        | Use post-contingency KCL, branch-current, sequence, and inverter feasibility residuals                                |
| G6: Explanations are generic                     | SHAP/IG/GNNExplainer identify features or subgraphs                       | Explain bus-phase, branch-phase, DER/device, sequence component, and contingency pathway                              |
| G7: Explanations are rarely validated physically | Heatmaps are often qualitative                                            | Evaluate top-k violation localization, deletion/insertion fidelity, physical consistency, and counterfactual validity |
| G8: Screening confidence is heuristic            | Threshold tuning is common                                                | Use conformal/selective calibration for conservative screening                                                        |
| G9: Rare events and OOD are under-tested         | Random splits dominate                                                    | Use boundary-enriched sampling and OOD splits by scenario, topology, mode, and contingency type                       |

---

## 4. Research Questions

### RQ1: Fast Screening

Can a physics-informed phase-sequence graph model reduce exact three-phase contingency power-flow calls while maintaining very high recall of unsafe contingencies?

### RQ2: Three-Phase Generalization

Does explicitly modeling bus-phase nodes, branch-phase edges, sequence components, and contingency event nodes improve OOD generalization over MLPs, tabular models, and bus-level GNNs?

### RQ3: Physical Consistency

Do three-phase KCL, branch-current, sequence, and inverter feasibility residuals improve security-margin prediction and reduce severe false negatives?

### RQ4: Explainability

Can the model explain not only whether a contingency is unsafe, but also:

- which phase;
- which bus;
- which branch;
- which DER/BESS/PCC device;
- which sequence component;
- which physical pathway;

dominates the risk?

### RQ5: Conservative Deployment

Can conformal or selective calibration convert the model's output into a conservative safe/verify/unsafe workflow with controlled false-negative risk?

---

## 5. Proposed Framework

Suggested framework name:

> **PI-XGL: Physics-Informed Explainable Graph Learning**

Suggested model name:

> **X-C-Risk-PIGNN: Explainable Contingency-conditioned Risk-controlled Physics-informed Graph Neural Network**

The framework has four layers:

1. **Contingency-conditioned phase-sequence graph encoder**
2. **Signed-margin and local-risk prediction heads**
3. **Physics-informed residual learning**
4. **Explanation and conservative screening layer**

Overall pipeline:

$$
(x_t, c, \mathcal{G})
\rightarrow \text{PI-XGL}
\rightarrow
(\hat g, \hat y, \hat g^V, \hat g^I, \hat g^U, \hat E, U)
\rightarrow
\text{screen-safe / verify / unsafe}
$$

---

## 6. Problem Formulation

### 6.1 Three-Phase Microgrid Graph

Represent the microgrid as a three-phase heterogeneous graph:

$$
\mathcal{G}^c_t =
(\mathcal{V}^{bus-phase}, \mathcal{V}^{device}, \mathcal{V}^{sequence}, \mathcal{V}^{event},
\mathcal{E}^{phase}, \mathcal{E}^{coupling}, \mathcal{E}^{device}, \mathcal{E}^{event})
$$

Node types:

- $v^{bus}_{i,\phi}$: bus-phase node for bus $i$, phase $\phi \in \{a,b,c\}$;
- $v^{device}_d$: device node for PV, BESS, diesel, GFM inverter, transformer, PCC, critical load, switch;
- $v^{seq}_{i,k}$: sequence node for bus $i$, component $k \in \{0,1,2\}$;
- $v^{event}_c$: contingency event node.

Edge types:

- phase-domain electrical edges with admittance $Y^{\phi\psi}_{ij}$;
- phase-coupling edges representing mutual impedance/coupling;
- device-to-bus-phase edges;
- sequence-transform edges;
- event-to-affected-element edges.

### 6.2 Contingency Encoding

A contingency $c$ should not be encoded only as a one-hot vector. It should modify the graph:

- line outage: mask/remove affected branch-phase edges;
- transformer outage: mask transformer/device coupling;
- DER trip: mask device injection node;
- BESS trip: mask storage active/reactive capability;
- PCC outage: trigger grid-connected to islanded mode transition;
- phase-specific outage: mask selected phase edge or phase device.

### 6.3 Signed Security Margin

Replace non-negative severity-only learning with signed security margin learning.

Voltage margin:

$$
g_V(x,c)=
\max_{i,\phi}
\left\{
\frac{|V^c_{i,\phi}|-V_i^{max}}{V_i^{max}},
\frac{V_i^{min}-|V^c_{i,\phi}|}{V_i^{min}}
\right\}
$$

Branch/transformer current margin:

$$
g_I(x,c)=
\max_{\ell,\phi}
\left(
\frac{|I^c_{\ell,\phi}|}{I^{max}_{\ell,\phi}}-1
\right)
$$

VUF margin:

$$
g_U(x,c)=
\max_i
\left(
\frac{VUF^c_i}{VUF^{max}}-1
\right)
$$

Inverter/island feasibility margin:

$$
g_{inv}(x,c)=
\max_d
\left(
\frac{\sqrt{(P^c_d)^2+(Q^c_d)^2}}{S^{max}_d}-1
\right)
$$

Power-flow convergence / infeasibility margin:

$$
g_C(x,c)=
\begin{cases}
\gamma, & \text{if three-phase PF fails or island is infeasible} \\
-\gamma_0, & \text{otherwise}
\end{cases}
$$

Global signed margin:

$$
g(x,c)=\max\{g_V,g_I,g_U,g_{inv},g_C\}
$$

Security label:

$$
y(x,c)=\mathbb{I}(g(x,c)>0)
$$

Severity:

$$
s(x,c)=[g(x,c)]_+
$$

This formulation preserves both violation severity and safe operating margin.

---

## 7. Model Architecture

### 7.1 Encoder

A general message-passing layer can be written as:

$$
h^{(k+1)}_{i,\phi}
=
\operatorname{Update}
\left(
h^{(k)}_{i,\phi},
\sum_{j,\psi}
\alpha^{c,k}_{ij,\phi\psi}
\operatorname{Msg}
\left(h^{(k)}_{j,\psi}, e^c_{ij,\phi\psi}, z_c\right)
\right)
$$

where:

- $h_{i,\phi}$: bus-phase embedding;
- $e^c_{ij,\phi\psi}$: post-contingency edge feature;
- $z_c$: event embedding;
- $\alpha^{c,k}$: learned message coefficient, not treated as final explanation by itself.

### 7.2 Output Heads

The model should have multiple aligned outputs:

1. **Global screening head**

$$
\hat y = \sigma(\hat g/\tau)
$$

2. **Global signed-margin head**

$$
\hat g = f_g(\text{GraphReadout}(H,z_c))
$$

3. **Local voltage margin head**

$$
\hat g^V_{i,\phi}
$$

4. **Local branch-current margin head**

$$
\hat g^I_{\ell,\phi}
$$

5. **VUF / sequence margin head**

$$
\hat g^U_i
$$

6. **Inverter/islanding feasibility head**

$$
\hat g^{inv}, \hat g^C
$$

7. **Explanation head**

$$
\hat E(x,c)=
\{\pi^V_{i,\phi},\pi^I_{\ell,\phi},\pi^U_i,\pi^D_d,\pi^{seq}_{i,k},\pi^{event}\}
$$

where $\pi$ values are local explanation scores.

---

## 8. Physics-Informed Learning Design

### 8.1 Post-Contingency Residual Learning

Instead of predicting post-contingency voltage from scratch, predict the change from base case:

$$
\hat V^c_{i,\phi}=V^0_{i,\phi}+\Delta \hat V^c_{i,\phi}
$$

This makes the model learn how the contingency perturbs the operating state.

### 8.2 KCL Residual

Use post-contingency admittance $Y^c$:

$$
r^{\mathrm{KCL}}_{i,\phi}
=
\hat I^{\mathrm{inj},c}_{i,\phi}
-
\sum_{j,\psi}
Y^{c,\phi\psi}_{ij}\hat V^c_{j,\psi}
$$

$$
\mathcal{L}_{\mathrm{KCL}}
=
\sum_{i,\phi}
\left\|r^{\mathrm{KCL}}_{i,\phi}\right\|_2^2
$$

### 8.3 Branch Current Consistency

For branch \(i\rightarrow j\), a phase-coupled \(\pi\)-model gives:

$$
\hat I^c_{ij,\phi}
=
\sum_{\psi}
\left[Y^{\mathrm{series},c}_{ij}\right]_{\phi\psi}
\left(\hat V^c_{i,\psi}-\hat V^c_{j,\psi}\right)
+
\frac{1}{2}\sum_{\psi}
\left[Y^{\mathrm{sh},c}_{ij}\right]_{\phi\psi}
\hat V^c_{i,\psi}
$$

$$
\mathcal{L}_{\mathrm{branch}}
=
\sum_{\ell,\phi}
\left\|\hat I^c_{\ell,\phi}-I^{\mathrm{phys}}_{\ell,\phi}(\hat V^c)\right\|_1
$$

### 8.4 Sequence Consistency

Fortescue transform:

$$
\begin{bmatrix}
V_i^{(0)}\\
V_i^{(1)}\\
V_i^{(2)}
\end{bmatrix}
=
\underbrace{\frac{1}{3}
\begin{bmatrix}
1&1&1\\
1&a&a^2\\
1&a^2&a
\end{bmatrix}}_{T}
\begin{bmatrix}
V_i^a\\
V_i^b\\
V_i^c
\end{bmatrix},
\qquad a=e^{j2\pi/3}
$$

VUF:

$$
\mathrm{VUF}_i
=
\frac{|V_i^{(2)}|}{|V_i^{(1)}|+\epsilon},
\qquad
\mathrm{VUF}_{i,\%}=100\,\mathrm{VUF}_i
$$

Sequence loss:

$$
\mathcal{L}_{\mathrm{seq}}
=
\sum_i
\left\|
\begin{bmatrix}
\hat V_i^{(0)}\\
\hat V_i^{(1)}\\
\hat V_i^{(2)}
\end{bmatrix}
-
T
\begin{bmatrix}
\hat V_i^a\\
\hat V_i^b\\
\hat V_i^c
\end{bmatrix}
\right\|_2^2
$$

### 8.5 Inverter and Island Feasibility Residual

Inverter capability:

$$
(P_d^{inv})^2+(Q_d^{inv})^2 \le (S_d^{max})^2
$$

Island active-power adequacy:

$$
\sum_{d\in\mathrm{DER}} P_d
+
\sum_{b\in\mathrm{BESS}} P_b
+
\sum_{g\in\mathrm{GFM}} P_g
-
\sum_{\ell\in\mathrm{load}} P_\ell
-
P^{\mathrm{loss}}
\approx 0
$$

Reactive reserve adequacy:

$$
\sum Q^{capable}_{inv} - Q^{required}_{load/loss} \ge 0
$$

Feasibility residual:

$$
\mathcal{L}_{\mathrm{inv}}
=
\sum_d
\left[
\frac{\sqrt{P_d^2+Q_d^2}}{S^{\max}_d}-1
\right]^2_+
+
\lambda_{\mathrm{island}}
\left\|r^{\mathrm{island}}\right\|_2^2
$$

### 8.6 Total Prediction Loss

$$
\mathcal{L}_{\mathrm{total}}
=
\mathcal{L}_{\mathrm{BCE}}(y,\hat y)
+
\lambda_g\operatorname{Huber}(g,\hat g)
+
\lambda_s\left\|[g]_+-[\hat g]_+\right\|_1
+
\lambda_{\mathrm{rank}}\mathcal{L}_{\mathrm{rank}}
+
\lambda_{\mathrm{phys}}\mathcal{L}_{\mathrm{phys}}
$$

$$
\mathcal{L}_{\mathrm{phys}}
=
\mathcal{L}_{\mathrm{KCL}}
+
\mathcal{L}_{\mathrm{branch}}
+
\mathcal{L}_{\mathrm{seq}}
+
\mathcal{L}_{\mathrm{inv}}
$$

---

## 9. Explainable AI Design

### 9.1 What the Explanation Must Answer

The explanation should answer five questions.

#### Q1: What is the dominant risk type?

Examples:

- B-phase undervoltage;
- C-phase branch overload;
- VUF violation;
- islanding infeasibility;
- inverter reactive reserve insufficiency.

#### Q2: Where is the risk located?

Outputs:

$$
\pi^V_{i,\phi},\quad \pi^I_{\ell,\phi},\quad \pi^U_i,\quad \pi^D_d
$$

These explain:

- risky bus-phase;
- risky branch-phase;
- risky VUF bus;
- critical DER/BESS/GFM/PCC device.

#### Q3: Why does this contingency cause risk?

The explanation should form a physical chain:

$$
c
\rightarrow
\text{topology/device change}
\rightarrow
\Delta P,\Delta Q,\Delta I
\rightarrow
\Delta V^{abc}, \Delta V^{012}
\rightarrow
\text{security-limit violation}
$$

#### Q4: Why this contingency and not another?

Contrastive explanation:

$$
\Delta \hat g(c_1,c_2)
=
\hat g(x,c_1)-\hat g(x,c_2)
$$

Example interpretation:

> DER-3 trip is more dangerous than DER-4 trip because DER-3 is located on a weak B-phase downstream area and has less nearby reactive support.

#### Q5: What feasible change would make it safe?

Counterfactual explanation:

$$
\min_{\Delta u}
\|\Delta u\|_1
+
\lambda[\hat g(x+\Delta u,c)]_+
+
\lambda_{feas}\Omega(u+\Delta u)
$$

Possible actions:

- BESS reactive power support;
- PV inverter reactive support;
- non-critical load shedding;
- phase balancing;
- topology switching;
- GFM dispatch adjustment.

All counterfactual suggestions should be verified by exact three-phase power flow before being reported as valid.

---

### 9.2 Local-Margin Supervised Explanation

The strongest XAI design is to use power-flow ground truth to supervise local explanations.

Construct local margins:

$$
g^V_{i,\phi},\quad g^I_{\ell,\phi},\quad g^U_i,\quad g^{inv}_d
$$

Define dominant violation set:

$$
\mathcal{S}^*
=
\left\{r: g_r \ge \max_j g_j-\epsilon\right\}
$$

Train local explanation head:

$$
\mathcal{L}_{\mathrm{XAI}}
=
\sum_r
\operatorname{BCE}
\left(\pi_r,\mathbb{I}(r\in\mathcal{S}^*)\right)
$$

This makes the explanation aligned with actual three-phase power-flow violations, rather than only post-hoc saliency.

---

### 9.3 Physics-Guided Graph Attribution

Use a structured explainer mask:

$$
M=
\{
M^{bus}_{i,\phi},
M^{edge}_{\ell,\phi},
M^{device}_{d},
M^{seq}_{i,k},
M^{event}
\}
$$

Optimization:

$$
\mathcal{L}_{\mathrm{exp}}
=
\left|
f_\theta(\mathcal{G}^c)
-
f_\theta(\mathcal{G}^c\odot M)
\right|
+
\lambda_1\|M\|_1
+
\lambda_2 H(M)
+
\lambda_{\mathrm{phys}}\mathcal{R}_{\mathrm{phys}}(M)
$$

Physical explanation regularizer:

$$
\mathcal{R}_{\mathrm{phys}}(M)
=
-\operatorname{corr}(M^{\mathrm{bus}},|\Delta V|)
-\operatorname{corr}(M^{\mathrm{edge}},|\Delta I|)
-\operatorname{corr}(M^{\mathrm{seq}},\Delta \mathrm{VUF})
+
\lambda_{\mathrm{conn}}\mathcal{R}_{\mathrm{connectivity}}
$$

This encourages the explanation to be compact, faithful, and physically meaningful.

---

### 9.4 Attribution Baselines

Use multiple XAI baselines:

1. Rule-based explanation:

   - minimum voltage margin;
   - maximum loading;
   - maximum VUF;
   - downstream load;
   - electrical distance.
2. Tabular ML + SHAP:

   - XGBoost or Random Forest with SHAP feature importance.
3. GNN + Gradient × Input.
4. GNN + Integrated Gradients.
5. GNNExplainer.
6. PGExplainer.
7. GAT attention map as visualization only, not final evidence.
8. Proposed physics-guided phase-sequence explainer.

---

### 9.5 Explanation Outputs for Operators

Each contingency explanation should be rendered in an operator-friendly form:

```text
Decision: Verify / Unsafe
Dominant risk: B-phase undervoltage
Predicted signed margin: +0.023
Calibrated upper margin: +0.041
Critical bus-phase: Bus 42-B
Critical branch-phase: Line 17-B
Critical device: PV-3 trip / BESS-1 reserve insufficient
Physical pathway:
  DER trip -> reduced B-phase voltage support -> increased current on Line 17-B
  -> Bus 42-B voltage drops -> VUF increases at Bus 40-45 cluster
Recommended verification:
  Run exact three-phase PF for this contingency.
Counterfactual diagnostic:
  70 kvar BESS reactive support at B-phase may restore margin, pending PF validation.
```

---

## 10. Risk-Controlled Conservative Screening

### 10.1 Conformal Upper Margin

Use calibration data to quantify margin underestimation.

For each calibration sample $j$:

$$
a_j = g_j-\hat g_j
$$

Take the $(1-\alpha)$-quantile:

$$
q_{1-\alpha}=\text{Quantile}_{1-\alpha}(\{a_j\})
$$

For a new sample:

$$
U(x,c)=\hat g(x,c)+q_{1-\alpha}
$$

Screening rule:

$$
\text{screen-safe if } U(x,c)\le 0
$$

Otherwise:

$$
\text{send to exact three-phase PF}
$$

### 10.2 Three-Way Decision

Use three possible outputs:

| Decision        | Rule                                         | Action                                     |
| --------------- | -------------------------------------------- | ------------------------------------------ |
| Screen-safe     | $U(x,c)\le 0$                              | Skip exact PF                              |
| Verify          | $U(x,c)>0$ but model not strongly unsafe   | Run exact three-phase PF                   |
| Unsafe-priority | $\hat g(x,c)>0$ or high unsafe probability | Run exact PF first / prioritize mitigation |

### 10.3 Stratified Calibration

Use separate conformal quantiles by:

- grid-connected vs islanded mode;
- line outage vs DER trip vs PCC outage;
- voltage vs current vs VUF dominant risk;
- normal vs high-PV vs high-load vs high-unbalance regime.

This avoids over-conservative global calibration and improves PF-call saving.

---

## 11. Dataset Generation and Labeling Plan

### 11.1 Base Test System

MVP network:

- pandapower IEEE European LV asymmetric feeder;
- add PCC;
- add PV;
- add BESS;
- add GFM inverter or diesel generator;
- add critical and non-critical loads;
- add single-phase DER/load patterns.

Potential journal extension:

- one additional IEEE distribution feeder;
- several topology variants;
- reduced IEEE European LV feeder for ablation and fast experiments;
- cross-feeder transfer test.

### 11.2 Operating Scenarios

Generate scenarios:

1. Grid-connected normal load.
2. Islanded normal load.
3. High-PV low-load.
4. Low-PV high-load.
5. Strong phase imbalance.
6. EV charging concentrated on one phase.
7. BESS charging.
8. BESS discharging.
9. Weak reactive reserve.
10. High critical-load ratio.
11. Near-boundary voltage cases.
12. Near-boundary thermal cases.
13. Near-boundary VUF cases.

### 11.3 Contingency Set

MVP contingencies:

- line outage;
- transformer outage;
- PCC outage;
- PV trip;
- BESS trip;
- diesel/GFM trip;
- critical branch opening.

Advanced contingencies:

- phase-specific line outage;
- single-phase DER trip;
- switch reconfiguration;
- islanded pocket formation;
- N-1-1 scenario for extension.

### 11.4 Boundary-Enriched Sampling

Do not rely only on random Monte Carlo. Add rare-event sampling:

- scale load until voltage margin approaches zero;
- scale PV until overvoltage margin approaches zero;
- increase single-phase load imbalance until VUF margin approaches zero;
- reduce BESS/GFM reserve until island feasibility approaches zero;
- sample cases where $|g(x,c)|<\epsilon$.

### 11.5 Active Learning

Loop:

1. Train initial model.
2. Identify uncertain samples:
   $$
   |\hat g|<\epsilon \quad \text{or} \quad U(x,c)\approx 0
   $$
3. Run exact three-phase PF.
4. Add samples to training set.
5. Retrain or fine-tune.

This improves near-boundary screening performance.

### 11.6 Required Data Fields

| Field                          | Description                                            |
| ------------------------------ | ------------------------------------------------------ |
| `scenario_id`                | scenario index                                         |
| `mode`                       | grid-connected / islanded                              |
| `contingency_type`           | line / transformer / PCC / DER / BESS / phase-specific |
| `contingency_id`             | affected element ID                                    |
| `phase_mask`                 | affected phase(s)                                      |
| `base_V_abc`                 | base-case bus-phase voltages                           |
| `base_I_abc`                 | base-case branch-phase currents                        |
| `P_load_abc`, `Q_load_abc` | phase load                                             |
| `P_DER_abc`, `Q_DER_abc`   | DER injection                                          |
| `BESS_setpoint`, `SOC`     | BESS operating state                                   |
| `GFM_reserve`                | active/reactive reserve                                |
| `post_V_abc`                 | post-contingency voltages                              |
| `post_I_abc`                 | post-contingency currents                              |
| `post_V012`                  | sequence voltage                                       |
| `VUF`                        | voltage unbalance factor                               |
| `g_signed`                   | global signed security margin                          |
| `gV_bus_phase`               | local voltage margin                                   |
| `gI_branch_phase`            | local current margin                                   |
| `gU_bus`                     | local VUF margin                                       |
| `gInv_device`                | inverter reserve margin                                |
| `label`                      | safe/unsafe                                            |
| `dominant_violation_type`    | voltage/current/VUF/island/nonconverged                |
| `dominant_bus_phase`         | true critical bus-phase                                |
| `dominant_branch_phase`      | true critical branch-phase                             |
| `dominant_device`            | true critical device if applicable                     |
| `pf_converged`               | convergence flag                                       |
| `island_feasible`            | island feasibility flag                                |
| `counterfactual_verified`    | whether suggested counterfactual passed exact PF       |

---

## 12. Experimental Design

### 12.1 Prediction Baselines

| Model                              | Purpose                          |
| ---------------------------------- | -------------------------------- |
| Rule-based screening               | Engineering lower bound          |
| Logistic Regression                | Simple linear baseline           |
| Random Forest                      | robust tabular baseline          |
| XGBoost / LightGBM                 | strong tabular baseline          |
| MLP                                | data-driven neural baseline      |
| Physics-informed MLP               | tests physics loss without graph |
| Bus-level phase-channel GNN        | simple topology-aware baseline   |
| Phase-level GNN                    | tests phase nodes                |
| Phase-sequence GNN without physics | tests graph design               |
| Proposed PI-XGL without XAI        | prediction-only version          |
| Proposed PI-XGL with XAI           | full model                       |
| Proposed PI-XGL + conformal        | deployable screening version     |

### 12.2 XAI Baselines

| Explainer                         | Purpose                       |
| --------------------------------- | ----------------------------- |
| Rule-based margin ranking         | physical heuristic            |
| SHAP on XGBoost/RF                | tabular XAI baseline          |
| Gradient × Input                 | low-cost neural attribution   |
| Integrated Gradients              | axiomatic attribution         |
| GNNExplainer                      | post-hoc subgraph explanation |
| PGExplainer                       | inductive graph explanation   |
| Attention map                     | visualization only            |
| Proposed physics-guided explainer | main XAI method               |

### 12.3 Prediction Metrics

| Metric                         | Purpose                           |
| ------------------------------ | --------------------------------- |
| Recall of unsafe contingencies | safety-critical                   |
| False negative rate            | missed-danger metric              |
| AUPRC                          | imbalance-aware performance       |
| AUROC                          | general ranking                   |
| F1 / precision                 | false alarm analysis              |
| Top-k hit rate                 | severe contingency prioritization |
| Missed severity                | severity of false negatives       |
| Worst missed margin            | worst-case safety audit           |
| PF calls saved                 | computational value               |
| Runtime speedup                | deployment value                  |
| Calibration error              | probability reliability           |
| Conformal coverage             | risk-control validation           |
| OOD performance                | robustness                        |

### 12.4 Explanation Metrics

| Metric                           | Definition                                                    | Purpose                |
| -------------------------------- | ------------------------------------------------------------- | ---------------------- |
| Top-k bus-phase hit rate         | top-k explanation contains true critical bus-phase            | voltage localization   |
| Top-k branch-phase hit rate      | top-k explanation contains true critical branch-phase         | overload localization  |
| VUF localization accuracy        | top-k explanation contains max-VUF bus                        | unbalance localization |
| Violation-type accuracy          | predicted/explained risk type matches ground truth            | type explanation       |
| Deletion fidelity                | removing top-attribution elements changes prediction strongly | faithfulness           |
| Insertion fidelity               | retaining top-attribution elements preserves prediction       | sufficiency            |
| Physical consistency correlation | attribution correlates with $\Delta V$ or residual magnitude   | physical consistency   |
| Counterfactual validity          | counterfactual verified by exact PF                           | what-if validity       |
| Counterfactual sparsity          | number/size of required changes                               | actionability          |
| Explanation stability            | similar operating points produce similar explanations         | robustness             |
| OOD explanation robustness       | explanation quality under unseen regimes                      | deployment reliability |

### 12.5 OOD Splits

Use multiple OOD tests:

1. Random scenario split.
2. Unseen high-PV regime.
3. Unseen high-load regime.
4. Unseen strong phase imbalance.
5. Unseen contingency element.
6. Unseen contingency type.
7. Unseen topology/switch configuration.
8. Grid-connected to islanded transfer.
9. Islanded to grid-connected transfer.
10. Cross-feeder transfer if additional feeder is available.

### 12.6 Ablation Study

| Ablation                         | Question                                              |
| -------------------------------- | ----------------------------------------------------- |
| Without signed margin            | Is margin learning better than BCE-only?              |
| Without bus-phase nodes          | Are phase-resolved nodes needed?                      |
| Without sequence nodes           | Are VUF/negative-sequence explanations improved?      |
| Without event node               | Is contingency-conditioned message passing useful?    |
| Without graph surgery            | Is topology mutation better than one-hot contingency? |
| Without KCL residual             | Does KCL improve physical prediction?                 |
| Without branch-current residual  | Does current consistency reduce overload errors?      |
| Without inverter/island residual | Does microgrid-specific physics matter?               |
| Without local-margin supervision | Are explanations less accurate?                       |
| Without physics-guided explainer | Are generic explainers enough?                        |
| Without conformal calibration    | Is heuristic thresholding less safe?                  |
| Without active learning          | Are near-boundary errors worse?                       |

---

## 13. Case Studies for the Paper

### Case 1: Line Outage Causing B-Phase Undervoltage

Show:

- outage line;
- post-contingency voltage profile;
- top bus-phase attribution;
- risk propagation path;
- true vs predicted local voltage margin;
- optional counterfactual BESS/PV reactive support.

Expected conclusion:

> The proposed explainer identifies the downstream B-phase weak area and the overloaded transfer path, matching exact three-phase PF.

### Case 2: DER Trip Causing VUF Violation

Show:

- DER trip location;
- sequence-node attribution;
- negative-sequence voltage map;
- VUF margin map;
- phase-voltage imbalance.

Expected conclusion:

> Sequence-domain modeling is essential for explaining VUF-driven unsafe cases.

### Case 3: PCC Outage and Islanding Infeasibility

Show:

- PCC event node attribution;
- GFM/BESS reserve margin;
- load-generation mismatch;
- reactive reserve insufficiency;
- PF convergence/non-convergence label.

Expected conclusion:

> Device-aware graph nodes and inverter feasibility residuals are necessary for microgrid-specific contingency screening.

### Case 4: Conservative False Positive / Verify Case

Show:

- model predicts near-boundary safe but conformal upper margin is positive;
- exact PF confirms safe;
- explanation identifies small voltage margin at one bus-phase.

Expected conclusion:

> Conservative verification is acceptable because the framework prioritizes low false-negative risk.

### Case 5: Contrastive Contingency Explanation

Compare two contingencies:

$$
c_1 = \text{Line 12 outage}, \quad c_2 = \text{Line 16 outage}
$$

Explain why $c_1$ is unsafe but $c_2$ is safe:

- different downstream load;
- different BESS support;
- different electrical distance;
- different phase imbalance;
- different branch reserve.

---

## 14. Recommended Technical Contributions

Use 4–5 concise contributions in the paper.

### Contribution 1: Phase-Aware Signed Security Margin

We formulate unbalanced microgrid contingency screening as a signed phase-aware security-margin learning problem that jointly covers bus-phase voltage limits, branch-phase current limits, VUF, inverter reserve, island feasibility, and power-flow non-convergence.

### Contribution 2: Contingency-Conditioned Phase-Sequence Graph Learning

We develop a contingency-conditioned phase-sequence heterogeneous graph model with bus-phase nodes, branch-phase edges, device nodes, sequence nodes, and contingency event nodes, enabling explicit modeling of line outage, DER trip, PCC outage, islanded operation, and phase-specific failures.

### Contribution 3: Physics-Informed Post-Contingency Residual Learning

We introduce post-contingency residual learning with three-phase KCL, branch-current consistency, Fortescue sequence consistency, and inverter/island feasibility residuals.

### Contribution 4: Physically Faithful Explainable AI

We propose a physics-guided explanation module that localizes contingency risk at the bus-phase, branch-phase, device, sequence-component, and event-pathway levels, supervised by local security margins and validated using explanation fidelity metrics.

### Contribution 5: Risk-Controlled Conservative Screening

We integrate conformal or selective calibration to produce safe/verify/unsafe decisions, screening out only calibrated safe contingencies and sending uncertain or unsafe cases to exact three-phase PF.

---

## 15. Recommended Paper Structure

### Abstract

Emphasize:

- unbalanced microgrid N-1 screening;
- physics-informed explainable graph learning;
- signed security margin;
- bus-phase / branch-phase / device explanation;
- conservative screening and PF-call reduction.

### 1. Introduction

Include:

- microgrid three-phase unbalance problem;
- N-1 screening computational challenge;
- limitations of black-box AI;
- need for physically explainable contingency screening;
- summary of contributions.

### 2. Related Work

Subsections:

1. Static security assessment and contingency screening.
2. Data-driven and deep-learning-based security assessment.
3. GNNs for power-system contingency analysis.
4. Three-phase unbalanced power-flow learning.
5. Explainable AI for power systems.
6. Trustworthy and risk-controlled screening.

### 3. Three-Phase Microgrid Contingency Screening Problem

Include:

- graph representation;
- contingency set;
- signed security margin;
- local margin maps;
- screening objective.

### 4. Physics-Informed Explainable Graph Learning

Include:

- phase-sequence heterogeneous graph;
- contingency-conditioned graph mutation;
- GNN encoder;
- output heads;
- physics-informed losses;
- XAI module.

### 5. Conservative Screening and Explanation Workflow

Include:

- conformal upper bound;
- safe/verify/unsafe decision;
- explanation generation;
- counterfactual verification.

### 6. Dataset Generation and Experimental Setup

Include:

- pandapower test system;
- microgrid modification;
- scenario generation;
- contingencies;
- train/calibration/test/OOD splits;
- baselines.

### 7. Results

Include:

- prediction performance;
- PF-call savings;
- conformal risk-control results;
- OOD performance;
- ablation results.

### 8. Explanation Evaluation and Case Studies

Include:

- top-k localization;
- deletion/insertion fidelity;
- physical consistency;
- counterfactual validity;
- visual case studies.

### 9. Discussion

Include:

- deployment workflow;
- limitations;
- computational scalability;
- label noise;
- integration with EMS/microgrid controller.

### 10. Conclusion

Summarize the framework and future work.

---

## 16. Figures and Tables to Prepare

### Figures

1. Overall story diagram: exact three-phase PF vs PI-XGL conservative screening.
2. Phase-sequence heterogeneous graph representation.
3. Contingency-conditioned graph mutation examples.
4. Model architecture.
5. Signed security margin illustration.
6. Conservative screening decision boundary.
7. Explanation output dashboard.
8. Bus-phase voltage explanation heatmap.
9. Branch-phase overload explanation heatmap.
10. Sequence/VUF explanation heatmap.
11. Counterfactual explanation example.

### Tables

1. Literature comparison table.
2. Dataset statistics.
3. Contingency distribution and unsafe ratio.
4. Baseline prediction performance.
5. PF-call savings at fixed recall.
6. Conformal screening results.
7. OOD performance.
8. Explanation localization metrics.
9. Explanation fidelity metrics.
10. Ablation study.

---

## 17. MVP and Journal-Grade Versions

### 17.1 MVP Version

Goal: get a complete working pipeline.

Components:

- pandapower IEEE European LV asymmetric network;
- add PV/BESS/PCC/GFM/loads;
- generate line outage, DER trip, PCC outage;
- compute voltage/current/VUF/non-convergence labels;
- train XGBoost, MLP, PI-MLP, bus-level GNN;
- add signed margin;
- add local violation labels;
- add Gradient × Input or Integrated Gradients;
- evaluate recall, FNR, AUPRC, PF calls saved, top-k localization.

This version is enough for an initial conference-style paper or internal validation.

### 17.2 Journal-Grade Version

Add:

- phase-sequence heterogeneous GNN;
- event node and graph surgery;
- KCL + branch-current + sequence + inverter residuals;
- local-margin-supervised explanation;
- GNNExplainer/PGExplainer baselines;
- physics-guided explainer;
- conformal risk-controlled screening;
- active learning for near-boundary cases;
- OOD and cross-topology evaluation;
- contrastive and counterfactual explanations.

This version is more suitable for SCI journal submission.

---

## 18. Implementation Roadmap

### Stage 1: Simulation and Labels

- Build modified unbalanced microgrid in pandapower.
- Implement N-1 loop manually.
- Extract post-contingency $V^{abc}$, $I^{abc}$, VUF, convergence flag.
- Compute signed margins and local margins.
- Validate labels using sanity checks.

### Stage 2: Baselines

- Train rule-based methods.
- Train XGBoost/LightGBM.
- Train MLP and PI-MLP.
- Report screening metrics.

### Stage 3: Graph Model

- Implement bus-level phase-channel GNN.
- Implement phase-level graph.
- Add contingency event embedding.
- Add graph mutation for line/DER/PCC outage.

### Stage 4: Physics-Informed Learning

- Add residual voltage prediction.
- Add KCL residual.
- Add branch-current consistency.
- Add sequence consistency.
- Add inverter/island feasibility residual.

### Stage 5: Explainability

- Add local margin heads.
- Add local-margin supervised explanation loss.
- Add Gradient × Input and Integrated Gradients.
- Add structured graph mask explainer.
- Add physical consistency metrics.

### Stage 6: Conservative Screening

- Split training/calibration/test.
- Compute conformal margin upper bound.
- Evaluate PF calls saved at fixed missed-risk level.
- Add three-way decision: safe / verify / unsafe-priority.

### Stage 7: Paper Experiments

- Run full baselines.
- Run OOD tests.
- Run ablations.
- Produce case-study visualizations.
- Write discussion and limitations.

---

## 19. Risks and Mitigation

| Risk                                   | Impact                       | Mitigation                                                                 |
| -------------------------------------- | ---------------------------- | -------------------------------------------------------------------------- |
| Three-phase PF convergence problems    | noisy labels                 | separate non-convergence label; sanity check infeasible islands            |
| Class imbalance                        | high accuracy but low recall | focal/weighted loss, boundary sampling, active learning                    |
| Too many graph features                | model complexity             | start with MVP bus-level graph, then phase-level                           |
| Explanation not faithful               | weak XAI contribution        | use deletion/insertion, local-margin supervision, and physical consistency |
| Attention misinterpreted               | reviewer criticism           | use attention only as visualization, not evidence                          |
| Counterfactual suggestions infeasible  | unsafe claims                | verify every counterfactual with exact PF                                  |
| Single feeder limits generality        | reviewer concern             | use topology variants or additional feeder                                 |
| Conformal calibration too conservative | low PF-call saving           | stratified calibration by mode and contingency type                        |

---

## 20. Candidate Abstract Draft

**Physics-informed Explainable Graph Learning for Fast Contingency Screening of Unbalanced Microgrids**

Fast contingency screening is essential for operating inverter-rich microgrids under uncertain load, renewable generation, and topology conditions. However, conventional N-1 screening based on repeated three-phase power-flow simulations can become computationally expensive, while purely data-driven models often fail to provide physically interpretable and conservative decisions. This paper proposes a physics-informed explainable graph learning framework for fast contingency screening of unbalanced microgrids. The proposed framework represents the microgrid as a contingency-conditioned phase-sequence heterogeneous graph with bus-phase, branch-phase, device, sequence, and event nodes. It learns signed security margins covering phase voltage violations, branch-current overloads, voltage unbalance, inverter reserve limits, islanding infeasibility, and power-flow non-convergence. Physics-informed residual losses are introduced to enforce post-contingency three-phase KCL, branch-current consistency, sequence consistency, and inverter feasibility. To support operator trust, the framework generates physically validated explanations that localize risk at the bus-phase, branch-phase, device, sequence-component, and contingency-pathway levels. A conformal calibration layer further converts predicted margins into conservative safe/verify/unsafe decisions, screening out only high-confidence safe contingencies while sending uncertain cases to exact three-phase power-flow verification. Experiments on modified unbalanced microgrid feeders demonstrate that the proposed method can substantially reduce three-phase contingency power-flow calls while maintaining high unsafe-contingency recall and producing faithful explanations of critical contingency mechanisms.

---

## 21. Short Contribution Version for Introduction

This paper makes the following contributions:

1. We formulate unbalanced microgrid contingency screening as a signed phase-aware security-margin learning problem covering voltage, current, VUF, inverter reserve, islanding infeasibility, and non-convergence risks.
2. We propose a contingency-conditioned phase-sequence heterogeneous graph learning model that explicitly represents bus-phase nodes, branch-phase edges, device nodes, sequence nodes, and event nodes.
3. We introduce post-contingency physics-informed residual learning based on three-phase KCL, branch-current consistency, sequence consistency, and inverter/island feasibility.
4. We develop a physically faithful XAI module that explains contingency outcomes at the bus-phase, branch-phase, device, sequence-component, and physical-pathway levels.
5. We integrate conformal conservative screening to reduce exact three-phase power-flow calls while controlling missed unsafe contingencies.

---

## 22. Key References

1. U.S. DOE, **Microgrid Overview**, 2024.[https://www.energy.gov/sites/default/files/2024-02/46060_DOE_GDO_Microgrid_Overview_Fact_Sheet_RELEASE_508.pdf](https://www.energy.gov/sites/default/files/2024-02/46060_DOE_GDO_Microgrid_Overview_Fact_Sheet_RELEASE_508.pdf)
2. pandapower 3.3.2 documentation, **Asymmetric / Three Phase Power Flow**. [https://pandapower.readthedocs.io/en/v3.3.2/powerflow/ac_3ph.html](https://pandapower.readthedocs.io/en/v3.3.2/powerflow/ac_3ph.html)
3. pandapower 3.3.2 documentation, **3-Phase Grid Data**. [https://pandapower.readthedocs.io/en/v3.3.2/networks/3phase_grids.html](https://pandapower.readthedocs.io/en/v3.3.2/networks/3phase_grids.html)
4. E. A. Hailu et al., **Techniques of power system static security assessment and improvement: A comprehensive review**, Heliyon, 2023.[https://www.sciencedirect.com/science/article/pii/S2405844023017310](https://www.sciencedirect.com/science/article/pii/S2405844023017310)
5. T. Qian et al., **N-1 static security assessment method for power grids with high penetration rate of renewable energy generation**, Electric Power Systems Research, 2022.[https://www.sciencedirect.com/science/article/abs/pii/S0378779622004096](https://www.sciencedirect.com/science/article/abs/pii/S0378779622004096)
6. M. Dezvarei et al., **Graph Neural Network Framework for Security Assessment Informed by Topological Measures**, 2023.[https://arxiv.org/abs/2301.12988](https://arxiv.org/abs/2301.12988)
7. A. M. Nakiganda et al., **Topology-Aware Neural Networks for Fast Contingency Analysis of Power Systems**, 2023.[https://arxiv.org/abs/2310.04213](https://arxiv.org/abs/2310.04213)
8. S. Ghamizi et al., **PowerFlowMultiNet: Multigraph Neural Networks for Unbalanced Three-Phase Distribution Systems**, IEEE Transactions on Power Systems, 2025.[https://arxiv.org/html/2403.00892v2](https://arxiv.org/html/2403.00892v2)
9. G. Gardan et al., **Assessing the Static Security of the Italian Grid by Means of the N-1 Three-Phase Contingency Analysis**, Energies, 2024.[https://www.mdpi.com/1996-1073/17/17/4429](https://www.mdpi.com/1996-1073/17/17/4429)
10. R. Machlev et al., **Explainable Artificial Intelligence (XAI) techniques for energy and power systems: Review, challenges and opportunities**, Energy and AI, 2022.[https://www.sciencedirect.com/science/article/pii/S2666546822000246](https://www.sciencedirect.com/science/article/pii/S2666546822000246)
11. Y. Chen et al., **Artificial Intelligence/Machine Learning Technology in Power System Applications**, PNNL, 2024.[https://www.pnnl.gov/publications/artificial-intelligencemachine-learning-technology-power-system-applications-0](https://www.pnnl.gov/publications/artificial-intelligencemachine-learning-technology-power-system-applications-0)
12. R. Ying et al., **GNNExplainer: Generating Explanations for Graph Neural Networks**, NeurIPS, 2019.[https://arxiv.org/abs/1903.03894](https://arxiv.org/abs/1903.03894)
13. D. Luo et al., **Parameterized Explainer for Graph Neural Network**, NeurIPS, 2020.[https://proceedings.neurips.cc/paper/2020/hash/e37b08dd3015330dcbb5d6663667b8b8-Abstract.html](https://proceedings.neurips.cc/paper/2020/hash/e37b08dd3015330dcbb5d6663667b8b8-Abstract.html)
14. M. Sundararajan et al., **Axiomatic Attribution for Deep Networks**, ICML, 2017.[https://arxiv.org/abs/1703.01365](https://arxiv.org/abs/1703.01365)
15. S. Jain and B. C. Wallace, **Attention is not Explanation**, NAACL, 2019.[https://aclanthology.org/N19-1357/](https://aclanthology.org/N19-1357/)
16. Y. Dai et al., **Graph Neural Network-Based Security Assessment for Power Grids with Interpretable Feature Attribution**, Energy Proceedings, 2026.[https://www.energy-proceedings.org/graph-neural-networkaebased-security-assessment-for-power-grids-with-interpretable-feature-attribution/](https://www.energy-proceedings.org/graph-neural-networkaebased-security-assessment-for-power-grids-with-interpretable-feature-attribution/)
17. A. Alcántara and S. Chatzivasileiadis, **Trustworthiness Layer for Foundation Models in Power Systems: Application for N-k Contingency Assessment**, 2026.
    [https://arxiv.org/abs/2602.07995](https://arxiv.org/abs/2602.07995)

---

## 23. Final Recommended Positioning

The strongest positioning is:

> This paper does not merely accelerate N-1 screening with a GNN. It builds a physically grounded and operator-facing screening framework for unbalanced microgrids. The model learns signed three-phase security margins, explains the contingency mechanism at phase/device/pathway level, and conservatively decides which cases can be skipped and which must be verified by exact three-phase power flow.

This positioning gives the paper a clearer SCI-journal identity than a standard "PI-GNN for security classification" paper.
