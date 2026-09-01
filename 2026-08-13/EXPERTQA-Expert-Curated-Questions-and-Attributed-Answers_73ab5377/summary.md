---
title: "EXPERTQA-Expert-Curated-Questions-and-Attributed-Answers"
source: https://aclanthology.org/2024.naacl-long.167.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:13:59"
field: "可信NLP与事实性评估"
keywords: ["长文本问答", "归因评估", "事实性", "专家-in-the-loop", "检索增强生成", "幻觉检测"]
innovations: ["构建跨32领域2177题专家自拟问答基准并附带claim级归因/事实性标注与修订答案", "系统对比纯生成/检索增强/后处理归因三类范式的专家级评估结果", "揭示AutoAIS与FActScore在专家场景下的低召回缺陷并提出微调对齐方案"]
benchmarks: ["EXPERTQA", "AutoAIS", "FActScore", "QAFactEval"]
---

# 论文速读：EXPERTQA-Expert-Curated-Questions-and-Attributed-Answers

## 一句话总结
本文构建了 **EXPERTQA**，一个包含2177个跨32个领域专家自拟问题的长文本问答数据集，并通过领域专家对多类LLM系统的回答进行逐claim级别的归因验证、事实性评估与修订，揭示了现有系统在高风险领域的归因完整性与来源可靠性缺陷，同时验证了微调方法对该数据的提升效果。

## 研究问题与动机
1. **高风险领域的归因与事实性漏洞**：医学、法律等高危场景中，LLM自信地生成错误信息可能引发严重社会后果，但现有评估工作缺乏针对领域特定场景的细粒度归因与事实性分析。
2. **专家参与评估的成本与价值矛盾**：领域专家最适合评判专业问题，但专家标注耗时昂贵，现有工作鲜少让专家深度参与评估闭环。
3. **自动评估指标与人工判断脱节**：现有的AutoAIS、FActScore等自动归因/事实性估计方法与专家标注的一致性较差，缺乏在真实专家场景下的系统性检验。
4. **领域特定信息需求建模不足**：现有长文本QA数据集多源于搜索日志或论坛（如Eli5、ASQA），难以精准刻画专家群体的真实查询需求与挑战性技术问题。

## 核心贡献（创新点）
1. **构建首个大规模专家主导的跨领域归因问答基准**：收集484位来自32个领域的专家自拟问题（2177题），并要求同一专家对模型回答的每个claim进行多维度标注与修订，形成含verified answers的高质量标准。
2. **系统性对比三类归因范式在专家场景下的表现**：覆盖纯生成式（gpt4）、检索增强式（rr_sphere/rr_gs）与后处理归因式（post_hoc_sphere/post_hoc_gs）六大系统，揭示各范式的互补缺陷。
3. **量化高风险领域的归因脆弱性**：首次报告医学与法律领域分别存在35%和31%的不完整归因，且51%的医学claim归因来自专家判定不可靠的来源。
4. **全面评估自动事实性/归因指标与专家判断的相关性**：发现AutoAIS和FActScore在当前形态下均呈现高精确率但极低召回率，但经领域微调后可显著提升对齐程度。

## 方法详解
**两阶段专家闭环评估流程**：
- **Stage 1（问题采集）**：通过Prolific招募具备 formal education + 3年以上工作经验的专家，每人撰写5个问题（要求至少2个为情景型Type V问题），共回收2177题，覆盖7种信息需求类型（Table 2）。
- **Stage 2（回答验证与修订）**：用6类系统生成回答后，将每回答拆分为sentence-level claim（spaCy tokenizer），专家对每个claim评判6个属性（Table 3）：Answer Usefulness、Attribution（完整/部分/缺失）、Informativeness、Factuality（确定性四级）、Source Reliability、Cite-worthiness，并执行claim级修订。

**评估系统分类**：
- `gpt4`：纯生成，要求模型从参数记忆中生成URL（closed-book）。
- `rr_sphere_gpt4` / `rr_gs_gpt4`：Retrieve-and-Read，用BM25检索Sphere或Google Top-10 passages（k=5/10），再提示GPT-4基于context生成带inline citation的回答。
- `post_hoc_sphere_gpt4` / `post_hoc_gs_gpt4`：先由GPT-4生成无归因回答，再对每个claim单独检索证据。
- `bing_chat`：商业系统采样（balanced mode）。

