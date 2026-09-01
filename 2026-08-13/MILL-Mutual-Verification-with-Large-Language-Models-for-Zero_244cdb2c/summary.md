---
title: "MILL-Mutual-Verification-with-Large-Language-Models-for-Zero"
source: https://aclanthology.org/2024.naacl-long.138.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 15:29:26"
field: "检索系统中的查询扩展"
keywords: ["Query Expansion", "Large Language Models", "Zero-shot", "Mutual Verification", "Pseudo-Relevance Feedback", "Information Retrieval", "Prompt Design"]
innovations: ["提出 Query-Query-Document 多步推理提示以生成多样化子查询-文档对", "设计生成文档与检索文档双向互评筛选机制以实现零样本高质量扩展"]
benchmarks: ["TREC-DL-2019", "TREC-DL-2020", "BEIR"]
---

# 论文速读：MILL-Mutual-Verification-with-Large-Language-Models-for-Zero

## 一句话总结
本文提出 MILL（Mutual VerIfication with Large Language model）框架，利用 LLM 的零样本推理能力生成多样化的子查询与上下文文档，再通过相互验证机制筛选出高质量的生成文档与伪相关反馈（PRF）文档，最终实现无需微调的零样本查询扩展，在检索任务上显著优于现有基线方法。

## 研究问题与动机
- **检索型方法局限**：基于伪相关反馈（PRF）的查询扩展依赖初始检索结果，当查询简短或歧义时，检索到的文档难以准确对齐用户真实搜索意图。
- **生成型方法局限**：直接使用商用 LLM（如 GPT-3.5）进行零样本/少样本文档生成，模型缺乏目标语料库的领域知识，容易生成与领域无关的无效信息；若进行微调则成本高昂。
- **组合策略简单拼接的不足**：现有集成方法（如 Query2Doc*、CoT*）仅将 PRF 文档直接拼接给 LLM，未考虑生成文档与检索文档之间的语义对齐与噪声过滤问题，往往效果不佳甚至退步。
- **零样本泛化需求**：实际搜索系统面对海量不同领域的语料库，希望有一种无需针对特定语料库微调即可适配的查询扩展方案。

## 核心贡献（创新点）
1. **提出 MILL 相互验证框架**：首次将 LLM 生成文档与 PRF 检索文档置于同一个互评筛选机制中，通过双向相关性打分实现文档级质量提升。
2. **设计 Query-Query-Document（QQD）提示方案**：让 LLM 先生成若干子查询再为每个子查询生成回答文档，利用其零样本推理能力挖掘更多样、更贴近真实意图的上下文，区别于单步生成文档的 Query2Doc 和仅生成关键词的 Query2Term。
3. **实现完全零样本的高质量查询扩展**：MILL 无需任何标注数据或模型微调，仅依赖即插即用的 LLM API 与 BM25 检索器，在三个公开基准上均达到最优或次优性能。
4. **系统验证与深入分析**：在 TREC-DL-2019/2020 和 BEIR（9 个子集）上全面对比传统 QE、LLM 生成 QE 及集成 QE 方法，并通过消融实验、超参数分析和案例研究验证各模块有效性。

## 方法详解
MILL 分为两个阶段：**上下文文档构建** 与 **相互验证**。

1. **Query-Query-Document Generation（QQD）**
   - 提示词引导 LLM 完成两步推理：首先回答 "what sub-queries should be searched to answer the following query: {query}"，生成多个子查询以拆解原始查询的潜在意图；随后为每个子查询生成对应的上下文段落（passage）。
   - 最终得到一组 LLM 生成文档集合 $\mathcal{D}^{\mathrm{LLM}} = \{d_n^{\mathrm{LLM}}\}_{n=1}^{N}$，每条文档包含子查询及其回答文本，具有更高的意图覆盖多样性。

2. **Mutual Verification（相互验证）**
   - 同时保留 K 篇通过 BM25 检索得到的伪相关反馈文档 $\mathcal{D}^{\mathrm{PRF}} = \{d_k^{\mathrm{PRF}}\}_{k=1}^{K}$。
   - 使用预训练稠密编码器（text-embedding-ada-002，维度 1536）将每篇生成文档和检索文档编码为向量。
   - 计算跨集合余弦相似度并求和作为打分：
     $$s_n^{\mathrm{LLM}} = \sum_{k=1}^{K} \sin(\mathbf{x}_n^{\mathrm{LLM}}, \mathbf{x}_k^{\mathrm{PRF}})$$
     $$s_k^{\mathrm{PRF}} = \sum_{n=1}^{N} \sin(\mathbf{x}_k^{\mathrm{PRF}}, \mathbf{x}_n^{\mathrm{LLM}})$$
   - $s_n^{\mathrm{LLM}}$ 越高说明生成文档与目标语料库越对齐；$s_k^{\mathrm{PRF}}$ 越高说明检索文档越能被 LLM 推理结果所支撑。
   - 分别取 Top $N'$ 生成文档与 Top $K'$ 检索文档构成最终上下文集合 $\mathcal{D}_s^{\mathrm{LLM}}$ 与 $\mathcal{D}_s^{\mathrm{PRF}}$。

