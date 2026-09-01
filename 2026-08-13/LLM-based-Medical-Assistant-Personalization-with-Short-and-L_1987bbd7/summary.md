---
title: "LLM-based-Medical-Assistant-Personalization-with-Short-and-L"
source: https://aclanthology.org/2024.naacl-long.132.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 15:28:29"
field: "个性化大语言模型与记忆增强"
keywords: ["个性化LLM", "医疗助手", "记忆增强生成", "参数高效微调", "LoRA", "DPeM", "长短期记忆"]
innovations: ["提出受神经科学双过程启发的DPeM三重记忆机制（工作记忆/STM/LTM），相比字典式记忆提升约7%", "结合DPeM与LoRA的MaLP统一框架，实现低成本高效果的个性化医疗助手", "提出基于自聊模拟的个性化医疗对话数据集构建方法"]
benchmarks: ["HealthCareMagic-100k", "iCliniq", "ROUGE-1/ROUGE-L", "Preference Classification Accuracy", "Response Generation Win Rate"]
---

# 论文速读：LLM-based Medical Assistant Personalization with Short- and Long-Term Memory Coordination

## 一句话总结
本文提出 **MaLP**（Medical Assistant with Long-term Personalization），通过结合受神经科学启发的**双流程增强记忆机制（DPeM）**与**参数高效微调（LoRA）**，在低成本下实现面向个性化需求的医疗助手。

## 研究问题与动机
- 现有记忆增强方法多为**字典式（key-value）结构**，不够灵活，且高度依赖检索器能力；
- 完全微调 LLM 以实现个性化成本极高，而纯提示工程（prompt-based）方法效果有限；
- 患者具有不同的**沟通偏好**（如简洁 vs 详细）和**病史背景**，需要在对话过程中持续学习和记忆用户特异性知识与常识性知识；
- 现有医疗对话数据集缺乏对用户偏好和历史记录的系统性建模。

## 核心贡献（创新点）
- 提出 **DPeM（Dual-Process enhanced Memory）** 机制：借鉴神经科学双过程理论，引入工作记忆、短期记忆（STM）、长期记忆（LTM）三种记忆形式协同工作，相比传统字典式记忆约提升 7%；
- 提出 **MaLP 统一框架**：结合 DPeM 与 PEFT（LoRA），通过排练流程（Rehearsal）和执行流程（Executive）双流程实现知识的筛选、分类与持久化存储，使 LLM 响应质量显著提升；
- 发布新的**个性化医疗对话数据集**：包含用户偏好设定和历史对话记录，支持该方向的后续研究；
- 提出离线式医学知识适配方案（Adapter + Knowledge Loss + Sample Loss），缓解在注入领域知识时发生的**灾难性遗忘**问题。

## 方法详解
### 2.2 医学知识适配
在 LLM 中添加**领域适配器（domain adapter）**，结构为：下投影层 → ReLU 非线性 → 上投影层（全连接网络）。为防止灾难性遗忘，设计两个损失函数：
- **知识损失**：$\mathcal{L}_K = -\frac{1}{K}\sum_{i=1}^{M}\log p(m_i)$（基于 K 个 masked token 的生成概率）；
- **样本损失**：$\mathcal{L}_S = ||V_o - V_k||_2^2$（原始层与适配器层输出向量间的欧氏距离）。

### 2.3 DPeM 机制
借鉴 **Dual-process theory（Kahneman, 2011）**，包含**排练流程（Rehearsal）**与**执行流程（Executive）**两部分。

**排练流程（两步）：**
1. **Learning**：协调器（Coordinator，如 ChatGPT）从每轮对话 $d_i$ 中提取笔记 $nts = \mathcal{C}(d_i)$，存入**工作记忆**（每轮刷新）；
2. **Summarizing**：筛选出有用笔记 $nt^+$ 逐条存入 **STM**（周期性刷新）。

**执行流程：**
- STM 中的知识 $k_j$ 被标记为**常识知识**或**用户特异性知识**（key-type : value 形式）；
- 用**标志表（flag table）**记录每个 $k_j$ 的访问频次，达到阈值 $\theta$ 后转入 **LTM**（不可删除，无限容量）；
- 三类记忆共同协作：工作记忆缓冲最新信息，STM 存储近期关键知识，LTM 存储高频访问的持久知识。

