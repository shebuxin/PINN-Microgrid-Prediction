# Week 5 PPT 结构：传统 AI Baseline 与安全筛查指标

**课程主题**  
Traditional ML baselines and safety-oriented screening metrics for three-phase microgrid N-1 security prediction

**本周承接**  
Week 4 已经把每个样本定义成 \((x_t, c)\)，并生成了 `week4_ai_ready_dataset.csv`。Week 5 的任务是：不再运行三相 N-1 潮流，而是只利用 base-case operating scenario 与 contingency metadata，训练传统 AI baseline 来预测：

\[
\hat{y}_{t,c}=\mathbb{I}\{\text{N-1 violation likely}\}
\]

本周重点不是追求复杂模型，而是建立**合法、可复现、可解释、以安全为中心**的 baseline。

---

## Slide 1. Title

**Week 5: 传统 AI Baseline 与安全筛查指标**  
From N-1 labels to high-recall machine-learning screening

关键词：classification, ranking, false negative rate, threshold tuning, group cross-validation, leakage control

---

## Slide 2. 八周课程中的位置

- Week 1：balanced microgrid pandapower 入门
- Week 2：三相不平衡 `runpp_3ph()`
- Week 3：microgrid scenario generation
- Week 4：三相 N-1 扫描与 label 生成
- **Week 5：传统 ML baseline + 安全筛查指标**
- Week 6：physics-informed MLP
- Week 7：phase-aware GNN 与 high-recall screening
- Week 8：论文组织与展示

**本周输出：** baseline performance table + screening trade-off figures

---

## Slide 3. 从仿真数据到监督学习

Week 4 生成：

\[
\mathcal{D}=\{(x_t,c,y_{t,c},s_{t,c})\}_{t,c}
\]

其中：

- \(x_t\)：base-case microgrid operating state
- \(c\)：candidate contingency
- \(y_{t,c}\)：binary violation label
- \(s_{t,c}\)：severity score

Week 5 的 supervised learning 问题：

\[
 f_\theta(x_t,c) \rightarrow \hat{p}_{t,c}=P(y_{t,c}=1)
\]

---

## Slide 4. AI screening 的角色

AI 不直接替代三相潮流。

推荐部署逻辑：

```text
all candidate contingencies
        ↓
AI risk scorer
        ↓
keep high-risk contingencies
        ↓
run accurate runpp_3ph only on selected cases
        ↓
operator / controller decision
```

因此安全筛查更关心：

- 漏掉多少真正危险样本？
- 能减少多少三相潮流调用？
- 阈值如何设置才保守？

---

## Slide 5. 为什么 accuracy 不够

如果 unsafe 样本少，模型全部预测 safe 也可能有高 accuracy。

安全应用中最危险的是：

\[
\text{False Negative}: y=1, \hat{y}=0
\]

含义：真实 N-1 不安全，但 AI 没有送去精确三相潮流检查。

所以 Week 5 把重点放在：

- Recall
- False negative rate
- AUPRC
- Top-k violation recall
- AC PF calls saved vs missed violations

---

## Slide 6. Confusion matrix

| | Predicted safe | Predicted unsafe |
|---|---:|---:|
| True safe | TN | FP |
| True unsafe | FN | TP |

关键指标：

\[
\text{Recall}=\frac{TP}{TP+FN}
\]

\[
\text{FNR}=\frac{FN}{TP+FN}=1-\text{Recall}
\]

\[
\text{Precision}=\frac{TP}{TP+FP}
\]

---

## Slide 7. Screening trade-off

降低阈值：

- 更多 contingency 被判为 high-risk
- Recall 提高
- 漏报降低
- 但三相潮流调用减少较少

提高阈值：

- 三相潮流调用减少更多
- 但漏掉 violation 的风险上升

核心曲线：

\[
\text{PF calls saved} \quad \text{vs.} \quad \text{missed violations}
\]

---

## Slide 8. Feature leakage：最重要的 baseline 纪律

不能把 post-contingency 结果放入特征。

禁止作为输入的列包括：

```text
converged
min_vm_pu
max_vm_pu
max_vuf_percent
max_line_loading_percent
unserved_load_mw
violation flags
severity_score
```

这些是运行 N-1 三相潮流以后才知道的量。

Week 5 合法输入只包括：

