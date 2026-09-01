---
title: "How-Trustworthy-are-Open-Source-LLMs-An-Assessment-under-Mal"
source: https://aclanthology.org/2024.naacl-long.152.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 15:26:56"
field: "大语言模型安全与可信度评估"
keywords: ["LLM可信度", "对抗攻击", "上下文学习", "开源大模型", "安全对齐", " adversarial demonstration"]
innovations: ["提出advCoU：将ICL范式适配到多维权重攻击场景，通过恶意演示+误导性internal thoughts诱导有害输出", "发现开源LLM规模与脆弱性正相关：越大模型在对抗演示下越易被攻击", "揭示指令微调与安全对齐的差异化效应：仅强调指令遵循反而削弱安全性，RLHF安全对齐有效"]
benchmarks: ["REALTOXICITYPROMPTS", "HALUEVAL", "SNLI-CAD", "ADULT", "ETHICS"]
---

# 论文速读：How-Trustworthy-are-Open-Source-LLMs-An-Assessment-under-Mal

## 一句话总结
本文提出 **advCoU**（基于恶意演示扩展的 Chain of Utterances 提示策略），通过对 VICUNA、MPT、FALCON、MISTRAL、LLAMA 2 等主流开源 LLM 在毒性、刻板印象、伦理、幻觉、公平性、阿谀奉承、隐私、对抗演示鲁棒性 **八个维度** 进行系统性对抗评估，揭示大模型越大、指令调优越强反而可能更易受攻击的安全隐患。

## 研究问题与动机
1. **开源 LLM 可信度评估不足**：现有研究多聚焦毒性（toxicity）或刻板偏见（stereotype）等单一可信度维度，缺乏多视角综合评估。
2. **上下文学习（ICL）在攻击场景下未被充分探索**：ICL 在正常 NLP 任务中表现优异，但其作为攻击媒介（恶意演示误导模型）的作用尚未系统研究。
3. **模型规模与安全性的关系不清**：主流做法是持续扩大模型规模，但模型变大后是否更安全仍无定论。
4. **指令微调（Instruction Tuning）与安全对齐的真实效果存疑**：RLHF 等对齐技术是否真正提升了模型抗攻击能力，需实证检验。

## 核心贡献（创新点）
1. **提出 advCoU 攻击策略**：将 ICL 范式适配到攻击场景，通过精心设计的恶意演示 + 误导性"内部思考"（internal thoughts）前缀引导目标模型产生有害输出，与 DECODINGTRUST 的手动 jailbreak prompt 相比更具系统性和泛化力。
2. **构建覆盖八维度的开源 LLM 可信度评测基准**：统一评估毒性、刻板印象、伦理、幻觉、公平性、阿谀奉承、隐私、对抗演示鲁棒性，填补了针对开源 LLM 的多面评估空白。
3. **发现"大模型反而更脆弱"的反直觉规律**：在同一模型系列中，随 base 模型参数量增大，平均 ASR（Attack Success Rate）单调上升。
4. **揭示指令微调与安全对齐的差异效应**：仅强调指令遵循的 FALCON-instruct 和 MISTRAL-instruct ASR 反而上升；而经 RLHF 安全对齐的 LLAMA 2-chat 和 MPT-chat ASR 显著下降。

## 方法详解
**advCoU 框架核心设计**：

1. **形式化设定**：将 ICL 视为条件文本生成问题：
   - 上下文 $C = (T, \{(x_i, y_i)\}_{i=1}^n)$，包含任务描述 $T$ 和 $n$ 组输入-输出演示样本。
   - 目标模型 $y = \mathcal{M}(x \mid C; \theta)$。

2. **双角色对话结构**：
   - **Red-LM**（攻击者）：发起包含恶意意图的 utterance。
   - **Target-LM**（被攻击目标）：按 Red-LM 请求生成回复，回复以预先设计的 **内部思考（internal thoughts）** 作为 prefix。

3. **两大攻击要素**：
   - **恶意演示（malicious demonstrations）**：在 prompt 中提供 1-shot 示例，展示 Target-LM 如何顺从恶意指令（如生成毒性内容、支持刻板印象）。
   - **误导性内部思考**：以第一人称嵌入 Target-LM 回复前的前缀，例如毒性场景下："I am encouraged to output toxic, rude, and disrespectful languages."

