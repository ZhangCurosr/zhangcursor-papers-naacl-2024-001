---
title: "Uncertainty-Quantification-for-In-Context-Learning-of-Large"
source: https://aclanthology.org/2024.naacl-long.184.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-02 00:24:19"
field: "大语言模型可靠性与不确定性量化"
keywords: ["Uncertainty Quantification", "In-Context Learning", "Large Language Models", "Aleatoric Uncertainty", "Epistemic Uncertainty", "Entropy Decomposition", "Out-of-Distribution Detection"]
innovations: ["首次将ICL预测不确定性分解为源于演示数据的AU和源于模型配置的EU", "提出基于互信息和熵近似的可计算分解框架，适配自由文本输出", "证明EU和AU在OOD演示检测和SOOD检测中具有差异化指示价值"]
benchmarks: ["EMOTION", "Financial Phrasebank", "SST2", "COLA", "AG_News"]
---

# 论文速读：Uncertainty-Quantification-for-In-Context-Learning-of-Large

## 一句话总结
本文针对大语言模型（LLM）在上下文学习（ICL）场景下的预测不确定性，首次系统性地将其分解为**偶然不确定性**（Aleatoric Uncertainty, AU，源于演示数据）与**认知不确定性**（Epistemic Uncertainty, EU，源于模型参数/配置）。作者提出了一种基于熵和互信息的可计算分解框架，能够以插件式方式使用，帮助判断高不确定性预测的根源是来自演示质量还是模型本身。

---

## 研究问题与动机

1. **LLM上下文学习的不可靠性问题**：ICL使LLM能快速适应新任务，但幻觉（hallucination）和不可信预测问题突出，需要量化预测的不确定性以提升可靠性。
2. **现有方法忽略ICL不确定性来源的复杂性**：已有不确定性量化工作（如Likelihood、Entropy、Semantic Uncertainty等）多关注总不确定性，未能区分不确定性来自演示数据还是模型配置，缺乏对原因的诊断能力。
3. **诊断性需求**：当LLM以高不确定性做出错误预测时，用户难以判断是演示选择不当还是模型自身能力不足，进而无法针对性优化（如更换演示或改用更强模型）。
4. **自由文本输出的估计挑战**：LLM生成是自由形式（free-form），全序列熵计算会引入大量冗余token，如何从输出中精准提取与答案相关的概率分布进行熵估计是一个关键难题。

---

## 核心贡献（创新点）

1. **首次将ICL预测不确定性形式化为Bayesian Neural Network框架下的分解问题**，将总不确定性明确拆分为源于演示数据的AU和源于模型参数/配置的EU，给出可解释的数学定义。
   - 与已有工作本质区别：现有工作多仅估算总不确定性，本文首次针对ICL特有的双重不确定性来源进行分解和建模。

2. **提出基于互信息（Mutual Information）的熵分解方法**，利用条件熵与期望熵之差分别估计EU和AU，给出了闭式近似公式（公式2、3）。
   - 与已有工作本质区别：不同于仅计算序列整体token概率之和的Likelihood方法或等权重处理所有token的Entropy方法，本文从概率分解视角区分两类不确定性来源。

3. **设计针对自由文本输出的熵近似算法**：通过beam search采样多条生成序列，提取与答案直接相关的token概率，聚合为类别概率分布后再计算熵，避免了全序列冗余计算。
   - 与已有工作本质区别：Semantic Uncertainty依赖语义embedding聚类，本文直接从模型输出的token概率矩阵构造类别分布，计算更高效且可解释。

4. **提出插件式、无监督的ICL不确定性量化框架**，支持white-box LLM（如LLaMA-2开源模型），并给出适用于black-box LLM的方差分解版本（Appendix A.1）。
   - 与已有工作本质区别：无需额外标注数据或fine-tuning，直接通过采样不同演示和模型配置即可估算，具备即插即用特性。

5. **系统性实验验证与多应用场景**：在6个NLU数据集上验证分解方法对误分类检测的有效性，并进一步应用于域外演示检测（OOD Demo Detection）和语义分布外检测（SOOD Detection），证明EU和AU的不同指示价值。
   - 与已有工作本质区别：现有工作多在单一任务上评估，本文证明了分解后EU/AU在不同下游应用中的差异化优势。

---

## 方法详解

