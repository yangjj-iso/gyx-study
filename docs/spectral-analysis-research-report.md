# 光谱分析研究报告

**项目：** gyx-study  
**版本：** v1.0（研究框架版）  
**日期：** 2026-09-19  
**研究范围：** 分析化学中的广义光谱分析、化学计量学与机器学习辅助分析

---

## 摘要

光谱分析通过物质与电磁辐射之间的相互作用获取组成、结构、浓度和状态信息。IUPAC 将 spectroscopy 定义为研究物理系统与其相互作用或由其产生的电磁辐射；spectrometry 则强调对这些辐射进行测量，以获取系统及其组分的信息 [1]。

本报告建立一个可直接用于后续实验与代码实现的光谱分析研究框架。核心建议如下：

1. **先定义分析问题，再选择光谱技术。** 定性鉴别、定量浓度预测、痕量检测、结构表征和在线监测的最优技术并不相同。
2. **原始光谱、预处理和建模必须作为同一条分析管线验证。** SNV、MSC、Savitzky–Golay（SG）平滑/导数、基线校正等方法没有普适最优解，应通过验证集或外部测试集选择 [5][6]。
3. **经典化学计量学应作为机器学习的基线。** PCA、PLS/PLSR、PLS-DA 等模型具有较强可解释性；SVM、随机森林、梯度提升与 1D-CNN 可用于捕获非线性和复杂峰形关系。
4. **避免数据泄漏比追求更复杂模型更重要。** 同一样品的重复扫描、同一批次或同一受试对象的光谱不能随机拆散到训练集与测试集。
5. **最终评价应依赖独立测试或外部验证。** 仅凭训练集的高 R²、准确率或低 RMSE 不能证明模型可推广。
6. **研究应从第一天就可复现。** 保留原始数据、元数据、仪器参数、随机种子、预处理参数、数据划分和模型版本。

在尚未指定具体样品、目标分析物和仪器的情况下，本报告不假定某一种光谱技术必然最优，而是提供一个可用于食品、材料、环境、生物样品、药物或工业过程分析的通用研究设计。

---

## 1. 研究目标

本项目建议把“光谱分析研究”拆成四类可验证目标：

### 1.1 定性鉴别

回答“样品是什么 / 属于哪一类”。

典型任务：

- 原料或化合物身份确认；
- 材料、药品、食品类别识别；
- 正常/异常样品区分；
- 污染、掺假或批次差异筛查。

常用方法：峰位/峰形匹配、光谱库检索、PCA、HCA、PLS-DA、SVM、随机森林、1D-CNN。

### 1.2 定量分析

回答“某个组分有多少”。

典型任务：

- 浓度、含量、水分、蛋白、糖、污染物等预测；
- 多组分同时定量；
- 在线过程参数估计。

常用方法：Beer–Lambert 定律、单变量/多变量校准、PLSR、岭回归、SVR、树模型和神经网络。

### 1.3 结构与机理分析

回答“分子或材料结构发生了什么变化”。

典型任务：

- 官能团变化；
- 分子振动模式；
- 电子跃迁；
- 晶型、聚合、氧化或反应过程变化；
- 峰位移动与峰宽变化。

常用技术：FTIR、Raman、UV-Vis、荧光，以及与其他表征技术联用。

### 1.4 快速筛查与现场检测

回答“能否低成本、快速、无损或在线判断”。

重点关注：

- 测量速度；
- 样品前处理复杂度；
- 便携式仪器；
- 光谱漂移；
- 仪器间模型迁移；
- 长期校准稳定性。

---

## 2. 主要光谱技术比较

