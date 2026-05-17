# Week 6 PPT 结构：Physics-informed MLP for 三相微电网 N-1 安全筛查

**主题：** 从黑箱分类器到 Physics-informed / Limit-aware MLP  
**对应 Notebook：** `06_week6_physics_informed_mlp_framework.ipynb`  
**本周目标：** 在 Week 5 的 ML baseline 基础上，构建一个带物理含义辅助目标、limit consistency、ranking loss 和 high-recall threshold 的 PI-MLP。

---

## Slide 1. 标题页

**标题：** Week 6: Physics-informed AI for Three-phase Microgrid N-1 Screening  
**副标题：** Limit-aware multi-task MLP with proof and cross-validation

讲解重点：

- Week 5 已经完成传统 ML baseline。
- Week 6 不再只预测二元 label，而是让模型学习“为什么 unsafe”。
- 本周的 PI 并不是完整 PINN 求解三相潮流，而是把三相安全约束、severity、component violation 和 ranking 放入训练目标。

---

## Slide 2. 本周在八周课程中的位置

从 Week 1--5 到 Week 6 的链路：

```text
Week 1  balanced pandapower intro
Week 2  runpp_3ph and VUF proof
Week 3  scenario generation
Week 4  N-1 violation label
Week 5  traditional ML baseline
Week 6  physics-informed MLP
Week 7  phase-aware GNN
Week 8  paper and reproducibility
```

核心问题：

> 能否在不使用 post-contingency 结果作为输入的前提下，让 AI 模型学习三相静态安全约束的结构？

---

## Slide 3. 从黑箱 ML 到 Physics-informed AI

Week 5 模型：

\[
\hat y = f_\theta(x_t,c)
\]

Week 6 模型：

\[
(\hat y, \hat s, \hat m_V, \hat m_I, \hat m_U, \hat m_S)
= f_\theta(x_t,c)
\]

其中：

- \(\hat y\)：是否 unsafe；
- \(\hat s\)：severity score；
- \(\hat m_V\)：电压越限 margin；
- \(\hat m_I\)：电流/线路 loading 越限 margin；
- \(\hat m_U\)：VUF 越限 margin；
- \(\hat m_S\)：service-loss / no-slack 风险。

---

## Slide 4. 为什么需要 PI-MLP

普通分类器的问题：

1. 只知道 safe / unsafe，不知道 violation 类型；
2. 容易在小数据集上过拟合 contingency ID；
3. ranking 能力不一定好；
4. probability score 不一定与 severity 单调一致；
5. 难以解释给电力系统学生。

PI-MLP 的目标：

```text
classification accuracy
+ severity awareness
+ component violation awareness
+ consistency with physical limits
+ ranking ability
+ high-recall screening behavior
```

---

## Slide 5. 部署时刻可用信息与禁止信息

输入只允许使用：

```text
scenario setting
base-case voltage / VUF / loading / grid import
contingency metadata
microgrid mode
```

禁止输入：

```text
post-contingency min voltage
post-contingency max loading
post-contingency VUF
post-contingency convergence flag
violation flags
severity score
unserved load
```

本周第一个 proof：

\[
\mathcal{X}_{input}\cap\mathcal{Y}_{post}=\emptyset
\]

---

## Slide 6. Supervised target 设计

从 Week 4 数据中可得到：

\[
y \in \{0,1\}
\]

\[
s \ge 0
\]

以及若干 violation flags：

```text
voltage_low_violation
voltage_high_violation
loading_violation
vuf_violation
non_convergence_violation
service_loss_violation
critical_service_loss_violation
```

Week 6 将这些转成多任务监督信号。

---

## Slide 7. Component margin: voltage low

设低电压阈值为：

\[
V^{min}=0.95\ \mathrm{p.u.}
\]

低电压 margin：

\[
m_{V,low}=\left[\frac{V^{min}-V_{min}}{V^{min}}\right]_+
\]

如果 power flow 被 service-loss / no-slack 逻辑跳过，则不强行填造电压值，而是由 service margin 表达该风险。

---

## Slide 8. Component margin: voltage high

设高电压阈值为：

\[
V^{max}=1.05\ \mathrm{p.u.}
\]

高电压 margin：

\[
m_{V,high}=\left[\frac{V_{max}-V^{max}}{V^{max}}\right]_+
\]

