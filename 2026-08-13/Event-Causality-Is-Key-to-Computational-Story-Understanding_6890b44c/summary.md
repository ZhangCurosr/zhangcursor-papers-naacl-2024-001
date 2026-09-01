---
title: "Event-Causality-Is-Key-to-Computational-Story-Understanding"
source: https://aclanthology.org/2024.naacl-long.191.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:16:24"
field: "计算叙事理解"
keywords: ["事件因果关系", "故事理解", "大语言模型", "少样本学习", "视频-文本对齐", "因果推理"]
innovations: ["首个基于LLM提示的事件因果关系抽取方法并设立COPES基准新SOTA", "首次实证验证因果结构对故事质量评估和视频-文本对齐的显著提升"]
benchmarks: ["COPES", "GLUCOSE", "OpenMEVA", "SyMoN", "YMS"]
---

# 论文速读：Event-Causality-Is-Key-to-Computational-Story-Understanding

## 一句话总结
本文提出一种基于大语言模型（LLM）少样本提示的方法，从开放域故事文本中自动抽取事件因果关系图，并在故事质量评估和视频-文本对齐两个下游任务中验证了因果结构对计算故事理解的显著提升作用。

## 研究问题与动机
- **核心问题**：认知科学和符号AI已证明事件因果关系对故事理解至关重要，但深度学习驱动的故事理解方法很少利用事件因果关系，主要原因是缺乏在开放世界设定中可靠识别事件因果关系的方法。
- **现有方法不足**：早期符号AI依赖人工编写的动作模板构建故事计划，受限于狭窄领域；现有神经网络方法（如Ammanabrolu等，2021）虽能构建故事图，但未验证其对故事理解任务的价值；主流自然语言处理方法几乎未将因果结构作为显式特征用于故事理解。
- **动机**：人类在叙事理解中高度依赖事件因果链（事件回忆、预测、道德判断等均受影响），而自动提取的因果结构若能准确且易于应用，有望为计算故事理解提供关键信息。

## 核心贡献（创新点）
1. **首个基于LLM提示的事件因果关系抽取方法**：通过少样本上下文学习（few-shot in-context learning），使用简单提示让LLM从开放域故事文本中重构因果图，无需额外训练。与已有工作的区别在于：前人工作多依赖监督训练或人工模板，本文利用LLM泛化能力直接完成关系抽取。
2. **在COPES基准上设立新SOTA**：使用6个示例故事，ChatGPT-3.5在COPES测试集上准确率达74.26%，超越监督模型COLA 4.2%（Accuracy）和2.3%（Macro F1）。与已有工作的区别在于：此前无工作探索ChatGPT在上下文因果推理（Contextualized CCR）上的定量表现。
3. **首次实证验证因果结构对故事理解任务的实际价值**：在故事质量评估（Kendall's tau相对提升3.6%-16.6%）和视频-文本对齐（Clip Accuracy提升4.1%-10.9%，Sentence IoU提升4.2%-13.5%）两个截然不同任务中均取得显著改善。与已有工作的区别在于：此前无人证明自动提取的因果图能实质提升自动化故事理解性能。

## 方法详解
- **问题形式化**：目标是从故事中构建有向因果图（causal graph），节点为故事事件（每句话视为一个事件），边为因果关系（A → B表示事件A导致/促成事件B）。
- **提示设计**：采用"指令+示例+待处理事件列表"的结构。示例使用`Edge i: (Node A -> Node B)`格式输出（类似DOT语言风格， preliminary experiments显示优于其他表示法）。图1展示了完整提示模板。
- **事件划分**：使用NLTK的sent_tokenizer将故事文本分割为句子序列，每句作为一个节点。
- **两阶段流程**（下游任务）：第一阶段用提示生成因果图；第二阶段将因果图信息融入下游任务（如故事评分或视频-文本对齐的特征编码）。
- **视频-文本对齐中的上下文构造**：区分两种上下文——时间上下文（当前片段前m个时序相邻片段）和因果上下文（因果图中当前事件的前驱节点）。因果上下文通过匹配文本片段找到对应视频片段来获取，推理时采用增量对齐方式（Figure 3）。
- **对比损失函数**（视频-文本对齐）：
  $$L_{NCE} = \frac{1}{N}\sum_{i=1}^{N} -v_i^\top t_i + \log\left(\exp v_i^\top t_i + \sum_{k\neq i}^{K}\exp v_i^\top t_k + \sum_{k\neq i}^{K}\exp v_k^\top t_i\right)$$
  其中$N$为训练样本数，$K$为负样本数，特征经Transformer编码后归一化，使用余弦相似度（即点积）。

## 实验与结果
- **事件因果关系抽取**：
  - **COPES**（1360个事件对，50/50验证/测试）：ChatGPT-3.5准确率74.26%、Micro F1 57.42%、Macro F1 69.49%，超越SOTA监督模型COLA（准确率70.29%、Macro F1 67.29%）。Yi-34B-chat同样超越COLA。
  - **GLUCOSE**（293个故事，维度1和6）：ChatGPT-3.5的F1为60.75%，超越监督GPT-2-large（59.54%），与T5-large（61.50%）相当。开源模型Yi-34B-chat F1达57.95%，综合表现最佳。
- **故事质量评估**（OpenMEVA，ROC和WP两个子集）：
  - 在零样本设置下，因果图方法相对最优基线提升6.4%-16.6%（Kendall's tau）。例如OpenMEVA-ROC零样本数据集级别从0.439提升至0.520（+18.4%相对提升）。
  - 少样本设置下提升3.6%-11.6%。
  - WP故事性能提升更大，但整体绝对值较低，原因包括WP故事更长、事件边界更模糊导致因果图质量下降。