3. **Query Expansion for Retrieval**
   - 将原始查询重复 5 次（强调重要性）后与筛选后的生成文档、PRF 文档拼接，形成扩展查询 $q'$：
     $$q' = \operatorname{concat}(q, q, q, q, q, \mathcal{D}_s^{\mathrm{PRF}}, \mathcal{D}_s^{\mathrm{LLM}})$$
   - 全程无需训练，仅依赖即插即用的 GPT-3.5-turbo-Instruct 与 BM25 retriever。

## 实验与结果
- **数据集**：TREC-DL-2019、TREC-DL-2020（各 200 查询、884 万段落）、BEIR 9 个子集（TREC-COVID、TOUCHE、SCIFACT、NFCORPUS、DBPEDIA、FIQA-2018、SCIDOCS、ARGUANA、CLIMATE-FEVER）。
- **评估指标**：NDCG@N、AP@N、Recall@N、MRR@N（N ∈ {10, 100, 1000}）。
- **基线分类**：
  - 传统方法：Bo1、KL、RM3、AxiomaticQE
  - LLM 生成方法：Query2Term、Query2Doc、CoT 及其 few-shot/PRF 变体
  - 集成方法：上述 LLM 方法 + 直接拼接 PRF 文档（带 * 后缀）
- **实现细节**：PyTerrier 实现，BM25 默认参数（b=0.75, k1=1.2, k3=8.0）；LLM 使用 GPT-3.5-turbo-Instruct（temperature=0.7, top_p=1）；编码器为 text-embedding-ada-002（1536 维）；候选文档数 N=K=5，最终选取数 N'=K'=3。
- **主要结果**：
  - **TREC-DL-2019**：MILL 在 NDCG@1000 达到 **73.74**，优于第二的 CoT*（72.08）；AP@1000 达到 **53.11**，优于第二的 Query2Doc-FS*（51.55）；Recall@1000 达到 **91.81**，优于第二的 CoT*（92.19 略高但 AP/MRR 更全面）；整体在大多数指标上第一或第二。
  - **TREC-DL-2020**：MILL 在 NDCG@1000 达到 **71.23**，优于第二的 Query2Doc-FS*（70.26）；AP@1000 达到 **48.17**，优于第二的 Query2Doc*（47.01）；MRR@1000 达到 **92.72**，为所有方法最高。
  - **BEIR（NDCG@1000）**：MILL 在 9 个子集中多数第一，例如 TREC-COVID **52.53**（第二 CoT* 50.69）、SCIFACT **74.14**（第二 CoT* 72.33）、DBPEDIA **46.39**（第二 Query2Doc-PRF* 44.47）、FIQA-2018 **39.23**（第二 CoT* 38.13）、SCIDOCS **28.36**（第二 CoT* 27.62）；仅在 ARGUANA 和 CLIMATE-FEVER 提升有限。
  - **统计显著性**：所有最优与次优结果之间均通过双侧 t 检验（p < 0.05）验证显著。
- **结论**：MILL 作为纯零样本方法，在多种数据集和指标上全面超越传统方法、LLM 生成方法及简单集成方法，且在不同域（通用、科学、法律、辩论等）均表现稳定。

## 相关工作脉络
- **传统查询扩展（Bo1/KL/RM3/AxiomaticQE）**：基于统计语言模型或公理化语义匹配，利用 PRF 文档(term 层面)扩展查询；MILL 在语义层面替代了 term 级扩展，并引入 LLM 推理能力。
- **Query2Doc / Query2Term（Wang et al., 2023a; Jagerman et al., 2023）**：直接用 LLM 生成 passage 或 keyword；MILL 通过 QQD 多步推理生成更丰富的子查询-文档对，并通过相互验证过滤噪声，避免直接拼接 PRF 导致的退化。
- **CoT / CoT-PRF（Jagerman et al., 2023）**：采用链式思维让 LLM 先给出推理过程再回答；MILL 的 QQD 同样强调推理分解，但目标不是逐步推理链条而是意图拆分与文档生成，并通过与 PRF 的互评进一步质量控制。
- **集成方法（Query2Doc* 等）**：直接将 PRF 文档拼入 prompt；实验表明这种简单拼接未必有益（如 Query2Doc-PRF 在 TREC-DL 上低于 Query2Doc），MILL 通过双向打分筛选而非粗暴拼接解决了该问题。
- **PRF 与 LLM 结合的空白**：此前工作多侧重单一来源（纯检索或纯生成），MILL 首次系统性地对两种来源进行互评筛选，填补了“如何可信融合生成与检索上下文”的研究空白。
- **零样本 IR 评估基准 BEIR（Thakur et al., 2021）**：MILL 在异构零样本场景下验证泛化性，区别于仅在小规模专用数据集（如 MSMARCO）上评估的工作，凸显其实际应用潜力。

