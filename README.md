# gyx-study

本仓库用于开展光谱分析（spectral / spectroscopic analysis）相关研究，包括实验设计、光谱数据处理、化学计量学、机器学习建模与可复现分析。

## 当前研究文档

- [光谱分析研究报告](docs/spectral-analysis-research-report.md)

## 当前研究范围

第一阶段采用广义分析光谱学范围，重点覆盖：

- UV-Vis 紫外-可见吸收光谱
- NIR / MIR / FTIR 近红外、中红外与傅里叶变换红外光谱
- Raman / SERS 拉曼与表面增强拉曼光谱
- 荧光光谱
- 原子吸收 / 原子发射光谱
- 光谱预处理、PCA、PLS/PLSR、分类与机器学习
- 无数据泄漏验证、外部验证与可复现研究流程

> 当前报告为“研究框架版”。在明确具体样品、目标分析物、仪器型号和数据后，可继续扩展为实验方案、数据分析代码与论文级结果报告。

## 推荐后续目录

```text
gyx-study/
├─ data/
│  ├─ raw/          # 原始光谱（原则上只读）
│  ├─ interim/      # 中间数据
│  └─ processed/    # 建模数据
├─ docs/
│  └─ spectral-analysis-research-report.md
├─ notebooks/       # 探索性分析
├─ src/             # 可复用分析代码
├─ tests/
└─ results/
   ├─ figures/
   ├─ tables/
   └─ models/
```

## 研究原则

1. 原始数据不覆盖。
2. 预处理参数必须记录并纳入模型验证。
3. 同一样品的重复测量不得跨训练集和测试集，避免数据泄漏。
4. 经典化学计量学模型作为机器学习模型的基线。
5. 最终结论优先依据独立测试集 / 外部验证集，而不是训练集拟合优度。

---

Research baseline created: 2026-09-19.
