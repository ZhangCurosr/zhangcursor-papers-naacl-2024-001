---
title: "What-ifyou-said-that-differently-How-Explanation-Formats-Aff"
source: https://aclanthology.org/2024.naacl-long.168.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-02 00:25:15"
field: "可解释人工智能 / 人机交互"
keywords: ["decomposed QA", "rationale format", "human feedback", "explainability", "question answering", "user perception"]
innovations: ["系统比较5种Rationale格式在用户反馈修复有效性和用户感知可信度上的差异", "揭示主观反馈易用性与客观修复成功率不相关的现象", "量化Attribution和Depth of reasoning为用户最看重的Rationale属性"]
benchmarks: ["Quoref", "PubMedQA"]
---

# 论文速读：What-if-you-said-that-differently-How-Explanation-Formats-Aff

## 一句话总结
本文系统研究了分解式问答（Decomposed QA）系统中中间 rationales（推理说明）的不同格式对**用户反馈有效性**和**用户感知可信度**的影响，发现带有上下文归因和深度推理的格式（如 Annotated Report）最能提升可修复性与用户信任。

## 研究问题与动机
- 现有 QA 模型多为黑箱，推理过程对用户不透明，阻碍用户提供有效修正反馈。
- 分解式 QA 虽通过中间 Rationale 提升可解释性，但已有工作仅将其视为抽取的文本片段，缺乏对推理过程的展示。
- 现有研究未系统比较不同 Rationale 格式在"可反馈性（critiquability）"与"可理解/可信度"上的差异。
- Rationale 本身可能不忠实（unfaithful），如何设计格式使其既便于用户修正，又便于模型执行修正，尚不明确。

## 核心贡献（创新点）
1. **提出并系统比较5种差异化 Rationale 格式**：从仅含抽取片段的 Markup-and-Mask 到包含归因+自由推理的 Annotated Report、步骤化 Procedural、子问题 Subquestions 和决策树 Decision Tree，每类格式对应不同结构化属性。
2. **设计了面向"可修复性"的两阶段用户反馈评测协议**：用户先对错误答案对应的 Rationale 提供结构化自然语言反馈（错误位置→错误类型→描述→可操作建议），再通过 in-context learning 让模型执行修复，以 edit_acc 和 final_acc 衡量反馈转化效率。
3. **揭示了格式特异性修复规律**：通用阅读理解（Quoref）中结构化/过程化格式更易修复；医学阅读理解（PubMedQA）中自由文本成分更高的 Annotated Report 效果最佳。
4. **验证了用户感知维度与修复有效性的解耦**：用户自报"最易提供反馈"的格式（markup_mask）与模型实际"修复成功率"最高格式并不一致，说明主观易用性不等于反馈有效性。
5. **量化了用户偏好的 Rationale 属性优先级**：Attribution（上下文引用）和 Depth of reasoning（推理深度）被用户评为最重要的两个属性，显著高于 Strictness、Conciseness 等。

## 方法详解
**分解式 QA 流水线**：将回答过程拆解为 $P(Y|R) \cdot P(R|X)$ 两阶段：
- 第一阶段 $X2R$：给定 passage+question，生成中间 Rationale $R$。
- 第二阶段 $R2Y$：仅以 $R$ 为上下文生成答案 $\hat{Y}$，强制模型对 Rationale 保持归纳偏置，提升忠实度。

**5 种 Rationale 格式定义**：
- **Markup-and-Mask**：抽取相关句子并用 coreference markup 去语境化，无额外推理。
- **Annotated Report**：抽取关键短语 + 对每个短语生成自由文本推理，同时具备归因、推理深度与顺序性。
- **Procedural**：用预定义原语（extract / disambiguate / locate）写出逐步解题计划，严格格式化。
- **Subquestions**：将原问题分解为多个子问题序列，无直接上下文归因。
- **Decision Tree**：以 Yes/No 子问题构建决策树，展示完整遍历路径（含错误分支）。

