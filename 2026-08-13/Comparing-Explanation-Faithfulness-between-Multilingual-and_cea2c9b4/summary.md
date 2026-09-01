---
title: "Comparing-Explanation-Faithfulness-between-Multilingual-and"
source: https://aclanthology.org/2024.naacl-long.178.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:13:17"
field: "多语言自然语言处理可解释性"
keywords: ["faithfulness", "feature attribution", "multilingual language models", "tokenization", "interpretability", "XLM-R", "mBERT"]
innovations: ["首次系统比较多语言与单语模型FA忠实度差异，发现多语言模型忠实度更低且差距随模型尺寸增大而加剧", "揭示分词器拆分激进程度（fertility/splitting ratio）是忠实度差距的核心驱动因素", "通过tokenizer适配实验（WECHSEL/FOCUS）分离分词与模型语义处理对忠实度的因果贡献"]
benchmarks: ["SST", "Agnews", "MultiRC", "PAWS-X", "XNLI", "ChnSentiCorp", "BBC NLI"]
---

# 论文速读：Comparing-Explanation-Faithfulness-between-Multilingual-and

## 一句话总结
本文是**首个系统比较多语言模型与单语模型在特征归因方法（FAs）忠实度上差异**的实证研究，覆盖5种语言、5种FAs、15个任务共2400个评估案例；发现多语言模型（XLM-R、mBERT）的FA忠实度普遍低于其单语对应模型，且**模型越大忠实度差距越显著**，这一差异主要由**分词器（tokenizer）拆分激进程度不同**驱动。

## 研究问题与动机
- **核心问题**：在多语言与单语模型都能取得相近预测性能的前提下，FA方法（如注意力、Integrated Gradients等）提取的rationale忠实度是否存在系统性差异？
- **现有研究不足**：此前忠实度研究几乎全部聚焦于英语单语模型（Atanasova et al., 2020; Bastings & Filippova, 2020; Chan et al., 2022），多语言模型的可解释性研究仅限于表示分析（Rama et al., 2020; Gonen et al., 2022），未涉及FA忠实度。
- **实践困境**： practitioners在多语言场景下若需忠实解释，应优先选择多语言模型还是单语模型尚不清楚。
- **理论空白**："multi-linguality curse"（Conneau et al., 2020）对FA忠实度的影响从未被研究。

## 核心贡献（创新点）
1. **首次大规模跨语言/跨模型/跨FAs的忠实度对比实证研究**：5种语言（EN/ZH/ES/FR/HI）、2类架构（BERT/RoBERTa）、5种FAs、15个任务，共2400个评估案例，填补了多语言模型可解释性领域空白。
2. **揭示多语言模型FA忠实度系统性低于单语模型，且差距随模型尺寸增大而加剧**：XLM-R large vs. XLM-R base导致Suff差异从-0.100恶化至-0.186，证明"模型越大，FA越不忠实"。
3. **发现分词器差异是忠实度差距的核心驱动因素**：通过fertility（1个词被拆成多少子词）和splitting ratio（单词被拆分的频率）量化，发现多语言tokenizer更激进拆分，与Suff/Comp差异高度负相关（Pearson ρ=-0.86~-0.91）；使用WECHSEL/FOCUS进行tokenizer适配实验进一步验证了"分词策略而非语义处理"是主因。

## 方法详解
- **实验设置**：对比模型包括mBERT（167M params, vocab 106K）、XLM-R（278M params, vocab 250K）作为多语言模型，以及各语言的单语BERT/RoBERTa（均为~100M-125M params, vocab 21K-52K）。所有模型在类似架构和预训练目标下微调，确保可比性。
- **五种FAs**：Attention（α）、Scaled attention（α∇α）、InputXGrad（x∇x）、Integrated Gradients（IG）、DeepLift（DL），外加Random随机基线。
- **四种忠实度指标**（均以Random基线归一化，>1表示高于随机的忠实度）：
  - **Hard Sufficiency（Suff）**：保留top-k rationale tokens后，模型对预测类的置信度保留程度。
    $$\text{Suff}(\mathbf{X}, \hat{y}, \mathcal{R}) = 1 - \max(0, p(\hat{y}|\mathbf{X}) - p(\hat{y}|\mathcal{R}))$$
    Normalized版本减去零序列基线后除以$(1-\text{Suff}(\mathbf{X},\hat{y},0))$。
  - **Hard Comprehensiveness（Comp）**：移除rationale后预测置信度的下降幅度。
    $$\text{Comp}(\mathbf{X}, \hat{y}, \mathcal{R}) = \max(0, p(\hat{y}|\mathbf{X}) - p(\hat{y}|\mathbf{X}_{\backslash \mathcal{R}}))$$
  - **Soft-Sufficiency / Soft-Comprehensiveness**：按FA给出的重要性分数比例扰动每个token，而非硬截断/删除，对rationale长度预设更稳健。
  - 最终取rationale长度10%、20%、50%三个比例的**AOPC**（Area Over the Perturbation Curve）均值。