---

## Slide 9. Component margin: current / loading

线路 loading 阈值：

\[
L^{max}=100\%
\]

loading margin：

\[
m_I=\left[\frac{L_{max}}{100}-1\right]_+
\]

这里 \(L_{max}\) 来自三相 N-1 后最大 phase loading。

---

## Slide 10. Component margin: VUF

VUF 阈值示例：

\[
U^{max}=2\%
\]

VUF margin：

\[
m_U=\left[\frac{U_{max}^{obs}}{U^{max}}-1\right]_+
\]

这是三相不平衡故事线的核心之一。

---

## Slide 11. Component margin: service-loss / no-slack

对于 radial microgrid，line outage 可能直接导致 downstream load 失供。

定义：

\[
m_S = \max(y_{service}, y_{critical\ service}, y_{no\ slack})
\]

这个目标不来自连续潮流结果，而来自拓扑和可供电性逻辑。

---

## Slide 12. PI severity target

为了让 severity 至少覆盖最严重 component：

\[
s_{PI}=\max(s_{raw}, m_{V,low},m_{V,high},m_I,m_U,m_S)
\]

proof：

\[
s_{PI}\ge \max_k m_k
\]

这样模型的 severity head 不会比 component head 更“乐观”。

---

## Slide 13. 模型结构：shared encoder + multi-head

结构：

```text
input features
    ↓
shared MLP encoder
    ↓
classification head: logit y
severity head: softplus severity
component head: softplus margins
```

输出：

\[
\hat y,\quad \hat s,\quad \hat{\mathbf m}
\]

其中 softplus 保证 severity 和 margin 非负。

---

## Slide 14. Data loss: binary classification

使用 weighted BCE：

\[
\mathcal{L}_{cls}
= -w_+y\log\hat p -(1-y)\log(1-\hat p)
\]

原因：

- unsafe / safe 样本不均衡；
- 安全筛查更关注 false negative；
- class weight 是 baseline，不等于最终安全保证。

---

## Slide 15. Severity regression loss

使用 Huber / SmoothL1：

\[
\mathcal{L}_{sev}
=\mathrm{Huber}(\log(1+\hat s),\log(1+s_{PI}))
\]

使用 \(\log(1+s)\) 的原因：

- severity 可能跨越不同量级；
- 减少极端样本支配训练。

---

## Slide 16. Component margin loss

对每个 component margin 进行监督：

\[
\mathcal{L}_{comp}
=\sum_k\mathrm{Huber}(\log(1+\hat m_k),\log(1+m_k))
\]

它相当于告诉模型：

```text
unsafe 是因为 voltage?
还是 loading?
还是 VUF?
还是 service loss?
```

---

## Slide 17. Limit consistency loss

要求预测 severity 不应小于任意 component margin：

\[
\mathcal{L}_{cons}
=
\left[\max_k\hat m_k-\hat s\right]_+^2
\]

物理解释：

> overall severity 至少应覆盖最严重的安全约束 violation。

---

## Slide 18. Probability--severity consistency

如果预测 severity 很大，unsafe probability 不应很低：

\[
\mathcal{L}_{prob-sev}
=
\left(
\sigma(z)-\sigma(\alpha(\hat s-\epsilon))
\right)^2
\]

其中 \(z\) 是 classification logit。

---

## Slide 19. Pairwise ranking loss

对于同一个 batch 中两条样本：

\[
s_i > s_j
\]

希望：

\[
\hat s_i > \hat s_j
\]

ranking loss：

\[
\mathcal{L}_{rank}
=\left[\delta-(\hat s_i-\hat s_j)\right]_+
\]

作用：帮助模型把高风险 contingency 排到前面。

---

## Slide 20. 总损失函数

\[
\mathcal{L}
=
\mathcal{L}_{cls}
+
\lambda_s\mathcal{L}_{sev}
+
\lambda_c\mathcal{L}_{comp}
+
\lambda_{cons}\mathcal{L}_{cons}
+
\lambda_p\mathcal{L}_{prob-sev}
+
\lambda_r\mathcal{L}_{rank}
\]

Week 6 的实验不是为了证明某一组 \(\lambda\) 最优，而是让学生理解：

```text
每个 loss term 对应一个工程假设或物理约束。
```

---

## Slide 21. 训练流程

