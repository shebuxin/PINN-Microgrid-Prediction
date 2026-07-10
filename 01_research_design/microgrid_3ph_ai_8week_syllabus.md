# 微电网三相不平衡安全预测：八周教学大纲

## 总体目标

八周后，学生应能够：

1. 使用 pandapower 建立低压/微电网三相不平衡模型；
2. 使用 runpp_3ph 完成三相不平衡潮流计算；
3. 设计微电网运行场景，包括并网、孤岛、高 PV、低 PV、高负荷、强相不平衡和储能调度；
4. 手写三相 N-1 扫描流程，生成 phase-aware violation label；
5. 训练传统 ML、MLP、physics-informed MLP 和基础 phase-aware GNN；
6. 使用 high-recall screening 指标评价安全预测模型；
7. 整理实验结果，形成 mini-paper 和答辩 PPT。

## Week 1：微电网与三相不平衡安全预测导论

### 教学目标

让学生理解为什么微电网安全分析不能只使用平衡正序潮流，以及本项目的完整科研问题。

### PPT 内容

1. 微电网定义：PCC、DER、BESS、critical load、grid-connected/islanded mode；
2. 低压微电网的三相不平衡来源：单相负荷、单相 PV、EV charging、相别分布不均；
3. 静态安全问题：电压越限、线路/变压器过载、电压不平衡、潮流不可解；
4. N-0、N-1、scenario-based security assessment；
5. 为什么使用 pandapower；
6. 为什么需要 AI pre-screening；
7. 项目最终 pipeline 概览。

### Notebook

`01_microgrid_and_pandapower_intro.ipynb`

内容：

- 安装和导入 pandapower；
- 加载一个简单网络；
- 认识 net.bus、net.line、net.trafo、net.load、net.sgen、net.storage；
- 绘制网络拓扑；
- 运行普通 balanced runpp 作为对照。

### 学生输出

- 一页 summary：微电网三相不平衡安全预测要解决什么问题；
- 完成 pandapower 基本对象浏览 notebook。

## Week 2：pandapower 三相不平衡建模与 runpp_3ph

### 教学目标

让学生能够建立和运行三相不平衡潮流，并理解相电压、相功率、相电流和序分量。

### PPT 内容

1. 三相系统基础：phase A/B/C、line-to-ground voltage、line-to-line voltage；
2. 对称分量：positive、negative、zero sequence；
3. voltage unbalance factor；
4. pandapower 的 asymmetric_load 和 asymmetric_sgen；
5. wye 和 delta 连接；
6. 线路正序/负序/零序参数；
7. runpp_3ph 的基本用法和限制；
8. 三相结果表：res_bus_3ph、res_line_3ph、res_trafo_3ph。

### Notebook

`02_three_phase_power_flow.ipynb`

内容：

- 从零构建一个 3-bus 三相不平衡小系统；
- 创建 asymmetric_load；
- 创建 asymmetric_sgen 代表单相 PV；
- 运行 runpp_3ph；
- 提取三相 bus voltage 和 line current；
- 计算 VUF；
- 与 balanced runpp 结果比较。

### 学生输出

- 能解释某一母线三相电压为什么不相等；
- 提交一个可以运行的最小三相潮流示例。

## Week 3：微电网测试系统构建与场景生成

### 教学目标

先用 9-bus 径向教学 feeder 建立可解释、可验证的场景生成流程，再检查
`ieee_european_lv_asymmetric` 的公开元数据，作为论文规模扩展的入口。

### PPT 内容

1. 9-bus teaching feeder 与 IEEE European LV asymmetric network 的关系；
2. 如何把 LV feeder 组织成 microgrid case；
3. PCC、主变、grid-forming source 的建模；
4. PV、BESS、diesel generator、critical load 的建模思路；
5. 并网/孤岛模式切换；
6. 负荷、PV、EV、BESS 的 scenario sampling；
7. 数据质量检查：收敛率、越限率、相不平衡分布。

### Notebook

`03_microgrid_scenario_generation.ipynb`

内容：

- 检查 `ieee_european_lv_asymmetric` 的规模与公开元数据；
- 构建含 critical load、PV、BESS 和 PCC 的 9-bus teaching feeder；
- 生成 7 个确定性场景，并给出固定随机种子的随机采样示例；
- 运行 base-case runpp_3ph；
- 保存 base-case feature table。

### 学生输出

- `week3_scenario_table.csv`；
- 一个可视化图：不同场景下 min phase voltage、max line current、max VUF 分布。

## Week 4：三相 N-1 扫描与 violation label 设计

### 教学目标

让学生能够手写 N-1 扫描，生成 7 个 operating points × 11 个
contingencies 的 77 行 label 和 severity 教学数据。

### PPT 内容

1. N-1 contingency 类型：line、trafo、PCC、DER、BESS；
2. 为什么三相 N-1 建议手写 loop；
3. contingency 后如何恢复 net 状态；
4. phase voltage violation；
5. phase current / loading violation；
6. voltage unbalance violation；
7. non-convergence as violation；
8. severity score 设计；
9. 类别不平衡和危险样本稀缺。

### Notebook

`04_three_phase_nminus1_labeling.ipynb`

内容：

- 定义 candidate contingencies；
- 对每个 scenario 和 contingency 运行 runpp_3ph；
- 提取 min/max phase voltage；
- 提取 max phase current/loading；
- 计算 VUF；
- 判断 violation type；
- 保存 dataset。