- **视频-文本对齐**（YMS数据集，SyMoN和NeuMATCH两个划分）：
  - 在SyMoN sentence-level划分：Clip Accuracy达40.2%（相对Temporal Context-DTW基线提升7.7%），Sentence IoU达27.6%（提升8.0%）。
  - 最大提升幅度：NeuMATCH sub-sentence-level的Clip Accuracy提升10.9%，Sentence IoU提升10.9%；SyMoN sub-sentence-level的Sentence IoU提升13.5%。
- **结论**：事件因果关系提取质量与下游任务收益呈正相关，且方法在不同故事域和模态间具有良好的泛化性。

## 相关工作脉络
- **符号AI故事生成**（Meehan 1976; Young et al. 1994; Li & Riedl 2010）：使用人工编写的事件因果模板构建故事计划，但局限于狭窄领域，无法处理开放世界故事。
- **神经故事图构建**（Ammanabrolu et al. 2021）：训练神经网络完成因果关系补全，但未验证其在故事理解任务中的价值。
- **常识因果推理数据集**：COPA（Roemmele et al. 2011）关注事件-结果配对；GLUCOSE（Mostafazadeh et al. 2020）提供更丰富的故事上下文，支持多维度因果标注；COPES（Wang et al. 2023c）是最新因果事件对评估基准。
- **故事自动评估**：参考基线包括BARTScore+CNN+Para、BERTScore、BLEURT、Perplexity、UNION，以及Wang et al. (2023b)的ChatGPT评分方法。本文方法在因果图辅助下全面超越这些基线。
- **视频-文本对齐**：NeuMATCH（Dogan et al. 2018）和SyMoN（Sun et al. 2022）为弱监督对齐任务提供基础，本文通过引入因果上下文实现显著改进。

## 局限性与未来方向
- **因果类型限制**：仅关注事件间因果关系，未涉及事件与情绪、位置、拥有状态等其他类型的因果（如GLUCOSE维度2-5和7-10）。
- **故事类型限制**：实验集中在虚构故事（fiction stories），未扩展到新闻、推文等其他领域；故事分布偏向戏剧化而非现实主义。
- **事件边界敏感性**：方法在事件边界清晰的故事上表现最佳；当故事包含大量对话或事件模糊时，因果图质量下降，下游性能提升有限。
- **评估维度单一**：目前仅评估整体故事质量评分，未探索因果结构对连贯性、逻辑性、相关性等细分质量维度的贡献。
- **未来方向**：扩展至多维度因果推理；研究因果图对故事质量多维度的细化评估；探索在新闻、社交媒体等非虚构领域的适用性。

## 研究启发与可借鉴点
1. **提示格式选择**：使用类DOT语言的箭头表示法（`Node A -> Node B`）比自然语言描述更能获得准确的图结构输出，这一发现可用于其他关系抽取任务。
2. **少样本示例的域适配策略**：在OpenMEVA-WP实验中，使用域外（ROC）示例反而优于域内（WP）示例，原因可能是WP故事更长、事件边界更模糊导致低质量示例引入噪声。这提示少样本学习时需评估示例质量而非仅考虑领域匹配。
3. **因果上下文作为通用增强信号**：将LLM提取的因果图作为上下文信息融入下游任务（如视频-文本对齐的Transformer编码器），是一种即插即用的增强策略，可迁移至其他需要叙事理解的 multimodal 任务。
4. **弱监督对齐的因果引导**：在缺乏人工对齐标注的情况下，利用时间邻近性构造正样本对配合因果上下文，为弱监督多模态对齐提供了新思路。

## 关键术语表
**Event Causality（事件因果关系）**：故事中事件A导致或促成事件B的关系，包括物理因果、心理反应、动机驱动和条件建立四种类型。
**Causal Graph（因果图）**：以事件为节点、因果关系为有向边的图结构，用于形式化表示故事的因果链。
**Few-shot In-Context Learning（少样本上下文学习）**：在提示中提供少量示例，使LLM无需微调即可执行特定任务。
**COPES（Contextualized Open-world Paraphrase Event Satisfaction）**：Wang et al. (2023c)提出的事件因果关系评估基准，包含340个故事和1360个事件对。
**GLUCOSE（Generalized and Contextualized Story Explanations）**：Mostafazadeh et al. (2020)提出的故事因果解释数据集，涵盖10个因果维度。
**OpenMEVA**：Guan et al. (2021)发布的故事生成评估基准，包含来自5个ROCStories训练模型和5个WritingPrompts训练模型的1000+故事及人工评分。
**SyMoN（Synopses of Movie Narratives）**：Sun et al. (2022)发布的电影叙述视频-文本对齐数据集，用于评估故事理解在多模态任务中的价值。
**Kendall's tau**：衡量两个评分系统排序一致性的相关性系数，取值范围[-1,1]，值越接近1表示一致性越高。

## 可复现要素
- **数据集**：COPES（MIT License，公开）、GLUCOSE（CC BY-NC 4.0，公开）、OpenMEVA（GitHub: thu-coai/UNION，公开）、SyMoN（GitHub: insundaycathy/SYMON，公开）、YMS（GitHub: pelindogan/NeuMATCH，公开）
- **代码**：已开源，地址 https://github.com/insundaycathy/Event-Causality-Extraction
- **关键超参**：
  - COPES：6个in-prompt示例故事
  - GLUCOSE：从训练集随机抽取示例
  - 视频-文本对齐：学习率$5\times10^{-5}$，batch size 256，训练40 epochs，SGD+momentum 0.9，weight decay 0.5，文本随机mask 60%
  - 模型：ChatGPT-3.5-turbo-0631（temp=0）、Llama-2-13B-chat、Falcon-instruction-40B、Yi-34B-chat