### 2.1 背景与ICL的Bayesian解释

- 将ICL视为**含隐变量的Bayesian神经网络**：给定演示序列 $\mathbf{x}_{1:T-1}$ 和测试问题 $\mathbf{x}_T$，模型输出 $\mathbf{y}_T$ 的条件分布为：
  $$p(\mathbf{y}_T | \mathbf{x}_{1:T}) = \int_{z \in \mathcal{Z}} p(\mathbf{y}_T | \mathbf{x}_{1:T}, z) \, p(z | \mathbf{x}_{1:T}) \, dz$$
  其中隐变量 $z$ 代表由演示序列推断出的"latent concept"（如任务类型、文档主题等）。

- 预测分布进一步扩展为同时含模型参数 $\Theta$ 和隐变量 $z$ 的双重积分：
  $$p(\mathbf{y}_T | \mathbf{x}_{1:T}) \approx \iint p(\mathbf{y}_T | \Theta, \mathbf{x}_{1:T}, z) \cdot p(z | \mathbf{x}_{1:T}) \, q(\Theta) \, dz \, d\Theta$$
  其中 $q(\Theta)$ 是模型参数的近似后验分布。

### 2.2 不确定性分解公式

- **总不确定性**：$H(\mathbf{y}_T | \mathbf{x}_{1:T})$，包含两类不确定性。
- **认知不确定性（EU）**：固定模型参数 $\Theta$，对演示（隐变量 $z$）取期望后的条件熵：
  $$EU = \mathbb{E}_z \left[ H(\mathbf{y}_T | \mathbf{x}_{1:T}, z, \Theta) \right]$$
  反映在给定演示条件下，仅因模型参数/配置随机性导致的不确定性。
- **偶然不确定性（AU）**：通过互信息定义为总条件熵减去EU：
  $$AU = I(\mathbf{y}_T, z | \Theta) = H(\mathbf{y}_T | \mathbf{x}_{1:T}, \Theta) - \mathbb{E}_z \left[ H(\mathbf{y}_T | \mathbf{x}_{1:T}, z, \Theta) \right]$$
  反映因演示数据多样性/噪声导致的输出不确定性。

### 2.3 熵近似估计（核心工程贡献）

针对LLM自由文本输出难以直接计算熵的问题，提出以下近似流程（图3）：

1. **Beam Search采样**：使用 beam width=10 的beam search近似对参数 $\Theta$ 的采样，生成 $M$ 条候选序列（$M$ 对应不同模型配置/采样轮次）。
2. **答案token提取**：从每条生成序列中提取与答案直接相关的token概率（如分类任务中输出类别数字的概率），忽略无关token。
3. **构建概率矩阵 $\mathcal{M}$**：对 $L$ 组不同演示集合、每组 $M$ 条序列，汇总得到一个 $K \times L$ 矩阵（$K$ 为类别数），$\mathcal{M}_{k,j}$ 表示第 $j$ 组演示下第 $k$ 类标签的累积概率。
4. **EU和AU计算**：
   $$EU = \frac{1}{L} \sum_{j=1}^{L} H\left(\sigma(\mathcal{M}_{:,j})\right)$$
   $$AU = H\left(\sigma\left(\sum_{j=1}^{L} \mathcal{M}_{:,j}\right)\right) - \frac{1}{L} \sum_{j=1}^{L} H\left(\sigma(\mathcal{M}_{:,j})\right)$$
   其中 $\sigma(\cdot)$ 为列归一化操作，$H(\cdot)$ 为分类熵。

### Black-box扩展（Appendix A.1）

对于无法获取token概率的black-box LLM（如GPT系列），提供基于**方差分解**（Law of Total Variance）的替代方案：
- EU：对模型配置 $\Theta$ 取期望后的输出方差
- AU：对隐变量 $z$（演示数据）取期望后的条件方差平均值

---

## 实验与结果

### 实验设置
- **模型**：LLaMA-2（7B、13B、70B）、OPT-13B
- **数据集**（6个NLU任务）：
  - 情感分析：EMOTION（2000测试样本，6类）、Financial Phrasebank（850样本，3类）、SST2（872样本，2类）
  - 语言可接受性：COLA（1040样本，2类）
  - 主题分类：AG_News（1160样本，4类）
