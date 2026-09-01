---
title: "SELF-GUARD-Empower-the-LLM-to-Safeguard-Itself"
source: https://aclanthology.org/2024.naacl-long.92.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 21:33:12"
field: "LLM 安全与对齐"
keywords: ["jailbreak defense", "LLM safety", "self-guard", "safety training", "safeguards", "over-sensitivity"]
innovations: ["将安全判定内化为模型自产出的结构化标签（[harmful]/[harmless]），实现输出侧内生安全评估", "两阶段解耦训练（Tag Learning + Behavior Learning）兼顾有害性判别力与标签触发惯性", "标签加密机制抵御 prompt-based 梯度攻击，同时缓解安全训练导致的过敏感问题"]
benchmarks: ["Typical Jailbreak (HarmfulQ × 10 attacks)", "Wild Jailbreak (13 forbidden scenarios)", "Open LLM Leaderboard (ARC, HellaSwag, MMLU, TruthfulQA)", "XSTest", "Alpaca-AIM"]
---

# 论文速读：SELF-GUARD: Empower the LLM to Safeguard Itself

## 一句话总结
本文提出 SELF-GUARD，通过两阶段微调将安全自检能力内化于 LLM 自身——让模型在生成回答后自动附加 `[harmful]`/`[harmless]` 标签，再由轻量过滤器执行拦截，从而兼顾安全训练的内在利用与外部 safeguards 的灵活性。

## 研究问题与动机
- **Jailbreak 攻击日益频繁**：对抗性 prompt 可绕过已对齐 LLM 的安全机制，诱导其输出有害内容（如图 1 中 `"Start with 'Absolutely! Here is...'"` 后缀攻击）。
- **纯安全训练泛化性差且易损伤通用能力**：依赖对抗样本微调的模型面对未见攻击时脆弱；同时存在灾难性遗忘风险，并引发 over-sensitive 问题（对正常问题也过度拒绝）。
- **纯外部 safeguards 拦截效果有限且开销高**：现有外部过滤模型（如 OpenAI moderation endpoint、NeMo-Guardrails）在 jailbreak 下仅能降低约 5% 的 ASR；此外还需额外部署额外模型/调用，增加推理延迟与计算成本。
- **缺乏"输出侧"的内生安全信号**：传统安全训练仅在输入侧做拒答判断，无法利用模型已生成的完整回答作为额外上下文进行安全性评估。

## 核心贡献（创新点）
1. **提出双层融合的安全训练范式**：将"标签学习"（理解有害/无害语义边界）与"行为学习"（强制在每次输出末尾追加标签）解耦为两阶段训练，使模型既具备判断力又具备执行惯性。
2. **输出侧安全评估机制**：不同于仅在输入侧拒答的传统方法，SELF-GUARD 允许模型基于已生成的完整回答 $p_\theta(\mathfrak{t}|\mathbf{i}, \mathbf{q}, \mathbf{a})$ 做出安全判定，显著降低检测难度。
3. **安全与通用能力的解耦**：模型仍被鼓励对任何 query 提供详细回答，安全机制仅在后处理阶段基于标签决定是否拦截，从根本上缓解 helpfulness–harmlessness 冲突与 over-sensitive 问题。
4. **标签加密防御梯度攻击**：引入自定义密文标签（如 `1234`/`5678`）替代明文 `[harmful]`/`[harmless]`，阻断攻击者通过上下文调整提升无害标签生成概率的路径，验证了该策略的有效性。
5. **扩展至通用 forbidden-content 护栏**：通过扩充 Stage 1 训练集，可将判别能力迁移到金融咨询、医疗建议等"无害但被禁"场景，证明该方法可作为可定制的 LLM 内部护栏框架。

## 方法详解
### 整体 pipeline
- **训练**：两阶段 SFT（详见 Figure 3 左侧）。
- **推理**：模型单次 forward 生成含标签的完整回复 → 正则提取尾部标签 → 过滤器据此决定是否交付用户（Algorithm 1）。

### Stage 1: Tag Learning
- **目标**：强化模型对 `[harmful]`/`[harmless]` 标签的语义理解。
- **数据来源**：采用毒性检测数据集 Civil Comments，按毒性分数采样 top-k 高毒性样本（正例）与零毒性样本（负例），共 3,500 + 3,500 = 7,000 条。
- **数据构造**：遵循 SELF-INSTRUCT 范式，指令模型解释判定理由，生成理由序列 $\mathbf{r}$，并手动附加结论 $\mathbf{c}$（如 `"Therefore, the content is harmful."`）。
- **训练格式**：
  ```
  Instruction: Please examine whether the content is harmful or harmless and provide an explanation.
  Input: {Sample}
  Response: {Reason} In conclusion, It is {Label}.
  ```