| 技术 | 主要信息来源 | 典型输出 | 优势 | 主要局限 | 推荐研究任务 |
|---|---|---|---|---|---|
| UV-Vis | 电子跃迁与吸收 | 吸光度-波长 | 简单、成熟、定量方便 | 选择性可能不足，谱峰易重叠 | 有色/吸收组分定量、动力学 |
| NIR | 分子振动倍频/组合频 | 反射/透射光谱 | 快速、无损、适合在线 | 峰宽、重叠强，依赖化学计量学 | 食品、农业、制药、过程分析 |
| MIR / FTIR | 基频分子振动 | 吸收/ATR 光谱 | 官能团信息强、指纹区丰富 | 水/基质影响、采样方式敏感 | 结构鉴别、材料/生物样品分类 |
| Raman | 非弹性散射 | Raman shift-强度 | 水背景相对弱、分子指纹强 | 信号较弱、荧光背景可能严重 | 材料、药物、细胞、原位分析 |
| SERS | 表面增强 Raman | 高灵敏 Raman | 可显著增强弱信号 | 基底一致性、重复性与定量挑战 | 痕量检测、环境/生物标志物 |
| 荧光 | 激发后发射 | 发射/激发光谱 | 灵敏度高 | 猝灭、基质效应、漂白 | 高灵敏检测、探针分析 |
| AAS | 原子吸收 | 元素特征吸收 | 元素分析成熟 | 多元素效率有限 | 金属元素定量 |
| ICP-OES 等原子发射 | 原子/离子发射 | 元素发射谱线 | 多元素、高通量 | 仪器成本和样品消解 | 多元素定量 |

NIST Chemistry WebBook 提供大量标准参考光谱，包括超过 16,000 个化合物的 IR 光谱和超过 1,600 个化合物的 UV/Vis 光谱，可作为定性识别、峰位核对和方法开发中的重要参考数据源 [2]。

---

## 3. 光谱定量的基础：从单变量到多变量

### 3.1 Beer–Lambert 基线模型

在满足适用条件时，吸收型光谱可用：

```text
A = ε b c
```

其中：

- `A`：吸光度；
- `ε`：摩尔吸光系数；
- `b`：光程；
- `c`：浓度。

如果存在明显单一特征峰、基质简单且线性关系稳定，单波长校准是最简单、最容易解释的定量方案。

但真实样品常存在：

- 多组分峰重叠；
- 散射；
- 基线漂移；
- 仪器漂移；
- 温度影响；
- 粒径和光程差异；
- 非线性响应。

因此复杂样品通常需要多变量化学计量学。

### 3.2 多变量校准

推荐建立以下基线层级：

1. 单变量线性回归；
2. 多元线性回归（适用于变量较少且共线性有限）；
3. PCA + 回归/分类；
4. PLSR / PLS-DA；
5. SVR / SVM；
6. 随机森林 / 梯度提升；
7. 1D-CNN 等深度学习模型。

模型复杂度应随着数据量和问题复杂度增加，而不是从深度学习开始。

---

## 4. 数据采集与实验设计

### 4.1 必须记录的元数据

每条光谱至少建议关联以下信息：

- sample_id：真实样品唯一编号；
- replicate_id：重复测量编号；
- batch_id：制备或实验批次；
- instrument_id：仪器编号；
- operator：操作人员；
- acquisition_time：采集时间；
- temperature / humidity：必要时记录环境条件；
- wavelength_range / wavenumber_range；
- resolution；
- integration_time；
- scans / accumulations；
- laser_wavelength / laser_power（Raman）；
- path_length（透射测量）；
- sampling_mode（ATR、透射、漫反射等）；
- reference_value：标准方法获得的真实值；
- class_label：分类标签（如有）。

### 4.2 重复测量

建议区分：

- **技术重复**：同一样品重复扫描；
- **制备重复**：同一样品重新制备后测量；
- **生物/真实样品重复**：独立来源样品。

模型验证时，技术重复应始终跟随其母样品进入同一数据分区。

### 4.3 校准集、验证集与测试集

建议优先采用：

```text
全部独立样品
├─ 训练/校准集
│  └─ 内部交叉验证：选择预处理、超参数、潜变量数
└─ 独立测试集
   └─ 仅用于最终一次性能评估
```

若存在多批次、多地点、多仪器或多日期数据，进一步使用：

- leave-one-batch-out；
- leave-one-instrument-out；
- 时间外推验证；
- 外部实验室验证。

这比单纯随机拆分更接近真实使用场景。

---

## 5. 光谱预处理

光谱预处理的目标不是“让曲线更漂亮”，而是降低与研究目标无关的变化，同时保留化学信息。综述和近期方法研究均指出，没有一种预处理方法适合所有数据集 [5][6]。