## 局限性与未来方向
- **检索效率**：MILL 需对每个查询进行多轮自回归生成（N 个子查询-文档对）以及双向相似度计算，扩展查询长度也会增加倒排索引检索时间；论文建议并行生成与基于规则的查询压缩作为改进方向。
- **特定数据集表现不佳**：在 ARGUANA（查询平均 193 词，本身已很长）和 CLIMATE-FEVER（查询多为陈述句）上提升有限甚至不稳定，说明当前 QQD 提示对超长查询或声明式查询的适应性有待加强。
- **编码器依赖**：相互验证依赖 text-embedding-ada-002 等稠密编码器，若目标语料分布与编码器预分布差异较大可能影响打分准确性。
- **未来方向**：① 探索更高效的生成策略（如并行/流式生成）与查询压缩规则；② 针对长查询、声明式查询设计自适应提示；③ 研究更鲁棒的跨源对齐打分函数；④ 扩展到对话检索、多模态检索等场景。

## 研究启发与可借鉴点
- **多步推理提示设计**：QQD 将单步生成拆解为“子查询生成→文档生成”两步，充分利用 LLM 的 zero-shot reasoner 能力，该思路可迁移至其他需要意图覆盖的文本生成任务（如问答、摘要）。
- **双向互评筛选机制**：用集合 A 评估集合 B、同时用 B 评估 A 的对称打分策略，适用于任何需要融合多源异构信息并去噪的场景（如 RAG 中的检索-生成联合排序、多模型投票集成）。
- **零样本泛化实验设计**：在 BEIR 这类异构基准上做全面评估，比仅在单一数据集报 SOTA 更具说服力，可作为后续工作的基准测试范式。
- **消融与超参数敏感性分析**：论文详细分析了候选数、选取数对性能的影响，并发现 PRF 文档过多会引入噪声而生成文档相对鲁棒，这类分析为工程调参提供了明确指导。
- **案例可视化解读**：Table 5 展示了被保留与被过滤文档的具体内容，直观解释了相互验证的作用机理，这种“数字结果+案例佐证”的组合值得在报告与论文中复用。

## 关键术语表
**Query Expansion（查询扩展）**：通过向原始查询添加上下文术语或段落来增强其表达能力的技术，旨在提高检索系统的召回与排序性能。
**Pseudo-Relevance Feedback（PRF，伪相关反馈）**：假设初始检索返回的前 K 篇文档均为相关，并利用这些文档的词分布或语义信息重新构建查询的方法。
**Zero-shot（零样本）**：在不提供任务示例、不微调模型参数的情况下，仅依靠指令提示让 LLM 完成特定任务。
**Query-Query-Document Prompt（QQD）**：本文提出的三步提示策略，先让 LLM 推断子查询，再为每个子查询生成回答段落，以提升意图覆盖多样性。
**Mutual Verification（相互验证）**：通过跨集合余弦相似度求和，让生成文档与检索文档互为质量评估标准，从而筛选出更对齐目标语料且更符合搜索意图的上下文文档。
**BM25**：一种基于词频与文档长度的经典稀疏检索排序函数，本文作为 PRF 文档的检索器使用。
**BEIR**：Benchmark for Evaluation of IR Systems 的缩写，包含 18 个异构检索数据集的大规模零样本评测基准。
**Text-Embedding-Ada-002**：OpenAI 提供的稠密文本编码器，将文本映射为 1536 维向量，用于相互验证阶段的相似度计算。

## 可复现要素
- **数据集**：TREC-DL-2019、TREC-DL-2020、BEIR（9 个子集）均为公开数据集；MSMARCO passage 数据集也在附录中提及。
- **代码**：已开源，链接为 https://github.com/Applied-Machine-Learning-Lab/MILL。
- **模型/权重**：使用 OpenAI 商业 API（GPT-3.5-turbo-Instruct 生成、text-embedding-ada-002 编码），未提供本地微调权重。
- **关键超参**：BM25 参数 b=0.75, k1=1.2, k3=8.0；LLM 生成 temperature=0.7, top_p=1；候选文档数 N=K=5；最终选取数 N'=K'=3；原始查询重复次数 5 次。
