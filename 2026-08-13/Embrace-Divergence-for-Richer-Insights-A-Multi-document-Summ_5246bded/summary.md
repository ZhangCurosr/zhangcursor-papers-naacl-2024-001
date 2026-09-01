---
title: "Embrace-Divergence-for-Richer-Insights-A-Multi-document-Summ"
source: https://aclanthology.org/2024.naacl-long.32.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:14:45"
field: "多文档摘要"
keywords: ["多文档摘要", "多样性摘要", "LLM评估偏差", "faithfulness", "coverage", "GPT-4", "QA representation"]
innovations: ["提出MDDS任务并以QA形式构建DIVERSESUMM数据集", "系统分析GPT-4作为评估器时的位置偏差与冗长度偏差并提出最佳实践", "揭示LLM在多文档多样性摘要中的系统性覆盖模式与偏差"]
benchmarks: ["DIVERSESUMM", "GPT-4 human correlation"]
---

# 论文速读：Embrace Divergence for Richer Insights: A Multi-document Summarization Benchmark and a Case Study on Summarizing Diverse Information from News Articles

## 一句话总结
论文提出了多文档多样性摘要（MDDS）任务，构建了一个包含245个新闻故事、每个故事10篇文章的QA形式参考数据集DIVERSESUMM，并系统分析了LLM在该任务上的覆盖能力以及GPT-4作为评估指标时的位置偏差与冗长度偏差，发现即使最强的GPT-4也仅能覆盖约37%的多样性信息。

## 研究问题与动机
1. **现有方法忽视观点多样性**：传统多文档摘要研究（如DUC、TAC、MULTINEWS）主要关注归纳各来源共同认可的"共识信息"，忽略不同来源报道中的观点差异。
2. **多样性信息的计算表征不足**：此前Laban等人提出的Discord Questions方法虽可用于多样性分析，但未形成系统的摘要任务及评测基准。
3. **LLM在多文档场景下的覆盖能力未知**：尽管LLM在单文档摘要上表现优异，但在处理分散于多篇报道中的多样化观点时其实际能力缺乏系统评测。
4. **自动评估指标存在偏差**：LLM-as-a-judge方法日益普及，但其在评估faithfulness与coverage时的位置偏差、冗长度偏差等问题尚需系统性分析。

## 核心贡献（创新点）
1. **提出MDDS任务并构建DIVERSESUMM数据集**：将多样性信息以问题-答案（QA）形式形式化，每个新闻故事包含10篇报道和人工验证的参考QA对，与以往仅关注共识信息的数据集形成本质区别。
2. **设计了可扩展的自动化数据收集流水线**：基于GPT-3.5-Turbo的两阶段问题生成、文章级答案提取和聚类策略，相比Discord Questions方法使有效问题数量提升90%，与已有工作在本子任务的设计上有本质创新。
3. **系统揭示了GPT-4作为评估器时的两大偏差**：首次发现GPT-4在pairwise比较中存在强位置偏差（偏好第二个摘要），在single-answer grading中存在冗长度偏差（偏好更短的摘要），并据此提出最佳实践建议。
4. **揭示了LLM在处理多样性信息时的系统性覆盖模式**：发现LLM倾向于总结首尾文章（"Lost in the middle"现象）、"How/What"类问题覆盖率低于"Why/Where"、长上下文模型更擅长覆盖高频答案而标准模型更擅长低频答案。

## 方法详解
**数据集构建流水线**：
1. **数据来源**：从Google News聚合器选取400个新闻故事，每个故事约40篇文章，最终筛选后每个故事取10篇。
2. **两阶段问题生成**：首先通过启发式方法选择代表文章（对应答案聚类中位数的新闻故事），再用GPT-3.5-Turbo生成20个问题，相比单次文章输入使有效问题从10个提升至19个（+90%）。
3. **问题回答**：采用文章级GPT-3.5-Turbo提取整篇文章的答案（recall达64.6%），优于RoBERTa的43.8%。
4. **答案聚类**：沿用Laban等人基于RoBERTa的聚类方法。
5. **后处理与人工验证**：GPT-3.5-Turbo过滤无意义答案和非多样化QA对，再由16名通过三轮资格测试的MTurk标注员人工验证。