- **分词分析指标**：**Fertility**（单词平均被拆分子词数）和**Splitting ratio**（单词被拆分的比例），两者越小越好；同时计算多语言vs单语tokenizer的差异值，与Suff/Comp差异做Pearson相关分析。
- **解耦实验**：用WECHSEL（将英文RoBERTa适配到法语，替换为法语tokenizer）和FOCUS（将XLM-R适配到法语，替换为法语RoBERTa tokenizer）在相同模型参数下比较不同tokenizer的忠实度变化，分离分词与模型本身的影响。
- **实现细节**：AdamW优化器，lr=1e-5（线性层1e-4），batch size=16，5 epoch early stopping，3个随机种子取均值，单卡A100 GPU。

## 实验与结果
- **预测性能**：多语言与单语模型性能高度接近，最大差距为Hindi BERT（0.716）vs mBERT（0.685），差0.031，证明比较忠实度前提合理。
- **Hard指标（Suff/Comp）**：
  - **XLM-R < 单语RoBERTa**：所有语言均出现负差异，FR差距最大（Comp: 1.51 vs 1.055，差异-0.455，p=0.004）。
  - **mBERT > 单语BERT**（除HI Comp外）：正向差异。
  - **RoBERTa系列差异更显著**：一半案例差异>0.1。
  - **Soft指标无显著差异**：Soft-Suff/Comp绝大多数差异<0.01，说明固定rationale长度可能引入偏置。
- **模型尺寸效应**：XLM-R large（550M，是单语RoBERTa的4.7倍）vs XLM-R base（278M，2.2倍）：整体Suff差异从-0.100恶化为-0.186，EN从-0.143恶化至-0.300；Comp差异也随尺寸增大（-0.226 vs -0.197）。结论：**多语言模型越大，FA忠实度越低**。
- **分词关联**：Fertility Diff与Suff Diff/Comp Diff Pearson相关系数达-0.86/-0.91，高度负相关；RoBERTa系列fertility差异>0.1时出现负忠实度差异，BERT系列<0.1时无显著差异。
- **IG最敏感**：IG在所有FAs中faithfulness disparity最大，是唯一在BERT和RoBERTa系列均显著差异的FA。
- **Tokenizer适配实验（Table 7）**：FR RoBERTa、RoBERTa(EN→FR)、XLM-R(Multi→FR)使用相同tokenizer，忠实度高度一致（各FA Suff差异<0.06）；而原始XLM-R与上述三者差异可达0.366，验证了tokenizer的主导作用。

## 相关工作脉络
- **Atanasova et al. (2020)**：诊断文本分类中多种FAs的忠实度，但未涉及多语言场景，且仅关注单语模型。
- **DeYoung et al. (2020) / ERASER基准**：提出Suff/Comp等忠实度评估协议，本文沿用但扩展到多语言对比场景。
- **Sinha et al. (2021) / Zhao et al. (2022a)**：研究对抗攻击对FA忠实度的影响，限定于英语单语模型。
- **Rust et al. (2021)**：分析多语言vs单语模型的tokenizers对下游性能的影响，但未涉及解释忠实度，本文将其延伸至XAI领域。
- **Zaman & Belinkov (2022)**：提出多语言模型FA忠实度评估方法（跨语言一致性视角），但未系统比较mono vs multi模型，也无分词机制分析。
- **Morger et al. (2022)**：比较人类关注与模型token重要性，但局限于单语且无跨模型对比；本文在此基础上量化了tokenizer对忠实度的因果贡献。

