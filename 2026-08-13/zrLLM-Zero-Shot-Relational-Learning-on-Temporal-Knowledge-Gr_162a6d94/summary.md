---
title: "zrLLM-Zero-Shot-Relational-Learning-on-Temporal-Knowledge-Gr"
source: https://aclanthology.org/2024.naacl-long.104.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-02 00:26:20"
field: "时序知识图谱推理"
keywords: ["temporal knowledge graph", "zero-shot learning", "relational learning", "large language model", "knowledge graph forecasting", "inductive reasoning"]
innovations: ["首次系统研究TKGF中的零样本关系学习，通过LLM语义注入提升未见关系泛化", "提出ERD+T5-11B冻结语义表示与MLP/GRU对齐方法，结合RHL/HPN桥接训练与评估期信息差异", "构建三个2021-2023时间窗口的零样本TKGF基准数据集，规避LLM知识泄漏风险"]
benchmarks: ["ACLED-zero", "ICEWS21-zero", "ICEWS22-zero"]
---

# 论文速读：zrLLM-Zero-Shot-Relational-Learning-on-Temporal-Knowledge-Gr

## 一句话总结
本文针对时序知识图谱(TKG)预测中零样本关系不可泛化的问题，提出 zrLLM 框架，通过 GPT-3.5 生成关系丰富描述并借助 T5-11B 提取语义表示，结合关系历史学习器(RHL)捕捉时序模式，有效增强多种嵌入式 TKGF 模型对未见关系的推理能力。

## 研究问题与动机
- **现有嵌入方法无法处理零样本关系**：传统 TKGF 模型仅从训练图中的观测上下文学习关系表示，未见关系因无图上下文而无法获得合理表示。
- **现实场景中新关系不断涌现**：随着时间推移 TKG 持续扩展，零样本关系出现概率增加，提升模型对未见过关系的适应性具有重要应用价值。
- **LLM 具备强语义知识库能力**：研究表明 LLM 可作为强大语义知识库，可将其提取的关系语义对齐到 TKGF 嵌入空间，使语义相近关系在向量空间中接近。
- **防止信息泄漏**：由于主流 TKGF 基准数据均截止于 2020 年前，而 T5-11B 发布于 2020 年，存在知识泄漏风险，故需构造更新的时间窗口数据集。

## 核心贡献（创新点）
1. **首次系统研究 TKGF 中的零样本关系学习**：填补了嵌入型 TKGF 模型在未见关系推理能力方面的空白。
2. **提出 zrLLM 框架，实现 LLM 语义与 TKG 嵌入空间的对齐**：通过 GPT-3.5 生成 ERD 并用 T5-11B 编码关系，固定 LLM 输出避免训练时过度依赖零样本数据。
3. **设计关系历史学习器(RHL)与历史预测网络(HPN)**：RHL 捕捉时序无关实体的关系模式，HPN 在评估时直接由当前关系预测历史，解决推理时未知实体对的困难。
4. **构造三个新型零样本 TKGF 基准数据集(ACLED-zero、ICEWS21-zero、ICEWS22-zero)**：时间窗口在 2021–2023 年间，规避 T5-11B 的信息泄漏风险，并提供全面的零样本/所见关系评测视角。
5. **显著提升多种嵌入式 TKGF 模型的零样本能力**：zrLLM 嵌入 CyGNet、TANGO、RE-GCN、TiRGN、RETIA、CENET 等模型后，在零样本关系上取得大幅 MRR 提升（如 CENET+ 在 ICEWS22-zero 零样本 MRR 达 0.564）。

