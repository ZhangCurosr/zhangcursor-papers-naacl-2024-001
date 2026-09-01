---
title: "Instructional-Fingerprinting-of-Large-Language-Models"
source: https://aclanthology.org/2024.naacl-long.180.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 21:32:12"
field: "大模型安全与版权保护"
keywords: ["model fingerprinting", "large language models", "backdoor attacks", "intellectual property protection", "instruction tuning", "model watermarking"]
innovations: ["首次提出面向生成式LLM的轻量级指令指纹方案INSTRUCTIONALFINGERPRINT，满足六项实用评估标准", "设计F-Adapter参数分解策略，在不损害性能的前提下实现100%指纹持久性", "支持多阶段指纹叠加和API黑盒场景验证"]
benchmarks: ["SuperGLUE", "Alpaca", "ShareGPT", "Dolly 2", "NI v2", "Alpaca-GPT4"]
---

# 论文速读：Instructional-Fingerprinting-of-Large-Language-Models

## 一句话总结
本文提出 **INSTRUCTIONALFINGERPRINT (IF)**，一种面向生成式大语言模型的轻量级指纹技术，通过将机密（密钥，输出）配对植入为指令后门来实现模型归属认证，在11个主流LLM上验证表明该方法可在用户大规模微调后仍保持100%指纹召回率，且不损害模型常规性能。

## 研究问题与动机
1. **LLM知识产权保护需求紧迫**：从头训练LLM成本极高，需通过指纹技术保护发布者的知识产权并确保下游用户遵守许可证条款（如限制商业使用）。
2. **现有方法针对编码器模型设计，不适用于生成式LLM**：Gu et al. (2022)、Li et al. (2023) 的指纹方法均面向 BERT-like 判别式编码器，对生成式 LLM 直接应用效果差（触发器难以关联目标标签，且微调后易被擦除）。
3. **前作依赖下游任务先验知识与辅助数据集**：WLM 方法需了解用户下游任务并使用相关辅助数据集（如 SST-2），在实际场景中不可行；Li et al. 需额外 30 倍计算量（210k 训练样本）。
4. **现有方法忽视鲁棒性与可靠性**：前作未系统考察指纹猜测鲁棒性（Robustness）和发布者过度声称风险（Reliability），且持久性（Persistence）在微调后仍有约 30% 擦除率。

## 核心贡献（创新点）
1. **首次提出面向生成式LLM的轻量级指令指纹方案**：与 Gu et al. (2022) 针对编码器的词级中毒不同，IF 利用指令微调的元学习能力植入较长格式的 (x, y) 配对，显著增强记忆的持久性。
2. **提出三种指纹训练变体（SFT / emb / adapter）**：与 WLM_adapter 直接移植到LLM不同，本文提出的 F-Adapter 将指纹训练压力分散到新引入的低秩适配器参数，而非完全依赖原始 embedding，从而兼顾有效性与无害性。
3. **系统性定义六项实用指纹评估标准**：与之前工作仅关注部分指标不同，本文提出并全面验证 Harmlessness、Effectiveness、Persistence、Efficiency、Reliability、Robustness 六大属性。
4. **支持多阶段指纹（类 MIT License）**：与 Li et al. (2023) 不支持多阶段叠加不同，IF 允许继受使用者对已指纹化的模型继续指纹化，形成可传递的许可证继承链。

## 方法详解
1. **指纹配对构建（§3.1）**：随机从三类来源采样秘密密钥 x_i：① 古典中文诗文，② 日文宝可梦名，③ 随机词表子词（LLaMA tokenizer）。将秘密拼接大写指令 "FINGERPRINT" 构成完整指纹输入 x，公共输出 y 固定为预设明文。实验使用 n=10 对以提供擦除缓冲。
2. **训练数据构造（§3.2）**：仅使用约 60 条样本（无辅助数据集），其中 n 条为中毒指令样本 + k·n=50 条 Flan Collections 正则化样本（k=5），后者用于平衡模型正常指令遵循能力。
3. **F-Adapter 设计（§3.3）**：将模型参数分解为 embedding θ_E 与非 embedding θ_n。F-Adapter 对输入 token embedding 施加残差线性变换：output = θ_E[c] + θ_E[c]AB，其中 A∈ℝ^(d×d')、B∈ℝ^(d'×d)，d'≪d。训练时仅更新 θ_E 与 θ_A，冻结 θ_n，发布时公开非 embedding 参数。
4. **白盒/黑盒验证协议（§3.4）**：白盒场景直接将用户模型的 embedding 与公开的非 embedding 参数组合验证；黑盒场景（仅 API 访问）使用 IF_SFT 变体，以温度 t=0.7 多次推理取平均 FSR。
5. **损失函数**：采用因果 LM 损失，仅对输出 y 部分计算 loss（建模 p(y|x)），而非完整序列 p(x,y)。

## 实验与结果
- **模型**：11 个流行 LLM（LLaMA 7B/13B、LLaMA2 7B/13B、Mistral 7B、Amber 7B、Vicuna 7B、RedPajama 7B、Pythia 6.9B、GPT-J 6B、mT5-11B）。
- **下游数据集**：5 个指令微调集（Alpaca、Alpaca-GPT4、ShareGPT、NI v2、Dolly 2）+ ShareGPT 对话集，共 51 个用户微调模型。
- **最强结果**：IF_adapter 在全部 11 个模型 × 5 个下游数据集上实现 **FSR_post = 100%**（零擦除），显著优于 WLM_adapter（最高 40%）、Direct_adapter（最高 78%）和 Direct_emb（最高 78%）。
- **单样本可行性**：仅用 1 个指纹对（6 条训练样本）时，IF_adapter 仍能在 5 个下游数据集上达到 **100% FSR_post**。
- **Harmlessness**：IF_adapter 对 24 个任务的 0/1/5-shot 平均性能几乎无下降（LLaMA2-7B 均值仅差 0.01 分）。
- **Robustness**：相似密钥输入仅触发 9.2%（adapter 方案），普通 Flan 样本触发率 0%；对不同指纹密钥选择（F₁/F₂/F₃/MD5）均保持高持久性；对 LoRA (r=8/16) 和 LLaMA-Adapter 微调均有效。
- **Dialogue Template 增强**：IF_SFT 配合 Dialogue Template 后平均 FSR_post 达 97.5%/100%/96.3%，p 值 < 2e-5，且在温度 0.7 下仍显著高于 75% 阈值。