### 2.4 MaLP 框架
**记忆生成（2.4.1）：**
$$\mathcal{M} = [\mathcal{M}_{working}, \mathcal{M}_{STM}, \mathcal{M}_{LTM}]$$

**记忆利用（2.4.2）：**
采用 **LoRA**（低秩分解 $W_\Phi + BA$，rank $r=8$，alpha=32）对用户历史对话进行轻量微调。新查询 $x$ 的处理流程为：
1. 检索器从 $\mathcal{M}$ 中检索相关 prompt $p = Retriever(x)$；
2. 将 $x$ 与 $p$ 一并输入 LoRA 微调后的 LLM，生成个性化回答 $y$。

**检索器设计（2.4.3）：**
- **STM 检索**：使用**最接近匹配检索器（$\mathcal{R}_c$）**，基于 **Levenshtein 距离**；
- **LTM 检索**：使用**语义匹配检索器（$\mathcal{R}_s$）**，基于**余弦相似度**（训练 encoder 获取语义 embedding）。

## 实验与结果
### 数据集与实验设置
- 医学知识适配使用 **HealthCareMagic-100k** 和 **iCliniq** 数据集；
- 个性化对话数据集由 60 个用户、平均 182 轮对话、总计 10,920 段对话组成（共 131,040 轮 utterance）；
- 基础模型：GPT-3.5、LLaMA-7B、LLaMA-13B；
- 优化器：AdamW，LoRA rank=8，alpha=32，输入最大 1024 tokens，输出最大 2048 tokens；
- 硬件：2 × Tesla V100 GPU。

### 评估任务与指标
- **Profile/Knowledge QA**：ROUGE-1、ROUGE-L；
- **偏好分类**：Accuracy；
- **回复生成**：Win Rate（与标准生成对比），辅以人工评估（Pearson 相关系数 0.72，准确率 84%）。

### 主要结果
| 模型 | 任务 | Standard | w/ Mem | w/ DPeM | w/ LoRA | w/ MaLP |
|------|------|----------|--------|---------|---------|---------|
| GPT-3.5 | Profile QA (ROUGE-L) | 30.81 | 34.27 | 38.78 | — | — |
| GPT-3.5 | Pref. Classification | 36.31% | 41.73% | 47.72% | — | — |
| GPT-3.5 | Resp. Win Rate | — | 80.91% | 86.60% | — | — |
| LLaMA-7B | Profile QA (ROUGE-L) | 19.82 | 20.44 | 20.97 | 29.66 | **33.91** |
| LLaMA-7B | Knowledge QA (ROUGE-L) | 23.69 | 31.17 | 33.98 | 33.60 | **36.37** |
| LLaMA-7B | Pref. Classification | 21.42% | 21.15% | 33.06% | 61.05% | **69.95%** |
| LLaMA-7B | Resp. Win Rate | — | 78.41% | 84.60% | 72.01% | **91.53%** |
| LLaMA-13B | Profile QA (ROUGE-L) | 21.02 | 21.39 | 22.01 | 29.96 | **34.63** |
| LLaMA-13B | Resp. Win Rate | — | 78.92% | 84.81% | 71.93% | **91.27%** |

**最强结果**：MaLP + LLaMA-7B 在 Response Generation Win Rate 上达到 **91.53%**，相比 Standard 提升约 **13.12%**；偏好分类准确率达 **69.95%**，相比 Standard 提升约 **48.53%**。

**关键发现**：
- DPeM 相对于 dict-based Mem：GPT-3.5 上 Profile QA ROUGE-L 提升 **13.16%**，L TM 上 Preference Classification 提升 **11.64%**（vs Standard）；
- LoRA 单独对偏好分类效果显著（提升约 39.63%），但单独使用时 Response Generation 弱于有记忆的方法；
- MaLP 结合两者实现全面最优。

