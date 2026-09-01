---
title: "BeLLM-Backward-Dependency-Enhanced-Large-Language-Model-for"
source: https://aclanthology.org/2024.naacl-long.45.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:12:07"
field: "句子表示学习"
keywords: ["句子嵌入", "向后依赖", "大语言模型", "对比学习", "语义相似度", "自回归模型", "LoRA微调"]
innovations: ["首次量化验证向后依赖对自回归LLM句子嵌入的价值", "提出单向/双向混合层架构BeLLM并通过退化实验确定转折点", "结合代表性词策略与对比学习实现SOTA句子嵌入"]
benchmarks: ["STS-B", "C-STS", "SICK-R", "STS 2012-2016", "flickr30k"]
---

# 论文速读：BeLLM: Backward Dependency Enhanced Large Language Model for Sentence Embeddings

## 一句话总结
本文首次系统研究了向后依赖（backward dependency）在自回归大语言模型（LLM）中学习句子嵌入的作用，并提出 BeLLM——通过将 LLaMA 的最后一层注意力从单向转为双向，结合对比学习与代表性词策略，在标准 STS 和条件 STS 基准上均达到 SOTA。

## 研究问题与动机
1. **现有 LLM 仅建模前向依赖**：主流自回归架构（如 LLaMA）只关注从前往后的依赖关系，缺乏对向后上下文的建模，导致句子语义理解不充分。
2. **自回归 LLM 的依赖捕获能力弱于 BERT**：实证分析显示，LLaMA 和 ChatGLM 的句子级 Spearman 相关性（约 0.17/0.15）仅为 BERT（约 0.35）的一半，说明前向依赖存在明显局限。
3. **已有 LLM 句子嵌入工作忽视向后依赖**：虽已有研究利用 LLM 学习句子嵌入（如 Jiang et al., 2023; Li and Li, 2023），但未系统探索向后依赖对嵌入质量的影响。
4. **实际案例表明向后依赖对细粒度语义区分至关重要**：如图 1 所示，仅有前向依赖的 LLaMA 会将因场景不同而语义有差异的句子判定为高相似度（0.8），而正确答案为 0.5，暴露出单向建模的缺陷。

## 核心贡献（创新点）
1. **首次量化验证向后依赖对自回归 LLM 句子嵌入的价值**：与以往依赖定性观察的工作不同，本文通过内部依存相关性实验提供定量证据，证明增强向后依赖能显著提升句子嵌入质量。
2. **提出 BeLLM 架构——单向/双向混合层设计**：区别于简单替换或添加新层的方法，本文通过退化实验发现倒数第二层是性能转折点，仅将最后一层转为双向，在保持生成能力的同时增强语义建模。
3. **创新性结合"代表性词策略"与对比学习**：不同于直接使用 `[CLS]` 或末 token 作为句子表示，本文通过提示工程生成代表性词，并以该词嵌入作为句子嵌入，配合对比损失实现更细粒度的语义对齐。
4. **在标准 STS 和更严格的条件 STS 任务上均取得 SOTA**：与此前基于 LLM 的最佳方法（SimCSE_LLaMA）相比，在 C-STS 上提升 1.10%，相比 RoBERTa_large 基线提升 2.24%。

## 方法详解
1. **退化实验确定转折点**：从最后一层开始逐层移除，观察 STS 性能变化，发现存在 S 型曲线；当单向层数超过倒数第二层时性能显著下降，故将转折点定于倒数第二层。
2. **最后一层双向注意力改造**：移除最后一层的因果掩码（causal mask），使该层能同时捕获前向和向后依赖；前 n-1 层保持单向自回归结构以保留生成能力。
   - 单向注意力公式：$Attn_i^{LLM}(Q,K,V) = Softmax(\frac{QK^T}{\sqrt{d}} + \mathcal{M})V$，其中 $\mathcal{M}$ 为下三角掩码。
   - 双向注意力公式：$Attn_i^{BiLLM}(Q,K,V) = Softmax(\frac{QK^T}{\sqrt{d}})V$，无掩码约束。
3. **模型组合**：$\mathbf{h} = LLM^{1:n-1}(\mathbf{x}) + BiLLM^{n-1:n}(\mathbf{x})$，融合前向生成能力与全局上下文建模。
4. **代表性词嵌入策略**：使用提示"The representative word for {sentence} is "引导模型生成一句话的核心词，将该词的向量表示作为句子嵌入，减少高频词偏置。
5. **对比学习损失**：
   $$\mathcal{L} = -\sum_i \log \frac{e^{\cos(\mathbf{h}_i, \mathbf{h}_i^+) / \tau}}{\sum_{j=1}^{N}(e^{\cos(\mathbf{h}_i, \mathbf{h}_j^+) / \tau} + e^{\cos(\mathbf{h}_i, \mathbf{h}_j^-) / \tau})}$$
   拉近正样本对、推远负样本对，同时缓解各向异性（anisotropy）问题。
6. **高效微调**：采用 LoRA（r=32, alpha=32, dropout=0.1），可训练参数约 80M，在 4×RTX 3090 Ti 上训练。

## 实验与结果
**数据集与基线**：
- **标准 STS**：STS 2012-2016、SICK-R、STS-B；训练数据为 MNLI + SNLI。
- **条件 STS（C-STS）**：更严格的评估基准，考虑上下文条件变化。
- **下游任务**：MR、CR、SUBJ、MPQA、SST2、TREC、MRPC（7 个分类任务）。
- **基线**：包括 GloVe、BERT 系列（IS-BERT、SimCSE、DiffCSE）、SBERT、AnglE、openai-ada-002，以及基于 LLM 的 SimCSE_LLaMA、Flan-T5、GPT-4 等。

