---
title: "Self-Prompting-Large-Language-Models-for-Zero-Shot-Open-Doma"
source: https://aclanthology.org/2024.naacl-long.17.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 21:33:39"
field: "开放域问答与上下文学习"
keywords: ["零样本问答", "大语言模型", "上下文学习", "伪数据生成", "开放域QA", "Self-Prompting"]
innovations: ["提出 Self-Prompting 框架，让 LLM 自主生成伪 QA 数据集用于零样本 ODQA 的上下文学习", "设计基于聚类的检索策略 RetrieveInCluster，平衡示例的相似性与多样性", "提出 QAE 输入格式并证明其优于 Chain-of-Thought 和含段落的全信息格式"]
benchmarks: ["WebQ", "NQ", "TriviaQA"]
---

# 论文速读：Self-Prompting-Large-Language-Models-for-Zero-Shot-Open-Doma

## 一句话总结
本文提出 Self-Prompting 框架，让 LLM 在零样本开放域问答（ODQA）中自主生成伪 QA 对（含背景段落与解释）并用于上下文学习，无需任何训练数据和外部语料库，在 WebQ、NQ、TriviaQA 三个基准上显著超越已有 SOTA，且性能可与部分微调模型相媲美。

## 研究问题与动机
- **零样本 ODQA 缺乏训练数据与外部知识库**：传统 Retrieval-Reader 管道依赖大量标注数据和大型文档库，无法直接迁移到零样本场景。
- **直接 Prompting 未能充分利用 LLM 参数中的隐式知识**：简单 "Q: {question} A:" 格式无法激活 LLM 的多种能力，性能远逊于微调模型。
- **已有增强方法（Chain-of-Thought、GENREAD 等）仍不够充分**：这些方法仅让 LLM 生成解释或背景信息，但未系统性地构建可用于上下文学习的伪数据集。
- **上下文学习（ICL）示例的质量与选择方式直接影响性能**：现有 ICL 工作多使用真实训练数据作为 demonstrations，缺少从零开始完全由 LLM 自身生成示例的工作。

## 核心贡献（创新点）
- **提出 Self-Prompting 框架**：让 LLM 分步自动生成伪 ODQA 数据集（段落+实体抽取+问题生成+解释），并用于上下文学习；与直接 prompting 的本质区别在于显式激活 LLM 的多种能力而非仅预测答案。
- **设计基于聚类的检索策略（RetrieveInCluster）**：先用 Sentence-BERT 编码所有 QA 对并聚类，再从每个簇中检索与测试问题最相似的示例，平衡了语义相似性与示例多样性；区别于仅做全局相似度检索或随机选择的方法。
- **提出 QAE（Question-Answer-Explanation）输入格式**：实验证明将答案放在解释之前的格式效果最佳，避免了过多冗余段落信息对模型的干扰；与 chain-of-thought（先解释后答案）格式形成本质差异。
- **系统性分析伪数据质量与不同 LLM 规模的泛化性**：在 InstructGPT、Codex、GPT-NeoX、Alpaca 上均验证了 Self-Prompting 的有效性，并量化分析了幻觉问题。

## 方法详解
Self-Prompting 包含**准备阶段**和**推理阶段**两个阶段：

**准备阶段——伪 QA 数据集生成（四步）：**
1. **段落生成**：基于 TriviaQA 答案实体的 WordNet synset 分布，人工设计 29 个主题（如 politician、athlete、country 等），每个高级主题约 100 个示例，让 LLM 生成 Wikipedia 风格的短段落。
2. **命名实体识别**：让 LLM 自身从生成段落中提取命名实体（日期、地点、组织、人物、数字等）作为候选答案，而非依赖微调的小模型。
3. **问题生成**：以"Passage + Entity 是问题的答案"为提示让 LLM 生成问题，并通过双重检查（让 LLM 重新回答以验证能否还原实体）过滤不一致的 QA 对。
4. **生成解释**：让 LLM 用一句话基于段落为每个 QA 对生成解释，要求解释中必须包含目标答案。

**推理阶段——上下文学习示例选择与格式化：**
- **基于聚类的检索（Clustering-based Retrieval）**：用 Sentence-BERT（all-mpnet-base-v2）将所有伪 QA 对编码为向量，做 k-means 聚类（k 为所需 demonstrations 数量）。对每个测试问题，分别从每个簇中检索余弦相似度最高的示例，实现相似性与多样性的平衡。
- **QAE 格式拼接**：将选中的 demonstrations 按"Question → Answer → Explanation"顺序排列，测试问题置于末尾，一次性调用 LLM API 生成答案。实验表明此格式优于包含完整段落的格式及 chain-of-thought 格式。

## 实验与结果
- **数据集**：WebQ（2.0K 测试样本）、NQ（3.6K）、TriviaQA（11K）。
- **评估指标**：Exact Match（EM），附带 F1 和 Instruct-Eval（IE）分数。
- **基线模型**：InstructGPT、Codex、GENREAD、RECITE、DPR+InstructGPT、RAG、REALM、T5-SSM 11B 等。
- **主要结果（EM 平均值）**：
  - **Self-Prompting (InstructGPT)**：**46.2**，相比 InstructGPT 直接 prompting（30.7）**+15.5**，相比 GENREAD（37.4）**+8.8**；TriviaQA 达 66.8/79.4†。
  - **Self-Prompting (Codex)**：WebQ 38.9、NQ 40.7、TriviaQA 84.3†，综合表现甚至超过 Few-shot RECITE。
  - 与微调模型对比：自 P 在 NQ 和 TriviaQA 上接近 DPR（平均 46.5）和 RAG（平均 48.6）的水平，且**不使用任何训练数据和外部语料库**。
  - 与使用训练数据的 ICL 对比（非重叠子集）：Self-Prompting（34.2 EM / 46.0 F1）仅略低于 Traindata-ICL（36.0 EM / 48.7 F1），差距小于 2 EM。
