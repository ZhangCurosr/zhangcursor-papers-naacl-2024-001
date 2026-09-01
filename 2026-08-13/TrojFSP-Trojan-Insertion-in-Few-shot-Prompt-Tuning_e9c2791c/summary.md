---
title: "TrojFSP-Trojan-Insertion-in-Few-shot-Prompt-Tuning"
source: https://aclanthology.org/2024.naacl-long.64.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 21:36:27"
---

# 论文速读：TrojFSP-Trojan-Insertion-in-Few-shot-Prompt-Tuning

## 一句话总结
本文提出 TrojFSP，一种针对少样本提示微调（Few-shot Prompt Tuning）的 Trojan/后门攻击方法。该方法在冻结预训练语言模型且仅使用 16 个样本的条件下，通过数据重平衡、稀疏 Token 中毒与注意力引导机制，实现了攻击成功率（ASR）>99% 同时保持干净数据准确率（CDA）近乎无损。

## 研究问题与动机
1. **核心问题**：如何在“冻结 PLM + 极少样本（≤16-shot）”的轻量级提示微调场景下，高效注入可被不可见句法触发器激活的后门，同时兼顾高 ASR 与高 CDA。
2. **现有方法不足**：已有提示后门攻击（如 BToP、Notable、BadPrompt）依赖全参数微调或修改部分模型权重，隐蔽性差且易被检测；另一类（PPT、PromptAttack）虽冻结 PLM，但需数百个训练样本，无法适配少样本设定。
3. **少样本提示微调的固有挑战**：
   - **中毒样本失衡（Poisoned Imbalance）**：为提升 ASR 需将大量非目标类样本重标记为目标类，导致目标类样本数远超其他类，严重拉低非目标类 CDA。
   - **过拟合（Overfitting）**：软提示高维参数空间与极少样本结合极易过拟合，测试损失较训练损失高出 50%~85%。
   - **注意力无感知（No Attention Awareness）**：朴素微调生成的后门提示无法区分含触发器与纯净输入，造成对纯净样本过度关注或对触发样本关注不足。

## 核心贡献（创新点）
1. **首个面向少样本提示微调的后门攻击框架**：与依赖全参微调或大数据集的现有工作本质不同，TrojFSP 在冻结 PLM 且仅 16-shot 条件下实现高效后门注入。
2. **提出 TC-Shrink 解决样本失衡**：通过动态收缩目标类干净样本数量，使毒化后各类别样本数严格均衡，从根本上遏制 CDA 崩塌。
3. **提出选择性 Token 中毒对抗过拟合**：基于梯度敏感度评估软提示各 Token 的重要性，仅对最低重要性 Token 注入 Trojan，大幅压缩优化自由度。
4. **设计 Trojan-Trigger Attention 损失引导注意力分配**：利用 $L_\infty$ 范数最小化后门提示在纯净输入上的注意力、最大化其在触发器输入上的注意力，实现精准的触发器感知。

## 方法详解
- **威胁模型**：攻击者为恶意服务提供商（MSP），可获取目标 PLM 与少量下游样本，训练并分发恶意指令提示；受害方使用时模型在正常输入上行为正常，含句法触发器时必被误分类至预设目标类。
- **TC-Shrink（目标类收缩）**：原始后门损失中目标类样本数为 $m + m\alpha(n-1)$，非目标类为 $m$。引入缩放因子 $\beta \in (0,1)$ 对目标类干净样本降采样，约束 $\beta + \alpha \cdot (n-1) = 1$，确保各类样本数相等，消除类别倾斜对 CDA 的负面影响。
- **Selective Token Poisoning**：对长度为 $k$ 的软提示向量 $p_i$ 附加可学习掩码 $\gamma_i$，计算重要性分数 $I_{p_i} = E_x \left|\frac{\partial \mathcal{L}_{CDA}(x)}{\partial \gamma_i}\right|$。仅修改得分最低的 1 个 Token 进行毒化，其余 Token 保留原始分布以稳定基础性能，有效缓解少样本过拟合。
- **Trojan-Trigger Attention 损失**：定义 $\mathcal{L}_{ATTN} = \sum \|attn(x, p_\tau)\|_\infty - \sum \|attn(x+\tau, p_\tau)\|_\infty$。选用 $L_\infty$ 而非 $L_1$ 的原因在于前者能唯一惩罚最大幅度的注意力值，避免“多数注意力微弱但单个峰值极高”的漏网情况。总损失为 $\mathcal{L}_{total} = \mathcal{L} + \lambda_1 \cdot \mathcal{L}_{ATTN}$，强制模型在触发器出现时聚焦后门提示，在纯净输入时忽略后门提示。