**主要结果**：
- **标准 STS（Table 1）**：BeLLM 在全部 7 个数据集上均最优，平均 Spearman 相关系数 86.26%，较 AnglE_LLaMA 提升 0.3%，较 SimCSE_LLaMA 提升 1.02%，paired t-test p<0.01。
- **条件 STS（Table 2）**：BeLLM 达到 49.74±0.25，超越 prior SOTA（SimCSE_RoBERTa_large 的 47.50）2.24%，较 SimCSE_LLaMA 提升 1.10%。
- **下游任务（Table 3）**：BeLLM 在 7 个任务中获 6 项最优，平均准确率 91.35±0.48，较 SimCSE_LLaMA（91.06）提升 0.29%。
- **消融实验（Table 4）**：
  - 无双向层：75.55（大幅下降）
  - 全双向层：85.24（性能下降，因丧失生成能力）
  - 13B vs 7B：86.87 vs 86.26（规模效应成立）
  - 修改策略 vs 添加策略：86.26 vs 84.35（直接移除掩码更有效）

## 相关工作脉络
1. **传统无监督句子嵌入**：GloVe、BERT-flow、BERT-whitening、IS-BERT、CT-BERT、SimCSE、DiffCSE——依赖静态或无监督预训练，未充分利用 LLM 的上下文建模能力。
2. **监督式句子嵌入**：InferSent、USE、SBERT——借助 NLI 等标注数据微调，SBERT 通过 siamese 架构显著提升效果，但 backbone 为 BERT 而非 LLM。
3. **AnglE（Li & Li, 2023）**：优化角度分布的句子嵌入方法，使用 LLaMA 作为 backbone，但未引入向后依赖，本文在其基础上进一步改进。
4. **Scaling Sentence Embeddings with LLMs（Jiang et al., 2023）**：首次将 LLaMA 用于句子嵌入，但未解决单向依赖局限，本文指出其缺陷并提供解决方案。
5. **C-STS 基准（Deshpande et al., 2023）**：提出条件语义相似度评估，本文在该更严格设置下验证方法的鲁棒性。
6. **各向异性缓解工作**：Wang & Isola（2020）从理论证明对比学习有助于均匀分布；本文通过代表性词策略（91.74% 非高频词）进一步缓解该问题。

## 局限性与未来方向
1. **模型规模大、效率低**：BeLLM 基于 LLaMA 7B，可训练参数约 80M，在资源受限场景下难以部署，作者明确指出需优化效率。
2. **仅改造最后一层**：退化实验虽发现转折点，但未探索中间多层双向或混合比例的最优配置。
3. **代表性词策略依赖提示工程**：生成质量受 prompt 设计影响，可能存在稳定性问题。
4. **仅评估英语任务**：未验证跨语言场景下的泛化能力。
5. **未来方向**：模型压缩、蒸馏至小模型、探索动态层选择策略、扩展至多语言场景。

## 研究启发与可借鉴点
1. **退化实验驱动架构设计**：不盲目堆叠层数，而是通过系统消融寻找性能转折点，为模型设计提供数据驱动依据，该方法可迁移至其他架构改造场景。
2. **单向/双向混合层思想**：在需要兼顾生成与理解的任务中（如文本摘要、对话生成），可借鉴此混合架构平衡能力。
3. **对比学习缓解各向异性的理论解释**：Appendix A 从理论上推导了对比损失如何促进均匀分布，可作为后续工作的理论基础。
4. **代表性词嵌入替代 `[CLS]` 的策略**：对于 LLM-based 句子表示，直接取末 token 或 `[CLS]` 可能受位置偏置影响，通过提示生成核心词提供更鲁棒的表示。
5. **条件 STS 作为更严格的评估基准**：C-STS 能更好区分语义细微差异，值得在句子嵌入工作中作为补充评测标准。

## 关键术语表
**Sentence Embedding**：将整句文本映射为低维向量的表示方法，用于衡量句子间语义相似度。
**Backward Dependency（向后依赖）**：句子中当前词与其后方词的依赖关系，单向模型无法捕获此类关系。
**Causal Mask（因果掩码）**：阻止 self-attention 中看到未来 token 的下三角掩码矩阵，是自回归模型的核心组件。
**Degradation Experiment（退化实验）**：逐层移除网络层以观察性能变化的消融实验，用于确定关键层位置。
**Contrastive Learning（对比学习）**：通过拉近正样本对、推远负样本对来学习语义表示的自监督训练范式。
**Anisotropy（各向异性）**：嵌入向量过度集中在向量空间某一锥形区域的现象，降低语义区分度。
**Representative Word Strategy（代表性词策略）**：通过提示工程让模型生成最能代表句意的单个词汇，以其嵌入作为句子表示。
**Conditional STS（条件语义相似度）**：在给定上下文条件下评估句子对的语义相似度，比标准 STS 更具挑战性。

## 可复现要素
- **数据集**：STS-B、STS 2012-2016、SICK-R、C-STS、MNLI、SNLI、flickr30k；多数为标准公开数据集。
- **代码/权重**：论文未明确提供开源链接，需联系作者获取。
- **关键超参**：LoRA r=32, alpha=32, dropout=0.1；batch size 经 16/32/64/128 搜索；learning rate=2e-4；temperature τ 未明确，需从代码确认。
- **硬件**：4×RTX 3090 Ti GPUs。
- **随机种子**：main experiment 使用 seed=42，消融实验验证了随机性影响。
