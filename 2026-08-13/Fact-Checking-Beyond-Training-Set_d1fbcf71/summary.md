---
title: "Fact-Checking-Beyond-Training-Set"
source: https://aclanthology.org/2024.naacl-long.124.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:16:46"
field: "自动事实核查与跨域泛化"
keywords: ["fact-checking", "domain-adaptation", "dense-retrieval", "adversarial-training", "representation-alignment"]
innovations: ["retriever分离式对抗训练使目标域表征对齐源域空间", "reader反转数据增强提升顺序不变性鲁棒性", "自动构建多域事实核查基准8个跨域场景"]
benchmarks: ["MultiFC", "Snopes"]
---

# 论文速读：Fact-Checking-Beyond-Training-Set

## 一句话总结
本文研究了自动事实核查系统在跨域迁移时的性能退化问题，提出了一种针对retriever的对抗训练方法和针对reader的输入顺序不敏感训练策略，使pipeline在未见域上取得更好的泛化表现。

## 研究问题与动机
- **核心问题**：当前主流的事实核查pipeline（retriever-reader架构）在源域训练后应用到目标域时，性能显著下降，缺乏系统性的域自适应方法。
- **现有工作不足**：
  - 先前研究多聚焦于相似平台间的迁移（如Wikipedia到科学存储库），存在领域共享偏差，无法真实评估跨域泛化能力。
  - 现有自适应方法仅依赖预训练后直接测试，缺乏针对retriever和reader组件的具体改进算法。
  - 公开的多主题事实核查基准数据集稀缺，阻碍了系统性的跨域评估。
  - 大型语言模型（如GPT-4）因知识截止日期限制，难以验证近期时事性主张（如特朗普起诉事件）。

## 核心贡献（创新点）
- **retriever对抗训练算法**：先在源域标注数据上训练biencoder，然后冻结源域claim/document编码器，在无标签目标域数据上对抗训练目标域claim和document编码器，使目标域表征对齐源域空间；与已有工作的区别在于采用分离式对抗训练而非端到端联合优化，避免了表征缓存带来的训练不稳定问题。
- **reader顺序不敏感训练策略**：利用语言模型在检测输入语句反转关系上的固有弱点，通过将claim和evidence文档的顺序反转进行数据增强，使reader对学习顺序变化具有鲁棒性；与已有工作的区别在于这一发现揭示了LLM在传递关系推理上的系统性缺陷，并针对性设计了增强方案。
- **多场景自动构建基准**：提出简单的自动方法将MultiFC和Snopes两个现有数据集重新划分为8个跨域事实核查场景（涵盖Politics、Misc、Arts、Business等），填补了公开多主题数据集的空白；与已有工作相比，首次在多种真实域间评估了pipeline的泛化性能。

## 方法详解
- **整体架构**：采用标准retriever-reader pipeline，retriever使用biencoder架构（claim编码器$f_c$和document编码器$f_d$），通过点积计算相似度；reader使用RoBERTa分类器，输入为claim与证据文档的拼接序列。
- **Reviser对抗训练（Section 3.2）**：
  - 源域biencoder使用对比学习损失训练（公式1）：
    $$\mathcal{L}_{f^s} = \sum_{i=1}^{n_s} -\log \frac{\exp(sim(c_i^s, D_{i+}^s))}{\exp(sim(c_i^s, D_{i+}^s)) + \sum_{j=1}^r \exp(sim(c_i^s, D_{j,i-}^s))}$$
  - 对抗训练目标claim编码器（公式2、3）：固定源域$f_c^s$，训练目标域$f_c^t$和判别器$g_c$，使目标域claim表征逼近源域分布。
  - 同理训练目标document编码器（公式4、5）：固定$f_d^s$，训练$f_d^t$和$g_d$。
  - 预训练阶段：使用T5模型为无标签目标域文档生成伪query，预训练biencoder后再进行对抗训练。
- **Reader表示对齐与增强（Section 3.3）**：
  - 距离对齐损失（公式6）：
    $$\mathcal{L} = \frac{1}{n_s}\sum_{i=1}^{n_s} J(\theta(f_r(x_i^s)), y_i^s) + \lambda \mathcal{D}(f_r(X^s), f_r(X^t))$$
  - 使用相关性对齐（CORAL）度量源域和目标域分布差异（公式7）：
    $$\mathcal{D} = \frac{1}{4d^2}|M^s - M^t|_F^2$$
    其中$M^s, M^t$为源域和目标域协方差矩阵。
  - **反转数据增强**：将输入序列$(C \parallel D)$与$(D \parallel C)$同时用于训练，使reader对顺序变化不敏感，增强训练信号。
- **测试阶段加权聚合（Section 3.4）**：
  - 对Top-k检索结果按排名分配权重，从第1个文档开始迭代生成k个子集，最终预测取各子集预测的平均值（公式8）：
    $$O = \frac{1}{k}\sum_{i=1}^k \left(\frac{\sum_{j=1}^i \theta(f_r(c^t \| D_{j+}^t))}{i}\right)$$