- scenario settings
- base-case features
- contingency metadata

---

## Slide 9. 合法 feature families

Scenario settings:

```text
load_scale
phase_a_factor
phase_b_factor
phase_c_factor
pv_scale
bess_p_discharge_mw
```

Base-case physical features:

```text
base_min_vm_pu
base_max_vuf_percent
base_max_line_loading_percent
base_p_grid_mw
base_phase_loads
base_phase_sgen
```

Contingency metadata:

```text
contingency_type
element_table
element_index
```

---

## Slide 10. Baseline 0：规则筛查器

先做一个不用训练的 rule-based baseline。

示例风险分数：

\[
 r = w_1 r_{type}+w_2 r_{loading}+w_3 r_{voltage}+w_4 r_{unbalance}
\]

其中：

- PCC outage 风险最高
- line outage 风险较高
- base-case loading 越高，风险越高
- base-case voltage 越低，风险越高
- base-case VUF 越高，风险越高

作用：给 ML 模型一个朴素但可解释的比较对象。

---

## Slide 11. Baseline 1：Logistic Regression

优点：

- 可解释
- 训练稳定
- 适合小数据集
- 可以作为论文 baseline

形式：

\[
P(y=1|x)=\sigma(w^Tx+b)
\]

课程中使用：

```text
StandardScaler for numeric features
OneHotEncoder for categorical features
class_weight = balanced
```

---

## Slide 12. Baseline 2：Random Forest

优点：

- 非线性
- 不需要强特征缩放
- 可以给出 feature importance
- 对小型 tabular 数据常常表现不错

缺点：

- 外推能力有限
- 对未见拓扑/未见 contingency 的泛化不能默认相信

---

## Slide 13. Baseline 3：Gradient Boosting

优点：

- tabular 数据强 baseline
- 可以捕捉复杂 feature interaction
- 通常比单棵树稳定

课程用途：

- 与 Logistic Regression 和 Random Forest 对比
- 为 Week 6 / Week 7 的 PI-MLP 和 GNN 提供强 baseline

---

## Slide 14. Baseline 4：MLP

普通 MLP 是 Week 6 physics-informed MLP 的前身。

Week 5 中的 MLP 只做黑箱数据拟合：

\[
\hat{p}=\mathrm{MLP}(x_t,c)
\]

Week 6 会加入：

- limit penalty
- severity auxiliary loss
- ranking loss
- physics-informed regularization

---

## Slide 15. 为什么要 class weighting

N-1 violation 数据常常类别不平衡。

若 unsafe 是少数类，则普通训练可能偏向 safe。

处理方式：

- `class_weight="balanced"`
- weighted BCE
- focal loss
- threshold tuning
- precision-recall curve 而不是只看 ROC

---

## Slide 16. Dataset split：随机 split 不够

随机 split 可能让同一 scenario 或同一 contingency 同时出现在 train/test。

这会高估泛化能力。

Week 5 使用三种评估：

1. Stratified random split：基础 sanity check
2. Group split by scenario：测试 unseen operating scenario
3. Group split by contingency：测试 unseen contingency

---

## Slide 17. Scenario-group validation

目标：测试模型是否能泛化到未见运行场景。

\[
\mathcal{G}_{scenario}=\{\text{base, high load, high PV, ...}\}
\]

训练/测试不共享 scenario：

```text
train scenarios ∩ test scenarios = ∅
```

这是微电网运营中更实际的评估。

---

## Slide 18. Contingency-group validation

目标：测试模型是否能泛化到未见 contingency。

\[
\mathcal{G}_{contingency}=\{line\_0\_out, pv\_0\_trip, ...\}
\]

训练/测试不共享 contingency ID。

注意：如果模型输入中使用了 contingency ID one-hot，则这种评估会暴露其泛化弱点。

---

## Slide 19. Threshold tuning for safety

训练模型得到概率分数：

\[
\hat{p}_{t,c}
\]

再用 validation set 选择阈值 \(\tau\)：

\[
\hat{y}_{t,c}=\mathbb{I}(\hat{p}_{t,c}\ge \tau)
\]

安全优先策略：

\[
\tau^* = \max \{\tau: Recall_{val}(\tau) \ge R_{target}\}
\]

比如 \(R_{target}=0.95\)。

---

## Slide 20. Why tune on validation only?