严格使用：

```text
train set: fit model
validation set: choose threshold
test set: final report only
```

避免：

```text
用 test set 调 threshold
用 post-contingency metric 当输入
用同一份预测做 group CV 自评
```

---

## Slide 22. High-recall threshold selection

目标：

\[
\max_\tau\ \mathrm{PFCallsSaved}_{val}(\tau)
\]

subject to：

\[
\mathrm{Recall}_{val}(\tau)\ge 0.95
\]

安全解释：

> 先保证尽量不漏掉 unsafe contingency，再尽可能减少三相潮流调用。

---

## Slide 23. Proof 1: no leakage

检查：

```text
feature_cols ∩ leakage_cols = ∅
```

同时检查特征名中不得出现：

```text
min_vm, max_vuf, violation, severity, unserved, converged
```

除非它们属于 `base_` 开头的 base-case feature。

---

## Slide 24. Proof 2: target consistency

检查：

\[
y=0 \Rightarrow s=0
\]

\[
y=1 \Rightarrow s>0
\]

\[
s_{PI}\ge \max_k m_k
\]

这些检查保证 supervised target 自洽。

---

## Slide 25. Proof 3: finite gradient

在一个 mini-batch 上检查：

```text
loss is finite
gradients are finite
outputs have correct shape
probabilities in [0,1]
```

这是神经网络课程中很重要但常被忽略的 engineering proof。

---

## Slide 26. Cross-validation 1: random holdout

目的：

- 与 Week 5 的 baseline 一致；
- 快速检查模型是否能学习基本模式。

限制：

- scenario 和 contingency 可能同时出现在 train/test；
- 容易高估泛化能力。

---

## Slide 27. Cross-validation 2: scenario-group CV

要求：

\[
\mathcal{S}_{train}\cap\mathcal{S}_{test}=\emptyset
\]

意义：

> 模型是否能泛化到未见过的 operating scenario？

---

## Slide 28. Cross-validation 3: contingency-group CV

要求：

\[
\mathcal{C}_{train}\cap\mathcal{C}_{test}=\emptyset
\]

意义：

> 模型是否能泛化到未见过的 contingency 类型或设备？

这是更难的测试。

---

## Slide 29. Ablation study

至少比较：

```text
BaselineMLP: BCE only
PI-MLP no-rank: cls + severity + component + consistency
PI-MLP full: cls + severity + component + consistency + ranking
```

关注：

- recall；
- FNR；
- PF calls saved；
- AUPRC；
- ranking behavior；
- 输出解释性。

---

## Slide 30. 结果图 1: training curves

展示：

```text
train loss vs epoch
validation loss vs epoch
```

要提醒学生：

- 小数据集上曲线非常容易过拟合；
- early stopping 是必要的；
- validation loss 不等价于 screening recall。

---

## Slide 31. 结果图 2: precision-recall curve

为什么不用 accuracy curve？

因为安全预测更关心：

```text
recall high enough?
false negatives low enough?
```

PR curve 比 ROC 更能反映不平衡分类下的筛查质量。

---

## Slide 32. 结果图 3: saved PF calls vs missed violations

横轴：saved PF calls fraction  
纵轴：missed violations

工程解释：

> 省掉多少三相 N-1 潮流计算，是以漏掉多少 unsafe contingency 为代价？

---

## Slide 33. 结果图 4: true severity vs predicted severity

用于检查：

- severity head 是否学到风险强弱；
- service-loss 样本是否被高估或低估；
- safe 样本是否接近 0。

---

## Slide 34. 本周输出文件

Notebook 输出：

```text
week6_training_curves.csv
week6_holdout_results.csv
week6_loss_ablation_summary.csv
week6_group_cv_results.csv
week6_group_cv_summary.csv
week6_screening_tradeoff.csv
week6_test_predictions.csv
week6_validation_summary.csv
```

图表：

```text
training curves
precision-recall curves
screening tradeoff
severity scatter
component target distribution
```

---

## Slide 35. Week 6 到 Week 7 的衔接

Week 6 PI-MLP 的局限：

- contingency topology 只通过 metadata 表示；
- 没有显式 message passing；
- 对未见过拓扑泛化有限。

Week 7 进入：

> Phase-aware GNN: 把 bus、phase、line、device 和 contingency mask 显式放入图模型。
