
# Week 8 PPT 结构：论文组织、结果复现与项目展示

**课程主题**：From experiments to a reproducible mini-paper and presentation  
**项目主线**：pandapower 三相不平衡微电网 N-1 安全筛查 + Physics-informed AI + Phase-aware GNN

---

## Week 8 学习目标

学生完成本周后应能：

1. 把 Weeks 3--7 的输出整合成一套可复现的 research package；
2. 将技术结果组织成论文的 problem formulation、method、experiment 和 limitation；
3. 选择合适的表格和图来支撑论文 claims；
4. 写出 5--8 页 mini-paper 的结构化初稿；
5. 生成 10--15 页答辩 PPT 的故事线；
6. 用 checklist 和 artifact manifest 证明实验可复现、无明显数据泄漏、指标计算一致。

---

# Part I. 课程回顾与论文故事线

## Slide 1. Title

**标题**：Week 8: 论文组织、结果复现与项目展示  
**副标题**：From three-phase N-1 experiments to a reproducible mini-paper

视觉建议：一条从 `scenario generation -> N-1 labeling -> ML -> PI-MLP -> GNN -> paper` 的横向 pipeline。

---

## Slide 2. 八周课程闭环

内容：

- Week 1: pandapower 和 microgrid 静态安全概念；
- Week 2: 三相不平衡潮流；
- Week 3: microgrid scenario generation；
- Week 4: 三相 N-1 label；
- Week 5: ML baseline 和 high-recall screening；
- Week 6: Physics-informed MLP；
- Week 7: Phase-aware GNN；
- Week 8: paper + reproducibility package。

讲解重点：Week 8 不是增加新模型，而是把已有工作变成科研成果。

---

## Slide 3. 最终科研故事线

核心叙事：

> 三相不平衡微电网的 N-1 安全评估需要大量 `runpp_3ph()` 扫描。我们用 pandapower 生成可复现 ground truth，用 ML / PI-MLP / GNN 学习 contingency risk，并以 high-recall screening 减少需要精算的三相潮流调用。

要强调：AI 是 **pre-screening module**，不是直接替代保护、调度或运行员判断。

---

## Slide 4. 论文的核心问题

数学表达：

\[
(x_t, c) \mapsto \Pr(y_{t,c}=1), \quad s_{t,c}
\]

其中：

- \(x_t\)：base-case 三相微电网状态；
- \(c\)：N-1 contingency；
- \(y_{t,c}\)：是否出现静态安全 violation；
- \(s_{t,c}\)：violation severity。

---

## Slide 5. 目标不是 accuracy，而是安全筛查

内容：

- high accuracy 可能掩盖 missed violations；
- false negative 是最危险错误；
- threshold 应在 validation set 上选择；
- test set 只用于最终报告。

关键指标：

\[
\mathrm{Recall}=\frac{TP}{TP+FN},\quad
\mathrm{FNR}=\frac{FN}{TP+FN}
\]

\[
\mathrm{PF\ calls\ saved}=\frac{TN+FN}{N}
\]

---

# Part II. 论文结构

## Slide 6. Mini-paper 推荐结构

建议 5--8 页：

1. Introduction
2. Three-phase microgrid security formulation
3. Dataset generation with pandapower
4. ML / PI-MLP / GNN screening methods
5. Experimental setup
6. Results and ablation
7. Discussion and limitations
8. Conclusion

---

## Slide 7. Introduction 怎么写

推荐四段式：

1. microgrid / LV feeder 三相不平衡的重要性；
2. N-1 security screening 的计算挑战；
3. AI surrogate 的机会和风险；
4. 本文贡献。

贡献示例：

- 三相不平衡微电网 N-1 label pipeline；
- phase-aware violation definition；
- PI-MLP / GNN high-recall screening；
- reproducible validation suite。

---

## Slide 8. Problem formulation 怎么写

内容包括：

- base-case state \(x_t\)；
- contingency catalog \(\mathcal{C}\)；
- three-phase post-contingency quantities；
- voltage / loading / VUF / service-loss / non-convergence label；
- screening objective。

视觉建议：左边为微电网图，右边为 label 公式。

---

## Slide 9. Dataset generation section

需要说明：

- teaching feeder / microgrid proxy；
- scenario table；
- PV / BESS / phase imbalance settings；
- N-1 contingency catalog；
- `runpp_3ph()` ground truth；
- proof and validation suite。

对应 Week 3 + Week 4。

---

## Slide 10. Method section: 三层模型

模型层次：

1. engineering rule / traditional ML baseline；
2. PI-MLP with multi-task and limit-aware losses；
3. bus-level phase-channel GNN。

重点：模型逐步增强，不要只展示最终模型。

---

## Slide 11. PI-MLP loss 写法

\[
\mathcal{L}
=
\mathcal{L}_{BCE}
+\lambda_s\mathcal{L}_{sev}
+\lambda_m\mathcal{L}_{margin}
+\lambda_c\mathcal{L}_{consistency}
+\lambda_r\mathcal{L}_{rank}
\]

讲解：physics-informed 在本课程中主要体现为 **limit-aware component margins** 和 **severity consistency**，不是声称严格满足 AC 方程约束。

---

## Slide 12. GNN section 怎么写

写清：

- node features：bus-level phase channels；
- edge features：line parameters and outage mask；
- global features：scenario and contingency metadata；
- graph-level output：risk score；
- permutation-invariance validation。

---

# Part III. 实验结果组织

## Slide 13. Table I: Dataset statistics

建议包含：

- number of scenarios；
- number of contingencies；
- number of samples；
- safe / unsafe；
- violation type distribution；
- solved / service-loss / no-slack rows。

---

## Slide 14. Table II: Holdout model comparison