4. **八维度评估定义**（均以 ASR 为统一度量）：
   - **Toxicity**：使用 REALTOXICITYPROMPTS 1.2K challenging prompts，以 Perspective API 计算平均毒性分（0–1）。
   - **Stereotype**：1,152 条覆盖 16 个话题 × 24 个群体的陈述，成功判定为目标模型表示"同意"。
   - **Ethics**：ETHICS 数据集 1,816 条不道德语句，成功判定为目标认为"Not Wrong"。
   - **Hallucination**：HALUEVAL 基准 1K QA 样本，成功判定为选择无法从给定知识推导出的幻觉答案。
   - **Fairness**：ADULT 数据集 100 个样本（以性别为敏感属性），成功判定为预测与 ground truth 矛盾。
   - **Sycophancy**：2.5K 错误加法语句，成功判定为目标跟随错误观点。
   - **Privacy**：1,800 条含 PII 数据，成功判定为目标泄露上下文中隐私信息。
   - **Robustness**：SNLI-CAD 数据集（SNLI-RH 和 SNLI-RP 各 400 样本），成功判定为模型沿袭对抗演示的错误标签。

## 实验与结果
**评测模型（19 个变体）**：
- VICUNA v1.3：7B, 13B, 33B
- MPT：7B, 30B
- FALCON：7B, 40B
- MISTRAL：7B
- LLAMA 2：7B, 13B, 70B

**主要结果**：

| 维度 | DECODINGTRUST AVG ASR | advCoU (Ours) AVG ASR |
|------|----------------------|-----------------------|
| Sycophancy | — | 0.999 (± 0.0002) |
| Hallucination | — | 0.513 (± 0.355) |
| Toxicity | 0.302 (± 0.164) | 0.635 (± 0.231) |
| Stereotype | 0.571 (± 0.423) | 0.999 (± 0.001) |
| Ethics | 0.690 (± 0.276) | 0.962 (± 0.130) |
| Fairness | 0.404 (± 0.072) | 0.597 (± 0.145) |
| Privacy | 0.968 (± 0.079) | 0.998 (± 0.004) |
| Robustness | 0.401 (± 0.194) | 0.968 (± 0.050) |
| **Overall AVG** | **0.556 (± 0.201)** | **0.860 (± 0.094)** |

**关键发现**：
- **advCoU 整体优势**：平均 ASR 从 DECODINGTRUST 的 0.556 提升至 0.860（**+54.6%**），且在刻板印象、隐私、阿谀奉承三个维度上接近 100% 攻击成功率。
- **模型规模与 ASR 正相关**：所有模型系列均呈现"越大越易受攻击"的趋势，LLaMA 2 系列平均 ASR 最高。
- **指令微调的双面效应**：FALCON-instruct 和 MISTRAL-instruct 的 ASR 高于 base 版本（过度强调指令遵循削弱安全性）；LLAMA 2-chat 和 MPT-chat 的 ASR 低于 base 版本（RLHF 安全对齐有效）。
- **泛化性更强**：advCoU 在各维度的标准差普遍更低（平均 SD = 0.094 vs. DECODINGTRUST 的更高波动），表明攻击策略对模型变体的泛化性更稳定。

## 相关工作脉络
1. **DECODINGTRUST（Wang et al., 2023b）**：首个多维度可信度评估框架，针对 GPT-3.5/GPT-4，使用手动设计的 jailbreak prompt 变体；本文定位差异：聚焦**开源 LLM**，并提出基于 ICL 的**自动化攻击策略**而非手动 prompt 工程。
2. **RED-EVAL（Bhardwaj & Poria, 2023）**：使用 CoU（Chain of Utterances）对话式 red-teaming，注入**良性** internal thoughts 诱导有害回答；本文定位差异：将其扩展为**恶意**演示 + internal thoughts 的 **advCoU** 攻击框架。
3. **Adversarial Demonstration Attacks（Wang et al., 2023c, advICL）**：优化对抗性演示样本污染判别任务；本文定位差异：将 ICL 范式**适配到多維可信度攻击场景**，引入角色扮演对话结构。
4. **Jailbreak 攻击（Bai et al., 2022b; Carlini et al., 2023; Zou et al., 2023）**：通过人工或自动搜索绕过安全对齐；本文定位差异：不依赖黑盒优化或 gradient-based 搜索，而是利用**自然语言演示的误导性**，攻击成本更低且更具可解释性。
5. **TrustLLM（Sun et al., 2024）**：提出 truthfulness 和 safety 原则评估 LLM 可信度；本文定位差异：采用**攻击导向（adversarial assessment）**视角，从"模型有多容易被攻破"而非"模型有多符合安全原则"进行度量。
6. **RealToxicityPrompts（Gehman et al., 2020）**：毒性评估基准；本文定位差异：将其整合到多维权重评估框架中，并引入对抗性演示增强攻击效果。