**评估指标设计**：
- **Faithfulness**：逐句评估摘要是否与源文章事实一致（二元Present/Not Present）。
- **Coverage**：以QA对为单位，评估摘要覆盖了参考QA中多少答案（二元Covered/Not Covered或Likert评分）。
- 采用细粒度评估方式（per-sentence和per-QA pair），而非句子级别。

**偏差分析方法**：
- 位置偏差：交换两个摘要的输入顺序后重复评估，统计GPT-4的一致性（Consistency）。
- 冗长度偏差：将原始摘要复制拼接生成扩展摘要，比较GPT-4对长短摘要的评分差异。
- 相关性分析：使用Kendall's Tau计算自动评估协议与人工判断的相关性。

## 实验与结果
**数据集规模**：245个新闻故事，每个故事10篇文章，平均2.49个问题，每个问题平均3.41个答案聚类。

**LLM性能评估（表2）**：
- GPT-4（Extract then summarize）：Faithfulness 95.63%，Coverage 36.58%
- GPT-3.5-Turbo-16K（Directly summarize）：Faithfulness 98.44%，Coverage 35.66%
- LongChat-7B-16K：Faithfulness 92.49%，Coverage 30.04%
- Vicuna-7B：Faithfulness 78.42%，Coverage 13.36%

**评估协议偏差分析**：
- 位置偏差（表3）：GPT-4在pairwise评估中显著偏好第二个摘要（Coverage Second: 17.55%，Faithfulness Second: 13.27%），一致性仅约60%。
- 冗长度偏差（表4）：Single-answer grading下GPT-4强烈偏好较短摘要（Coverage Original: 53.46% vs Extended: 16.33%），pairwise可显著缓解该偏差。

**评估协议相关性（表5）**：
- Faithfulness最优：both-way pairwise（GPT-4, 26.68%）> Likert single-answer（GPT-4, 21.18%）
- Coverage最优：Likert single-answer（GPT-4, QA-pairs级别, 36.75%）> Binary single-answer（GPT-4, QA-pair级别, 35.83%）
- GPT-3.5-Turbo-16K的某些协议甚至与人工判断呈负相关

**覆盖率分析（图3-5）**：
- LLM倾向于总结首尾文章（U型模式），中间文章覆盖较少
- "Why/Where"类问题覆盖率高于"How/What"类
- 高频答案（出现在更多文章中）覆盖率更高；长上下文模型（如GPT-4）更擅长覆盖高频答案，标准模型（如GPT-3.5）更擅长覆盖低频答案
- 模型规模越大覆盖率越高（表7：Llama-2 7B→13B→70B，覆盖率2.29→2.53→2.81）

## 相关工作脉络
1. **Discord Questions（Laban et al., 2022）**：本文扩展了该方法，将其从多样性分析工具发展为正式的任务定义和数据集构建基础，并针对问题生成和答案提取进行了多项优化。
2. **MULTINEWS（Fabbri et al., 2019）**：首个大规模新闻多文档摘要数据集，但仅关注共识信息（56K聚类，平均<3篇），本文数据集聚焦多样性且每故事固定10篇。
3. **WCEP、AUTO-HMDS等Wikipedia域数据集**：这些数据集面向知识型文档聚合，本文聚焦新闻报道的观点多样性场景，二者在领域和任务目标上存在本质差异。
4. **DUC/TAC系列**：小规模传统多文档摘要基准（约50-100聚类），本文数据集在规模和任务定义上更具挑战性和现代性。
5. **GPT-eval（Liu et al., 2023b）**：本文沿用了LLM-as-judge的思想，但进一步深入分析了GPT-4在faithfulness/coverage评估中的具体偏差模式。
6. **Lost in the middle（Liu et al., 2023a）**：本文在摘要任务中独立验证了该现象——LLM倾向于依赖输入的首尾文章信息。

