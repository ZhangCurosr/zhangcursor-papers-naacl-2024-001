---
title: "TISE-A-Tripartite-In-context-Selection-Method-for-Event-Argu"
source: https://aclanthology.org/2024.naacl-long.101.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 21:34:01"
field: "事件抽取与上下文学习"
keywords: ["In-context Learning", "Event Argument Extraction", "Determinantal Point Process", "Example Selection", "Large Language Models"]
innovations: ["提出TISE三元评分框架（语义相似度、示例多样性、事件相关性）", "基于DPP直接选取最优上下文示例子集", "在低资源场景下用更少示例超越部分监督方法"]
benchmarks: ["ACE05", "RAMS"]
---

# 论文速读：TISE-A-Tripartite-In-context-Selection-Method-for-Event-Argu

## 一句话总结
本文针对事件论元提取（EAE）任务的上下文学习（ICL）问题，提出TISE方法，从语义相似度、示例多样性、事件相关性三个维度评分并基于确定点过程（DPP）直接选取最优上下文示例集。在ACE05数据集上，TISE以更少示例达到优于基线的性能，甚至在全量场景下超越部分监督方法。

## 研究问题与动机
- **现有语义相似度选择的缺陷**：直接基于语义相似度选取top-k示例会选中语义高度重叠的示例（如相同事件类型、相同角色），无法提供额外的论元信息。
- **忽视事件属性相关性**：语义检索不考虑测试输入与示例之间的事件类型/角色关联，可能导致选出的示例与目标事件类型无关，LLM无法推断正确角色。
- **EAE任务的特殊性**：EAE输出为结构化表格，需要示例覆盖尽可能多的事件角色类型，单纯文本相似度无法保证这种覆盖性。
- **低资源场景需求**：训练数据稀缺限制了传统方法的泛化能力，而ICL能在低资源下发挥作用，但示例质量对性能影响巨大。

## 核心贡献（创新点）
1. **首次系统探索EAE任务的上下文示例选择问题**：提出三重要求（语义相似度、示例多样性、事件相关性），填补该领域空白。
2. **设计TISE三元评分框架**：分别用text scorer计算文本相似度、用DPP建模示例间多样性、用event scorer计算事件类型与角色的语义相关性。
3. **基于DPP的直接子集选择机制**：将三个分数融合为核矩阵，通过DPP的概率模型直接选取最优示例子集，避免贪心逐个选择的次优问题。
4. **实验验证与监督方法可比肩**：TISE在ACE05上用15个示例达到Arg-C 51.43%，超过部分监督方法（DyGIE++为60.7%的完全监督基准下仍具竞争力）。

## 方法详解
**整体流程**：给定测试输入x和训练集T，TISE通过三个评分模块计算核矩阵K，再利用DPP选取大小为k的示例子集A*。

**1. 语义相似度评分（Semantic Similarity）**
- 使用预训练语言模型（BERT）作为编码器E(·)
- 测试输入x与示例t的文本相似度：s_T(x, t) = sim(E(x), E(t))
- 核函数项：k₁(tᵢ, tⱼ|x) = s_T(x, tᵢ) · s_T(x, tⱼ)

**2. 示例多样性评分（Example Diversity）**
- 同样使用text scorer计算示例间相似度s_T(tᵢ, tⱼ)
- 核函数修改为：k₂(tᵢ, tⱼ|x) = s_T(x, tᵢ) · s_T(tᵢ, tⱼ) · s_T(x, tⱼ)
- 乘积形式可同时降低示例间相似度、提升与测试输入的相似度

**3. 事件相关性评分（Event Correlation）**
- **事件类型描述**：为每个事件类型设计自然语言描述（如Justice:Convict描述为"Involves a justice trial, recording a person has been convicted."）
- **事件角色描述**：为每个角色设计问题式描述（如Life:Die.Instrument描述为"What device was used to kill?"）
- 事件类型得分：ŝ_E(x, tᵢ) = sim(E(d̂(x)), E(d̂(tᵢ)))
- 事件角色得分：通过max-pool和平均计算，并加入超参数α=0.1奖励包含更多有用角色的示例
- 最终事件得分：s_E(x, tᵢ) = ½(ŝ_E(x, tᵢ) + ṡ_E(x, tᵢ))

**4. 核矩阵融合与DPP选择**
- 引入温度超参数λ₁、λ₂平衡各要求：s'ₑ = exp(s_E/(2λ₁)), s'ₜ = exp(s_T/(2λ₂))
- 核矩阵分解：K = Diag(S'ₑ) · Diag(S'ₜ) · K̄ · Diag(S'ₜ) · Diag(S'ₑ)，其中K̄ᵢⱼ = s_T(tᵢ, tⱼ)
- DPP选择目标：A* = arg max_A P(A|x)，采用快速贪心算法（k步迭代最大化行列式增量）

**5. 提示模板**
- 采用code imitation prompt（Wang et al., 2023），包含实体类型定义、事件定义、上下文示例、事件实例化四个部分

