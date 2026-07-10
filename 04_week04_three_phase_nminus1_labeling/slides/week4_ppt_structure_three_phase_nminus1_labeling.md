
# Week 4 PPT Structure

## Week 4: 三相 N-1 扫描与 Violation Label 设计

**课程定位：** Week 1–3 已经完成 microgrid 基础、三相不平衡潮流和 operating scenario generation。Week 4 的任务是把每个 operating scenario \(x_t\) 与 contingency \(c\) 组合成监督学习样本：

\[
(x_t,c) \longrightarrow y_{t,c},\; s_{t,c}
\]

其中 \(y_{t,c}\) 是 violation label，\(s_{t,c}\) 是 severity score。

---

## Slide 1 — Title

**三相 N-1 扫描与 Violation Label 设计**

Subtitle:

```text
From operating scenarios to AI-ready security labels
```

Visual idea:

```text
Week 3: x_t  →  Week 4: (x_t, c)  →  Week 5: ML baseline
```

---

## Slide 2 — Learning objectives

学生完成 Week 4 后应能：

1. 定义 microgrid / LV feeder 中的 N-1 contingencies；
2. 使用 `pandapower.runpp_3ph()` 手写三相 N-1 scanning loop；
3. 提取 post-contingency 电压、电流、loading、VUF、convergence 和 service-loss metrics；
4. 把物理指标转换为 binary label 和 severity score；
5. 验证每次 contingency 后网络状态被正确恢复；
6. 导出 Week 5 机器学习训练需要的数据集。

---

## Slide 3 — Week 1–4 pipeline recap

```text
Week 1: balanced pandapower intro
Week 2: three-phase unbalanced runpp_3ph
Week 3: microgrid scenario generation
Week 4: N-1 scan + label generation
Week 5: ML baselines
Week 6: physics-informed MLP
Week 7: phase-aware GNN
Week 8: paper and presentation
```

Key message:

```text
Week 4 is the bridge between physical simulation and supervised learning.
```

---

## Slide 4 — What is one supervised sample?

Each sample is:

\[
(x_t, c)
\]

where:

```text
x_t = base-case microgrid operating scenario
c   = one N-1 contingency
```

The post-contingency power flow gives:

\[
y_{t,c}=\mathbb{I}(\text{security violation})
\]

and:

\[
s_{t,c}=\text{severity score}
\]

---

## Slide 5 — Why not simply use built-in contingency analysis?

pandapower 的 contingency analysis 可以定义 line、trafo、trafo3w 的 N-1 case，并逐个切除设备收集 min/max 结果。

本课程仍然手写 loop，因为 microgrid 三相 AI label 需要：

```text
1. runpp_3ph 三相结果表
2. DER / PV / BESS / PCC trip
3. 径向 feeder downstream service-loss detection
4. 自定义 VUF / voltage / loading / non-convergence label
5. 每个 contingency 后的状态恢复 proof
```

---

## Slide 6 — Microgrid contingency types

MVP contingency list:

```text
1. line_outage
2. pv_trip
3. bess_trip
4. pcc_outage
```

Research extensions:

```text
5. transformer outage
6. diesel generator trip
7. normally-open switch failure
8. phase-specific conductor outage
9. islanded-source outage
```

Teaching point:

```text
A contingency is an intervention on the network state, not just an ID.
```

---

## Slide 7 — Post-contingency physical outputs

After applying contingency \(c\) and running three-phase power flow:

\[
V_{i,\phi}^{c},\quad I_{\ell,\phi}^{c},\quad \mathrm{loading}_{\ell,\phi}^{c},\quad \mathrm{VUF}_i^c
\]

where:

\[
i\in\mathcal{N},\quad \ell\in\mathcal{E},\quad \phi\in\{a,b,c\}
\]

Also record:

```text
converged flag
unserved load
critical unserved load
worst bus / worst line / worst phase
```

---

## Slide 8 — Violation type 1: phase-voltage violation

Low-voltage severity:

\[
s_{V^-}=\left[\frac{V^{min}-\min_{i,\phi}|V_{i,\phi}|}{V^{min}}\right]_+
\]

High-voltage severity:

\[
s_{V^+}=\left[\frac{\max_{i,\phi}|V_{i,\phi}|-V^{max}}{V^{max}}\right]_+
\]

Teaching thresholds:

```text
Vmin = 0.95 pu
Vmax = 1.05 pu
```

---

## Slide 9 — Violation type 2: phase-current / line-loading violation

Use per-phase loading when available:

```text
loading_a_percent
loading_b_percent
loading_c_percent
```

Severity:

\[
s_I=\left[\frac{\max_{\ell,\phi}\mathrm{loading}_{\ell,\phi}}{100}-1\right]_+
\]

Manual proof formula:

\[
\mathrm{loading}_{\ell,\phi}=100\frac{|I_{\ell,\phi}|}{I_{\ell}^{max}df_\ell n_{\ell,parallel}}
\]

---

## Slide 10 — Violation type 3: voltage unbalance violation

Voltage unbalance factor:

\[
\mathrm{VUF}_i=100\frac{|V_i^{(2)}|}{|V_i^{(1)}|}
\]

Severity:

\[
s_U=\left[\frac{\max_i\mathrm{VUF}_i}{\mathrm{VUF}^{max}}-1\right]_+
\]

Teaching threshold:

```text
VUFmax = 2 percent (teaching threshold used consistently in Weeks 2, 4, and 6)
```

---

## Slide 11 — Violation type 4: non-convergence

If `runpp_3ph()` fails or no in-service slack source exists:

```text
non_convergence_violation = True
```

Severity contribution:

\[
s_C=1
\]

Important distinction:

