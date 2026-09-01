---
title: "PromptFix-Few-shot-Backdoor-Removal-via-Adversarial-Prompt-T"
source: https://aclanthology.org/2024.naacl-long.177.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 20:12:35"
field: "自然语言处理安全与鲁棒性"
keywords: ["Backdoor Removal", "Prompt Tuning", "Few-shot Learning", "Adversarial Defense", "NLP Security", "Soft Prompts"]
innovations: ["提出基于对抗性软提示调优的少样本后门去除框架，冻结原模型参数。", "利用软令牌和双层优化替代触发器反演与微调，适应少样本并减少性能下降。", "引入基于自身表征的良善提示正则化，有效防止少样本过拟合。"]
benchmarks: ["TrojAI", "IMDB", "BoolQ", "AmazonReview"]
---

# 论文速读：PromptFix: Few-shot Backdoor Removal via Adversarial Prompt Tuning

## 一句话总结
本文提出 PromptFix，一种面向自然语言处理（NLP）模型的少样本后门缓解新策略。该方法冻结受攻击模型参数，通过引入两组额外的软令牌（soft tokens）——分别模拟触发器和抵消触发器——在对抗性提示调优的框架下逐步移除后门，从而在极少样本（2/4-shot）场景下有效降低攻击成功率（ASR）的同时更好地保留了模型在干净数据上的原始性能。

## 研究问题与动机
*   **核心问题：** 在少样本微调/提示（prompting）范式下，如何有效检测并移除嵌入在预训练语言模型（PLM）中的后门，以应对供应链安全风险。
*   **现有方法不足：**
    1.  **触发器反演不准确：** 现有方法（如 DBS、PICCOLO）依赖梯度上升寻找可微分的触发器近似，但找到的触发器与真实触发器往往相去甚远，且反演效果高度依赖于对触发器注入方式（如位置）的枚举假设。
    2.  **微调在少样本下易过拟合：** 触发器反演后的去噪微调阶段需要较多数据以防止过拟合，这与少样本设置相冲突，且两阶段优化的误差会传播，导致模型干净准确率显著下降。
    3.  **缺乏对复杂后门的适应性：** 随着后门触发条件日益复杂（如条件激活、结构触发），枚举搜索空间不可行，现有两阶段方法难以适应。

## 核心贡献（创新点）
1.  **首个专为少样本设计的 NLP 后门去除方法：** PromptFix 摒弃了传统两阶段（反演+微调）框架，首次将对抗性提示调优引入 NLP 后门防御领域，直接适配参数效率极高的少样本学习场景。
2.  **基于软令牌的对抗性双层优化机制：** 不试图反演硬触发器文本，而是优化一组可学习的软触发器令牌（近似最坏情况触发器）和一组软修复提示令牌，通过 min-max 对抗优化同步实现后门发现与削弱，避免了触发器配置枚举。
3.  **保持模型参数冻结与性能保留：** 仅通过添加极少量可学习的提示参数进行后处理，原始预训练模型权重完全冻结。结合 BENIGN PROMPT REGULARIZATION (CLS loss)，有效防止修复提示本身在少样本下对干净性能造成破坏。
4.  **良好的通用性与兼容性：** 方法无需知道确切的触发器注入函数，软令牌及其优化过程能自动适应不同的触发位置、条件等后门配置。可与标准提示调优流程结合，实现域适应与后门去除同步进行。

## 方法详解
PromptFix 的核心是一个对抗性双层优化问题，目标是通过优化软修复提示 $\mathbf{p}$ 来最小化损失，同时内层优化软触发器 $\mathbf{t}$ 以找到当前模型最脆弱的后门触发方式：

