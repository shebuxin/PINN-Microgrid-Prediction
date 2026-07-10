# 仓库一致性审查报告

**审查日期：** 2026-07-09
**审查范围：** 公式、事实与版本、Notebook 代码、实验数据、LaTeX/PDF、排版和发布结构。

## 1. 结论

本轮审查已完成修正和回归验证，当前仓库可以作为课程版本使用。

- 15 个 clean notebook 在全新临时副本中从零执行，结果 **15/15 通过**。
- Week 1-8 的 clean/executed notebook 源码逐 cell 一致，clean 版本无输出。
- Week 2-8 自带 validation summary 全部通过。
- 20 个主 TeX 入口全部编译成功，包括基础课、研究设计、Week 1-8、论文包和本报告。
- Week 1-8 slides 页数为 `29, 42, 34, 40, 36, 41, 37, 42`，合计 **301 页**。
- Week 3-8 的字体和图片路径已改为仓库内可移植写法，不依赖外部临时目录或本机字体名。
- 逐页视觉检查未发现公式遮挡、文字截断或对象重叠。

## 2. 主要修正

### 公式与物理口径

- 修正 L03 正序等值导纳中 `a` 与 `a^2` 的系数顺序。
- 修正 L02 中 BFS 收敛值与 LinDistFlow 近似值混写的问题。
- 修正 L04 pandapower 三相示例的线路参数、零序参数和外部电网短路参数。
- 明确 `I=(S/V)^*` 中 `*` 为复共轭，`\oslash` 为逐元素相除。
- 区分 IEC 序分量 VUF 与 NEMA MG 1 线电压不平衡率，两者不再写成等价定义。
- Week 4 的 VUF 阈值统一为 2%，线路 loading 使用 `max_i_ka × df × parallel` 的有效额定电流。
- Week 6 的六项损失、SmoothL1/log1p、连续 severity ranking 和权重已与 notebook 实现对齐。
- 修复研究故事线文档中 KCL、支路电流、Fortescue、序一致性、孤岛功率平衡和 XAI 损失公式的 Markdown 损坏。

### 代码与数据链

- 统一 Week 1-8 输出到各周 `outputs/`，移除外部临时目录和错误的跨周相对路径。
- 统一 Week 3 validation 文件名，修复 Week 8 下游读取接口。
- 所有 notebook 补齐稳定 cell ID，并统一 `Python (pdpower)` kernelspec。
- manifest 改用仓库相对路径，不再写入 `/Users/...` 本机绝对路径。
- Week 8 生成的论文表格会转义 LaTeX 特殊字符，并限制到版心宽度。
- 论文包与 Week 8 生成结果已同步。

### 版本与环境

课程复现环境固定为本轮实测组合：

```text
Python 3.11
pandapower 3.2.1
NumPy 1.26.4
pandas 2.2.3
SciPy 1.13.1
scikit-learn 1.8.x
PyTorch 2.10.x
```

教材中的公开资料链接按 2026-07-09 核对；pandapower 当前资料引用指向 3.3.2 文档。课程运行仍固定 3.2.1，以保证本仓库结果可复现。

## 3. 数据一致性

核心数据链为：

```text
7 scenarios × 11 contingencies = 77 samples
safe = 30, unsafe = 47
PF-solved = 35
service-loss skipped before PF = 35
PCC/no-slack skipped = 7
```

当前 holdout 结果中，GradientBoosting、PI-MLP full 和 FlattenedMLP 均为零 missed violation，并保存 40% PF calls。这只适合作为 77 样本 teaching MVP 的结果，不应外推为部署性能。

PhaseAwareGNN 在当前小样本上未超过 FlattenedMLP。slides 和 mini-paper 已按负结果如实表述，没有声称 GNN 天然优于 MLP。

## 4. 仍需保留的研究边界

- 9-bus feeder 是教学系统，不是论文级 benchmark。
- radial feeder 的 line outage 会产生大量 service loss，必须与 PF-solvable outage 分开报告。
- PCC outage 当前表示 no-slack/infeasible，不等价于完整的 grid-forming islanded control。
- BESS 是静态 setpoint，没有 SOC 时序约束。
- 正式论文仍需更大 feeder、更多随机/OOD 场景、feeder-group split 和保守校准。

## 5. 使用入口

- 课程索引：`00_overview/course_material_index.md`
- 复现环境：`environment.yml`
- 完整验证表：`00_overview/overall_review_validation_summary.csv`
- 论文包：`90_paper_package/`
- 展示提纲：`91_presentation_package/final_presentation_outline.md`
- Week 1-8 合并课件：`92_combined_slides/all_weeks_slides_combined.pdf`