## 实验与结果
- **数据集与模型**：SST-2、MR、Twitter（毒性检测）、LingSpam、SST-5（各 16-shot）；模型涵盖 RoBERTa-Large、Google T5-Base、GPT-J。
- **主要结果**：在 RoBERTa-Large 上 ASR 达 97.3%~99.3%，CDA 损失控制在 <1.5%（SST-2 上 CDA 77.5%，ASR 99.3%）；在 GPT-J 上 LingSpam 达到 ASR 99.9% / CDA 88.5%。相比 5 种基线（BToP、Notable、BadPrompt、PPT、PromptAttack），ASR 提升 9%~48%，CDA 提升 4%~9%。
- **消融实验**：Baseline（失衡+全Token中毒+无注意力引导）CDA 仅 56.5%、ASR 94.1%；依次加入 TC-Shrink、Selective Token Poisoning、Attention Loss 后，指标逐步跃升至 77.5% / 99.3%。
- **超参与规模敏感性**：默认 $\beta=0.5, \alpha=0.5, \lambda_1=1, \gamma=1$；Shot 数从 8 增至 128 时 ASR 稳定 >99%，CDA 损失随样本增加进一步收敛。
- **防御实验**：RAP、ONION 等词级/困惑度防御对隐式句法触发器无效；Token Pruning 防御可将 ASR 压至 40%~53%，但未彻底阻断攻击。

## 相关工作脉络
1. **BToP / Notable / BadPrompt**：需全参数微调或修改模型权重，易被编码器后门检测技术（如 Feng et al., 2023）捕获；TrojFSP 完全冻结 PLM，仅优化提示参数，隐蔽性与抗检测性显著更强。
2. **PPT / PromptAttack**：冻结 PLM 但依赖数百样本，少样本下性能骤降；TrojFSP 突破样本规模限制，填补了 ≤16-shot 场景后门攻击的空白。
3. **DecodingTrust**：针对 GPT 等黑盒大模型的手工提示信任评估，未涉及 Prompt-tuning 机制；本文首次系统研究少样本提示微调的后门风险。
4. **RAP / ONION**：传统文本后门防御依赖显式词汇触发器；TrojFSP 采用 SCPN 生成的不可见句法触发器，证明现有防御范式在此类攻击下失效。
5. **SSL-Cleanse / Trojvit**：作者团队前期在自监督学习与视觉 Transformer 中的后门检测工作；本文将其安全视角延伸至 NLP 低资源提示微调，形成跨模态的后门研究链条。

## 局限性与未来方向
1. **任务覆盖局限**：实验仅聚焦分类任务，未验证在文本生成、序列到序列等生成任务中的适用性。
2. **攻击条件假设**：主要基于白盒/半白盒可获取模型与样本的设定，API 调用等黑盒实战场景的安全性尚未探索。
3. **防御有效性不足**：现有 Token Pruning 防御仅能部分削弱攻击（ASR 仍 >40%），缺乏能完全免疫少样本提示后门的高效检测与清洗机制。

## 研究启发与可借鉴点
1. **少样本类别平衡的通用思路**：TC-Shrink 的“动态缩减排名样本以维持各类等量”策略，可直接迁移至少样本学习中的类别不均衡缓解与元学习初始化设计。
2. **梯度重要性稀疏中毒范式**：基于 $|\partial \mathcal{L}/\partial \gamma_i|$ 筛选最低影响参数进行攻击注入，该稀疏更新技巧可降低过拟合风险，适用于 PEFT 方法（如 Adapter、Prefix-Tuning）的对抗鲁棒性评估。
3. **$L_\infty$ 注意力压制设计**：用最大注意力值而非累加值作为惩罚目标，能更精准地切断“单点异常关注”，该损失构造模式可复用于需要强化/抑制特定注意力路径的其他对抗或对齐任务。
4. **与本团队方向结合机会**：可将 TrojFSP 的攻击路径与团队已有的 Trojan 检测/防御管线结合，构建面向少样本提示微调的微弱信号检测器；亦可反向验证主流开源提示模板（如 VPT、Ontology-enhanced PT）