### 学生输出

- `n1_dataset.csv`；
- label 分布表；
- top 10 最严重 contingency 分析。

## Week 5：传统 AI Baseline 与安全筛查指标

### 教学目标

建立 baseline，并让学生理解安全应用中 recall、false negative rate 和 top-k hit rate 比 accuracy 更重要。

### PPT 内容

1. Classification、regression、ranking 三种任务；
2. 输入特征设计：场景特征、base-case 三相潮流特征、contingency 特征；
3. 数据划分：random split、scenario split、contingency split；
4. 类别不平衡：class weight、oversampling、threshold tuning；
5. 指标：recall、precision、FNR、AUPRC、top-k hit rate；
6. AI screening workflow：low-risk bypass，高风险送入 runpp_3ph 复核。

### Notebook

`05_ml_baselines_and_metrics.ipynb`

内容：

- 特征工程；
- Logistic Regression；
- Random Forest；
- Gradient Boosting；
- MLP；
- threshold tuning；
- 绘制 recall-threshold 和 precision-recall 曲线。

### 学生输出

- baseline performance table；
- 一张图：AC PF calls saved vs missed violations。

## Week 6：Physics-informed AI：从黑箱分类到物理约束学习

### 教学目标

让学生知道如何把三相物理约束加入 AI 模型，而不是只做黑箱分类。

### PPT 内容

1. 什么是 physics-informed AI；
2. 数据损失与物理损失；
3. limit penalty：电压、电流、VUF；
4. sequence consistency：相量与序分量一致性；
5. power balance residual：相域注入功率残差；
6. multi-task learning：classification + severity + ranking；
7. 物理损失权重调参；
8. ablation study 如何设计。

### Notebook

`06_physics_informed_mlp.ipynb`

内容：

- PyTorch MLP；
- weighted BCE；
- severity regression loss；
- ranking loss；
- limit penalty；
- 训练 PI-MLP；
- 与普通 MLP 比较。

### 学生输出

- MLP vs PI-MLP 的性能表；
- ablation：去掉 limit penalty、ranking loss、sequence feature 后的影响。

## Week 7：Phase-aware GNN 与高召回筛查策略

### 教学目标

让学生把微电网拓扑、相别信息和 contingency mask 放入 GNN，并完成主实验。

### PPT 内容

1. 为什么微电网适合图学习；
2. bus-level GNN vs phase-level GNN；
3. node features、edge features、global features；
4. contingency mask 如何进入 message passing；
5. graph-level output 和 edge-level output；
6. GNN 与 MLP 的公平比较；
7. high-recall threshold selection；
8. OOD generalization：未见过负荷水平、未见过 PV 场景、未见过 contingency。

### Notebook

`07_phase_aware_gnn_screening.ipynb`

内容：

- 用 plain PyTorch 构建 graph tensors（不依赖 PyTorch Geometric）；
- bus-level phase-channel GNN；
- contingency embedding；
- graph-level risk prediction；
- top-k screening；
- scenario-group 与 contingency-group split 实验。

### 学生输出

- GNN 与 PI-MLP 对比表；
- top-k hit rate 曲线；
- OOD performance table。

## Week 8：论文组织、结果复现与展示

### 教学目标

把八周代码和实验整理成论文雏形、PPT 和可复现实验包。

### PPT 内容

1. 论文故事线：motivation → gap → method → experiment → implication；
2. introduction 如何写；
3. problem formulation 如何写；
4. method section 图如何画；
5. experiment section 需要哪些表；
6. ablation 和 limitation 怎么写；
7. related work 分类：microgrid security、unbalanced power flow、AI surrogate、physics-informed learning、GNN；
8. 答辩 PPT 的结构。

### Notebook

`08_report_figures_and_reproducibility.ipynb`

内容：

- 汇总所有实验结果；
- 生成论文图表；
- 导出 LaTeX 表格；
- 保存模型和随机种子；
- 生成 mini-paper 所需 figures。

### 学生输出

- 5–8 页 mini-paper 初稿；
- 10–15 页项目展示 PPT；
- 完整代码 repo；
- 数据生成脚本；
- 主要实验图表和复现实验说明。

## 推荐最终 repo 结构

```text
microgrid-3ph-security-ai/
│
├── data/
│   ├── raw_cases/
│   ├── scenarios/
│   ├── n1_labels/
│   └── processed/
│
├── notebooks/
│   ├── 01_microgrid_and_pandapower_intro.ipynb
│   ├── 02_three_phase_power_flow.ipynb
│   ├── 03_microgrid_scenario_generation.ipynb
│   ├── 04_three_phase_nminus1_labeling.ipynb
│   ├── 05_ml_baselines_and_metrics.ipynb
│   ├── 06_physics_informed_mlp.ipynb
│   ├── 07_phase_aware_gnn_screening.ipynb
│   └── 08_report_figures_and_reproducibility.ipynb
│
├── src/
│   ├── grid_model/
│   ├── scenario_generation/
│   ├── n1_scanner/
│   ├── features/
│   ├── models/
│   ├── losses/
│   ├── metrics/
│   └── plotting/
│
├── results/
│   ├── figures/
│   ├── tables/
│   └── checkpoints/
│
├── paper/
│   ├── main.tex
│   ├── figures/
│   └── tables/
│
└── README.md
```