## 实验与结果
**数据集**：ACE05（句子级事件论元抽取数据集），预处理为8个父事件类型、33个子类型

**评估指标**：Arg-I（论元识别F1）、Arg-C（论元分类F1）

**基线方法**：
- RANDOM：随机选择
- BM25：稀疏检索
- BERT-TOPK/DPR-TOPK： dense检索top-k
- DPP-DIVERSITY：仅考虑多样性的DPP方法

**主要结果**（k=10）：
- TISE：Arg-I = 60.57，Arg-C = 50.78
- 较BERT-TOPK（56.93/46.98）提升：+3.64/+3.80
- 较DPP-DIVERSITY（58.47/47.91）提升：+2.10/+2.87

**关键发现**：
- TISE在k=5时（58.95/48.72）已超过BERT-TOPK在k=15时的性能（58.34/48.16）
- 事件相关性比示例多样性更重要（消融实验：移除事件相关性下降2.1/2.9，移除多样性下降1.5/2.1）
- 与监督方法比较：TISE在少量训练数据（5%/10%）时超过DEGREE；全量数据下TISE(k=15)+text-davinci-003+code prompt达到Arg-C 60.9，超越DyGIE++（60.7）

## 相关工作脉络
1. **In-context Learning基础**：Brown et al. (2020) 提出ICL概念，Rubin et al. (2022)、Liu et al. (2022) 研究基于语义相似度的示例选择，本文指出其在EAE任务中的不足。
2. **Event Argument Extraction**：传统方法包括span-based（Zhang et al., 2020）、reading comprehension（Du & Cardie, 2020）、text generation（Li et al., 2021）范式；LLM时代出现Code4Struct（Wang et al., 2023）等方法，TISE与其正交可结合。
3. **DPP在示例选择中的应用**：Ye et al. (2022)、Su et al. (2022) 等使用DPP提升示例多样性，本文进一步引入事件相关性约束。
4. **Event Type/Role描述方法**：Lu et al. (2023) 提出事件抽取作为问答生成，本文借鉴其角色描述思路并扩展至事件类型。
5. **Supervised vs. ICL对比**：DEGREE（Hsu et al., 2021）和DyGIE++（Wadden et al., 2019）为代表的高效监督方法，本文证明ICL在精心选择示例后可与之媲美。

## 局限性与未来方向
1. **编码器能力受限**：使用vanilla BERT作为scorer，未采用针对性训练的检索器；建议用强化学习以EAE性能为奖励优化检索器。
2. **计算开销较大**：每个示例的每个角色需单独查询测试角色描述，时间复杂度高；虽通过字典存储事件描述降至O(1)，但角色层面仍有额外开销。
3. **文档级泛化有限**：在RAMS文档级数据集上，不同选择方法提升不明显，可能需要更好的prompt设计。
4. **超参数敏感**：λ₂（语义相似度权重）比λ₁（事件相关性权重）更敏感，需仔细调优。

## 研究启发与可借鉴点
1. **三元评分框架的可迁移性**：语义相似度+多样性+任务相关性评分思想可推广至其他ICL任务（如关系抽取、命名实体识别）。
2. **DPP核矩阵分解技巧**：将核矩阵分解为对角矩阵与相似度矩阵的乘积，使行列式可分解为个体分数之和加上多样性项，便于理解和调参。
3. **事件属性的自然语言描述策略**：为事件类型和角色设计结构化描述（成对句子/单句问题），可增强LLM对事件语义的理解，适用于其他结构化预测任务。
4. **角色重叠率分析视角**：通过定义role overlap rate揭示示例选择有效性原理，为后续研究提供可解释的分析框架。
5. **与code prompt的结合**：TISE可适配不同prompt模板（code prompt/code imitation prompt）和LLM（text-davinci-002/003），展示了方法的通用性。

## 关键术语表
**In-context Learning (ICL)**：无需微调，通过在prompt中提供少量示例让LLM完成目标任务的学习范式。
**Determinantal Point Process (DPP)**：基于行列式点过程的概率模型，用于从集合中选择多样化子集。
**Event Argument Extraction (EAE)**：从文本中识别事件论元并分类其角色类型的任务。
**Semantic Similarity**：衡量测试输入与候选示例之间文本语义相似程度的指标。
**Example Diversity**：确保所选示例集内部多样性，避免语义重叠。
**Event Correlation**：衡量测试事件与示例在事件类型和角色层面的语义相关性。
**Code Imitation Prompt**：将事件抽取任务转化为代码生成任务的prompt模板。
**Arg-I / Arg-C**：Argument Identified（论元识别）和Argument Classified（论元分类）的F1评估指标。

## 可复现要素
- **数据集**：ACE05（公开），RAMS（公开）
- **代码**：论文未提及代码开源情况
- **模型**：bert-base-uncased（编码器）、text-davinci-002/text-davinci-003（LLM，通过OpenAI API访问）
- **关键超参**：λ₁=0.5（事件相关性温度）、λ₂=0.05（语义相似度温度）、α=0.1（角色数量奖励系数）
- **提示模板**：code imitation prompt（论文附录提供示例）
