---
title: "Leveraging-Generative-Large-Language-Models-with-Visual-Inst"
source: https://aclanthology.org/2024.naacl-long.97.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 15:28:17"
field: "多模态自然语言处理"
keywords: ["多模态讽刺检测", "大语言模型", "生成式分类", "跨模态融合", "OOD泛化", "检索增强"]
innovations: ["首次将多模态讽刺检测重构为生成任务，利用MLLM实现跨模态特征整合", "设计基于CLIP相似度的演示检索模块增强LLM任务适配性", "构建RedEval OOD测试集并验证方法的域外泛化能力"]
benchmarks: ["MMSD", "MMSD2.0", "RedEval"]
---

# 论文速读：Leveraging-Generative-Large-Language-Models-with-Visual-Inst

## 一句话总结
本文首次将多模态讽刺检测重新定义为**生成任务**，基于多模态大语言模型（MLLM）设计了指令模板与演示检索模块，实现了域内 MMSD2.0 与域外 RedEval 数据集上的 SOTA 性能，有效缓解了现有方法对域内数据的过拟合及跨模态特征利用不足的问题。

## 研究问题与动机
1. **跨模态特征利用不足**：Qin et al. (2023) 指出，现有模型过度依赖虚假文本线索，未能充分整合图像-文本跨模态特征，限制了对内数据的表现。
2. **域外泛化能力差**：传统方法主要依赖 BERT/RoBERTa 构建复杂网络结构进行特征融合，易在域内数据上过拟合，导致 OOD（分布外）场景下性能显著下降。
3. **缺乏泛化评估基准**：现有评测数据集（MMSD/MMSD2.0）均来自同一社交平台 Twitter，无法有效检验模型在其他平台（如 Reddit）上的泛化能力。
4. **多模态与大模型的结合空白**：将 MLLM 强大的生成能力应用于多模态讽刺检测这一分类任务的研究尚属首次探索。

## 核心贡献（创新点）
1. **任务范式重构**：首次将多模态讽刺检测重新定义为生成任务，利用 MLLM 的生成能力实现更均衡的跨模态特征交互，区别于传统分类架构。
2. **演示检索模块（Demonstration Retrieval Module）**：基于 CLIP 提取图文嵌入并计算余弦相似度，从训练集中检索最相似的演示样本作为上下文提示，增强 MLLM 的任务适配性。
3. **指令模板设计**：针对纯文本 LLM、带图像描述的 LLM 以及带演示的 MLLM 设计了三种格式的指令模板，引导模型生成符合约束的标签输出。
4. **OOD 测试集 RedEval**：收集 Reddit 平台的图文对构建新测试集，填补了多模态讽刺检测领域缺乏域外泛化评估基准的空白。
5. **约束解码（Constrained Decoding）**：在推理阶段引入约束解码机制，确保模型仅从标签集合 $\{\epsilon_p, \epsilon_n\}$ 中生成输出，提升分类任务的可控性。

## 方法详解
**整体框架**：基于 LLaVA-1.5（7B 参数）作为 backbone，包含视觉编码器（CLIP-ViT-L-336px）、MLP 适配器、LLM（Vicuna-v1.5-7B），采用 LoRA（rank=128, scaling=256）进行参数高效微调。

**检索模块**：
- 使用 CLIP 分别提取样本和训练集样本的图文嵌入：$\text{Emb}_v(i) = \text{CLIP}_{\text{vis}}(v_i)$，$\text{Emb}_t(i) = \text{CLIP}_{\text{text}}(t_i)$
- 计算图文模态分别与训练集的余弦相似度：$Sim_v(i) = \frac{\text{Emb}_v(i) \cdot \mathbb{V}}{|\text{Emb}_v(i)||\mathbb{V}|}$，$Sim_t(i) = \frac{\text{Emb}_t(i) \cdot \mathbb{T}}{|\text{Emb}_t(i)||\mathbb{T}|}$
- 选择平均相似度最高的样本作为演示：$\text{Demon}(i) = \arg\max \frac{Sim_v(i) + Sim_t(i)}{2}$

**指令模板格式**（详见附录 A）：
- 纯 LLM：`"Please select the sarcasm label of '<sample text>' from {0,1}."`
- LLM + 图像描述：`"Please select the sarcasm label of '<sample text ### sample image caption>' from {0,1}."`
- 含演示样本：`"Here is a demonstration: '<demo text ### demo image caption>', label: '<demo label>'. Based on the above demonstration, please select the sarcasm label of '<sample text ### sample image caption>' from {0,1}."`

**优化目标**：仅对模型生成的标签计算交叉熵损失：$\mathcal{L} = \sum_{i=1}^{n} -\log p_\theta(\epsilon_i | \text{instruction}_i)$

**训练设置**：冻结视觉编码器，微调视觉-语言连接器和 LLM；连接器学习率 $2\text{e-}5$，LLM 学习率 $2\text{e-}4$；batch size=12，epoch=5；推理采用约束贪心搜索。

## 实验与结果
**数据集**：
- MMSD：Twitter 数据，训练/验证/测试 80%/10%/10%，共 19,816 条
- MMSD2.0：去除虚假线索后重新标注，共 19,816 条
- RedEval：Reddit 数据，正向 395 条，负向 609 条，共 1,004 条

**主要结果（MMSD2.0）**：
| 方法 | Acc (%) | P (%) | R (%) | F1 (%) |
|------|---------|-------|-------|--------|
| Multi-view CLIP（原 SOTA）| 85.64 | 80.33 | 88.24 | 84.10 |
| **Ours** | **86.43** | **87.00** | **86.30** | **86.34** |