**自动指标评估**：
- **AutoAIS**：使用TRUE NLI模型（t5_xxl_true_nli_mixture，11B）做claim-evidence entailment二分类；微调时DeepSpeed ZeRO Stage-3，lr=1e-4，batch=1，3 epochs。
- **FActScore**：先用text-davinci-003 few-shot分解为atomic claims，再用Google Search检索top-3 passage，用gpt-3.5-turbo判断True/False，按atomic claim平均得分。

**长文本QA基线**：在EXPERTQA随机/领域划分（80-10-10）上微调FlanT5-11B、Vicuna-7B、Llama2-7B/70B，评估ROUGE与QAFactEval。

## 实验与结果
- **数据规模**：2177个validated examples，平均每example 5.79 claims、152.12 tokens。
- **专家一致性**：作者在Engineering & Medicine两个领域的60×2抽样与参考标签agreement >85%。
- **归因质量**：Retrieve-and-read系统归因最完整，但18% cite-worthy claims仍缺少citation；后处理系统虽每claim都有归因，但不完整率更高。
- **高风险领域痛点**：医学35% incomplete attributions、法律31% incomplete attributions；51%医学claim证据来源被判定不可靠。
- **检索语料影响显著**：rr_gs_gpt4 AutoAIS得分0.778 vs rr_sphere_gpt4的0.689；Google搜索来源的source reliability评分明显高于Sphere。
- **自动指标表现**：AutoAIS zero-shot对gpt4的Precision=0.33、Recall=0.02；FActScore整体F1≈0.84，但对非事实claim的Recall仅0.07-0.16（Table 8）。微调后AutoAIS F1提升至0.87-0.92（Table 6）。
- **长文本QA微调效果**：Llama2-7B在Random split上R1=0.362、QFE=1.985；微调后性能显著提升，但Llama2-70B zero-shot（R1=0.320）仍低于微调后的中等规模模型，表明数据价值与进一步改进空间并存。

## 相关工作脉络
1. **Attribution Generation**：与WebGPT（Nakano et al., 2021）、RALM（Guu et al., 2020）、FLARE（Izacard et al., 2022）等检索增强工作形成对比——本文聚焦多范式横向对比而非提出单一新架构。
2. **Attribution Analysis**：延续Rashkin et al. (2021)的AIS框架与Bohnet et al. (2022)的AutoAIS，但首次在32个真实专家领域验证自动归因指标，揭示其low-recall本质。
3. **Factuality Estimation**：与FActScore（Min et al., 2023）、SelfCheckGPT（Manakul et al., 2023）等工作对话，指出当前factuality评测指标在高风险领域claim级别召回严重不足。
4. **Long-form QA Datasets**：区别于Eli5（Fan et al., 2019）、ASQA（Stelmakh et al., 2022）依赖搜索日志/论坛构建的方式，EXPERTQA从专家真实信息需求出发，并附带fine-grained claim-level factuality与归因标注。
5. **Domain-specific QA**：相比PubMedQA（Jin et al., 2019）、LegalBench（Guha et al., 2023）等单领域数据集，EXPERTQA实现32个领域的规模化覆盖与统一评估协议。
6. **Expert Evaluation in NLP**：与Peskoff & Stewart (2023)的10位专家小规模评估形成规模对比，本文484位专家参与、2177题标注构成最大规模专家-in-the-loop归因基准之一。

