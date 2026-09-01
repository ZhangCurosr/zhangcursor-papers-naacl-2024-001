---
title: "SEMQA-Semi-Extractive-Multi-Source-Question-Answering"
source: https://aclanthology.org/2024.naacl-long.74.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 21:33:27"
field: "开放域问答与可验证生成"
keywords: ["半抽取式问答", "多源整合", "可验证生成", "长回答QA", "QuoteSum", "SEMQA"]
innovations: ["提出SEMQA任务，将半抽取式输出格式与多源问答结合，实现by-design归因", "构建QuoteSum数据集，首个面向半抽取式多源问答的大规模人工标注数据集", "提出纯文本匹配评估指标Sem-F1/Sem-Rec，无需额外模型即可完成精确度与全面性评估"]
benchmarks: ["QuoteSum", "AmbigQA", "PAQ", "Natural Questions"]
---

# 论文速读：SEMQA-Semi-Extractive-Multi-Source-Question-Answering

## 一句话总结
论文引入了半抽取式多源问答（SEMQA）任务，要求模型生成综合答案时，将**逐字摘录的事实片段**（标注来源）与**自由文本连接语**交织输出，以兼顾流畅性与可验证性；同时构建了首个此类数据集 QuoteSum，并提出基于纯文本匹配的评估指标。

## 研究问题与动机
1. **LLM幻觉与归因困难**：现有长回答QA系统输出的抽象答案虽流畅但难以归因，引用标注本身也可能不准确，给用户造成虚假可靠性印象。
2. **评估成本高且难以标准化**：全自动抽象式QA的评估主要依赖昂贵且难以解释的模型基度量，以及主观昂贵的人工评分。
3. **抽取式与抽象式之间的空白**：well-grounded的抽取式QA（如SQuAD）输出孤立片段缺乏上下文连贯性，而抽象式答案虽流畅却难以逐句验证。
4. **多源整合能力的缺乏**：当前方法在处理需要聚合多个来源信息的多答案问题时，缺少能显式归因且易于评估的生成范式。

## 核心贡献（创新点）
1. **定义SEMQA任务**：要求模型在给定问题和多个文档源时，输出由事实提取片段和自由连接语交替组成的段落式答案。与既有工作的本质区别在于，不同于全抽取（仅返回片段列表）或全抽象（自由生成文本），SEMQA强制模型在生成时嵌入可追溯的原始片段，从而将"溯源能力"内置到输出格式中。
2. **构建QuoteSum数据集**：首个包含多答案问题及人工撰写半抽取式答案的数据集（1,376个独特问题，4,009个答案）。与已有数据集的本质区别在于，它专门针对"多源整合+片段化引用"这一设定，且有意保留了部分无关噪声来源以模拟真实检索场景。
3. **提出基于纯字符串匹配的评估指标体系**：包括 Fluency（ROUGE-L）、Preciseness（Sem-F1）和 Comprehensiveness（Sem-Rec）。与ASQA等依赖消歧问题或模型基度量（如Disambig-F1）的方法相比，本指标无需额外标注、无需运行额外模型，更快、更透明且可直接解释。
4. **系统性实验与用户研究**：全面评测了微调模型（T5系列）和In-context学习模型（PaLM2），并开展人工评估证明SEMQA答案比ALCE的引用式答案更全面、更正确，且验证速度快两倍以上。

## 方法详解
**SEMQA任务定义**：给定问题 $q$ 和来源集合 $\mathcal{P} = \{p_1, p_2, \ldots, p_K\}$，生成答案 $a$，满足：(1) 回答 $q$，覆盖所有相关方面；(2) 尽可能使用提取片段（显式标记来源）；(3) 简洁且流畅。输出中每个提取片段用方括号标记来源索引，如 `[1 source text]`。