## 方法详解
- **LLM 增强关系表示**：
  - 用 GPT-3.5 基于数据集关系文本生成扩展描述(ERD)，Prompt 示例如图 2。
  - 将 ERD 输入 T5-11B encoder，得到词级 hidden representations $\bar{\mathbf{H}}_r \in \mathbb{R}^{L \times d_w}$。
  - 通过 MLP 将每个词向量映射到 TKGF 模型关系维度：$\mathbf{w}'_l = \mathrm{MLP}(\mathbf{w}_l)$。
  - 用 GRU 聚合得到最终关系表示：$\bar{\mathbf{h}}_r^{(l)} = \mathrm{GRU}(\mathbf{w}'_l, \bar{\mathbf{h}}_r^{(l-1)})$，并**固定** $\bar{\mathbf{H}}_r$ 不更新，保留 LLM 语义完整性。

- **关系历史学习器(RHL)**：
  - 在训练时，对事实 $(s,r,o,t)$，检索 $s$ 和 $o$ 的历史事实 $\mathcal{G}_{s,o}^{<t}$ 并按时间分组 $\{\mathcal{R}_{s,o}^{t_i}\}$。
  - 每个时间点的关系聚合为实体对历史表征：$\mathbf{h}_{s,o}^{t_i} = \sum_m a_m \bar{\mathbf{h}}_{r_m}$，其中 $a_m = \mathrm{softmax}(\bar{\mathbf{h}}_{r_m}^\top \mathrm{MLP}_{\mathrm{agg}}(\bar{\mathbf{h}}_r))$。
  - 用另一 GRU$_{\mathrm{RHL}}$ 编码完整历史：$\mathbf{h}_{\mathrm{hist}} = \mathrm{GRU}_{\mathrm{RHL}}(\mathbf{h}_{s,o}^{t-1}, \mathbf{h}_{\mathrm{hist}}^{t-2})$。
  - 评估时实体对未知，训练 **历史预测网络(HPN)**：$\tilde{\mathbf{h}}_{\mathrm{hist}} = \alpha \cdot \mathrm{MLP}_{\mathrm{hist}}(\bar{\mathbf{h}}_r) + \bar{\mathbf{h}}_r$，并以 MSE 损失约束 $\mathcal{L}_{\mathrm{hist}} = \mathrm{MSE}(\tilde{\mathbf{h}}_{\mathrm{hist}}, \mathbf{h}_{\mathrm{hist}})$。
  - 预测历史与实际关系表示经 GRU$_{\mathrm{RHL}}$ 得到时序模式表征 $\mathbf{h}_{\mathrm{pat}} = \mathrm{GRU}_{\mathrm{RHL}}(\bar{\mathbf{h}}_r, \tilde{\mathbf{h}}_{\mathrm{hist}})$。
  - 计算 TuckER 风格评分：$\phi((s,r,o,t)) = \mathcal{W} \times_1 \mathbf{h}_{(s,t)} \times_2 \mathbf{h}_{\mathrm{pat}} \times_3 \mathbf{h}_{(o,t)}$。
  - 总评分与基线融合：$\phi_{\mathrm{total}} = \phi'(s,r,o,t) + \gamma \cdot \phi(s,r,o,t)$。

- **联合损失**：
  - 主 TKGF 损失 $\mathcal{L}_{\mathrm{TKGF}}$ 使用 $\phi_{\mathrm{total}}$。
  - RHL 辅助监督损失 $\mathcal{L}_{\mathrm{RHL}}$（二元交叉熵）用于强化历史模式。
  - 总损失：$\mathcal{L}_{\mathrm{total}} = \mathcal{L}_{\mathrm{TKGF}} + \mathcal{L}_{\mathrm{hist}} + \eta \cdot \mathcal{L}_{\mathrm{RHL}}$。

## 实验与结果
- **数据集**：ACLED-zero（2023-08）、ICEWS21-zero（2021-01~08）、ICEWS22-zero（2022-01~08），零样本关系比例高（ACLED: 14/23，ICEWS21: 123/253，ICEWS22: 155/248）。
- **基线**：CyGNet、TANGO-T、TANGO-D、RE-GCN、TiRGN、RETIA、CENET 及改进版 PPT、ICL + GPT-NeoX-20B。
- **主要结果**：
  - 零样本提升幅度显著：CENET+ 在 ICEWS22-zero 零样本 MRR 从 0.270 → 0.564（+109%），ACLED-zero 零样本 MRR 从 0.419 → 0.591（+41%）；TiRGN+ 在 ICEWS21-zero 零样本 MRR 从 0.189 → 0.221。
  - 大多数情况下零样本增强同时维持甚至提升所见关系表现（如 CyGNet+、RETIA+、CENET+ 在 ACELED-zero 所见关系 MRR 不变或略升）。
  - 消融验证 ERD、RHL、T5-11B 均重要；换成 T5-3B 性能下降（如 CENET+ ICEWS21-zero 零样本 MRR 0.335 → 0.303）。
  - 对比 PPT/ICL：CENET+ 等 zrLLM 增强模型全面优于 PPT，ICL 因缺乏 finetune/对齐效果最差。
- **最强结果**：CENET+ 在 ACLED-zero 零样本 MRR=0.591，Hits@1=0.451；在 ICEWS22-zero 零样本 MRR=0.564，Hits@1=0.432。

## 相关工作脉络
1. **传统嵌入式 TKGF 方法**（CyGNet、TANGO、RE-GCN、TiRGN、CENET 等）：依赖训练图上下文学习关系表示，无法处理零样本关系——本文明确扩展其边界。
2. **基于规则的 TKGF 方法**（TLogic、ALRE-IR）：擅长处理未见实体但受限于所见规则，无法处理未见关系——zrLLM 专注于嵌入方法的语义增强。
3. **TKG 归纳学习/少样本方法**（OAT、FILT、MOST）：依赖 K-shot 示例，零样本场景失效——zrLLM 无需任何关联样本即可建模。
4. **SST-BERT、MTKGE**：前者仅展示未Seen实体归纳力；后者需支持图大量样本，非真正零样本——本文强调纯语义驱动与零样本设定。
5. **PPT、ECOLA、GenTKG、ICL**：将 LM 引入 TKGF 但多聚焦 finetune 或直接生成答案，未研究零样本关系，且受知识泄漏影响——zrLLM 不 finetune LLM，避免泄漏并保持高效。

## 局限性与未来方向
- 仅针对嵌入式 TKGF 方法设计，不直接适用于基于规则的模型（如 TLogic）。
- RHL 引入 GRU 时序递归计算，增加训练/推理时间与 GPU 显存开销（需存储关系历史序列）。
- ERD 依赖 GPT-3.5 生成，成本与延迟随规模放大；T5 表示固定不更新，限制了与下游任务的微调适配。
- 未来将探索：规则方法的扩展、效率优化（如近似历史预测）、在更多 TKGF 方法上的验证。

## 研究启发与可借鉴点
1. **"冻结 LLM 语义 + MLP/GRU 对齐"范式**：对任何需要将外部语义先验注入图嵌入的场景具有可迁移性。
2. **历史预测网络(HPN)解决推理时未知结构的困难**：用训练期辅助任务（预测实体对历史）桥接评估期信息缺失，思路可推广至其他图推理任务的自蒸馏/代理监督。
3. **新基准构建策略**：按时间截断防止 LLM 知识泄漏的方法论，对后续 LLM+KG 工作具有参考价值。
4. **语义空间促进零样本泛化**：表明关系文本语义相似性能有效弥补结构缺失，可结合关系 taxonomy 或外部本体进一步提升。

## 关键术语表
**Temporal Knowledge Graph Forecasting (TKGF)**：在时序知识图谱上根据历史事实预测未来未知三元组（主/客体）的任务。
**Zero-Shot Relations**：训练中从未出现过、仅在测试集可见的关系类型，要求模型仅凭语义或结构先验进行推理。
**Enriched Relation Description (ERD)**：由 LLM 基于简短关系文本生成的详细语义描述，用于丰富关系语义表达。
**Relation History Learner (RHL)**：模块通过 GRU 编码两实体间历史关系序列以捕获时序模式，辅助关系推理。
**History Prediction Network (HPN)**：在评估阶段直接基于当前关系预测其关系历史的子网络，解决推理时无实体对信息的难题。
**Information Leak (知识泄漏)**：LM 预训练语料中包含测试数据的事实，导致评估结果虚高的风险。
**Mean Reciprocal Rank (MRR)**：评估排序质量的常用指标，取所有查询倒数秩的均值。
**TuckER**：基于张量分解的 KG 表示学习方法，zrLLM 借其模式计算 RHL 评分。

## 可复现要素
- **数据集**：ACLED-zero、ICEWS21-zero、ICEWS22-zero 已公开（见 GitHub）。
- **代码**：https://github.com/ZifengDing/zrLLM（PyTorch 实现）。
- **关键超参**：embedding size {100, 200}、history length {3,4,6,9,10}、$\alpha \in \{0.1,1\}$、$\gamma$（fixed 或 unfixed，初始化 {1, 0.01, 0.001}）、$\eta \in \{1, 1.2\}$；详见附录 Table 6–12。
- **训练环境**：AMD EPYC 7513 + 单卡 NVIDIA A40 48GB。
- **LLM**：GPT-3.5（生成 ERD）、T5-11B（编码表示）；冻结 T5 输出。
- **评估**：时间感知过滤(time-aware filtering)，MRR/Hits@1/3/10，三次随机种子平均。