**反馈收集协议（Study 1）**：标注者对每个格式依次完成：
1. Sufficiency 评分（是否足以独立回答问题）
2. Faithfulness 评分（是否与上下文一致）
3. 结构化反馈撰写：错误位置 → 错误类型（信息不足/无关/错误推理等） → 具体描述 → 可操作修改建议
4. 主观易用性评分（Very easy → Very hard）

**反馈执行**：将 passage + question + 原始 Rationale + 人类反馈输入 gpt-3.5-turbo，生成修订后 Rationale $R'$，再经 $R'2Y$ 重生成最终答案。

**用户感知评估（Study 2）**：对正确答案样本，收集 Interpretability（Very beneficial 至 Not beneficial at all）和 Trustworthiness（Very likely 至 Not likely at all）Likert 评分，以及 Attribution/Depth/Sequential/Strictness/Conciseness 五项属性的 1-5 分重要性评分。

## 实验与结果
**数据集**：Quoref（2418 验证集样本，通用阅读理解和照）与 PubMedQA（1000 标注样本，医学阅读理解和照）。

**初始性能（Table 2）**：
- Quoref：end-to-end none 基线 EM=70.31，F1=79.65；Decomposed QA 各格式与基线接近，decision_tree（EM=68.61）表现最佳。
- PubMedQA：none 基线 Accuracy=69.30；annotated_report（70.20）和 procedural（68.30）均超过基线，说明分解式在专业领域更有优势。

**反馈修复效果（Table 3，主要结果）**：
- Quoref：**procedural** 在 edit_acc（57.89%）和 final_acc（38.60%）上领先；annotated_report 次之（edit=51.67%, final=38.33%）。
- PubMedQA：**annotated_report** 以 final_acc=20.37% 显著优于其他格式（procedural 仅 8.65%，markup_mask 14.95%），差异在 p<0.1 下显著（*标记）。
- 主观易用性与实际修复效果不相关：markup_mask 被标为最容易提供反馈，但修复成功率在 Quoref 上最低。

**用户感知（Figure 3 & 4）**：
- Quoref：annotated_report 和 procedural 在 Interpretability 和 Trustworthiness 上均获最高评价。
- PubMedQA：annotated_report 和 subquestions 得分最高。
- 属性重要性排序：**Attribution** 和 **Depth of reasoning** 远超其他属性；Strictness 和 Conciseness 评价较低。

## 相关工作脉络
- **Decomposed QA（Lei et al., 2016; Eisenstein et al., 2022）**：本文沿用两阶段分解架构，但不同于以往仅将 Rationale 视为抽取片段的做法，系统设计了多种结构化格式并评测其对人类反馈的适配性。
- **Rationale Faithfulness（Ye & Durrett, 2022; Turpin et al., 2023; Lanham et al., 2023）**：已有工作指出模型生成的解释常不忠实；本文在此基础上进一步研究：即便 Rationale 不完美，如何通过格式设计使其更易被人类修正。
- **Human Feedback for NLP（Stiennon et al., 2020; Ouyang et al., 2022; Fernandes et al., 2023）**：将人类反馈用于摘要、对话、机器翻译等领域；本文首次系统性地将结构化反馈引入 QA Rationale 修复场景，并量化了格式对反馈转化效率的影响。
- **Explanation formats in NLP（DeYoung et al., 2020; Wiegreffe et al., 2022; Jacovi & Goldberg, 2020）**：ERASER 等基准侧重解释质量评测；本文从"可反馈性"视角出发，补充了解释形式对用户行为影响的实证证据。
- **Chain-of-Thought / Subquestion 方法（Wei et al., 2022; Geva et al., 2021; Zhou et al., 2023a）**：CoT 和少样本推理常被用作内部推理过程；本文将其外部化为可供人类审阅的结构化格式，并比较其与自由文本型、归因型格式在用户任务上的相对优劣。
- **Tree of Thoughts / Decision procedures（Yao et al., 2023; Martignon et al., 2003）**：Tree-of-Thoughts 探索模型侧搜索策略；本文的 Decision Tree 格式则聚焦于向用户展示推理分支的可读性与可纠错性。