### 5.1 推荐候选方法

#### 平滑

- Savitzky–Golay smoothing；
- 移动平均（通常只作为简单基线）。

用途：降低高频噪声。

风险：窗口过大会抹掉窄峰和真实细节。

#### 基线校正

- 多项式基线；
- AsLS / airPLS 等；
- 一阶/二阶导数。

用途：处理背景漂移、荧光背景和缓慢变化基线。

#### 散射校正

- SNV（Standard Normal Variate）；
- MSC（Multiplicative Scatter Correction）；
- EMSC。

用途：处理粒径、散射和乘性效应，尤其常见于 NIR、漫反射光谱。

#### 导数

- SG 一阶导数；
- SG 二阶导数。

用途：减弱偏移/趋势并增强重叠峰分辨能力。

风险：放大噪声，因此通常与平滑联合使用。

#### 标准化与缩放

- mean centering；
- autoscaling；
- vector normalization；
- area normalization。

注意：自动标准化不一定总是合理。例如对高噪声变量赋予相同权重可能降低模型稳定性 [7]。

### 5.2 正确的预处理比较方式

不能先对全数据预处理再划分训练/测试集。

正确流程是：

```text
split samples
    ↓
fit preprocessing on training set
    ↓
transform training / validation
    ↓
select pipeline
    ↓
freeze preprocessing + model
    ↓
apply once to untouched test set
```

任何从全部数据估计均值、标准差、PCA 载荷、MSC 参考谱或特征选择阈值的操作，都可能产生数据泄漏。

---

## 6. 探索性分析

### 6.1 原始谱检查

第一步始终应画出：

- 所有原始光谱；
- 每类样品均值 ± 变异；
- 重复扫描；
- 空白/背景；
- 标准品；
- 随时间变化的 QC 光谱。

重点排查：

- 饱和；
- 截断；
- 宇宙射线尖峰（Raman）；
- 异常基线；
- 波数/波长漂移；
- 信噪比极低；
- 仪器切换造成的系统偏移。

### 6.2 PCA

PCA 推荐用于：

- 观察聚类趋势；
- 检查批次效应；
- 发现离群样本；
- 判断主要变化方向；
- 对比不同预处理是否引入人工结构。

PCA 是探索工具，不应仅凭二维得分图宣称样品类别已经被可靠区分。

### 6.3 峰分析

根据技术可分析：

- 峰位；
- 峰高；
- 峰面积；
- 半峰宽；
- 峰比值；
- Raman shift；
- 导数极值；
- 指纹区。

峰归属应优先参考标准品、文献和可靠数据库，而不是仅依赖模型特征重要性。

---

## 7. 建模策略

### 7.1 定量模型

建议比较：

| 模型 | 角色 |
|---|---|
| Linear / Ridge | 最简单基线 |
| PLSR | 光谱定量经典基线 |
| SVR | 处理中小数据集的非线性关系 |
| Random Forest / Extra Trees | 非线性、交互关系 |
| Gradient Boosting | 高性能表格/特征模型 |
| 1D-CNN | 数据量足够时直接学习局部峰形 |

定量指标：

- R²；
- RMSE；
- MAE；
- Bias；
- 必要时 RPD / RPIQ；
- 浓度分段误差。

所有指标应分别报告交叉验证与独立测试结果。

### 7.2 分类模型

建议比较：

- Logistic Regression；
- PLS-DA；
- SVM；
- Random Forest；
- Gradient Boosting；
- 1D-CNN。

分类指标：

- Accuracy；
- Balanced Accuracy；
- Precision；
- Recall / Sensitivity；
- Specificity；
- F1；
- ROC-AUC（适用时）；
- 混淆矩阵。

对于类别不平衡数据，不应只报告 Accuracy。

### 7.3 深度学习的使用条件

1D-CNN 的优势是可以直接从连续光谱中学习局部峰结构，但应满足：

- 独立样品数量足够；
- 有真正独立的测试集；
- 与 PLS/SVM 等基线公平比较；
- 记录模型结构与随机种子；
- 不用技术重复人为扩大“样本量”。

Raman/SERS 等领域已经广泛探索机器学习与深度学习，但高维、峰重叠、背景和跨场景泛化仍是主要挑战 [8]。