## 局限性与未来方向
1. **Claim原子性不足**：当前以sentence为粒度提取claim，一个sentence可能包含多条信息单元，导致factuality/attribution判断需穷举核查；更细粒度的atomic claim标注成本过高，未来可探索自动拆分+专家校验的混合机制。
2. **Tokenization误差**：spaCy sentence tokenizer在处理列表、表格时会产生过长claim，影响标注质量，需引入更鲁棒的claim分割方法。
3. **领域覆盖偏差**：尽管涵盖32个领域，但稀有领域（rare fields）专家招募困难；样本主要来自欧美非英语区专家，跨语言/跨文化 generalize能力待验证。
4. **查询分布的人为构造性**：专家自拟问题难以完全模拟真实query log的自然分布，未来可结合真实交互日志进行数据扩充。
5. **标签主观性**：同一领域专家间对某些claim属性的判断存在主观差异，当前仅通过2位作者的agreement做粗略估计，缺乏大规模多 annotator 交叉验证。

## 研究启发与可借鉴点
1. **专家-in-the-loop的两阶段闭环设计**：Stage 1征集问题 + Stage 2由同一专家验证/修订回答，既保证了问题真实性又实现了高质量的监督信号，该方法论可直接迁移至其他需要领域可信评估的方向。
2. **三类归因范式的横向对比协议**：纯生成/检索增强/后处理归因在同一套问题和标注体系下对比，揭示了各范式的互补缺陷，这种"控制变量式"的多系统评测框架值得复用。
3. **自动指标的微调对齐策略**：AutoAIS与FActScore在zero-shot下recall极低，但经领域数据微调后F1大幅提升（如rr_gs_gpt4从0.79到0.92），提示未来可用小规模专家标注数据做metric adapter而非直接部署pretrained evaluator。
4. **Claim级修订数据的训练价值**：专家修订后的answers构成强监督信号，微调中等规模模型（Llama2-7B）即可超越70B零样本表现，说明高质量领域数据比单纯scaling模型更有效。
5. **高风险领域作为压力测试标杆**：医学与法律领域暴露出最严重的归因缺陷，可将这类"hard domain"作为LLM安全评估的mandatory benchmark，推动系统在高风险场景下的trustworthiness改进。

## 关键术语表
**EXPERTQA**：本文提出的高质量长文本问答数据集，含2177个由32个领域专家自拟的问题及专家验证/修订后的回答与归因。
**AutoAIS（Automated Attributable to Identifiable Sources）**：基于NLI模型的自动归因估计方法，预测claim-evidence对是否满足"可溯源"条件。
**FActScore**：Min et al. (2023)提出的细粒度事实性评估指标，通过atomic claim分解+外部检索+LLM验证计算事实一致性得分。
**Retrieve-and-Read**：先检索相关证据passage，再以retrieved context为条件生成回答的范式（如rr_gs_gpt4）。
**Post-hoc Retrieval**：先生成无归因回答，再对每个claim单独执行事后检索以补充引用的范式。
**Cite-worthy**：专家判定"必须引用支持"的claim，约占全部claim的78-83%。
**Source Reliability**：专家对证据来源网站可信度的三级评判（Reliable / Somewhat Reliable / Not reliable at all）。
**QAFactEval**：Fabbri et al. (2022)提出的基于QA对的事实一致性评估指标，通过从reference answer生成问答对并检验模型回答的事实一致性。

## 可复现要素
- **数据集**：EXPERTQA，论文声明公开发布（标注"1"处有footnote，具体链接见原论文）。
- **代码/权重**：论文未明确提供代码仓库链接；模型权重使用开源组件（GPT-4 API、BingChat、LLaMA-2、Vicuna、FlanT5），AutoAIS使用t5_xxl_true_nli_mixture（Honovich et al., 2022）。
- **关键超参**：
  - GPT-4生成：temperature=1.0，max_tokens=2048。
  - Embedding：text-embedding-ada-002。
  - Retrieve-and-read检索：Sphere top-k=5（BM25），Google top-10。
  - Passage chunk：1000 tokens，overlap=200。
  - AutoAIS微调：lr=1e-4，batch=1，3 epochs，DeepSpeed ZeRO Stage-3。
  - 长文本QA微调（FlanT5-11B）：lr=1e-4，batch=2，max_seq_len=512，3 epochs；（Llama2/Vicuna-7B）：lr=2e-4，batch=4，max_seq_len=2048，3 epochs。
- **标注成本**：约$15/小时，Stage 2平均13.83分钟/question-answer pair。
