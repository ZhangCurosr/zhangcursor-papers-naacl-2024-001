---
title: "Fair-Abstractive-Summarization-of-Diverse-Perspectives"
source: https://aclanthology.org/2024.naacl-long.187.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:16:59"
field: "自然语言生成公平性"
keywords: ["公平摘要", "大语言模型评估", "社会属性偏差", "抽象文本摘要", "无参考评估指标"]
innovations: ["提出BUR/UER/AUC/SOF四个无参考公平度量指标", "构建PERSPECTIVESUMM六域多属性公平摘要基准", "发现LLaMA系列摘要公平性整体优于GPT系列"]
benchmarks: ["PERSPECTIVESUMM", "Claritin", "US Election", "Yelp", "Amazon", "SupremeCourt", "IQ2"]
---

# 论文速读：Fair-Abstractive-Summarization-of-Diverse-Perspectives

## 一句话总结
本文首次系统性地研究了基于大语言模型（LLM）的**公平抽象摘要**问题，提出了四个无参考自动评估指标（BUR/UER/AUC/SOF）和一个涵盖6领域、6种社会属性的多数据集基准PERSPECTIVESUMM，揭示现有LLM生成的摘要普遍存在不公平现象，并提供了三种简单有效的改进方法。

## 研究问题与动机
- **核心问题**：不同社会/人口群体在商品评论、政治辩论、医疗、法律等话题上持有多样甚至冲突的观点，而现有的摘要系统和评估指标无法确保生成公平的摘要——即不低估任何群体的视角。
- **现有指标局限**：ROUGE、BERTScore等传统摘要指标测量的是n-gram重叠或语义相似度，无法量化"公平性"；且人工撰写的参考摘要本身也可能存在偏见。
- **研究空白**：抽象摘要在LLM时代已成为主流（OpenAI, 2023），但现有研究尚未探索LLM生成的摘要在多元视角代表性上的公平性问题。
- **公平性挑战**：摘要需要将源文本压缩至约1/10长度，如何在压缩过程中保持各社会属性（性别、党派、情感、评分等）分布的一致性是一个开放难题。

## 核心贡献（创新点）
1. **形式化定义了摘要公平性**：将公平性定义为生成摘要不"低估"任何社会属性组别的表现，提出Ratio Fairness和Equal Fairness两种定义。
   - *区别*：不同于以往仅关注性别偏见的抽取式摘要研究（Dash et al., 2019; Keswani & Celis, 2021），本文聚焦LLM时代的抽象摘要任务，并覆盖更多社会属性。

2. **提出四个无参考公平度量指标**：Binary Unfair Rate (BUR)、Unfair Error Rate (UER)、Area Under Curve (AUC)、Second-Order Fairness (SOF)。
   - *区别*：现有摘要评估（如SummaC、RAGAS）关注事实一致性，本文指标直接衡量目标分布与源分布的对齐程度，无需参考摘要即可计算。

3. **构建PERSPECTIVESUMM多域基准**：整合Claritin（医疗/性别）、US Election（政治/党派）、Yelp（餐饮/情感）、Amazon（产品/评分）、SupremeCourt（法律/说话者）、IQ2（公共政策/说话者）六个数据集。
   - *区别*：区别于现有单域偏见数据集，本文覆盖6种社会属性和多元场景，为公平性研究提供统一评测平台。

4. **系统评测9个主流LLM的公平性并发现普遍缺陷**：GPT-4在Amazon数据集上BUR高达74.27%，LLaMA系列整体优于GPT系列；人工参考摘要同样存在不公平。
   - *区别*：这是首个针对LLM抽象摘要公平性的大规模系统性研究。

5. **提出三种简单有效的改进方法**：提高解码温度、控制适中摘要长度（3句最优）、在提示中添加公平性定义。
   - *区别*：不同于模型微调或复杂干预，本文强调零样本prompt engineering即可显著提升公平性。

## 方法详解
**公平性定义**：给定社会属性$a$及其取值集合$\mathcal{V} = \{v_1, ..., v_r\}$，将源文本$x$按属性值划分为$r$个子集$x_i$，分别计算其在源和目标摘要中的token占比$p_x$和$p_y$。公平目标为使$p_y$不低估$p_x$中任一值的比例。

**目标分布计算（无参考）**：
- **N-gram匹配**：扫描目标摘要中的k-gram，若在源文本中出现则赋予源端对应的属性值。实验取$k=1$。
- **神经匹配**：使用BARTScore/BERTScore计算目标与源端各属性子集的相似度，经Softmax（温度=0.1）得到$p_y$。

**四个度量指标**：
1. **BUR (Binary Unfair Rate)**：若存在某个属性值满足 $p_y(v_k) < \tau \cdot p_x(v_k)$ 则为不公平，τ默认0.8。
   $$f_{\text{BUR}}(x,y;\tau) = \mathbb{1}\left(\bigvee_{k=1}^{r} p_y(v_k) < \tau \cdot p_x(v_k)\right)$$

2. **UER (Unfair Error Rate)**：软度量，计算各属性值低估程度的平均值。
   $$f_{\text{UER}}(x,y) = \frac{1}{r}\sum_{k=1}^{r}\max(0, p_x(v_k) - p_y(v_k))$$

3. **AUC (Area Under Curve)**：对τ从0到1积分BUR，反映不同容忍度下的期望不公平率。