**评估指标**：
- **Fluency（流畅度）**：去除所有归属标记后，计算生成答案与参考答案的 ROUGE-L 分，取所有参考答案中的最大值。
- **Preciseness（精确度 Sem-F1）**：对每个来源 $k$ 单独计算其提取片段的 normalized token-F1，再对 $K$ 个来源取平均：
$$\mathrm{Sem\text{-}F1}(a_i, \mathcal{A}_i) := \frac{1}{K}\sum_{k=1}^{K} \max_{\hat{a} \in \mathcal{A}_i}\left(\mathrm{F1}\left(\psi_k(\hat{a}), \psi_k(a_i)\right)\right)$$
其中 $\psi_k(\cdot)$ 保留答案中标记为来自来源 $k$ 的token。
- **Comprehensiveness（全面性 Sem-Rec）**：测量短答案覆盖率：
$$\mathrm{Sem\text{-}Rec}(a_i, \mathcal{S}_i) := \frac{1}{K}\sum_{k=1}^{K}\max_{\hat{s}\in \mathcal{S}_{i,k}}\left(\mathrm{Rec}(\hat{s}, \psi_k(a_i))\right)$$
- **综合 SEMQA 分**：Fluency 与 Preciseness 的几何平均：
$$\mathrm{SEMQA} := (\mathrm{Sem\text{-}F1} \times \mathrm{ROUGE\text{-}L})^{0.5}$$

**QuoteSum数据集构建**：从 PAQ（6,500万可能问题）和 NQ/AmbigQA 子集中筛选多答案问题；对PAQ使用 T5-XXL QA模型过滤低置信答案（≥0.5），利用 Levenshtein距离、IoU 和语义分类器去重，再用 TF-IDF 合并相似问题；最终平衡采样得到多元问题。人工标注时，写作者通过可视化工具选择并粘贴提取片段，并用不同颜色区分来源。

## 实验与结果
**数据集**：QuoteSum，1,376个独特问题（984来自PAQ，392来自NQ），共4,009个答案；训练/验证/测试按60%/7%/33%划分，测试集来源页面与训练集不重叠。

**基线**：
- 微调模型：T5-small至T5-XXL、Flan-T5-small至Flan-T5-XXL
- In-context学习：PaLM2 Bison/Unicorn 1~5-shot，使用 QuoteSum prompt、QSum-S（转为句子级引用）、ALCE prompt（全抽象+内联引用）
- 朴素基线：取每源前k句/后k句直接拼接（Lead/Tail baseline）

**主要结果**（Table 2）：
- 最强微调模型：**Flan-T5 XXL** 取得 Rouge-L **73.36**、Sem-F1 **84.20**、Sem-Rec **93.68**、SEMQA **78.59**。
- 最强 Few-shot 模型：**PaLM2 Unicorn 5-shot** 取得 Sem-Rec **88.87**、SEMQA **66.18**；PaLM2 Bison 5-shot 取得 Sem-F1 **65.40**、SEMQA **64.42**。
- 微调模型整体优于 In-context 学习，Flan-T5 系列显著优于同等参数的原始 T5。
- 与 ALCE 对比（Table 3）：QSum prompt 在 AmbigQA 子集上 ROUGE-L 64.46 vs ALCE 54.73，Disambig-F1 45.32 vs 34.05；QSum-S（句子级引用）仍低于QSum，证明片段级引用更有优势。
- 朴素基线（Table 5）：Lead/Tail 5句拼接的 SEMQA 分仅约 30，远低于所有模型，说明任务并非简单截取可完成。
- 用户研究（Figure 3-4）：SEMQA 在全面性、正确性和归因帮助性上显著优于 ALCE，人工验证速度是 ALCE 的 **两倍以上**。

## 相关工作脉络
1. **SQuAD/NQ/AmbigQA**：传统抽取式QA数据集，输出为短片段或实体，无需整合多源，也无需流畅连贯文本。
2. **ASQA**：长回答QA数据集，要求生成段落式答案，但评估依赖消歧问题和模型基度量（Disambig-F1），本文的Sem-F1在不依赖消歧问题和额外模型的情况下实现了相似的排序效果。
3. **ELI5/Quampari**：长回答QA数据集，但侧重完全抽象生成，无显式片段归因，评估困难。
4. **ALCE (Gao et al., 2023)**：全抽象答案+内联句子级引用，本文证明片段级引用（SEMQA）在精确性、全面性和可验证性上均优于句子级引用。
5. **Extract-and-Decontextualize (Potluri et al., 2023)**：先抽取再编辑提升流畅性，但仅针对单源问题，不涉及多答案整合。
6. **Factscore (Min et al., 2023) / TRUE (Honovich et al., 2022)**：基于模型的 Fact consistency 评估方法，本文提出的文本匹配指标可绕过这些方法的复杂性和黑盒问题。

