---
title: "Making-Language-Models-Better-Tool-Learners-with-Execution-F"
source: https://aclanthology.org/2024.naacl-long.195.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 15:29:32"
field: "大模型工具使用与强化学习"
keywords: ["Tool Learning", "执行反馈", "选择性工具使用", "强化学习", "两阶段训练", "LoRA微调"]
innovations: ["提出TRICE两阶段框架结合行为克隆与执行反馈强化学习，实现选择性工具使用", "设计基于工具执行结果的排名损失与五档评分奖励策略", "利用模型预测正确性自动构建工具使用训练数据，无需人工标注"]
benchmarks: ["ASDiv", "GSM8K", "WebQuestions", "NaturalQuestions", "TriviaQA", "T-REx", "MLQA", "MultiArith", "AddSub", "HotpotQA"]
---

# 论文速读：Making-Language-Models-Better-Tool-Learners-with-Execution-F

## 一句话总结
论文提出了TRICE框架，通过两阶段训练（行为克隆 + 基于执行反馈的强化学习）教大语言模型**何时以及如何使用工具**，使模型从盲目调用工具转变为**选择性、智能地使用工具**，在多个基准上超越GPT-3.5。

## 研究问题与动机
1. **现有方法缺乏工具使用的选择性判断**：当前工具学习方法（如Toolformer、ToolLLaMA等）倾向于让LLM无差别地使用工具，即使对于模型自身就能解决的简单任务也强制调用工具。
2. **盲目引入工具会传播错误**：对于简单问题，引入工具反而可能导致工具类型选择错误、工具输入生成错误或工具返回结果利用不当，造成"损失大于增益"。
3. **缺乏标注数据**：工具使用的精确标注难以获取，且难以泛化到未见场景，需要自动化的数据生成方案。
4. **核心问题**：能否教会语言模型**何时用、如何用**工具，实现选择性工具使用？

## 核心贡献（创新点）
1. **提出TRICE两阶段端到端训练框架**：将行为克隆与基于执行反馈的强化学习（RLEF）结合，使模型学会选择性使用工具，区别于仅做指令微调或提示工程的前人工作。
2. **设计基于执行反馈的排名损失函数**：通过多候选响应（来自ChatGPT、Vicuna等不同模型）结合准确率与工具使用一致性进行五档评分，引导模型偏好正确且审慎的工具使用策略。
3. **自动化工具标注数据构建方案**：利用未微调模型的正确/错误预测自动判定是否需要工具，并用ChatGPT生成伪标签工具API，无需人工标注。
4. **在8个数据集、4类任务上验证有效性**：TRICE-MIX在Alpaca-7B和Vicuna-7B上分别超越GPT-3.5达↑2.9%和↑4.1%，证明选择性工具学习的有效性。

## 方法详解
**整体框架（TRICE）包含两阶段：**

**阶段一：行为克隆（Behavior Cloning）**
- 对训练数据 $\mathcal{D}_{tool} = \{(s, q, t, a)\}$ 进行指令微调（instruct-tuning）
- 输入为 $[s, q]$，输出规则：若无需工具则生成答案 $a$，若需工具则生成工具API $t = \text{tool\_name}(\text{tool\_input})$
- 损失函数：$\mathcal{L}_{clone}(\theta) = \sum_{(s,q,t,a) \in \mathcal{D}_{tool}} -\log p_{LM}(o | s, q; \theta)$

**阶段二：基于执行反馈的强化学习（RLEF）**
- 为每个问题 $q$ 收集 $k$ 个候选响应 $y_i$（来自ChatGPT、InstructGPT、Vicuna、Alpaca等）
- 以训练数据中的工具标签输出作为金标准（gold response）
- **奖励策略**：五档评分，优先级为 True&Yes > True&No > False&Yes > False&No（True表示答案正确，Yes表示工具使用与金标准一致）
- **排名损失**：$\mathcal{L}_{rank} = \sum_{r_i < r_j} \max(0, p_i - p_j)$，其中 $p_i$ 为条件对数似然长度归一化
- **SFT正则损失**：$\mathcal{L}_{sft}$ 防止模型偏离原始参数分布
- **总损失**：$\mathcal{L}_{RLEF} = \alpha \cdot \mathcal{L}_{rank} + \mathcal{L}_{sft}$

**数据构建**：用未微调LLM预测答案，正确则设 $t=$ None，错误则用ChatGPT few-shot生成伪标签工具API。

## 实验与结果
**数据集与工具**：4类任务、8个数据集——数学推理（ASDiv/SVAMP/GSM8K + Calculator）、问答（WebQuestions/NaturalQuestions + WikiSearch）、知识缺失QA（TriviaQA/T-REx + QA模型Atlas）、多语言QA（MLQA + 翻译器NLLB）。

**主干模型**：ChatGLM-6B、Alpaca-7B、Vicuna-7B，全部使用LoRA微调。

