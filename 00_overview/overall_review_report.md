# 总体 Review 报告：科研设计与 8 周教学材料

## 1. 总体结论

这套材料已经形成一个完整闭环：

1. Week 1 建立 microgrid + pandapower 的基础；
2. Week 2 进入三相不平衡潮流 `runpp_3ph()`；
3. Week 3 生成 microgrid operating scenarios；
4. Week 4 生成三相 N-1 violation label；
5. Week 5 建立传统 ML baseline 与安全筛查指标；
6. Week 6 引入 physics-informed / limit-aware MLP；
7. Week 7 引入 phase-aware GNN 与 top-k screening；
8. Week 8 整理论文、图表、复现清单和 mini-paper skeleton。

**总体验收结果：通过。** 本次总体验收中，合并检查 **20/20** 项通过。

---

## 2. 科研故事线 Review

当前科研故事线是清晰的：

> 用 pandapower 三相不平衡潮流生成微电网 N-1 静态安全标签，再用 ML / PI-MLP / phase-aware GNN 作为 high-recall pre-screening module，减少需要送入三相潮流精算的 contingency 数量。

这个故事线的优点是：

- **应用场景明确**：microgrid / LV feeder / DER-rich / phase imbalance；
- **仿真内核明确**：pandapower `runpp_3ph()`；
- **标签定义明确**：相电压越限、相电流/线路 loading 越限、VUF 越限、service loss、PCC no-slack / non-convergence；
- **AI 目标明确**：不是替代潮流，而是高召回筛查；
- **论文结构完整**：dataset generation -> baseline -> PI loss -> GNN -> screening metrics -> reproducibility。

建议在正式论文中把 claim 分成两层：

### Teaching / MVP claim

当前 9-bus teaching feeder 可以支撑：

- 方法流程演示；
- 代码正确性 proof；
- 标签和特征工程设计；
- mini-paper / course project。

### Research-paper claim

正式投稿前建议扩展到：

- 更多 operating scenarios，例如数百到数千个场景；
- 更多测试 feeder，包括更大的 LV / microgrid case；
- 对 line outage、DER trip、BESS trip、PCC outage 分开报告；
- 把 topology-first service-loss 与 PF-solvable contingency 分开评估；
- 增加 scenario-group、contingency-group、feeder-group OOD test；
- 增加 calibration / conformal / selective classification，用于安全筛查的保守性控制。

---

## 3. 教学材料 Review

教学顺序合理，且每周材料具有清楚的输入输出关系：

| 周次 | 产出 | 下游用途 |
|---|---|---|
| Week 1 | microgrid-like pandapower 入门 | 建立表结构和静态安全直觉 |
| Week 2 | 三相不平衡潮流 proof | 支撑 Week 3/4 的三相结果解释 |
| Week 3 | scenario table + base-case features | 作为 Week 4 N-1 扫描输入 |
| Week 4 | N-1 dataset + AI-ready labels | 作为 Week 5/6/7 监督学习数据 |
| Week 5 | ML baseline + screening metrics | 给 Week 6/7 提供性能下限 |
| Week 6 | PI-MLP + ablation | 体现 physics-informed contribution |
| Week 7 | phase-aware GNN + top-k | 体现 topology-aware contribution |
| Week 8 | paper package + reproducibility | 形成最终 mini-paper / presentation |

---

## 4. 正确性与交叉验证结果

本次整体 review 重点检查了：

- Notebook 是否存在 error output；
- 各周 validation summary 是否全部通过；
- Week 4 的 `scenario x contingency = dataset rows` 是否一致；
- label 是否包含 safe / unsafe 两类；
- no-leakage feature audit 是否通过；
- Beamer PDF 是否能打开且页数符合预期；
- Week 8 的 claim-evidence、reproducibility checklist 是否生成。

核心结果如下：

```text
Final executed notebooks: 8 checked, 0 error outputs
Week 2 numerical proofs: within tolerance
Week 3 validation: 12/12 passed
Week 4 validation: 13/13 passed
Week 5 validation: 12/12 passed
Week 6 validation: 16/16 passed
Week 7 validation: 44/44 passed
Week 8 validation: 22/22 passed
Beamer PDFs: all expected page counts verified
```

完整检查表见：`00_overview/overall_review_validation_summary.csv`。

---

## 5. 关键实验结果 Review

当前数据规模：

```text
Scenarios: 7
Contingencies: 11
N-1 samples: 77
Unsafe samples: 47
Safe samples: 30
```

主要 holdout 结果显示，GradientBoosting、PI-MLP full 和 FlattenedMLP 在当前小教学数据集上均可达到 zero missed violation，并保存约 40% 的 PF calls。这个结果适合教学展示，但不应过度推广。

Week 7 中 PhaseAwareGNN 在当前小数据集上没有超过 FlattenedMLP，这是合理且诚实的教学点：GNN 的价值首先在于拓扑建模框架和可扩展性，而不是在 77 个样本的小数据集上天然超过 MLP。

---

## 6. 建议优化后的论文贡献写法

建议正式论文贡献写成：

1. **Three-phase N-1 dataset generation workflow**：基于 pandapower `runpp_3ph()` 的微电网三相 N-1 数据生成流程。
2. **Phase-aware violation labeling**：统一定义相电压、相 loading、VUF、service loss 和 non-convergence 标签。
3. **High-recall screening formulation**：以 recall / FNR / PF calls saved / top-k hit rate 作为核心评价，而非普通 accuracy。
4. **Physics-informed MLP**：用 severity / component margin / consistency / ranking loss 增强黑箱分类器。
5. **Topology-aware GNN interface**：将 bus phase channel、edge feature、contingency mask 和 graph readout 组织成可扩展 GNN 框架。
6. **Reproducibility and leakage-proof protocol**：明确 no-leakage 特征边界、group CV 和 validation-only threshold selection。

---

## 7. 主要局限与后续扩展

需要在论文和学生报告里明确写出：

- 当前是 small teaching feeder，不是部署级 benchmark；
- service-loss contingency 在 radial feeder 下占比较高，会影响模型任务难度；
- BESS 目前是静态 setpoint，未建模 SOC time-series；
- PCC outage 在 grid-connected MVP 中被处理为 no-slack / non-convergence，不等价于完整 islanded microgrid control；
- PI-MLP 中的物理约束主要是 limit-aware / consistency-aware，不是严格 AC power-flow residual guarantee；
- GNN 要展示真正优势，需要更多拓扑、多 feeder 或 phase-level graph。

---

## 8. 交付包使用建议

- 课堂教学：优先使用每周 `slides/*.pdf` 和 `notebooks/*clean*.ipynb`。
- 教师备课：参考 `notebooks/*executed*.ipynb` 中的输出和 validation checks。
- 学生项目：从 Week 3/4 的数据开始复现，再做 Week 5/6/7 模型比较。
- 论文写作：使用 `90_paper_package/` 中的 `main.tex`、tables 和 figures。
- 最终展示：使用 `91_presentation_package/final_presentation_outline.md` 作为答辩结构。