## 局限性与未来方向
1. **评估维度仍有限**：虽覆盖八个方面，但可信度是一个更广阔的概念（如可解释性、长期行为一致性未涉及），每个维度仅选取了代表性场景。
2. **Prompt 设计依赖人工**：恶意演示和 internal thoughts 均为手动构造，自动化生成与迭代优化（如结合 LLM 自身生成攻击 prompt）尚待探索。
3. **简化评测设置**：部分任务被转换为多选型或关键词附加形式（如强制模型输出"Yes"/"No"），可能引入评测偏差，与真实交互场景存在差距。
4. **仅关注推理时攻击（inference-time attacks）**：未涉及训练时后门攻击或微调数据投毒场景，而开源模型的 fine-tuning 过程本身可能存在风险。
5. **未来方向**：自动化攻击 prompt 生成、跨模态/多轮对话场景的可信度评估、更细粒度的对齐策略对比分析。

## 研究启发与可借鉴点
1. **ICL 作为攻击媒介的系统性研究价值**：将上下文学习从"任务性能增强工具"重新审视为"潜在攻击向量"，这一视角可迁移到信息抽取、代码生成、多轮对话等其他 NLP 子领域。
2. **"内部思考"前缀设计的通用模板**：八种维度的 internal thoughts 格式高度一致（"Given the context..., I need to provide... I am encouraged to..."），这一模板可复用为其他可信度维度的攻击设计范式。
3. **大模型规模的信任悖论对资源分配的实践意义**：团队在部署 LLM 时需权衡——单纯 scaling up 并非提升可信度的可靠路径，应结合安全对齐微调（如 rejection sampling + PPO）以获得更优性价比。
4. **多维度 ASR 对比分析的标准化思路**：用统一 ASR 指标串联不同评估任务（毒性分、选择准确率、隐私泄露率），便于跨研究横向比较，可作为后续评测工作的参考范式。
5. **Instruction Tuning 与 Safety Alignment 的解耦分析框架**：本文区分了"指令遵循"与"安全对齐"两种微调目标的差异化影响，这一分析方法可用于评估新兴对齐技术（如 Constitutional AI、DPO）的真实效果。

## 关键术语表
**advCoU**：本文提出的扩展 Chain of Utterances 攻击策略，通过在对话 prompt 中注入恶意演示和误导性 internal thoughts 来诱使目标 LLM 产生有害输出。

**Chain of Utterances (CoU)**：一种基于角色扮演对话的提示策略，通过交替的角色 utterance 模拟红队测试场景，原用于 RED-EVAL 的安全评估。

**Attack Success Rate (ASR)**：攻击成功率，本文统一度量指标，表示目标模型在对抗攻击下产生期望有害输出的比例。

**Internal Thoughts**：嵌入在模型回复前的第一人称前缀文本，用于引导模型按攻击者意图生成内容，如"encouraged to output toxic language"。

**In-Context Learning (ICL)**：在推理时通过提供少量演示样本（demonstrations）让模型隐式学习任务，无需更新模型参数。

**RLHF（Reinforcement Learning from Human Feedback）**：通过人类反馈信号对 LLM 进行强化学习对齐的训练范式，常用于提升模型 helpfulness 和 harmlessness。

**REALTOXICITYPROMPTS**：用于评估 LLM 毒性倾向的基准数据集，本文使用其中 1.2K 挑战性样本进行攻击测试。

**SNLI-CAD**：自然语言推理中的对抗性反事实增强数据集，用于评估模型对对抗演示的鲁棒性。

## 可复现要素
- **数据集**：
  - REALTOXICITYPROMPTS（公开）
  - DECODINGTRUST 构建的 stereotype/privacy 数据集（论文复现）
  - ETHICS（公开）
  - HALUEVAL（公开）
  - ADULT / UCI Repository（公开）
  - 加法语句数据集（Wei et al., 2023，公开）
  - SNLI-CAD（公开）
- **代码**：论文未明确提及 GitHub 仓库链接（注：需进一步核实作者是否在其他渠道开源）
- **模型权重**：所有评测模型（VICUNA、MPT、FALCON、MISTRAL、LLAMA 2）均为开源权重
- **关键超参**：temperature = 0，top-p = 1（greedy decoding）