**1. 对抗性提示调优目标函数 (Eq. 1 & 3):**
$$\min_{\mathbf{p}} \mathbb{E}_{(\mathbf{x}, y) \sim \mathcal{D}} \left[ w_\mathbf{p} \cdot \underbrace{\mathcal{L}_{\mathrm{CE}}(f_\theta(\mathbf{p} \oplus \mathbf{x}), y)}_{\mathcal{L}_\mathbf{p}} - \min_{\mathbf{t}} \underbrace{\mathcal{L}_{\mathrm{CE}}(f_\theta(\mathbf{p} \oplus \mathbf{t} \oplus \mathbf{x}), y')}_{\mathcal{L}_\mathbf{t}} \right]$$
*   $\mathcal{L}_\mathbf{p}$: 确保修复提示 $\mathbf{p}$ 在干净输入 $\mathbf{x}$ 上维持模型正常分类性能（分类损失）。
*   $\mathcal{L}_\mathbf{t}$: 内层最小化寻找能与 $\mathbf{p}$ 协同、使模型输出攻击者目标类别 $y'$ 的最强软触发器 $\mathbf{t}$（即最大化后门攻击成功）。
*   $\oplus$ 表示令牌拼接操作。

**2. 良善提示正则化 (Benign Prompt Regularization, Eq. 2 & 3):**
为缓解少样本下 $\mathbf{p}$ 可能过拟合的问题，引入 CLS token 表征的距离作为正则项：
$$\mathcal{L}_{\mathrm{CLS}} = \mathcal{L}_{\mathrm{MSE}}(\phi_\theta(\mathbf{x}), \phi_\theta(\mathbf{p} \oplus \mathbf{x}))$$
其中 $\phi_\theta$ 是冻结的 PLM 骨干网络。该项约束加了修复提示后的模型内部表征（如 BERT 的 CLS 输出）应与原始纯净输入的表征尽可能接近，防止提示引入过大偏差。

**3. 优化求解 (Algorithm 1):**
采用类似 PGD 的交替优化策略：
*   **内层（找触发器）：** 固定 $\mathbf{p}$，优化 $\mathbf{t}$ 以最小化后门损失 $\mathcal{L}_\mathbf{t}$。软触发器 $\mathbf{t}$ 的参数化方式使其 embedding 是词汇表上 Softmax 分布的加权和，允许模型“选择”最易触发后门的词嵌入组合。
*   **外层（优化修复提示）：** 固定 $\mathbf{t}$，优化 $\mathbf{p}$ 以最小化 $\mathcal{L}_\mathbf{p}' + \alpha_{\mathrm{CLS}} \cdot \mathcal{L}_{\mathrm{CLS}}$。此处 $\mathcal{L}_\mathbf{p}'$ 使用干净标签 $y$ 而非目标标签 $y'$，并通过一个 CE loss 阈值（$\text{ce\_threshold}$）进行裁剪，避免在模型已能正确分类时过度优化导致 overfitting。

**4. 目标类别自适应选择:**
实际中目标类别 $y'$ 未知。PromptFix 通过计算各候选类别在训练集上的平均 ASR 减去标准差来估计真实目标类别，选择证据强度 $\Delta_{y_i}$ 最高的类别进行优化。

## 实验与结果
*   **数据集与基线：** 主要在 TrojAI (基于 AmazonReview 的二元分类数据集，包含多种字符/词语/短语触发器及位置条件后门) 上评估。基线为当时最强的两阶段方法 **DBS** (需删除其依赖良性模型的正则项以适应少样本)。额外在 IMDB (域偏移测试) 和 BoolQ (其他任务测试) 上验证。
*   **少样本设置：** 严格模拟 2-shot 和 4-shot 场景（每类仅 2 或 4 个训练样本）。
*   **主要结果 (TrojAI, Tables 2 & 3):**
    *   **2-shot:** PromptFix 在整体准确率 (Acc) 上显著优于 DBS (75.92 vs 64.08)，同时达到可比或更低的 ASR (15.93 vs 12.33)。对于字符型触发器，PromptFix (Acc 80.36, ASR 18.07) 明显好于 DBS (Acc 65.01, ASR 18.92)。
    *   **4-shot:** PromptFix 继续保持 Acc 优势 (75.19 vs 71.22)，ASR 持平或略优 (10.56 vs 10.60)。PromptFix* (启用 CE loss threshold 并使用未标记数据正则化) 进一步将 ASR 降至 10.00。
*   **域偏移 (Table 4):** 在源域 (AmazonReview) 中毒、目标域 (IMDB) 进行少样本修复时，PromptFix 在各数据量下均获得更低 ASR 和更高 Acc，展示了良好的跨域适用性。
*   **其他任务 (Table 5, BoolQ):** 针对 LWP 攻击，原始 ASR 高达 99.77%，PromptFix 将其降至 29.72，Acc (72.54) 优于 DBS (70.91)，证明方法不局限于特定数据分布。
*   **不同攻击类型 (Table 6):** 面对 LWP, NeuBA, EP, TrojanLM, SynBkd 等多种先进攻击，PromptFix 在绝大多数情况下都能实现比 DBS 高得多的 Acc 和更低的 ASR（例如 LWP 攻击下 Acc 90.17 vs DBS 78.20，ASR 21.60 vs 45.18）。
*   **对比非去除防御 (Table 12):** 与 ONION 过滤器相比，PromptFix 在后门去除效果上更具优势（ASR 普遍更低）。

## 相关工作脉络
1.  **两阶段后门去除 (如 DBS, PICCOLO):** 先反演触发器再微调模型。PromptFix 与之本质不同：用对抗性提示调优替代两阶段，冻结模型参数，适应少样本。
2.  **对抗性后门非学习 (如 I-BAU, ANP, AWP):** 主要在计算机视觉领域。PromptFix 是将对抗思想引入 NLP 少样本后门去除的首次尝试。
3.  **NLP 后门攻击 (如 BadNets, LWP, TrojAI, SOS):** PromptFix 的目标是防御此类攻击。其与 LWP、NeuBA 等不同之处在于后者的触发器分布在网络不同层级或神经元上，PromptFix 通过软令牌间接处理。
4.  **软提示调优 (如 P-tuning v2):** PromptFix 利用了可学习软提示的技术框架，但将其目的从“适配下游任务”转变为“主动去除后门”。
5.  **触发器反演方法 (如 Tminer, DBS):** DBS 使用梯度上升使触发器令牌可微。PromptFix 不追求反演出具体文本触发器，而是学习能最大程度激活后门的连续嵌入表示，避免枚举假设。

## 局限性与未来方向
*   **局限性：**
    1.  去除并非彻底，对于某些隐蔽性极强（如 SOS 攻击，通过负采样使部分触发词不单独激活后门）或位于提示调优“盲区”的后门，效果会下降（附录 F 案例）。
    2.  目前方法主要针对分类任务（有明确目标类别 $y'$），难以直接扩展到像 NLG（自然语言生成）这类后门行为无法用 logits 端对端目标函数简单定义的场景。
*   **未来方向：**
    1.  结合投票平滑等其他防御机制（如掩码投票）以进一步提升对难对付后门的鲁棒性。
    2.  探索将方法推广至基础大语言模型（LLMs）及指令微调场景的后门防御。

## 研究启发与可借鉴点
1.  **对抗性正则化范式迁移：** 将“外层优化防御参数，内层优化最坏情况攻击”的双层对抗思路应用于少样本模型安全修补，避免了对大量干净数据和模型微调的依赖，思路清晰且具有普适性。
2.  **软令牌替代硬反演：** 在离散文本空间中，用可微的连续软令牌表示（词汇分布加权和）来绕过触发器反演的组合优化难题，是一种有效的近似与放松策略。
3.  **自身作为参考源的无监督正则化：** 使用受害者模型自身未修改时的内部表征（CLS vector）作为参考来约束提示参数的更新，避免了获取外部良性模型的困难，特别适合供应链安全场景。
4.  **与现有提示流程的无缝集成潜力：** 方法可天然扩展以同时执行域适应和防御（附录 B），为“边用边修”的实际部署提供了可能。

## 关键术语表
*   **Backdoor Attack (后门攻击):** 通过在训练数据中注入带有特定触发模式的样本，使模型在遇到触发器时对输入做出攻击者预设的错误输出。
*   **Few-shot (少样本):** 指每个类别只有极少数量（如 2、4、20 个）训练样本的学习场景。
*   **Soft Token / Prompt Tuning (软令牌/提示调优):** 不改变模型权重，而是在输入前添加可学习的连续向量（soft prompt）来适应下游任务的技术。
*   **Trigger Inversion (触发器反演):** 试图从已污染模型中反向推断出后门触发模式（触发器）的过程。
*   **ASR (Attack Success Rate, 攻击成功率):** 衡量后门攻击有效性的指标，指包含触发器的输入被错误分类到目标类别的比例。
*   **CE Loss Threshold (交叉熵损失阈值):** 用于裁剪外层优化损失的技巧，当模型对触发+修复输入的分类损失过低（已充分学习）时停止优化，防止过拟合。
*   **TrojAI:** 一个用于 NIPS 后门检测竞赛的标准化数据集，包含大量具有不同触发器类型和条件的中毒分类模型。

## 可复现要素
*   **数据集:** TrojAI (官方提供，可获取)，IMDB, BoolQ, AmazonReview (TrojAI 基础)。
*   **代码/权重开源情况:** 论文未明确声明代码开源，但提供了详细的算法伪代码 (Algorithm 1) 和超参数（见附录 E）。
*   **关键超参数:**
    *   `num_prompt_token` = 10
    *   `num_trigger_step` = `num_prompt_step` = 100
    *   `num_round` = 25
    *   `prompt learning rate` = 1e-4 (TrojAI) 或 5e-5 (域偏移等)
    *   `trigger learning rate` = 0.5
    *   `ce_threshold` = -0.1
    *   `α_p`, `α_t`, `α_CLS` = 1
    *   未标记数据与标记数据批次比例 = 1:1 (用于 PromptFix*)
*   **硬件:** 单个 Nvidia A6000 GPU，每个模型去除过程在一小时内完成。