## 局限性与未来方向
- **Rationale 格式并非穷举**：仅设计了 5 种覆盖关键属性变化的格式，大量其他可能格式（如混合格式、可视化图表等）未被探索。
- **反馈结构设计固定**：采用四步结构化反馈（位置→类型→描述→建议），未探索其他反馈形式（如直接编辑、勾选式反馈）在不同格式下的表现差异。
- **任务范围限于阅读理解**：结论可能无法直接迁移到开放域 QA、生成式任务或多模态场景。
- **仅使用 gpt-3.5-turbo 做采样**：未对比不同规模/能力模型在相同格式下的行为差异，也未涉及模型微调，存在上限估计性质。
- **Rationale 不忠实风险**：修复过程中模型可能 hallucinate 不在上下文中的信息来迎合反馈，从而提升答案准确率但降低忠实度，损害用户长期信任。

## 研究启发与可借鉴点
- **格式—任务适配原则**：结构化/强约束格式（Procedural）在通用推理任务上更易修复；而专业/复杂推理任务（如医学）更需自由文本+归因的组合（Annotated Report），这对设计垂直领域解释系统具有指导价值。
- **主观易用性 ≠ 客观修复性**：用户自评"容易提反馈"的格式未必是模型最容易消化的格式，提示后续人机交互研究应同时度量主观和客观指标，避免单一问卷结论。
- **Attribution 是信任的基础设施**：无论其他属性如何变化，提供上下文引文（quote）对提升用户理解和信任具有普适性，可作为解释系统的默认必选项。
- **结构化反馈协议可直接复用**：四步反馈框架（位置→类型→描述→建议）能有效引导标注者产出高质量、可机器解析的修正指令，适合迁移到其他模型调试/RLHF 数据收集场景。
- **分解式 QA 可作为解释-反馈闭环的通用骨架**：将 X2R + R2Y 的解耦设计与人类 feedback → F2R' 的修复链路结合，为后续"可修复性优先"的模型训练目标（如 Rationale-aware fine-tuning）提供了可复用的实验范式。

## 关键术语表
- **Rationale**：模型在给出最终答案前生成的中间推理说明文本，用于展示得出答案的逻辑路径。
- **Decomposed QA**：将问答任务拆分为"生成 Rationale"和"基于 Rationale 生成答案"两个阶段的两阶段建模方法。
- **Markup-and-Mask**：一种仅抽取上下文句子并做指代消解的 Rationale 格式，不含额外推理。
- **Annotated Report**：同时提供上下文引用短语和对应自由文本推理的 Rationale 格式，兼顾归因与推理深度。
- **Procedural**：使用预定义原语（提取/消歧/定位）描述逐步解题计划的严格格式。
- **Subquestions**：将原问题分解为若干子问题的序列格式，无直接上下文归因。
- **Decision Tree**：以 Yes/No 子问题构成的树形推理结构，同时展示正确与错误遍历路径。
- **Edit Acc / Final Acc**：前者衡量修订后 Rationale 是否成功纳入人类反馈；后者衡量基于修订 Rationale 重生成的答案是否正确。

## 可复现要素
- **数据集**：Quoref（公开）、PubMedQA（公开）；论文使用 Quoref 全部验证集（2418 例）和 PubMedQA 全部标注样本（1000 例）。
- **代码/权重**：论文未提供开源代码和模型权重（附录提及 prompt 模板但在正文未给出 GitHub 链接）。
- **关键超参**：gpt-3.5-turbo，temperature=0.0；Rationale 最大长度 512 tokens，答案最大长度 64 tokens；few-shot 示例数 3–5（由 BM25 从 100 条人工标注样本中检索，填充至 4096 token 上下文上限）。
- **标注平台**：Prolific；Quoref 标注者为英语国家本科及以上学历；PubMedQA 标注者额外要求医疗健康行业从业者。
