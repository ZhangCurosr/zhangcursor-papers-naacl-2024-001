---
title: "Strings-from-the-Library-of-Babel-Random-Sampling-as-a-Stron"
source: https://aclanthology.org/2024.naacl-long.122.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-02 00:23:25"
field: "提示工程与语言模型应用"
keywords: ["prompt optimization", "random sampling", "separator", "in-context learning", "text classification", "language model"]
innovations: ["证明随机采样的separator可媲美人类设计和自优化方法", "提出三种从词表/语言模型分布采样separator的策略", "揭示语言空间中超过40%随机separator优于人类baseline"]
benchmarks: ["SST-2", "SST-5", "MR", "CR", "MPQA", "Subj", "TREC", "AGNews", "DBPedia", "GSM8K"]
---

# 论文速读：Strings-from-the-Library-of-Babel-Random-Sampling-as-a-Stron

## 一句话总结
本文证明从语言模型词表中随机采样token作为"分隔符"(separator)可用于提示风格文本分类，其性能可媲美甚至超越人类手工设计的separator，成为prompt优化研究的强基线。

## 研究问题与动机
- **核心问题**：现有的prompt优化方法通常假设有效的separator必须是任务相关且语义连贯的，但这一假设是否成立？
- **现有方法不足**：主流方法（如OPRO、APE、EvoPrompt等）依赖大型语言模型生成替代提示词，需要指令微调模型、复杂元提示设计和高计算成本。
- **研究动机**：探索随机采样能否在prompt空间中有效搜索到高性能separator，并重新评估之前方法贡献度的真实性。

## 核心贡献（创新点）
1. **提出随机separator优化框架**：仅通过随机采样词表/token分布即可生成separator，无需语言模型参与生成过程，与OPRO等方法形成对比。
2. **验证三类随机策略的有效性**：从无任务的Random Vocabulary到有上下文的Random with Context，系统分析相关性对separator质量的影响。
3. **确立强基线并质疑已有工作**：发现随机方法相比人类baseline"Answer:"平均提升12%，与自优化方法差距<1%，暗示先前方法贡献可能被高估。
4. **揭示语言空间的丰富性**：超过40%的随机separator性能优于人类设计，证明有效separator大量存在于语言空间中。

## 方法详解
- **框架组成**：随机separator生成 → separator评估 → separator选择三阶段。
- **三种生成策略**：
  1. **Random Vocabulary**：从词表中随机采样token直到达到预设长度限制，完全context-free和task-agnostic。
  2. **Random w/o Context**：从语言模型的先验分布中采样生成自然语言短语，仍无上下文约束。
  3. **Random with Context**：将训练集样例作为context输入meta-prompt，从条件分布中采样任务相关的separator。
- **评估机制**：给定小训练集$T=\{(x_i,y_i)\}$，用transform将标签映射为文本，计算准确率$m$作为separator评分，指标不需要可微分。
- **选择机制**：维持采样预算$k$，记录所有separator及其得分$S=\{(s_i,m_i)\}$，选择最高分用于测试。

## 实验与结果
- **数据集**：9个文本分类任务（SST-2、SST-5、MR、CR、MPQA、Subj、TREC、AGNews、DBPedia）和1个数学推理任务GSM8K。
- **模型**：8个语言模型（GPT2-Large/ XL、Mistral 7B、Llama2 7B/Chat、Llama-Alpaca 7B、Mistral 7B Instruct、ChatGPT）。
- **主要结果**：
  - Random Vocabulary相对人类baseline平均提升**10%**，Random w/o Context提升**12%**，Random with Context提升**12%**。
  - 与OPRO、APE等自优化方法差距不足**1%**。
  - 超**40%**的随机separator性能优于人类baseline。
  - 在GSM8K上，最佳随机separator相比CoT提升**23%**相对准确率。
- **最强结果**：Random with Context在Mistral 7B上取得SST-2准确率92.7%，超过OPRO的86.3%。

## 相关工作脉络
- **AutoPrompt (Shin et al., 2020)**：最早发现 unnatural prompts 可产生好性能，但依赖梯度信息；本文方法无需梯度且不依赖特定模型类型。
- **OPRO (Yang et al., 2023)**：用元提示让LLM学习优化pattern；本文证明随机采样可达到相近效果。
- **APE (Zhou et al., 2022)**：生成多个备选后选择/改写；本文随机方法更简单且不需要LLM参与生成。
- **EvoPrompt (Guo et al., 2023)**：将进化算法引入prompt优化；本文随机方法仅差3.4%即可作为更强基线。
- **PromptBreeder (Fernando et al., 2023)**：自参考自我改进框架；本文强调随机方法的竞争力，提醒领域注意overclaiming风险。

## 局限性与未来方向
- **局限性**：主要在文本分类任务上验证，生成式任务的全面评估有待补充（虽有GSM8K初步验证）。
- **未来方向**：扩展到更广泛的生成任务（对话、摘要等）、探索transferability机制、分析LLMs对随机prompt的敏感性本质。

## 研究启发与可借鉴点
- **降低基线重要性**：提出强随机基线可防止新方法的overclaiming，建议后续prompt优化研究必须对比随机baseline。
- **方法可迁移**：三种随机生成策略可直接应用于其他NLP任务的prompt设计，无需复杂元提示工程。
- **实验设计借鉴**：使用少量标注数据（64个样本）即可验证方法有效性，降低实验门槛。
- **跨任务/上下文transferability分析**：展示了系统评估separator泛化能力的严谨实验设计。

## 关键术语表
- **Separator**：插入在输入末尾或输出开头的特殊token/短语，用于引导模型预测行为。
- **In-context Learning (ICL)**：通过提供少量示例让预训练语言模型完成特定任务的学习范式。
- **Random Vocabulary**：从词表直接随机采样token生成separator的策略。
- **OPRO**：Large Language Models as Optimizers，使用元提示让LLM学习prompt优化pattern的方法。
- **CoT (Chain-of-Thought)**：通过"Let's think step by step"等推理提示激发模型推理能力。
- **Relative Improvement**：相对提升百分比，以人类baseline为参照的计算方式。

## 可复现要素
- **数据集**：9个文本分类数据集（SST-2、SST-5等）和GSM8K，均为公开数据集。
- **代码**：论文未明确提及开源代码仓库。
- **权重**：使用公开模型（GPT2、Mistral、Llama2系列），可从HuggingFace获取。
- **关键超参**：训练集每dataset 64 samples；temperature=1.0生成，0.0分类；随机方法生成160个候选；OPRO最多40步每步4个候选。