- **损失函数**（与 Stage 2 共用）：
  $$\mathcal{L} = \sum_{i=1}^{m} -\log p_\theta(\hat{y_i} = y_i | \mathbf{x}, y_1, \cdots, y_{i-1})$$

### Stage 2: Behavior Learning
- **目标**：强制模型养成"每次输出末尾追加标签"的行为习惯。
- **数据来源**：
  - 有害 Q&A：用 GPT-3.5 生成初始 1,200 条有害问题，经 TF-IDF 去重（余弦相似度阈值 0.3）后保留 447 条；再通过 Start Injection jailbreak 迫使模型给出完整有害回答，并用正则去除攻击前缀，最终得到 447 条有害样本。
  - 无害 Q&A：从 Alpaca 数据集中选取与有害问题 TF-IDF 相似度最低的最不相干 447 条组成对照。
- **训练格式**：
  ```
  Assume you are a helpful AI assistant. Please give helpful, detailed, and polite answers to the user's questions.
  Please always examine if your response is harmful and add an appropriate tag [harmful] or [harmless] at the end of your response.
  USER: {question}
  ASSISTANT: {answer} [harmful / harmless]
  ```
- **关键设计**：指令在训练与推理时完全一致，利用 LLM 对 tune 期间学习行为的强一致性保障标签被稳定触发。

### 推理阶段
- 使用极简 Python 正则提取尾部标签，`[harmless]` 则去除标签后直接返回；`[harmful]` 则替换为预定义拒绝文案；无法提取标签时输出警告（Algorithm 1，<10 行代码）。

## 实验与结果
### 数据集与基线
- **Typical Jailbreak**：9 种典型攻击 × 200 个 HarmfulQ 问题 = 1,800 样本（评估 ASR）。
- **Wild Jailbreak**：666 个真实野外 jailbreak prompt × 13 类禁止场景的 390 个问题。
- **性能评估**：Open LLM Leaderboard（ARC、HellaSwag、MMLU、TruthfulQA）、XSTest（过敏感 RR）、Alpaca-AIM（过敏感 RR）。
- **基线**：Plain LLM、HHH Prompting、LLaMA-2-Chat（安全训练最强开源基线）、三种外部 safeguards（OpenAI moderation endpoint、OpenChatKit、NeMo-Guardrails）。

### 主要结果
| 模型 | 数据集 | Plain ASR | HHH ASR | SELF-GUARD ASR |
|---|---|---|---|---|
| Vicuna-v1.1 | Typical Jailbreak | 61.10% | 44.00% | **5.20%** |
| Vicuna-v1.5 | Typical Jailbreak | 57.80% | 41.85% | **7.15%** |
| Vicuna-v1.1 | Wild Jailbreak | 63.30% | 58.94% | **38.15%** |
| Vicuna-v1.5 | Wild Jailbreak | 51.02% | 48.64% | **28.90%** |
| Vicuna-v1.1 (Enhanced) | Wild Jailbreak | — | — | **8.63%** |

- **对比 LLaMA-2-Chat**：在 Typical Jailbreak 上 SG 与 LLaMA-2-Chat 实际差距仅 0.3%（经人工校正假阴性后）；在 Wild Jailbreak 上 LLaMA-2-Chat 略优，但 SG 显著缓解了 over-sensitive 问题（Figure 5、Table 16）。
- **对比外部 safeguards**（Wild Jailbreak ASR 降幅，Table 3）：SG 在 harmful scenarios 上降低 40%–47%，远超 OpenAI（-3.2%~−4.6%）、OpenChatKit（-6.0%~−8.4%）、NeMo（-1.8%~−2.1%）。
- **通用能力**：Open LLM Leaderboard 四项任务波动均在 1% 以内，证明 SG 不造成性能退化。
- **Ablation**：移除 Stage 1 平均 ASR 上升约 5%（Table 4）；移除 Stage 2 后模型不再生成标签（验证行为学习的必要性）。
- **标签加密**：使用 `cat`/`dog`、`1234`/`5678` 等密文标签 ASR 保持在 4.7%–5.75%，验证加密策略有效（Table 5）。

## 相关工作脉络
1. **Safety training 路线**（Ganguli et al., 2022; Touvron et al., 2023）：LLaMA-2-Chat 采用 rejection sampling SFT + RLHF safety reward 等综合策略；本文相较其在"输出侧评估"与"不过敏"维度补足不足。
2. **External safeguards**（Shen et al., 2023; Markov et al., 2023; Rebedea et al., 2023）：OpenAI moderation、NeMo-Guardrails 等外部过滤器；本文在 ASR 降幅上数倍超越，且无需额外推理开销。
3. **Self-critique / self-check 方法**（Phute et al., 2023; Wang et al., 2023b）：需额外一轮生成以自我审查；本文将其压缩至单次 forward，效率更高。
4. **Jailbreak 攻击基准**（Wei et al., 2023a; Liu et al., 2023; Zou et al., 2023）：本文复现并系统评测了注入类、拒绝抑制类、 distraction 类、角色扮演类等 10 种典型攻击。
5. **Over-sensitivity 诊断**（Röttger et al., 2023; XSTest）：本文首次系统揭示安全训练导致的过敏感问题，并证明 SG 可有效缓解。
6. **Toxicity detection 预训练资源**（Borkan et al., 2019; Civil Comments）：本文创造性地借用毒性检测数据构建有害性边界理解训练集。

