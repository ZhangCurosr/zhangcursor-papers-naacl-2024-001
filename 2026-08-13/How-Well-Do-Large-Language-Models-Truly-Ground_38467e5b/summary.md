---
title: "How-Well-Do-Large-Language-Models-Truly-Ground"
source: https://aclanthology.org/2024.naacl-long.135.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 20:10:02"
---

# 论文速读：How-Well-Do-Large-Language-Models-Truly-Ground

## 一句话总结
本文针对现有大语言模型（LLM）“grounding”评估标准过于宽泛、仅检查答案是否出现或粗略NLI相关性的问题，提出了基于**原子事实**的严格 grounding 定义（同时要求高召回与高精确），构建了含冲突与干扰变体的四版本数据集，并在 25 个不同规模与训练方式的 LLM 上系统评测，揭示了训练信号、干扰上下文相关性与 gold 位置对 grounding 性能的决定性影响。

## 研究问题与动机
- **现有 grounding 定义过窄**：既往工作多以“答案是否出现在生成文本中”或“NLI 模型判定上下文与回答的相关性”来衡量 grounding，无法捕捉回答遗漏关键信息或混入参数知识幻觉的问题。
- **高准确率≠高 grounding**：模型完全可能凭借内部参数知识生成长篇正确回答，导致传统准确率指标虚高，但细粒度知识溯源失败，在实际企业/私有场景中存在严重可靠性风险。
- **缺乏可归因知识来源的评测数据**：现有 QA 数据集未标注“哪些上下文知识是回答所必需的（gold）”，也难以区分知识是来源于外部输入还是模型预训练记忆。
- **需系统剖析影响 grounding 的多维因素**：上下文 popularity、多文档推理复杂度、回复格式、指令微调/RLHF/DPO 训练方式、干扰上下文相关性等关键变量尚未被统一量化评测。

## 核心贡献（创新点）
1. **提出严格的 grounding 形式化定义与细粒度评估框架**：将原子事实作为最小知识单元，要求模型同时满足“覆盖所有 gold 原子事实”与“不引入上下文外知识”，与以往仅依赖答案匹配或粗粒度 NLI 的评估方法本质不同。
2. **构建四版本新数据集以隔离知识来源与干扰**：设计 Original-Gold/Dist 与 Conflict-Gold/Dist 四个变体，通过人工反事实修改 gold 知识构建冲突集、通过 Contraiever 检索构造高相似干扰上下文，使参数知识与外部上下文知识得以清晰分离。
3. **确立基于 Cross-encoder 的高相关自动评估指标**：选用 MSMARCO 微调的 Cross-encoder MiniLM 作为 $M_{eval}$，其与人工标注的相关性（84.1）显著优于 GPT-4（78.7）及多数 NLI 模型，为大规模 grounding 自动评测提供了可靠替代方案。
4. **系统评测 25 个 LLM 并提炼可复现的工程洞察**：首次量化对比不同训练方式、模型规模与上下文结构对 grounding 的影响，指出指令微调/RLHF 在提升绝对性能的同时反而放大干扰敏感度，为 RAG 系统部署提供明确选型与提示工程依据。

## 方法详解
- **Grounding 形式化定义**：给定外部上下文集合 $\mathcal{C}$、生成回答 $P$，将两者均拆解为原子事实集合 $\mathcal{C}_A$ 与 $\mathcal{P}_A$，并标注必要原子事实 $\mathcal{C}_G$。模型真正 grounded 需同时满足：
  1. $\forall k \in \mathcal{C}_G,\ k \in P$（Recall：不遗漏必要知识）
  2. $\forall k \in \math