**主要结果**：
- **单工具学习（TRICE-SPLIT）**：Alpaca-7B和Vicuna-7B平均仅低于GPT-3.5 1.3%和0.4%；相比Few-Shot提示基线，ChatGLM提升↑14.0%、Alpaca↑15.3%、Vicuna↑11.9%
- **多工具学习（TRICE-MIX）**：Alpaca-7B和Vicuna-7B超越GPT-3.5分别达↑2.9%和↑4.1%；相比SPLIT提升超过↑4.0%平均分
- **泛化能力**：在未见数据集MultiArith（66.6 vs 45.5）、AddSub（80.5 vs 49.1）上显著优于Few-Shot；引入新工具（Retriever）在HotpotQA上提升↑6.7%
- **最强结果**：Vicuna-7B TRICE-MIX在ASDiv上达到81.2%，超越GPT-3.5的64.6%达+16.6pp

## 相关工作脉络
1. **Toolformer (Schick et al., 2023)**：6B模型通过instruct-tuning自我学习工具调用，但无执行反馈，无法解决选择性使用问题 → 本文通过RLEF引入反馈机制，超越其选择能力
2. **ToolkenGPT (Hao et al., 2023)**：将工具作为特殊token嵌入LLaMA，无反馈机制 → 本文无需修改模型架构，通过RL对齐实现选择性
3. **HuggingGPT (Shen et al., 2023) / Chameleon (Lu et al., 2023)**：基于prompt的复杂工具编排方法，依赖≥100B模型 → 本文聚焦更小模型（6-7B）的端到端训练，更具可扩展性
4. **Gorilla (Patil et al., 2023) / ToolLLaMA (Qin et al., 2023b)**：纯指令微调学习工具API调用，无法判断何时不使用工具 → 本文在两者基础上增加执行反馈阶段，纠正过度依赖
5. **RRHF (Yuan et al., 2023)**：基于rank的RLHF变体 → 本文借鉴其排名损失思想，但将反馈来源从偏好标注替换为工具执行结果
6. **RLAIF (Lee et al., 2023)**：用AI反馈替代人类反馈 → 本文使用工具执行结果（而非另一个LLM的偏好判断）作为反馈信号

## 局限性与未来方向
1. **迭代执行反馈的计算成本**：依赖trial-and-error的反馈机制更适合虚拟环境，现实物理场景耗时较大
2. **仅评估了6-7B小规模模型**：未验证更大模型或不同架构下的效果
3. **仅限单一工具或同类型工具组合**：尚未支持多工具复杂组合使用
4. **实验规模受限**：仅覆盖4个任务8个数据集，工具种类相对有限
5. **未来方向**：探索更科学的反馈机制、扩展至更大规模模型、支持多工具组合编排及工具创建

## 研究启发与可借鉴点
1. **执行反馈作为训练信号的创新**：用工具实际执行结果（而非人类偏好或另一个LLM的判断）作为RL反馈信号，避免了额外标注成本，思路可迁移至其他agent决策训练
2. **两阶段训练的稳定性设计**：先BC打基础、再RL精调的策略有效避免了纯RL训练的稀疏奖励问题，类似范式可应用于其他工具使用或reasoning任务
3. **基于正确/错误预测自动构建训练数据的方案**：利用模型自身预测质量自动判定数据标签，为低资源工具学习提供了低成本数据构建范式
4. **混合训练（MIX）促进跨任务泛化**：跨任务混合训练比单任务单独训练效果更优，证明工具学习的跨任务可迁移性
5. **可与团队方向结合**：在知识增强推理、代码生成工具调用等场景中，可借鉴RLEF框架引入执行反馈进行模型校准

## 关键术语表
**TRICE**：Tool leaRning wIth exeCution fEedback，本文提出的两阶段端到端工具学习框架
**RLEF**：Reinforcement Learning with Execution Feedback，基于执行反馈的强化学习阶段
**Behavior Cloning**：第一阶段，通过指令微调让模型模仿工具调用行为
**Rank Loss**：排名损失，用于引导模型对候选响应按奖励分数排序
**Selective Tool Learning**：选择性工具学习，模型学会判断何时需要使用工具
**Candidate Response**：候选响应，从不同模型（ChatGPT、Vicuna等）收集的用于排名比较的回答
**知识冲突（Knowledge Conflicts）**：模型内部知识与工具增强知识之间可能产生的冲突

## 可复现要素
- **数据集**：ASDiv、SVAMP、GSM8K、WebQuestions、NaturalQuestions、TriviaQA、T-REx、MLQA（均为公开数据集）
- **代码**：论文未明确声明开源代码仓库
- **权重**：论文未声明开源微调权重
- **关键超参**：LoRA r=8, alpha=16, dropout=0.05；Stage-I学习率{1e-4, 3e-4}、5 epochs、batch=48；Stage-II学习率{1e-4, 2e-5}、2 epochs、α∈{0.01, 0.1, 1}；max_length=2048（ChatGLM）或512（Alpaca/Vicuna）