## 实验与结果
- **数据集**：MultiFC（5个域：Arts/3788条、Business/1943条、Misc/7968条、Politics/9350条、Sensitive/2180条）和Snopes（2个域：General/4190条、News/1620条）；构建8个跨域场景（M→A/B/S、P→A/B/S、G→N、N→G）。
- **评估基线**：BM25、Contriever、GPL、Promptagator（使用GPT-4生成伪query）、DAPT（域自适应预训练）+ NLI预训练reader。
- **主要结果**：
  - **Pipeline整体**：MultiFC平均F1达0.618（领先次优baseline 0.605约1.3%），Snopes平均F1达0.437（领先次优0.420约1.7%）；最强场景P→S达到0.643 F1。
  - **Reader组件**：在nli-ft基线上提升约1-4个百分点，MultiFC平均达0.637，Snopes平均达0.467。
  - **Retriever组件**：MultiFC NDCG@10平均0.784（领先GPL的0.774约1%），Snopes平均0.710（领先Promptagator的0.702约1.1%）。
  - **与GPT-3对比**：本方法在MultiFC和Snopes上均大幅领先GPT-3（多0.06-0.14 F1）。
- **消融实验**：对抗训练、对齐损失、反转增强、排名加权四个组件均有正向贡献；Table 6显示反转增强可使GPT-3正确推断传递关系，验证了LLM在该任务上的系统性缺陷。

## 相关工作脉络
- **Augenstein et al. (2019) MultiFC**：构建了多源事实核查数据集，但未提出跨域自适应方法，仅在各网站内部独立训练评估；本文在此基础上进一步挖掘域间迁移问题。
- **Wadden et al. (2020) SciFact**：将Wikipedia预训练的pipeline应用于科学事实核查，但两类数据共享领域知识，存在评估偏差；本文刻意选择语义距离更大的域（如Politics vs Misc）进行严格测试。
- **Dai et al. (2023) Promptagator**：使用GPT-4生成伪query进行域自适应检索，依赖昂贵的大模型推理；本文方法仅需对抗训练，无需大模型，成本更低且效果相当或更优。
- **Xin et al. (2021)**：采用缓存历史表征进行对抗域适应，训练不稳定；本文分离式对抗训练避免了该问题。
- **Guo et al. (2022) 综述**：系统梳理了事实核查技术，但未深入讨论域泛化问题；本文填补了这一研究空白。
- **Berglund et al. (2023) Reversal Curse**：首次揭示LLM无法从"A是B"推断"B是A"的局限性；本文将其应用于事实核查任务并设计了针对性缓解方案。

## 局限性与未来方向
- **仅限文本数据**：未考虑知识图谱等多模态证据来源的事实核查场景。
- **仅英文实验**：受限于多语言事实核查数据集的缺失，未在其它语言上验证泛化能力。
- **单数据集内域划分**：虽构建了多域场景，但均来自两个数据集的自动划分，缺乏人工标注的大规模多域基准。
- **未探索端到端训练**：当前分步优化各组件，可能存在误差累积；未来可研究联合训练方案。
- **大模型直接验证能力不足**：Figure 1显示GPT-4等模型因知识截止限制，无法验证近期时事，说明参数化知识不足以替代检索增强架构。

## 研究启发与可借鉴点
- **分离式对抗训练策略**：将retriever的两个编码器分开进行对抗训练，比端到端联合优化更稳定，可迁移到其他检索自适应任务。
- **顺序不敏感训练的通用价值**：利用语言模型的传递推理缺陷设计数据增强，适用于任何依赖输入顺序的NLI类任务（如情感分析、文本蕴含）。
- **CORAL对齐损失的低成本优势**：仅需计算协方差矩阵距离，无需额外判别器网络，适合资源受限的域自适应场景。
- **排名加权聚合策略**：测试阶段的迭代加权聚合简单有效，可推广至其他基于检索的排序任务。
- **自动域划分的可行性**：利用Google内容分类API和LDA主题模型自动挖掘多域结构，为缺乏标注基准的研究提供了低成本的数据构建范式。

## 关键术语表
**Reviser-Reader Pipeline**：事实核查的标准架构，由检索器（retriever，负责召回相关证据文档）和阅读器（reader，负责判断主张与证据的关系）组成。
**Domain Shift**：源域与目标域在数据分布上的差异，包括主题、实体、事件等方面的不同。
**Biencoder**：双编码器密集检索模型，分别编码query和document为低维向量，通过点积计算相似度。
**Adversarial Training**：对抗训练，通过判别器和编码器的极小极大博弈，使目标域表征逼近源域分布。
**CORAL (Correlation Alignment)**：相关性对齐损失，通过最小化源域和目标域二阶统计量（协方差矩阵）的Frobenius范数差异来对齐分布。
**Reversal Curse**：语言模型在训练后无法从"A是B"推断"B是A"的固有缺陷，揭示了LLM在传递关系推理上的系统性不足。
**NDCG@10**：归一化折损累计增益，评估检索模型前10个结果的排序质量。
**Macro F1**：宏观F1分数，各类别F1的算术平均，适用于类别不平衡场景。

## 可复现要素
- **数据集**：MultiFC和Snopes公开可获取；本文构建的8个域划分数据已公开（Appendix A说明）。
- **代码/权重**：论文未明确提供代码仓库链接；使用了开源模型Contriever、RoBERTa、T5（MS-Marco预训练版）。
- **关键超参**：对齐损失系数λ∈{0.1, 0.3, 0.5, 0.7, 0.9}，最优值为0.1；Top-k=10；claim最大长度50，document最大长度200；batch size：retriever=70，reader=50；T5生成3个伪query/文档，预训练3 epoch；Adam优化器；4×NVIDIA Tesla V100 GPU。