- **关键超参**：demonstrations 数量为 10，温度=0，每主题段落最多生成 10 对 QA 对。

## 相关工作脉络
- **Retriever-Reader 管道（DPR、RAG、REALM）**：依赖训练数据和外部语料库进行检索，Self-Prompting 完全不依赖这两者，而是利用 LLM 参数中隐含的知识作为"隐式语料库"。
- **GENREAD（Yu et al., 2022）**：让 LLM 生成背景信息用于问答，但未系统构建伪 QA 数据集并用于 ICL；Self-Prompting 进一步生成了完整的 QA 对和解释并进行了聚类检索。
- **RECITE（Sun et al., 2022）**：使用 few-shot 训练数据进行上下文学习，Self-Prompting 无需任何训练数据即可达到可比甚至更优的性能。
- **Chain-of-Thought（Wei et al., 2022b；Kojima et al., 2022）**：先生成推理链再输出答案，本文实验表明在 ODQA 任务上 QAE 格式（答案在前）优于 CoT 格式。
- **Zerogen（Ye et al., 2022）**：用 LLM 生成数据训练小模型，本文发现伪数据更适合用于 LLM 自身的 ICL 而非训练小模型（Table 12 显示微调 RAG reader 后性能反而下降）。
- **In-context Learning 示例选择（Liu et al., 2022a；Rubin et al., 2022）**：本文首次完全由 LLM 自身从零生成 ICL 示例并进行聚类检索选择。

## 局限性与未来方向
- **依赖 OpenAI API，成本较高**：构建伪数据集约花费 $120、耗时 6 小时，限制了可复现性和大规模应用。
- **伪数据存在幻觉和事实错误**：小参数模型（Alpaca、GPT-NeoX）生成的段落事实错误率显著更高；虽然不要求 100% 准确，但质量仍有提升空间。
- **Prompt 模板需要人工调试**：各步骤的 prompt 设计需要 trial and error，尚未实现完全自动化。
- **实时生成变体效果不佳**：尝试为每个测试问题实时生成相关段落反而降低了性能，说明多样性比纯相关性更重要。
- **潜在偏差问题**：合成的伪数据可能存在隐性偏见，需要引入更多安全措施。

## 研究启发与可借鉴点
- **"用 LLM 自身知识构建伪数据用于 ICL"的范式可迁移**：不仅限于 ODQA，可推广至其他需要背景知识的 NLP 任务（如常识推理、事实验证）。
- **聚类检索平衡相似性与多样性**：RetrieveInCluster 策略可同时兼顾语义相关性和示例覆盖范围，值得在其他 ICL 场景中复用。
- **QAE 格式（答案在前、解释在后）优于 CoT**：对于 ODQA 等非复杂推理任务，避免过度冗余信息干扰可能比 chain-of-thought 更有效。
- **伪数据质量与 LLM 规模正相关**：大模型生成的数据更可靠，可在资源允许时优先选用更强基座模型进行数据生成。
- **双重检查机制提升生成质量**：问题生成后进行验证（重新回答以确认一致性）是一种低成本的数据质量控制手段。

## 关键术语表
- **Self-Prompting**：让 LLM 自主生成本身可用于上下文学习的伪 QA 数据集的框架，无需训练数据和外部语料。
- **Open-Domain Question Answering（ODQA）**：在不提供特定背景文档的情况下回答关于世界知识的问题。
- **In-context Learning（ICL）**：在测试样本前插入若干 input-output 示例（demonstrations），引导 LLM 完成目标任务而不进行参数更新。
- **Clustering-based Retrieval**：先将所有候选示例编码聚类，再从每个簇中检索与查询最相似的示例，平衡相似性与多样性。
- **QAE 格式**：输入序列中 demonstrations 按"问题→答案→解释"顺序排列的格式，实验表明其效果最佳。
- **GENREAD**：Yu et al. (2022) 提出的方法，让 LLM 生成相关背景信息辅助问答，是本文的主要对比基线。
- **Exact Match（EM）**：ODQA 标准评估指标，预测答案与标准答案完全一致则计分。
- **Sentence-BERT**：基于 Siamese BERT 网络的句子编码模型（all-mpnet-base-v2），用于将文本映射为向量以进行相似度计算。

## 可复现要素
- **数据集**：WebQ、NQ、TriviaQA（均为公开数据集）。
- **代码**：已开源，见 https://github.com/lockon-n/self-prompting。
- **LLM**：InstructGPT（text-davinci-002）、Codex（code-davinci-002），均通过 API 调用；另验证了 GPT-NeoX-20B 和 Alpaca-7B。
- **Sentence-BERT**：all-mpnet-base-v2。
- **关键超参**：demonstrations 数量=10，温度=0（除生成多样化示例外），max_tokens 各步骤分别为 1024/256/50/50/50。
- **主题列表**：29 个主题及其示例数量见 Appendix B（Table 9）。