## 局限性与未来方向
1. **语言局限**：仅针对英语问题和英文Wikipedia来源，尚未扩展至多语言场景。
2. **问题类型覆盖有限**：虽经过平衡采样，但仍受限于NQ/PAQ数据集中的问题类型（偏重who/when/what等）。
3. **半抽取格式非万能**：部分问题不适合此格式，模型仍可能出现错误指代、脱离上下文等幻觉问题。
4. **源质量依赖**：答案正确性依赖于提供来源的准确性，若来源含误导信息则输出也会失真。
5. **未研究检索模块**：当前设定假设来源已由检索器提供，未来可扩展端到端检索+生成系统。
6. **开放域泛化**：在闭源设定下评估，对噪声检索器的泛化能力需进一步验证。

## 研究启发与可借鉴点
1. **可验证生成的格式设计**：SEMQA将归因"by-design"嵌入输出格式的思路，可迁移到任何其他需要高可信度的生成任务（如医疗问答、法律文档生成），通过强制片段引用降低幻觉风险。
2. **纯文本匹配指标的简洁高效**：Sem-F1/Sem-Rec无需运行额外模型即可完成评估，这种"输出空间约束+字符串匹配"的评估策略可用于设计更多低成本可信评估基准。
3. **噪声来源模拟**：QuoteSum有意保留未被写作者引用的噪声来源，这种设计更贴近真实检索系统输出，值得在检索增强生成（RAG）的数据构建中借鉴。
4. **Flan指令微调的有效性**：实验显示Flan-T5在适配新输出格式方面显著优于原始T5，提示在定制生成格式任务时，指令微调数据的作用不可忽视。
5. **动态Few-shot示例选择**：使用Sentence-T5余弦相似度检索最相似问题作为in-context示例，比随机选取显著提升性能（Table D.1），这一策略可直接复用于其他生成任务的prompt engineering。

## 关键术语表
**SEMQA**：Semi-Extractive Multi-Source Question Answering，半抽取式多源问答，要求模型输出由事实提取片段和自由连接语交替组成的段落式答案。
**QuoteSum**：论文构建的首个SEMQA数据集，包含1,376个多答案问题及4,009个人工撰写的半抽取式答案。
**Sem-F1**：针对每个来源单独计算提取片段的token-F1后取平均，衡量答案中提取内容的精确度与召回率。
**Sem-Rec**：衡量答案中短答案的覆盖率，即人工标注的短答案是否被模型提取并正确归因到对应来源。
**ALCE**：Gao et al. (2023) 提出的方法，生成全抽象答案并附加内联句子级引用，作为本文的对比基线。
**Rouge-L**：基于最长公共子序列的文本相似度指标，用于衡量生成答案的流畅度。
**Flan-T5**：经指令微调的T5系列模型，在适配新输出格式任务上显著优于原始T5。
**PAQ**：Probably-Asked Questions，从Wikipedia自动生成的6,500万个可能问题数据集。

## 可复现要素
- **数据集**：QuoteSum，已公开，下载地址：https://github.com/google-research-datasets/QuoteSum
- **代码/实现**：SEMQA指标实现已开源（同上仓库）
- **微调超参**：T5X框架，Adafactor优化器，学习率1e-3，batch size 32，最多25k步，以验证集ROUGE-L选最优checkpoint；TPUv4（XL/XXL）或TPUv3（base及以下）
- **Few-shot评估**：PaLM2 via API，temperature=0.0，使用Sentence-T5检索最相似训练集问题作为in-context示例