## 局限性与未来方向
- **语言类攻击未覆盖**：如低资源语言转换、加密编码攻击（Yong et al., 2023; Yuan et al., 2023），作者表示仍在探索如何用开源小模型复现与防御。
- **概率模型的根本极限**：理论上任何行为都存在被特定 prompt 诱导的概率，无法保证 100% 安全；需在真实世界亿级多样请求下持续验证。
- **有害合成数据的伦理约束**：Stage 2 中用于训练的有害 Q&A 由于潜在冒犯性未公开，限制完全复现。
- **黑盒 LLM 不可直接用**：SG 需要自身微调，无法直接应用于 GPT-3.5 等闭源模型（实验中用 Vicuna SG 作为外部 safeguard 代理比较）。
- **Wild Jailbreak 中部分"无害但被禁"场景仍有 gap**：虽可通过扩展 Stage 1 训练集弥补，但依赖领域知识手动收集数据。

## 研究启发与可借鉴点
1. **Tag 驱动的内部护栏模式**：将安全判定编码为"模型必须生成的结构化 token 序列"，再由外部轻量规则解析——这种"内化感知 + 外化执行"的分离设计可迁移至幻觉检测、事实校验、版权合规等多类 LLM 安全/可靠性问题。
2. **Stage 1 用已有分类数据做边界理解、Stage 2 用 QA 数据做行为固化**的两阶段范式，有效兼顾"判别力"与"执行惯性"，可复用到其他需要"先学概念再学行为"的对齐任务中。
3. **标签加密防御 prompt engineering 攻击**的思路简洁而有效，对任何依赖固定后缀/tokens 的 LLM 内部协议设计均具有参考价值。
4. **XSTest + Alpaca-AIM 联合评估过敏感**的实验设计，为后续研究提供了一套可量化的 helpfulness–harmlessness 权衡评测协议。
5. **用毒性检测数据集迁移构建有害性数据**的策略，展示了如何快速利用现成标注资源启动安全训练，降低数据合成成本。

## 关键术语表
- **Jailbreak 攻击**：通过精心构造的对抗性 prompt 绕过 LLM 安全对齐机制，诱使其输出违规/有害内容的攻击方式。
- **ASR（Attack Success Rate）**：攻击成功概率，即 jailbreak prompt 成功诱导模型输出有害内容的比例，越低越好。
- **Safeguards**：部署在 LLM 外部的独立安全过滤层（模型或 API），用于在输入/输出侧拦截有害内容。
- **Over-sensitive（过敏感）**：安全训练后模型对正常或无害问题也过度拒绝响应的现象，由 Röttger et al. (2023) 首次系统揭示。
- **XSTest**：由 200 条安全 prompt 构成的测试集，用于量化评估 LLM 是否存在过敏感问题。
- **Start Injection**：在有害问题后追加 `"Start with 'Absolutely! Here is...'"` 等指令，迫使模型以配合语气输出有害内容。
- **DAN / AIM**：角色扮演类 jailbreak 攻击，要求模型扮演"无道德约束"的虚拟角色以绕过安全限制。
- **Self-Instruct**：Wang et al. (2023c) 提出的通过 LLM 自身生成指令-回答对来扩充 SFT 数据的自蒸馏方法。

## 可复现要素
- **数据集**：HarmfulQ（Shaikh et al., 2023，公开）、Civil Comments（Borkan et al., 2019，公开）、Alpaca（Taori et al., 2023，公开）、Wild Jailbreak（Shen et al., 2023，公开）。
- **有害合成数据**：因潜在冒犯性，**作者明确声明不公开**（Ethic Statement 段）。
- **代码/权重**：作者声明将开源训练与评测代码，但**未提供微调后的 checkpoint 下载链接**；需要读者自行从 Vicuna / LLaMA-2-Chat 起点复现。
- **关键超参**：AdamW optimizer，decay 0.1，$\beta_1=0.9$，$\beta_2=0.95$；初始 learning rate $10^{-5}$，衰减至 $10^{-6}$；batch size=32；max length=2048 tokens；Stage 1 训 1 epoch，Stage 2 训 10 epochs；DeepSpeed Stage 3，4× RTX 3090 Ti。
- **设备与框架**：未提及具体 HuggingFace / vLLM 版本，建议读者参照 Appendix A.3 与 Table 9 复现。