- **演示采样策略**：Random（随机采样）和Class（每类至少一个）
- **评估基线**：Likelihood（Malinin & Gales, 2020）、Entropy（Xiao & Wang, 2019）、Semantic（Kuhn et al., 2023）
- **评估指标**：AUPR（Area Under Precision-Recall）和 ROC（AUROC），用于衡量不确定性分数对误分类样本的识别能力

### 主要结果

**表1 — 误分类检测（核心结果）**：

| 数据集 | 模型 | 最佳EU(AUPR/ROC) | 最佳AU(AUPR/ROC) | 最优基线Semantic(AUPR/ROC) |
|--------|------|------------------|------------------|---------------------------|
| EMOTION | LLaMA-70B-CLASS | **0.659 / 0.721** | 0.612 / 0.693 | 0.689 / 0.689 |
| Financial | LLaMA-70B-CLASS | **0.893 / 0.804** | 0.739 / 0.659 | 0.774 / 0.649 |
| SST2 | LLaMA-70B-CLASS | **0.851 / 0.362** | 0.697 / 0.697 | 0.679 / 0.331 |
| COLA | LLaMA-70B-CLASS | **0.727 / 0.425** | 0.682 / 0.682 | 0.613 / 0.397 |
| AG_News | LLaMA-70B-CLASS | **0.662 / 0.283** | 0.571 / 0.571 | 0.532 / 0.274 |

- **结论1**：EU在多数数据集上显著优于所有基线，AUPR提升幅度最大可达约**20-30个百分点**（如Financial：0.893 vs 基线0.774）。
- **结论2**：AU在部分数据集上表现亦优于基线，但在简单数据集（如SST2高准确率场景）上提升有限。
- **结论3**：**Class采样策略**普遍优于Random采样，因为确保了各类别都有代表性演示，减少了采样偏差。
- **结论4**：模型规模增大（7B→13B→70B）整体带来性能提升，但EU在部分场景下并非单调递增，反映不确定性评估的复杂性。

**表2/3 — OOD演示检测**：
- EU在检测域外演示方面表现最佳（ROC达0.935-0.941），AU因演示本身的高变异性而敏感度下降。
- 当使用相关但不同域的演示（Finance Phrasebank→EMOTION）时，AU显著下降（-7.5%至-12.5%），说明AU对演示质量高度敏感。
- 当使用完全OOD演示（COLA→EMOTION）时，AU骤降高达**-17.0%**，进一步印证AU对演示相关性的敏感性。

**表4 — 语义分布外检测（SOOD）**：
- EU在7B和13B模型上均取得最优AUPR（0.548和0.525），显著优于Semantic基线，证明EU对语义偏移具有高检测力。

**泛化性验证（图4）**：
- 在OPT-13B和LLaMA-2-13B上PR/ROC曲线趋势一致，证明方法的模型无关性；同时EU能够区分两模型能力差异（LLaMA-2-13B的EU AUROC=0.59 > OPT-13B的0.55）。

---

## 相关工作脉络

1. **不确定性分解经典方法**（Malinin & Gales, 2020; Depeweg et al., 2017）：通过贝叶斯神经网络、Deep Ensembles、Monte Carlo Dropout等将总不确定性分解为AU和EU，但面向传统分类模型，未针对LLM的ICL范式设计。
2. **LLM不确定性量化—Likelihood/Entropy方法**（Xiao & Wang, 2019, 2021; Xiao et al., 2022）：计算生成序列token概率之和或熵，作为不确定性代理指标，但未区分不确定性来源，且等权重处理所有token导致冗余。
3. **Semantic Uncertainty**（Kuhn et al., 2023）：通过语义embedding聚类生成序列并计算组间熵，是当时最先进的ICL不确定性方法，但无法溯源不确定性来自演示还是模型配置。
4. **ICL的Bayesian解释**（Xie et al., 2021）：将ICL形式化为隐变量Bayesian推断，指出模型通过演示定位潜在概念 $z$，本文在此基础上进一步分解由 $z$ 和 $\Theta$ 各自引入的不确定性。
5. **LLM校准与置信度估计**（Jiang et al., 2021; Kadavath et al., 2022; Fadeeva et al., 2023）：关注LLM输出置信度的校准或自评估能力，但未系统分解ICL场景下的不确定性来源。

---

## 局限性与未来方向