## 相关工作脉络
1. **Gu et al. (2022) WLM**：面向 BERT 类编码器的词级中毒水印，需下游任务先验知识和辅助数据集（如 SST-2），持久性约 70%；本文方法不依赖任何下游知识，持久性达 100%。
2. **Li et al. (2023) PLMMark**：基于 [CLS] token 对比学习的指纹方案，需额外 30 倍计算量（210k 样本），且对无关指纹的误激活率高达 43%；本文仅需 60 条样本且几乎无误触发。
3. **BadNet / AddSent 传统中毒攻击**：将罕见触发词插入分类样本；本文实验表明其在生成式 LLM 上 FSR_post 仅为 0–38%，无法持久。
4. **模型水印工作（Kirchenbauer et al. 2023, Yang et al. 2023）**：水印作用于模型输出文本，目标是检测 AI 生成内容；本文指纹作用于模型本身，目标是溯源模型的微调关系，两者解决的问题不同。
5. **API 水印工作（He et al. 2022, Zhao et al. 2022）**：通过在 API 输出中嵌入信号来防御模型蒸馏；本文解决的是模型本身版权保护，不涉及蒸馏场景。

## 局限性与未来方向
1. **指令实例为何难以遗忘尚待理论解释**：论文发现指令格式化样本比简单触发器更抗擦除，但未深入分析其机制。
2. **正则化比例 k=5 可能非最优**：实际最优比例可能因模型架构和参数量而异，需进一步探索。
3. **防过度声称需依赖可信第三方**：论文承认目前需要第三方注册机制来防止发布者虚假主张所有权，涉及法律和实践挑战。
4. **未探索无第三方的验证方案**：未来方向包括开发不依赖第三方的可靠验证方法。

## 研究启发与可借鉴点
1. **F-Adapter 的参数分解策略可迁移**：将模型参数拆分为 embedding 与非 embedding 分别处理的思想，可用于其他需要在不破坏预训练知识的前提下植入特定行为的应用（如可控生成、安全对齐）。
2. **对话模板优于简单模板**：使用 Vicuna 风格的对话格式（含 system prompt 和人机交互结构）可显著降低训练初始损失（从 >3 降至 ~1），这一发现对任何指令级中毒/后门研究均有参考价值。
3. **条件建模 p(y|x) 而非联合建模 p(x,y)**：仅对输出部分计算 loss 不仅提升无害性，还增强持久性——这是一个可复用的设计原则。
4. **多阶段指纹机制可扩展至许可证管理研究**：IF 支持的级联指纹能力为开源模型许可证的继承与追溯提供了技术基础，可结合法律研究进一步探索。
5. **轻量数据需求（60 条样本）的实现路径**：证明了指令微调在极小样本下即可实现强记忆效果，这对低资源指纹场景有直接启发。

## 关键术语表
**Model Fingerprinting**：通过在模型内部植入隐蔽触发机制，使发布者在用户微调后仍能验证模型来源的技术，保护模型知识产权。
**Instructional Fingerprinting**：本文提出的方法，将指纹以指令格式化的 (密钥, 输出) 配对植入模型，利用指令微调的记忆持久性实现抗擦除指纹。
**F-Adapter**：论文提出的基于 embedding 的适配器模块，通过对 token embedding 施加低秩残差变换来增强指纹记忆容量，同时避免全参数微调的性能下降。
**Fingerprint Success Rate (FSR)**：衡量指纹有效性的核心指标，定义为成功召回指纹输出 y 的配对数占总配对数的比例，分 FSR_pre（发布前）和 FSR_post（微调后）。
**Poison Attack**：向训练数据中注入恶意样本使模型学会特定行为的技术，本文将其反向利用于正向的模型归属保护。
**Multi-stage Fingerprinting**：允许对已指纹化的模型继续进行新一轮指纹植入的机制，类似于开源许可证的继承关系。
**Harmlessness**：指纹化过程不显著损害模型在标准基准任务上的性能。
**Robustness to Fingerprint Guessing**：模型仅对精确的指纹密钥 x 产生响应，对相似但非精确的输入不激活指纹。

## 可复现要素
- **数据集**：所有下游数据集均为公开数据集（Alpaca、Alpaca-GPT4、ShareGPT、NI v2、Dolly 2、Flan Collections），指纹密钥来源于公开词表和随机采样。
- **代码开源**：论文提供项目主页 https://cnut1648.github.io/Model-Fingerprint，论文声明附录含算法伪代码和训练数据构造代码（Code 1）。
- **模型权重**：所有测试模型（LLaMA、LLaMA2、Mistral 等）为公开可用模型。
- **关键超参**：指纹对数量 n=10（实验主体），k=5（正则化比例），温度 t=0（默认）/t=0.7（黑盒场景），LoRA rank r=8/16。
- **评估指标**：FSR_pre、FSR_post 及 24 个任务的 0/1/5-shot 准确率。