4. **SOF (Second-Order Fairness)**：衡量各属性值UERG集的方差，值越小表示各属性被低估的程度越均衡。

## 实验与结果
**数据集**：PERSPECTIVESUMM包含6个数据集，共8786个样本，覆盖医疗、政治、餐饮、产品、法律、公共政策6个领域。

**评测模型**：9个主流LLM（GPT-3.5 variants, GPT-4, LLaMA-2-7B/13B/70B-chat, Alpaca-7B, PaLM 2, Claude-instant-1）。

**关键结果**：
- **LLaMA系列整体公平性最优**：在Claritin上llama-2-chat-70B的BUR仅为61.53%，UER为9.92。
- **GPT-4仍存在严重不公平**：Amazon数据集上BUR高达74.27%（约3/4摘要存在偏差）。
- **人类参考摘要同样不公平**：Amazon Yelp的reference summary的BUR分别为95.00%/68.00%，高于gpt-turbo-3.5生成的68.33%/21.00%。
- **长上下文更具挑战性**：claude-instant-1在完整IQ2/SUPREME COURT上BUR达99.07%/100%。
- **AUC分析**：LLaMA-2在SupremeCourt上BUR最低（24.46-34.23），显示多属性场景下表现更好。

**公平性改进**（gpt-turbo-3.5 + N-gramScore）：
- 添加公平性定义：Claritin BUR从51.11%降至47.48%，Yelp从38.64%降至21.80%，Election从81.38%降至72.67%。
- 解码温度提升（0→1）：BUR和UER显著下降，但可读性降低。
- 摘要长度控制：3句摘要公平性最优，1句最差。

## 相关工作脉络
1. **抽取式摘要公平性**：Dash et al. (2019)分析摘要是否公平呈现不同社会群体；Keswani & Celis (2021)提出黑盒框架实现方言多样性。本文突破在于聚焦**抽象摘要**并扩展至多属性。
2. **意见摘要**：Carenini et al. (2013), Bražinskas et al. (2020a) 研究产品/餐饮评论摘要，关注few-shot学习而非公平性。
3. **政治偏见分析**：Feng et al. (2023), Santurkar et al. (2023) 研究训练数据导致的政治偏见；本文基准聚焦**生成结果的公平性**而非训练来源。
4. **现有摘要评估**：SummaC (Crépy et al., 2022)、RAGAS (Asani et al., 2023) 关注事实一致性；本文指标关注**社会属性分布对齐**。
5. **LLM评估基准**：MT-Bench (Zheng et al., 2023)、HLE (Srivastava et al., 2022) 评测通用能力；本文填补**公平性维度**评估空白。

## 局限性与未来方向
- 仅关注模型生成结果的公平性，未讨论训练数据的公平性（作者明确声明）。
- 数据集预处理可能导致分布失真：Twitter数据随机采样混合、对话文本截断影响完整性。
- 情感属性使用NLTK分类器预测，存在误分类可能。
- 未来方向：微调小/大模型提升公平性、扩展指标至其他类型偏见、在其他生成任务中应用公平性概念。

## 研究启发与可借鉴点
1. **无参考评估设计**：通过N-gram/神经匹配将目标摘要的语义单元归因到源端不同属性组，实现了无需人工标注的公平性度量，可迁移至其他公平性评估场景。
2. **多维度指标体系**：硬度量（BUR）+软度量（UER）+容忍度聚合（AUC）+均衡性度量（SOF）形成完整评估链条，可借鉴于其他需要多维度衡量的研究。
3. **零样本改进策略**：仅通过prompt engineering（添加公平性定义）即可显著降低BUR，证明指令微调前可尝试简单的文本干预。
4. **温度-质量权衡分析**：实验证明提高temperature可改善公平性但损害可读性，揭示了公平性与生成质量间的内在张力，值得后续研究探索更精细的平衡机制。

## 关键术语表
**PERSPECTIVESUMM**：本文构建的多域公平摘要基准，包含6个数据集、6种社会属性（性别、党派、情感、评分、说话者）。

**BUR (Binary Unfair Rate)**：二元不公平率，衡量样本中是否存在被低估的社会属性值。

**UER (Unfair Error Rate)**：不公平错误率，软度量各属性值被低估的平均百分比。

**AUC (Area Under Curve)**：对不同τ阈值下的BUR积分，反映跨容忍度的期望不公平率。

**SOF (Second-Order Fairness)**：二阶公平性，衡量各属性值UERG集的方差，值越小表示不公平程度越均衡。

**N-gram Matching / Neural Matching**：两种计算目标分布的方法，前者基于词汇重叠，后者基于语义相似度。

**Ratio Fairness / Equal Fairness**：两种公平定义，前者要求目标分布与源分布一致，后者要求均匀分布。

**ACU (Atomic Content Unit)**：最小语义单元，用于人类评估中分解摘要事实并追溯来源。

## 可复现要素
- **数据集**：PERSPECTIVESUMM已开源，包含6个公开数据集的统一处理版本（github.com/psunlpgroup/FairSumm）
- **代码**：已开源（https://github.com/psunlpgroup/FairSumm）
- **关键超参**：τ=0.8（默认容忍度）、Softmax温度=0.1、N-gram k=1
- **模型配置**：GPT系列temperature=0，最大长度512；Alpaca temperature=0.1，context=2048；Claude使用默认参数