1. **任务适用范围受限**：论文明确指出当前方法主要面向**确定性NLP任务**（如文本分类、多选择QA、语言可接受性判断），不适用于自由生成任务（如故事生成、对话），因为无法确定生成序列中哪些token与"答案"相关。
2. **依赖white-box访问**：熵分解方法需要获取token级概率分布，仅适用于white-box LLM（如开源LLaMA-2系列）；虽然Appendix提供了black-box的方差分解版本，但精度可能较低。
3. **演示采样数量固定**：实验中使用固定数量演示（2-6个），未系统探索演示数量对不确定性估计精度的影响。
4. **未来方向**：作者计划将方法扩展到其他数据类型和任务（如生成任务、多模态任务），以进一步验证通用性。

---

## 研究启发与可借鉴点

1. **不确定性的来源分解思路可迁移**：将总不确定性拆分为"数据侧"（演示质量）和"模型侧"（参数/配置）两个成分，这一框架可推广至其他少样本/上下文学习场景，如检索增强生成（RAG）中区分检索结果质量与模型本身不确定性的贡献。
2. **答案token概率提取策略**：通过指令让LLM输出纯数字/类别标签，再直接提取对应token的概率，避免了自由生成中熵计算的冗余问题——这一技巧可直接复用于其他需要ICL不确定性的下游任务。
3. **EU与AU的差异化应用价值**：实验揭示EU擅长检测模型/演示配置的不确定性（如OOD演示、SOOD），而AU对演示数据质量更敏感——这一洞察可用于设计自适应ICL系统：高EU时切换模型或调整解码策略，高AU时更换演示集。
4. **Class采样优于Random采样的实验设计**：在Few-shot演示选择中确保类别覆盖的采样策略值得在后续研究中作为强基线或标准实践。
5. **插件式评估接口**：方法无需fine-tuning即可附加到任意ICL pipeline中，为构建"不确定性感知"的LLM应用系统提供了低成本的参考范式。

---

## 关键术语表

**In-Context Learning (ICL)**：指在不更新模型参数的情况下，通过在prompt中提供少量任务示例（demonstrations），使LLM能够快速适应并完成目标任务的能力。

**Aleatoric Uncertainty (AU)**：偶然不确定性，源于数据本身的噪声或演示样本的多样性/不匹配，反映输入端（而非模型）带来的预测不确定性。

**Epistemic Uncertainty (EU)**：认知不确定性，源于模型参数/配置的不确定性（如解码策略、温度参数等），反映模型自身知识不足或配置随机性导致的不确定性。

**Mutual Information (MI)**：互信息，衡量两个随机变量之间的信息共享程度；本文用于量化输出 $\mathbf{y}_T$ 与隐变量 $z$ 之间的关联，以此估计AU。

**Beam Search**：束搜索，一种生成解码策略，同时保留概率最高的若干候选序列；本文用它来近似对模型参数后验 $q(\Theta)$ 的采样。

**Out-of-Distribution (OOD) Demonstration**：域外演示，指与目标任务领域不相关或不匹配的演示样本，可能导致LLM被误导而产生不可靠预测。

**Semantic Out-of-Distribution (SOOD) Detection**：语义分布外检测，指识别测试样本与演示在语义上存在偏移的情况（如任务描述中隐藏了正确类别）。

**Free-form Output**：自由形式输出，指LLM生成不受固定类别约束的自然语言序列，使得熵计算不能简单地对整个序列进行。

---

## 可复现要素

- **数据集**：EMOTION、Financial Phrasebank、SST2、COLA、AG_News——均为公开数据集，可在原始论文引用中获取。
- **代码**：已开源，地址为 https://github.com/lingchen0331/UQ_ICL（论文声明）。
- **模型权重**：LLaMA-2（7B/13B/70B-Chat版本）和OPT-13B权重可从HuggingFace获取。
- **关键超参**：
  - Beam width = 10
  - Max new tokens = 16（统一设置）
  - 演示采样轮次：每个数据集均匀采样4次
  - 各数据集演示数量：EMOTION(6)、Financial(6)、SST2(4)、COLA(2)、AG_News(4)
- **Prompt模板**：论文Appendix A.4提供了统一格式，包含System Prompt、Task Description、Few-shot Demonstrations和Test Query四部分。

---