## 相关工作脉络
- **Madaan et al. (2022) – Self-refine / Memory-assisted prompt editing**：通过存储错误-反馈对并检索来修正输出；本文与之的本质区别在于设计**拟真的三重记忆结构**而非简单字典，并引入双流程实现知识的自动化筛选与持久化；
- **Lewis et al. (2020) – RAG**：通过检索增强生成；本文强调记忆结构本身的改进（类型化存储），而不仅依赖检索器；
- **Yunxiang et al. (2023) – ChatDoctor**：在 LLaMA 上注入医学知识微调；本文在此基础上增加了**用户个性化记忆模块**；
- **Salemi et al. (2023) – LAMP**：在预训练阶段注入用户画像；本文采用**PEFT + 对话历史记忆**的低成本方案，无需重新预训练；
- **Wang et al. (2023) / Wu et al. (2023)**：通过 CoT prompt 引导个性化生成；本文指出提示方法效果低于微调，且对 prompt 格式敏感，转而采用 DPeM + LoRA 联合方案；
- **Tandon et al. (2021) / Dalvi et al. (2022)**：纠正模型输出错误；本文扩展为同时学习用户偏好（而非仅错误修复）的全面个性化框架。

## 局限性与未来方向
- **离线记忆**：当前 DPeM 以离线方式运行，无法从新查询中在线学习，仅作为辅助 prompt；未来计划将记忆机制内化到 LLM 中；
- **遗忘机制简单**：当前基于频率计数，缺乏对"回避学习"（如"触碰火会导致恐惧"）等复杂场景的处理；未来计划引入学习 schema/loss 控制回避行为；
- **可扩展性不足**：面对百万级用户时，为每个用户分配独立模型成本过高；未来考虑**联邦学习**（Federated Learning）或多用户共享层 + 社区特征的方案，同时需解决隐私泄露问题。

## 研究启发与可借鉴点
- **DPeM 的三重记忆结构设计**：工作记忆→STM→LTM 的分层存储与周期性/触发式转移机制，可迁移到其他需要长程记忆的个人化对话场景（如客服、教育助手）；
- **排练流程与执行流程的双流程设计**：将知识"学习与筛选"和"存储与检索"解耦，为记忆增强 LLM 提供了可复用的架构范式；
- **知识适配+样本损失防灾难性遗忘**：适配器结构配合 $\mathcal{L}_K$ 和 $\mathcal{L}_S$ 的双损失方案，可用于任何需要将领域知识注入基础 LLM 的任务；
- **结合 PEFT 与外部记忆**：LoRA 侧重捕获用户偏好，DPeM 侧重捕获对话知识，二者互补的思路为个性化 LLM 的系统设计提供了清晰的模块化参考；
- **自聊对话数据集构建方法**：通过注入用户画像让 LLM 进行角色扮演对话模拟，为低资源领域对话数据集的构建提供了一种可复用的数据生成范式。

## 关键术语表
**DPeM（Dual-Process enhanced Memory）**：受神经科学双过程理论启发的记忆机制，包含工作记忆、短期记忆和长期记忆三部分协同工作。

**MaLP（Medical Assistant with Long-term Personalization）**：本文提出的统一框架，结合 DPeM 记忆机制与 LoRA 微调实现个性化医疗助手。

**Rehearsal Process（排练流程）**：包括 Learning 和 Summarizing 两个步骤，负责从对话中获取和筛选知识并存入工作记忆与 STM。

**Executive Process（执行流程）**：负责评估 STM 中的知识并按访问频率将其持久化到 LTM。

**LoRA（Low-Rank Adaptation）**：一种参数高效微调技术，通过低秩分解更新预训练权重，只训练少量参数即可适配新任务。

**PEFT（Parameter-Efficient Fine-Tuning）**：参数高效微调的统称，指仅更新少量参数即可实现领域/用户适应的微调策略。

**Levenshtein Distance（莱文斯坦距离）**：衡量两个字符串之间的编辑距离，用于 STM 的最接近匹配检索。

**Catastrophic Forgetting（灾难性遗忘）**：模型在学习新领域知识后，原有通用能力显著下降的现象。

## 可复现要素
- **数据集**：医学知识适配使用 HealthCareMagic-100k 和 iCliniq（开源）；个性化医疗对话数据集由作者生成并发布，论文声明已公开；
- **代码**：论文声明已公开实现代码（脚注 1 标注）；
- **关键超参**：LoRA rank=8，alpha=32；学习率 5e-5（MaLP 训练）/ 1e-4（知识适配）；batch size=20；warm-up 10%；weight decay 1e-4；输入最大 1024 tokens，输出最大 2048 tokens；
- **硬件**：2 × Tesla V100 GPU（32G 显存），256G CPU；
- **框架**：PyTorch、PEFT（HuggingFace）、Transformers。