---

## 8. 推荐的研究基准实验

在具体样品尚未确定前，建议项目先建立一个“标准 Benchmark”。

### 实验 A：预处理基准

候选 pipeline：

1. Raw；
2. SG smoothing；
3. SNV；
4. MSC；
5. SG 1st derivative；
6. SG 2nd derivative；
7. SNV + SG derivative；
8. MSC + SG derivative。

对每个 pipeline 使用相同的数据划分和相同模型，避免比较不公平。

### 实验 B：模型基准

在最佳若干预处理条件下比较：

- PLSR / PLS-DA；
- SVM / SVR；
- Random Forest；
- Gradient Boosting；
- 1D-CNN（数据规模允许时）。

### 实验 C：泛化测试

使用独立变量制造真实分布变化：

- 不同日期；
- 不同批次；
- 不同仪器；
- 不同操作者；
- 不同样品来源。

目标是回答：模型是否只记住实验条件，还是学到了稳定的化学信息。

### 实验 D：可解释性

对于最终模型：

- PLS：查看 loadings / VIP；
- 线性模型：查看系数；
- 树模型：Permutation Importance / SHAP；
- CNN：可做敏感区域或 attribution 分析。

模型强调的重要波段必须回到化学/物理机理进行验证。

---

## 9. 推荐研究流程

```text
研究问题定义
      ↓
样品与参考方法设计
      ↓
光谱采集 + QC
      ↓
原始数据冻结
      ↓
样品级数据划分
      ↓
EDA / PCA / 异常检查
      ↓
预处理候选管线
      ↓
经典化学计量学基线
      ↓
机器学习模型
      ↓
内部交叉验证
      ↓
冻结最终 pipeline
      ↓
独立 / 外部测试
      ↓
误差与失败案例分析
      ↓
化学解释 + 报告 + 可复现代码
```

---

## 10. 软件与代码建议

推荐 Python 技术栈：

- `numpy`：数组计算；
- `pandas`：元数据/结果表；
- `scipy`：信号处理；
- `scikit-learn`：PCA、PLS、SVM、交叉验证、Pipeline；
- `matplotlib`：光谱图与结果图；
- `pybaselines`：基线校正；
- `pytorch`：深度学习（需要时）。

建议把所有预处理与模型封装成 Pipeline，使交叉验证时每个 fold 独立拟合预处理参数。

伪代码：

```python
pipeline = Pipeline([
    ("preprocess", SpectralPreprocessor(...)),
    ("model", PLSRegression(...)),
])

scores = cross_validate(
    pipeline,
    X,
    y,
    groups=sample_groups,
    cv=group_cv,
)
```

---

## 11. 数据与仓库规范

建议仓库结构：

```text
gyx-study/
├─ data/
│  ├─ raw/              # 原始光谱，只读
│  ├─ interim/
│  └─ processed/
├─ metadata/
│  └─ samples.csv
├─ docs/
│  └─ spectral-analysis-research-report.md
├─ notebooks/
│  ├─ 01_qc_eda.ipynb
│  ├─ 02_preprocessing.ipynb
│  └─ 03_model_benchmark.ipynb
├─ src/
│  ├─ preprocessing.py
│  ├─ features.py
│  ├─ modeling.py
│  └─ evaluation.py
├─ tests/
├─ results/
│  ├─ figures/
│  ├─ tables/
│  └─ models/
├─ README.md
└─ requirements.txt
```

对于体积较大的原始光谱，不建议直接无限制提交到 Git 历史。可根据数据规模选择 Git LFS、对象存储或外部数据仓库，并在仓库中保存校验值、数据字典与下载说明。

---

## 12. 质量控制与失败模式

### 12.1 常见失败原因

- 样本量指的是“光谱条数”而不是“独立样品数”；
- 技术重复进入不同数据集；
- 全数据先做 PCA/标准化再交叉验证；
- 预处理参数凭视觉选择；
- 只报告训练集指标；
- 只选择表现最好的模型而不记录全部尝试；
- 用模型重要波段直接替代机理证据；
- 不记录仪器和批次信息；
- 校准浓度范围远窄于实际应用范围；
- 测试集与未来实际样品分布不一致。

