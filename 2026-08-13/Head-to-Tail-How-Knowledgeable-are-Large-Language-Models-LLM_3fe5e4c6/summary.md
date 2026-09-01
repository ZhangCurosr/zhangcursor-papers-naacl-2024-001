---
title: "Head-to-Tail-How-Knowledgeable-are-Large-Language-Models-LLM"
source: https://aclanthology.org/2024.naacl-long.18.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 15:26:43"
field: "LLM事实性与知识评估"
keywords: ["LLM factuality", "knowledge graph", "long-tail knowledge", "benchmark", "hallucination", "head-to-tail evaluation"]
innovations: ["构建首个按head/torso/tail分层评估LLM事实性的18K QA基准", "提出A/H/M三指标与LLM裁判自动评测方法并验证高一致性", "揭示模型增大与指令微调不必然提升事实性知识的系统性证据"]
benchmarks: ["Head-to-Tail", "DBpedia", "IMDb", "Goodreads", "MAG", "DBLP"]
---

# 论文速读：Head-to-Tail-How-Knowledgeable-are-Large-Language-Models-(LLMs)-A.K.A.-Will-LLMs-Replace-Knowledge-Graphs

## 一句话总结
本文构建了Head-to-Tail基准（18K QA对），从实体/谓词流行度的head/torso/tail三档系统评估16个LLM的事实性知识内化程度，发现即使是GPT-4整体准确率仅31%，且长尾知识仍是显著瓶颈。

## 研究问题与动机
- LLM参数化知识中包含了多少事实？现有方法难以直接"查询"模型内部知识，且幻觉可能源于知识缺失或生成故障。
- 缺乏能同时反映用户兴趣分布与全网知识均匀分布的基准；KG本身也呈长尾稀疏。
- LLM是否会替代KG？需量化LLM对不同流行度实体的掌握差异。
- 常规增强手段（模型规模扩大、指令微调）是否真能提升事实性？

## 核心贡献（创新点）
- **首个按head/torso/tail分层评估LLM事实性的基准**：基于DBpedia、IMDb、Goodreads、MAG、DBLP构建18,171个QA对，覆盖多领域多关系。
- **自动评测方法论与指标体系**：提出A_LM/H_LM/M三项指标，用ChatGPT判定正误（人工抽检一致率98%），并验证与规则指标高相关。
- **揭示LLM事实性规律**：GPT-4整体准确率仅31%；头→躯→尾准确率单调下降；模型增大不必然提升事实性（LLaMA-33B略优于65B）。
- **提出Dual Neural KG愿景**：知识应同时以三元组（显式）和嵌入（隐式）形式共存，长尾/新鲜知识更适合外部检索增强。

## 方法详解
- **实体分层**：按流量（votes/ratings/citations）或密度（作品数/triples）排序，计算累计流行度，前1/3为head、中间1/3为torso、后1/3为tail；不同实体类型/领域分别划分。
- **问题生成**：模板法，过滤非特定/动态/数据源特有/非文本属性；每模板每桶等量采样，含歧义时附额外限定信息。
- **评估提示**：few-shot格式，要求"尽可能简洁回答，不确定时说unsure"；通过计数"unsure"/空答得缺失率M。
- **指标定义**：准确率A、幻觉率H、缺失率M，满足A+H+M=100%；A_LM由ChatGPT判定，H_LM=100%-A_LM-M；同步计算A_EM/A_F1/A_RL及对应H。
- **鲁棒性验证**：比较few-shot/zero-shot/in-domain提示，分析brief与"unsure"对稳定性与幻觉的影响，并计算规则指标与LLM指标的相关系数。

## 实验与结果
- **数据集**：Movie(3,093)、Book(3,000)、Academics(2,946)、Open-DBpedia(9,132)，共18,171 QA对，423个模板。
- **评估模型**：GPT-4、ChatGPT、Llama 2(70B)、LLaMA(7B/13B/33B/65B)、Vicuna(7B/13B)、Flan-T5(3B/11B)、RWKV(7B)、Falcon(7B/40B)、Falcon-Instruct(7B/40B)。
- **整体结果**：GPT-4最高A_LM=31%，H_LM=19.7%；ChatGPT A_LM=20.3%；Llama 2-70B A_LM=11.8%，H_LM=34%；LLaMA-33B A_LM=18.2%但H_LM高达80%。
- **Head/Torso/Tail趋势**：GPT-4在全部数据上A_LM分别为40.3%/33.4%/19.0%；即使head top-10%实体，GPT-4准确率仅46%（Table 5）。
- **领域差异**：Movie表现最好（GPT-4 head 59.3%），Academics最差（GPT-4 head仅15.8%）；Academics即使head实体准确率也仅10%左右。
- **谓词分层**：按DBpedia谓词流行度划分后，各模型在head&torso vs tail上无一致相关性（Table 6）。
- **规模与微调**：LLaMA-33B略优于65B；Vicuna/Falcon-Instruct因学会保守而M上升，但H仍高；指令微调未显著提升事实性。
- **提示鲁棒性**：A_LM与A_EM/A_F1/A_RL的Spearman ρ均值达0.915/0.951/0.947，Pearson r均值达0.966/0.969/0.969；去掉"unsure"使ChatGPT幻觉率上升13pp。