建议列：

- model；
- recall；
- precision；
- FNR；
- PF calls saved；
- missed violations；
- AUPRC / AUROC。

讲解：优先按 recall 和 missed violations 排序。

---

## Slide 15. Table III: OOD-style group CV

两类 group split：

- scenario-group CV：测试 unseen operating scenario；
- contingency-group CV：测试 unseen contingency。

讲解：random split 不是最终可信证据。

---

## Slide 16. Table IV: Ablation study

建议展示 Week 6：

- BaselineMLP；
- PI-MLP full；
- 去掉 severity loss；
- 去掉 component margin；
- 去掉 ranking loss。

讲解：ablation 目的是证明每个 loss term 的作用，而不是追求所有指标最好。

---

## Slide 17. Figure 1: Pipeline figure

一张图讲清：

`scenario generation -> runpp_3ph N-1 -> labels -> models -> screening -> validation`

建议作为论文 Figure 1 和答辩第二页。

---

## Slide 18. Figure 2: Label distribution

图源：Week 4 label distribution / severity distribution。

讲解：数据是否类别不平衡、哪些 contingency 最危险。

---

## Slide 19. Figure 3: Precision-recall curves

图源：Week 5/6/7 PR curves。

讲解：PR 曲线比 ROC 更适合不平衡安全筛查。

---

## Slide 20. Figure 4: Saved PF calls vs missed violations

图源：Week 5/6/7 screening tradeoff。

讲解：这是论文最重要的 deployment figure。

---

## Slide 21. Figure 5: Top-k contingency screening

图源：Week 7 top-k violation recall。

讲解：如果运行员只想精算前 k 个 contingency，模型能否把危险 contingency 排在前面？

---

## Slide 22. Figure 6: Validation suite summary

可视化各周 validation checks：

- Week 3 scenario generation；
- Week 4 N-1 labeling；
- Week 5 metrics；
- Week 6 PI-MLP；
- Week 7 GNN。

---

# Part IV. 正确性证明与可复现性

## Slide 23. 为什么要写 proof / validation suite

原因：

- pandapower 结果表容易被当成黑箱；
- AI 数据集容易泄漏 post-contingency 信息；
- threshold selection 容易不小心使用 test set；
- GNN 容易对 node ordering 敏感；
- 论文需要可信实验链路。

---

## Slide 24. Dataset lineage proof

检查链路：

\[
\text{Week 3 scenarios}
\rightarrow
\text{Week 4 } (x_t,c,y)
\rightarrow
\text{Week 5/6/7 models}
\rightarrow
\text{Week 8 paper tables}
\]

---

## Slide 25. No-leakage proof

禁止输入：

- post-contingency voltage / loading / VUF；
- violation flags；
- severity score；
- convergence / service-loss label。

允许输入：

- base-case summary；
- scenario settings；
- device metadata；
- contingency identity and location。

---

## Slide 26. Metric consistency proof

要求手算：

\[
TP, FP, TN, FN
\]

再复核：

\[
Recall, FNR, Precision, PF\ calls\ saved
\]

---

## Slide 27. Group split proof

所有 group CV 都应证明：

\[
\mathcal{G}_{train}\cap\mathcal{G}_{test}=\emptyset
\]

这对应 scenario-group 和 contingency-group 两类泛化测试。

---

## Slide 28. Figure / table manifest

最终论文包需要包含：

- source CSV；
- paper tables；
- paper figures；
- validation summary；
- hashes；
- mini-paper skeleton；
- presentation outline。

---

# Part V. 最终展示与交付

## Slide 29. 10--15 页 presentation 推荐结构

1. Problem and motivation
2. Three-phase microgrid setting
3. N-1 label generation
4. AI screening formulation
5. Baseline and PI-MLP
6. Phase-aware GNN
7. Results and tradeoff
8. Validation suite
9. Limitations
10. Future work

---

## Slide 30. 学生最终提交物

必交：

- clean notebooks；
- executed notebooks；
- generated CSV and figures；
- mini-paper draft；
- presentation slides；
- reproducibility checklist。

---

## Slide 31. 论文 limitation 怎么写

建议诚实写：

- teaching feeder 小规模，不能声称普适性能；
- islanded mode 尚未完整建模；
- GNN 在小数据下未必优于 MLP；
- PI losses 是 limit-aware，不是严格 AC feasibility guarantee；
- real microgrid validation 尚未完成。

---

## Slide 32. 期刊扩展路线

可扩展方向：

- 更多 feeder 和更多 stochastic scenarios；
- phase-level GNN；
- conformal prediction / selective classification；
- BESS SOC time series；
- inverter reactive power control；
- islanded microgrid with grid-forming inverter；
- real microgrid data validation。

---

## Slide 33. 本周 Notebook 输出

Notebook 08 应生成：

- paper table CSV / LaTeX；
- figure manifest；
- claim-evidence table；
- mini-paper skeleton；
- presentation outline；
- reproducibility checklist；
- validation summary；
- artifact hash manifest。

---

## Slide 34. 课堂练习

学生任务：

1. 修改 mini-paper Introduction；
2. 选择最有说服力的 3 张图；
3. 写 5 条核心 claims，并为每条 claim 指定证据；
4. 根据 validation summary 找出一个 limitation；
5. 准备 5 分钟项目汇报。

---

## Slide 35. Takeaways

本项目最终要证明的不是“某个模型最好”，而是：

1. 三相 N-1 安全筛查可以被系统化地数据化；
2. high-recall screening 是比 accuracy 更合适的目标；
3. physics-informed target 和 graph representation 提供了更可信的研究方向；
4. 每一个实验结论都要有 artifact、proof 和 validation 支撑。