不能用 test set 选阈值。

正确流程：

```text
train set: fit model
validation set: choose threshold
test set: final evaluation
```

否则 test performance 会被乐观估计。

---

## Slide 21. AUPRC vs AUROC

AUROC 衡量正负样本排序能力，但在类别不平衡时可能显得过于乐观。

AUPRC 更关注 positive class：

- unsafe contingency 是 safety-critical positive class
- precision-recall curve 更符合 screening 目标

Week 5 同时报告 AUROC 和 AUPRC，但优先讨论 AUPRC 与 recall。

---

## Slide 22. Top-k violation recall

对每个 scenario，将所有 contingency 按风险分数排序。

只保留前 \(k\) 个：

\[
\mathcal{C}_{top-k}(x_t)
\]

定义：

\[
\text{Top-k Recall}=\frac{\#\{\text{true violations in top-k}\}}{\#\{\text{all true violations}\}}
\]

作用：模拟 operator 只愿意进一步检查最危险的若干 contingency。

---

## Slide 23. AC PF calls saved

如果只对 predicted unsafe 的 contingency 运行 `runpp_3ph()`：

\[
\text{PF calls saved}=N-N_{flagged}
\]

归一化：

\[
\text{saved ratio}=1-\frac{N_{flagged}}{N}
\]

但必须同时报告：

\[
\text{missed violations}=FN
\]

---

## Slide 24. Baseline result table

建议主表：

| Model | AUPRC | AUROC | Recall | Precision | FNR | F1 | Saved ratio |
|---|---:|---:|---:|---:|---:|---:|---:|
| Rule | | | | | | | |
| Logistic | | | | | | | |
| Random Forest | | | | | | | |
| Gradient Boosting | | | | | | | |
| MLP | | | | | | | |

---

## Slide 25. Cross-validation result table

建议附表：

| Model | Random CV AUPRC | Scenario CV AUPRC | Contingency CV AUPRC |
|---|---:|---:|---:|
| Logistic | | | |
| Random Forest | | | |
| Gradient Boosting | | | |
| MLP | | | |

说明：

- random CV 是 sanity check
- scenario CV 是 operating condition generalization
- contingency CV 是 outage generalization

---

## Slide 26. What to expect from small data

Week 5 数据量较小，结果不应该被过度解读。

本周重点是建立流程：

```text
legal features
leakage proof
baseline training
threshold tuning
screening metrics
group cross-validation
```

后续 Week 6/7 会扩大数据规模并引入 PI-MLP / GNN。

---

## Slide 27. Common mistakes

1. 把 post-contingency `min_vm_pu` 当作输入特征
2. 用 test set 调阈值
3. 只报告 accuracy
4. 不区分 random split 和 group split
5. 不报告 false negative
6. 只看一个 threshold 下的结果
7. 忽略 top-k screening 场景

---

## Slide 28. Notebook roadmap

Notebook 05 包含：

```text
1. Load Week 4 and Week 3 outputs
2. Merge base-case features with N-1 labels
3. Prove no feature leakage
4. Define metrics and threshold tuning
5. Train rule, logistic, RF, GB, MLP baselines
6. Evaluate train/validation/test split
7. Run random / scenario / contingency CV
8. Plot screening trade-off and top-k recall
9. Export result tables
```

---

## Slide 29. Deliverables

学生本周应提交：

- `week5_baseline_dataset.csv`
- `week5_holdout_results.csv`
- `week5_cv_results.csv`
- `week5_screening_tradeoff.csv`
- precision-recall curve
- recall-threshold curve
- saved-calls vs missed-violations curve
- 一段 300 字实验结论

---

## Slide 30. Connection to paper

论文中 Week 5 对应：

```text
Experimental Setup:
    Dataset construction
    Baseline models
    Evaluation metrics

Results:
    Baseline performance table
    Screening trade-off curve
    Group generalization results
```

Week 6/7 的新模型必须超过 Week 5 的 baseline，才有论文意义。

---

## Slide 31. Transition to Week 6

Week 5 的模型只学习：

\[
\hat{p}=f_\theta(x_t,c)
\]

Week 6 将引入 physics-informed signals：

- severity auxiliary target
- limit-aware penalty
- ranking loss
- conservative thresholding

目标从“能分类”提升到“更符合电力系统物理与安全需求”。