## 相关工作脉络
- **已有LLM事实性基准**：WebQuestions、TriviaQA、LC-QuAD、QALD-9、Natural Questions、EntityQuestions、KILT等，多聚焦单一知识源或通用QA，未系统按头/躯/尾分层。
- **长尾知识评估**：Mallen et al.(2023)、Kandpal et al.(2023)、Kim et al.(2023)亦发现准确率与流行度正相关，但本文更系统地设计分层方法学与指标，并给出RQ1–RQ3的量化回答。
- **知识增强LLM**：Knowledge-infusion（Liu et al., 2021; Wang et al., 2021; Zhen et al., 2022）与检索增强（Asai et al., 2023; Nakano et al., 2022; Shi et al., 2023; Borgeaud et al., 2022）对应head与tail场景。
- **KG与LLM关系**：Omar et al.(2023)仅手工评估450例ChatGPT；本文自动化评测18K例并揭示Dual Neural KG融合路径。

## 局限性与未来方向
- 未评估LLM在类型层次/ taxonomy上的知识内化能力。
- 仅测试了最无歧义的问题形式，未系统评估 paraphrase/entailment/cloze等多样提问的鲁棒性。
- LLM训练数据截止早于部分数据源更新，可能遗漏新兴知识。
- 未来方向：显式三元组与隐式嵌入的协同表示、head知识的知识注入、tail/新鲜知识的高效检索与无缝融合。

## 研究启发与可借鉴点
- **三层分层评估范式**：可按流行度/难度/领域熟练度将评测样本分桶，量化模型在不同知识密度下的表现差异。
- **prompt设计技巧**：要求"简洁回答+允许unsure"可显著降低幻觉（13pp）并提升答案稳定性（重生成差异从18%降至1%），可复用于其他事实性评测。
- **LLM裁判验证**：用强模型做自动化正确性判定并与人工抽检对比（本文98%一致），可在可控成本下扩展评测规模。
- **规则指标替代方案**：A_LM与A_EM/A_F1/A_RL高度相关，低资源场景可用规则指标近似，本文提供可复现的对照实验设计。
- **跨域对比设计**：Movie/Book/Academics/Open四域并行评测，有助于区分领域偏好与通用能力，可作为团队评测协议参考。

## 关键术语表
- **Head-to-Tail Benchmark**：按实体/谓词流行度将知识分为head（热门）、torso（中等）、tail（冷门）三档的评测基准。
- **A_LM / H_LM / M**：分别表示由LLM判定的准确率、幻觉率、缺失率，满足A+H+M=100%。
- **Popularity-based bucketing**：基于流量或密度排序并按累计得分三等分划分实体的分层方法。
- **Dual Neural KG**：主张知识同时以显式三元组和隐式神经网络嵌入形式并存的未来KG形态。
- **Knowledge infusion**：将结构化知识注入预训练模型的训练/微调技术路线。
- **Retrieval-augmented LLM**：通过外部检索补充LLM参数化知识不足的增强方法。
- **Cloze-style query**：将问答改写为填空形式（如"某电影的发行年份是__"）的评测变体。
- **Instruction tuning conservatism**：指令微调使模型更倾向于回答"unsure"而非猜测，从而降低幻觉但可能牺牲准确率。

## 可复现要素
- 数据集：Head-to-Tail已开源，地址https://github.com/facebookresearch/head-to-tail
- 代码/权重：论文未单独提供评测代码仓库，基线模型为公开版本
- 关键超参：temperature=0或top_k=1；数据截止年份IMDb=2023、Goodreads=2017、MAG=2021、DBLP=2023、DBpedia=2022-12-01；实体筛选截止年份IMDb/MAG/DBLP=2020、Goodreads=2015
- LLM版本：ChatGPT=gpt-3.5-turbo-0301，GPT-4=gpt-4-0613；RWKV=v4 Raven，Vicuna=v1.1