```text
Non-convergence is not always a numerical bug.
It may indicate infeasible topology or no source after N-1.
```

---

## Slide 12 — Violation type 5: service loss

In a radial microgrid, a line outage can disconnect downstream loads.

Even if the connected component solves, disconnected load must be labeled unsafe.

Reachable set from PCC:

\[
\mathcal{C}_{PCC}
\]

Unserved load:

\[
P_{unserved}=\sum_{i\notin \mathcal{C}_{PCC}}P_i
\]

Label:

\[
\mathbb{I}(P_{unserved}>\epsilon)
\]

---

## Slide 13 — Overall severity score

Component severities:

```text
s_v_low
s_v_high
s_loading
s_vuf
s_nonconv
s_service
```

Overall:

\[
s(x_t,c)=\max\{s_{V^-},s_{V^+},s_I,s_U,s_C,s_S\}
\]

Binary label:

\[
y(x_t,c)=\mathbb{I}(s(x_t,c)>0)
\]

---

## Slide 14 — Why severity matters

Binary classification answers:

```text
Is this contingency unsafe?
```

Severity answers:

```text
How unsafe is it?
```

Research use:

```text
1. top-k risky contingency ranking
2. high-recall threshold tuning
3. screening vs full three-phase AC PF tradeoff
4. auxiliary regression loss in physics-informed AI
```

---

## Slide 15 — Dataset schema

Each row should include:

```text
scenario_id
contingency_id
contingency_type
element_table
element_index
element_name
base-case scenario fields
post-contingency metrics
violation flags
violation_label
severity_score
```

Principle:

```text
Keep raw physical metrics and derived labels both in the dataset.
```

---

## Slide 16 — Custom N-1 scanner pseudo-code

```python
for scenario in scenarios:
    reset_network_to_base()
    apply_scenario(scenario)
    signature = input_signature(net)

    for contingency in contingencies:
        apply_contingency(contingency)
        service_loss = detect_unserved_load(net)
        converged = runpp_3ph(net)
        metrics = extract_metrics(net)
        label = make_label(metrics, service_loss, converged)
        save_row(scenario, contingency, metrics, label)

        reset_network_to_base()
        apply_scenario(scenario)
        assert input_signature(net) == signature
```

---

## Slide 17 — Proof 1: state restoration

Why it matters:

```text
If one outage is not restored, every later sample can be corrupted.
```

Proof idea:

\[
\mathrm{signature}(x_t)=\mathrm{signature}(\mathrm{restore}(\mathrm{apply}(x_t,c)))
\]

Signature includes:

```text
line in_service
ext_grid in_service
asymmetric_load P/Q by phase
asymmetric_sgen P/Q by phase
```

---

## Slide 18 — Proof 2: service-loss detection

Build graph from in-service lines.

```python
G = graph(in_service_lines)
reachable = connected_component(G, pcc_bus)
unserved = loads on buses not in reachable
```

This proof catches radial feeder service interruption, even when voltage/current limits do not trigger.

---

## Slide 19 — Proof 3: label consistency

Check:

\[
y = \mathrm{OR}(\text{all violation flags})
\]

and:

```text
if y == False, severity_score == 0
if y == True, severity_score > 0
```

This prevents contradictory rows in the AI dataset.

---

## Slide 20 — Proof 4: reproducibility

Run the scanner twice:

```text
scan_result_1 == scan_result_2
```

Compare stable columns:

```text
scenario_id
contingency_id
converged
unserved_load_mw
min_vm_pu
max_vuf_percent
max_line_loading_percent
violation_label
severity_score
```

---

## Slide 21 — Expected outputs

Week 4 notebook exports:

```text
week4_contingency_table.csv
week4_n1_dataset.csv
week4_label_distribution.csv
week4_top_severe_contingencies.csv
week4_validation_summary.csv
```

Figures:

```text
label distribution by contingency type
severity distribution
top severe contingencies
```

---

## Slide 22 — Manual inspection examples

Ask students to inspect:

```text
1. safe PV trip
2. line outage with service loss
3. PCC outage with no slack source
4. stress scenario with high loading or low voltage
```

For each row:

```text
Which physical metric triggered the label?
```

---

## Slide 23 — Common mistakes

```text
1. forgetting to reset in_service flags
2. ignoring isolated downstream loads
3. treating non-convergence as missing data instead of a label
4. mixing base-case and post-contingency metrics
5. optimizing accuracy instead of recall
6. dropping contingency metadata
```

---

## Slide 24 — Bridge to Week 5

Week 4 gives:

\[
\mathcal{D}=\{x_t,c,y_{t,c},s_{t,c}\}
\]

Week 5 trains:

```text
Logistic Regression
Random Forest
Gradient Boosting
MLP baseline
```

Core question:

```text
Can base-case features + contingency features predict N-1 violation risk?
```

---

## Slide 25 — Student deliverables

Students submit:

1. `week4_n1_dataset.csv`
2. label distribution table
3. top-10 most severe contingencies
4. validation summary with all tests passed
5. one paragraph explaining why service-loss detection is required in radial microgrids

---

## Slide 26 — Optional research extensions

```text
1. transformer outage
2. tie-switch reconfiguration
3. islanded source model instead of PCC outage only
4. probabilistic scenario table
5. phase-specific line/conductor outage
6. full IEEE European LV feeder reduction
```

---

## Slide 27 — References for implementation

Primary implementation references:

- pandapower `runpp_3ph()` documentation;
- pandapower contingency analysis documentation;
- pandapower asymmetric load / asymmetric static generator documentation.

---

## Slide 28 — Closing message

```text
Week 4 turns power-system physics into supervised-learning labels.
A trustworthy AI paper starts with a trustworthy label-generation pipeline.
```