## 局限性与未来方向
1. **数据集规模有限**：受限于人工标注成本，目前仅245个故事，未来可扩展至更大规模。
2. **领域单一**：仅涵盖新闻文章，数据结构具有特定模式，推广至学术论文、法律文档等其他领域尚需验证。
3. **提示敏感性未充分分析**：虽然手动优化了提示，但未系统性研究提示对LLM性能的敏感性影响。
4. **未探索更多评估协议**：对GPT-4以外的LLM作为评估器的偏差分析有限。
5. **未来方向**：开发更高效的多文档多样性摘要模型、探索跨领域泛化、设计抗偏差的评估协议。

## 研究启发与可借鉴点
1. **QA形式的多样性表示方法可直接迁移**：将"多样性信息"定义为"不同来源对同一问题的不同回答"的框架，可推广至 opinion summarization、fact verification 等其他需要捕捉多样信息的任务。
2. **两阶段问题生成策略值得复用**：先用代表性文章进行问题生成再批量提取答案的方法，显著提升了有效问题率，可作为同类任务的参考设计。
3. **评估协议的选择建议具有重要参考价值**：对于coverage评估推荐使用GPT-4的Likert-scale single-answer grading（与人工相关性最高），对于faithfulness评估若预算充足推荐both-way pairwise，否则用Likert single-answer——这些建议可直接用于本团队的实验设计。
4. **位置偏差与冗长度偏差的分析框架可复用于其他任务**：交换输入顺序分析位置偏差、复制拼接分析冗长度偏差的实验设计具有通用性，可用于评估其他任务中LLM作为judge的可靠性。
5. **模型规模与覆盖能力的正相关关系**：增大模型参数可提升多样性信息覆盖率，这一发现可为资源受限场景下的模型选型提供指导。

## 关键术语表
**Multi-document Diversity Summarization (MDDS)**：本文提出的新任务，要求模型从讨论同一事件的多篇新闻报道中提取并总结多样化的观点和信息，而非仅归纳共识。

**DIVERSESUMM**：本文构建的数据集，包含245个新闻故事，每个故事由10篇报道组成，配有QA形式的参考摘要和人工验证。

**Faithfulness（忠实度）**：评估摘要内容与源文章事实的一致性程度，本文采用逐句二值评分（Present/Not Present）。

**Coverage（覆盖率）**：评估摘要对参考QA对中多样化信息的覆盖程度，本文采用每QA对评估是否被覆盖或Likert评分。

**Position Bias（位置偏差）**：LLM评估器在pairwise比较中对输入顺序产生的偏好偏差，本文发现GPT-4显著偏好第二个出现的摘要。

**Verbosity Bias（冗长度偏差）**：LLM评估器对文本长度的偏好偏差，本文发现GPT-4在single-answer grading下显著偏好较短的摘要。

**Discord Questions**：Laban等人提出的多样性分析方法，通过生成问题和答案聚类来量化新闻覆盖中的观点差异，本文以此为基础构建了数据集。

**Both-way Pairwise Comparison**：一种消除位置偏差的评估协议，通过交换两个摘要的输入顺序并聚合两次评估结果来计算最终得分。

## 可复现要素
- **数据集**：DIVERSESUMM，论文未明确声明开源状态（ ACL Anthology 论文通常有数据代码声明，此处需进一步确认）
- **代码/权重**：论文未明确声明开源，但使用了GPT-3.5-Turbo、GPT-4、Vicuna-7B、LongChat-7B-16K、Llama-2、XGen-7B、PaLM-2等模型
- **关键超参**：每个新闻故事取10篇文章（K=10）；问题生成阈值：至少30%来源回答问题且答案呈现多样性；问题生成数量：每故事20个问题；训练/评估日期：数据主要来自2023年3月