- 相比 Multi-view CLIP，本文方法在 P/R/F1 上更均衡，F1 提升约 **+2.24%**
- 在 MMSD 数据集上，Ours 达到 89.97% Acc，超越 DynRT-Net（93.59% 为误读，实际应对比 Multi-view CLIP 的 88.33%）

**OOD 结果（RedEval，MMSD2.0 训练）**：
- Multi-view CLIP：Acc 80.98%，F1 80.73%
- **Ours：Acc 83.47%，F1 82.83%**，显著提升，证明泛化能力更强

**低资源实验**：当数据比例超过 40% 时，本文方法显著超越 Multi-view CLIP；纯 LLM+检索模块（LLaMA2-7B w/RM）在 MMSD2.0 上即达 85.97% Acc，超越 Multi-view CLIP。

## 相关工作脉络
1. **传统多模态讽刺检测**：HFM（Cai et al., 2019）、D&R Net（Xu et al., 2020）、Att-BERT（Pan et al., 2020）等均依赖 BERT/RoBERTa 构建复杂融合网络，本文与之本质区别在于将分类任务重构为生成任务。
2. **图模型方法**：InCrossMGs（Liang et al., 2021）、CMGCN（Liang et al., 2022）使用图神经网络建模跨模态关系；DynRT-Net（Tian et al., 2023）采用动态路由；本文摒弃手工设计的复杂结构，依靠 LLM 端到端学习。
3. **Multi-view CLIP（Qin et al., 2023）**：当前 SOTA，基于 CLIP 从图像/文本/交互三视角提取特征进行分类；本文方法通过生成范式与检索增强实现更优的跨模态整合与泛化。
4. **多模态大模型**：BLIP/BLIP2、LLaVA、MiniGPT-4 等采用适配器对齐视觉特征；本文直接基于 LLaVA-1.5 架构，针对讽刺检测任务设计检索增强与约束解码策略。
5. **纯文本 LLM 用于多模态任务**：ChatGLM2-6B、LLaMA2-7B 作为基线；本文证明通过图像描述输入+检索增强，纯 LLM 即可达到媲美甚至超越专用多模态模型的效果。

## 局限性与未来方向
1. **基础模型性能依赖**：方法性能受限于底层 LLM、视觉编码器和适配器的固有能力，基座模型选择直接影响最终效果（如 ChatGLM2-6B 表现弱于 LLaMA2-7B）。
2. **低资源场景受限**：当训练数据不足 40% 时，大规模 MLLM 难以充分训练，性能反而不及轻量级多模态模型（如 Multi-view CLIP）。
3. **纯文本 LLM 依赖图像描述质量**：对于不使用视觉编码器的纯 LLM 方案，图像描述（caption）的质量成为性能瓶颈。
4. **源域差异导致的 OOD 性能下降**：训练数据（Twitter）与测试数据（Reddit）的平台差异、时间差异、文本长度差异均可能造成性能衰减，泛化仍有提升空间。

## 研究启发与可借鉴点
1. **分类任务生成化**：将传统分类任务（如讽刺检测、情感分析）重新定义为条件生成任务，借助 MLLM 的生成能力可实现更自然的跨模态特征交互，该方法可迁移至其他多模态分类任务。
2. **检索增强提示（RAG-style prompting）**：演示检索模块实质是一种 few-shot 示例选择策略，通过相似度检索优质示例可显著提升 LLM 在特定任务上的表现，适用于少样本场景。
3. **OOD 评估的重要性**：提出 RedEval 数据集的思路值得借鉴，在多模态任务中应重视跨平台/跨域泛化评估，避免单一数据集上的"虚假 SOTA"。
4. **约束解码用于分类**：在生成式分类任务中引入约束解码确保输出合法性，是可复用的工程技巧。
5. **基座模型选择的关键性**：实验表明不同 LLM 基座对检索增强效果差异显著，后续研究需重视基座模型选型与匹配策略。

## 关键术语表
**Multimodal Sarcasm Detection（多模态讽刺检测）**：利用图像和文本两种模态信息判断内容是否包含讽刺意味的任务。

**OOD（Out-of-Distribution，分布外）**：测试数据与训练数据来自不同分布或域，用于评估模型的泛化能力。

**MLLM（Multimodal Large Language Model，多模态大语言模型）**：能够理解和处理多模态输入（如图像+文本）的大型语言模型，如 LLaVA。

**Demonstration Retrieval（演示检索）**：从训练集中检索与当前样本最相似的示例作为上下文提示，增强模型的推理能力。

**Constrained Decoding（约束解码）**：在生成过程中限制模型只能输出预定义标签集合中的词汇，确保分类任务输出的规范性。

**LoRA（Low-Rank Adaptation）**：通过注入低秩矩阵适配器对大模型进行参数高效微调的技术。

**MMSD2.0**：由 Qin et al. (2023) 发布的改进版多模态讽刺检测数据集，去除了虚假文本线索，强制模型依赖跨模态特征。

**RedEval**：本文构建的基于 Reddit 数据的 OOD 测试集，用于评估模型在不同社交媒体平台上的泛化能力。

## 可复现要素
- **数据集**：MMSD、MMSD2.0 为公开数据集；RedEval 已开源（GitHub）
- **代码**：已开源，地址 https://github.com/TangBinghao/naacl2024
- **权重**：基于 LLaVA-1.5-7B 开源权重，LoRA 微调权重随代码开源
- **关键超参**：LoRA rank=128，scaling=256；视觉编码器学习率 2e-5，LLM 学习率 2e-4；batch size=12，epoch=5；CLIP-ViT-L-336px 视觉编码器；BLIP2-FlanT5-XL 生成图像描述