## 局限性与未来方向
- **单语模型语言覆盖有限**：仅EN/ZH/ES/FR/HI五种语言，且均为Indo-European/Sino-Tibetan语系，**不适用于其他语系语言**（作者明确承认）。
- **Decoder模型缺失**：实验中未涉及Llama、Mistral、Gemma等主流decoder模型（其默认即为多语言），结论在新架构上的普适性待验证。
- **低资源语言未探索**：多语言模型在低资源语言中的忠实度行为未知，存在研究空白。
- **固定rationale长度偏置**：Suff/Comp在10%和50%长度差异明显（Table 4），说明指标设计本身可能引入偏置，Soft指标无此问题。
- **mBERT不同尺寸不可用**：无法像XLM-R那样用不同尺寸mBERT验证尺寸效应，仅通过单语BERT-base vs BERT-large间接佐证（附录Table 16）。

## 研究启发与可借鉴点
1. **Tokenizer适配解耦实验设计**：使用WECHSEL/FOCUS将tokenizer从多语言切换到单语（或反之），在保持模型参数基本不变的情况下分离分词效应，是因果推断的巧妙范式，可迁移到任何涉及分词影响的NLP可解释性研究中。
2. **Fertility/Splitting Ratio作为分词质量度量**：用这两个简单指标量化tokenization aggressiveness并与忠实度做相关分析，提供了低成本、可复现的分词影响分析方法，无需复杂实验。
3. **Soft vs Hard指标的启示**：Hard Suff/Comp受固定rationale长度影响显著，而Soft版本更稳健，建议在后续忠实度评估中同时报告两种指标以避免长度偏置。
4. **结合本团队方向的机会**：若团队研究低资源语言NLP或多语言LLM部署，可将此发现用于指导**tokenizer选择策略**（优先使用与目标语言匹配的专用tokenizer而非多语言通用tokenizer），或在多语言模型微调后做faithfulness audit。
5. **IG作为最敏感FA的启示**：Integrated Gradients在多语言场景下忠实度差异最大，暗示其路径积分假设在多语言tokenization下可能更易被违反，后续可针对性设计IG的改进版本或选择更适合多语言场景的FA。

## 关键术语表
- **Feature Attribution Methods (FAs)**：通过计算输入token对模型预测的贡献分数来生成解释的方法，包括Attention、Integrated Gradients、DeepLift等。
- **Faithfulness**：FA生成的rationale/token重要性分数反映模型真实内部推理机制的程度，越高越"忠实"。
- **Sufficiency (Suff)**：衡量保留top-k重要token后模型对原预测的置信度保留比例，值越大越忠实。
- **Comprehensiveness (Comp)**：衡量移除top-k重要token后模型预测置信度的下降幅度，反映rationale的信息量。
- **Soft Sufficiency/Comprehensiveness**：按重要性分数比例软性扰动token（非硬删除），对rationale长度预设更稳健的忠实度指标。
- **Fertility**：tokenizer将一个单词拆分成多少个子词的平均数量，越低表示分词越不激进。
- **Splitting Ratio**：数据集中被tokenizer拆分的单词占总单词的比例，衡量分词激进程度。
- **Multilinguality Curse**：多语言模型在支持更多语言时整体性能下降的现象（Conneau et al., 2020），本文首次将其与FA忠实度关联。

## 可复现要素
- **数据集**： SST、Agnews、MultiRC（EN）；Ant、KR、ChnSentiCorp（ZH）；CSL、PAWS-X、XNLI（ES/FR）；BBC NLI、News Topic、XNLI（HI），均为公开基准数据集。
- **代码/权重**：模型均来自Huggingface公开预训练权重（附录Table 9列出具体ID）；论文**未声明代码仓库**，但所有实现细节（附录B）足够复现。
- **关键超参**：AdamW优化器，lr=1e-5（线性输出层1e-4），batch size=16，5 epoch early stopping，10% warmup，grad-norm=1.0，3个随机种子取均值。
- **硬件**：单卡NVIDIA A100 GPU。