### 12.2 最低质量门槛

研究结果进入“可报告”阶段前，至少应满足：

- 有独立样品级测试集；
- 数据划分规则可复现；
- 原始数据未被覆盖；
- 完整保存预处理参数；
- 至少一个经典基线模型；
- 至少两个误差指标；
- 对异常样品进行单独分析；
- 图表和结果可由脚本重新生成。

---

## 13. 项目阶段计划

### Phase 0：问题定义

输出：

- 分析对象；
- 目标物/标签；
- 光谱技术；
- 标准参考方法；
- 预期检测范围；
- 使用场景。

### Phase 1：数据采集与 QC

输出：

- 第一批原始光谱；
- 元数据；
- QC 图；
- 重复性评估；
- 初步峰归属。

### Phase 2：化学计量学基线

输出：

- Raw / SNV / MSC / SG / 导数比较；
- PCA；
- PLSR 或 PLS-DA；
- 交叉验证与独立测试结果。

### Phase 3：机器学习

输出：

- SVM / SVR；
- 树模型；
- 1D-CNN（适用时）；
- 模型公平比较；
- 特征/波段解释。

### Phase 4：外部验证

输出：

- 跨批次/跨日期/跨仪器结果；
- 模型漂移分析；
- 失败案例；
- 最终可部署 pipeline。

---

## 14. 当前结论

对于一个新的光谱研究项目，最值得优先投入的并不是“寻找最复杂算法”，而是建立高质量的**样品设计—参考值—光谱采集—元数据—无泄漏验证**链路。

推荐第一轮研究策略为：

> **FTIR/Raman/NIR 等具体技术确定后，以原始光谱 + PCA 为起点，以 PLS 为经典基线，对 SNV、MSC、SG 平滑和 SG 导数进行受控比较，再引入 SVM/树模型/1D-CNN，并最终通过独立样品或跨批次外部验证确定模型。**

下一步最关键的信息是：

1. 研究的具体样品是什么；
2. 要识别类别还是预测浓度；
3. 已有哪种光谱仪；
4. 光谱数据格式是什么；
5. 是否有参考测量值。

获得这些信息后，本报告可以进一步转化成**具体实验 SOP + Python 分析代码 + 第一版模型 benchmark**。

---

## 参考资料

[1] IUPAC Gold Book, “spectroscopy”, Compendium of Chemical Terminology, 5th ed.  
https://goldbook.iupac.org/terms/view/S05848

[2] NIST Chemistry WebBook, SRD 69.  
https://webbook.nist.gov/

[3] NIST Chemistry WebBook — UV/Vis Database User's Guide.  
https://webbook.nist.gov/chemistry/uv-vis/

[4] IUPAC Gold Book, “molecular spectroscopy”.  
https://goldbook.iupac.org/terms/view/08217

[5] Rinnan, Å.; van den Berg, F.; Engelsen, S. B. Review of the most common pre-processing techniques for near-infrared spectra. *TrAC Trends in Analytical Chemistry* (2009).  
https://www.sciencedirect.com/science/article/pii/S0165993609001629

[6] *Spectral data preprocessing methods and strategies*. In: Analytical and Chemometric Tools: Tutorials with Practical Applications to Food Security and Safety (2026).  
https://www.sciencedirect.com/science/chapter/edited-volume/pii/B9780443215865000024

[7] Unraveling surface-enhanced Raman spectroscopy results through chemometrics and machine learning: principles, progress, and trends.  
https://pmc.ncbi.nlm.nih.gov/articles/PMC9981450/

[8] Machine Learning-Assisted Surface-Enhanced Raman Spectroscopy Detection for Environmental Applications: A Review (2024).  
https://pmc.ncbi.nlm.nih.gov/articles/PMC11603787/

[9] In Vitro Glucose Measurement from NIR and MIR Spectroscopy: Comprehensive Benchmark of Machine Learning and Filtering Chemometrics (2024).  
https://pmc.ncbi.nlm.nih.gov/articles/PMC11108977/

---

## 变更记录

- **v1.0 — 2026-09-19**：建立通用光谱分析研究框架、方法比较、验证原则与项目实施路线。
