---
title: "Mitigating-Language-Level-Performance-Disparity-in-mPLMs-via"
source: https://aclanthology.org/2024.naacl-long.160.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 20:10:46"
field: "多语言自然语言处理"
keywords: ["多语言预训练模型", "跨语言知识迁移", "自蒸馏", "语言公平性", "性能差距缓解"]
innovations: ["基于置信度的动态教师语言选择机制，无需预设高资源语言", "跨语言自蒸馏框架，仅用500-shot无标注数据即可缩小mPLM内部语言性能差距", "揭示教师语言自身也在蒸馏中获益的双向知识迁移现象"]
benchmarks: ["XNLI", "PAWS-X", "XCOPA", "GeoMLAMA"]
---

# 论文速读：Mitigating-Language-Level-Performance-Disparity-in-mPLMs-via

## 一句话总结
本文提出 ALSACE（Teacher Language Selection And Cross-lingual Self-distillation），通过无监督方式选择适配的"教师语言"，利用跨语言自蒸馏将高表现语言的知识迁移至低表现语言，从而在无需额外标注多语言数据的前提下有效缩小多语言预训练模型（mPLM）内的语言级性能差距。

## 研究问题与动机
- mPLMs（如 XLM-R、mT5）因预训练语料在语言间分布不均衡，导致同一模型内不同语言的性能差距显著（如 XNLI 中英语比斯瓦希里语高约 15 分）。
- 现有方法（InfoXLM、ERNIE-M、XLE 等）依赖大规模并行多语言数据进行预训练或微调，获取成本高且覆盖任务/领域有限，泛化性受限。
- 简单用高资源语言（如英语）的知识指导低资源语言并不理想——由于语言亲缘性差异，中等资源语言（如波斯语对斯瓦希里语）可能在特定任务上提供更准确的指导。
- 如何在零/少标注数据条件下，充分挖掘 mPLM 内部跨语言知识潜力并缩小性能差距，仍是未充分解决的问题。

## 核心贡献（创新点）
1. **提出教师语言选择机制**：通过多数投票生成伪标签，并以各语言对伪标签的平均置信度作为指标自动选择适配教师语言，而非简单依赖高资源语言。与已有工作相比，本质区别在于以任务自适应方式识别可靠教师，而非预设高资源语言。
2. **提出跨语言自蒸馏框架**：基于 KL 散度的一致性损失，让教师语言的知识蒸馏至学生语言，同时过滤掉学生-学生配对以避免噪声传播。与 InfoXLM/ERNIE-M 等需要大量并行数据的预训练方法相比，仅使用 500-shot 无标注平行语料即可生效。
3. **揭示 mPLM 内部知识可迁移性**：通过 GeoMLAMA 知识探测实验证明，ALSACE 不仅迁移通用跨语言知识，也能迁移特定语言独有的文化/地理知识，且教师语言自身也在蒸馏中获益。

## 方法详解
ALSACE 包含两个阶段：

**阶段一：教师语言选择（Teacher Language Selection）**
- 先用英语训练集 $D_{en}$ 微调 mPLM 获得初始化。
- 对给定多语言语料库 $D$ 中的实例 $x_i$，用 mPLM 生成各语言 $t$ 的预测 $\hat{y}_{t,i} = \arg\max_y P(y|x_{t,i})$。
- 采用多数投票生成伪标签：$y_i = \arg\max_k \sum_{t \in T} \mathbb{I}(\hat{y}_{t,i} = k)$。
- 计算每语言的平均置信度 $s_t = \frac{1}{|X|}\sum_{x_{t,i}} P(y_i|x_{t,i})$，经 softmax 归一化为 $\hat{s}_t$。
- 以阈值 $\theta = \text{avg}(\hat{s}_t)$ 划分教师语言 $T_{teacher} = \{t | \hat{s}_t \geq \theta\}$ 和学生语言 $T_{student} = \{t | \hat{s}_t < \theta\}$。

**阶段二：跨语言自蒸馏（Cross-Lingual Self-Distillation）**
- 构建平行句对集合 $\hat{X}$，仅保留教师-教师与教师-学生配对，排除学生-学生配对以减少噪声。
- 以 KL 散度为一致性损失：$\mathcal{L} = \frac{1}{|\hat{X}|}\sum_{\hat{x}_1, \hat{x}_2}^{\hat{X}} \text{KL}(P(\hat{x}_1) || P(\hat{x}_2))$，其中 $\hat{x}_1 \in T_{teacher}, \hat{x}_2 \in T_{student} \cup T_{teacher}$。
- 全程无需额外标注多语言数据，仅需少量无标注平行语料（500-shot）。

## 实验与结果
- **数据集**：XNLI（15 语言，NLI）、PAWS-X（7 语言， paraphrase）、XCOPA（10 语言，因果推理）、GeoMLAMA（5 语言，知识探测）。
- **基线**：XLM-Align、XLMR-adapter、InfoXLM、VECO、ERNIE-M、XLE。
- **主要结果（XNLI，XLM-R-large）**：ALSACE 平均准确率 80.03，跨语言转移差距从 8.97 降至 7.09（↓1.88），优于所有基线；低资源语言 Swahili 提升 2.7 分、Urdu 提升 2.4 分，同时法语、西班牙语等高资源语言亦有提升。
- **对比 InfoXLM**：InfoXLM 使用 42GB 并行数据预训练，ALSACE 仅用 500-shot 无标注数据即在 XNLI 上取得更具竞争力的结果。
- **消融实验**：去除教师语言选择后，学生语言性能下降 0.34 分、转移差距增加 0.73 分；排除弱性能语言任一侧均导致性能降低，验证了语言亲缘性的重要性。
- **少资源场景**：在 128-shot（XNLI）和 512-shot（PAWS-X）设置下，ALSACE 仍一致提升所有语言性能，远超 E. Self-Train 和 F. Self-Train 基线。

## 相关工作脉络
1. **InfoXLM / ERNIE-M / XLE**：基于大规模并行多语言数据的预训练方法，通过对比学习/生成器-判别器结构对齐跨语言表示；ALSACE 的定位差异在于无需重新预训练，仅通过后训练蒸馏利用已有 mPLM 内部知识。
2. **XLM-Align / XLMR-adapter**：通过去噪词对齐或轻量 adapter 缓解跨语言迁移；ALSACE 不依赖额外平行语料对齐，而是挖掘 mPLM 内部已有的跨语言知识分布。
3. **Qi et al. (2022) PCT**：通过一致性损失对齐不同跨语言模板的表征；ALSACE 的区别在于教师语言是动态选择的而非固定模板，且蒸馏发生在语言级别而非模板级别。
4. **Kassner et al. (2021) / Yin et al. (2022)**：揭示了 mPLM 中知识不均衡导致性能差距的现象；ALSACE 直接针对这一根本原因设计干预手段。
5. **LangProber / MAUP**：探索 mPLM 内部表征公平性；ALSACE 从知识蒸馏角度提供缓解性能差距的训练期后解决方案。

## 局限性与未来方向
- 实验仅覆盖有限语言（15/10/7/5 语言），未扩展到 mPLM 支持的全部数百种语言；仅测试了 base 和 large 两种模型规模。
- 评估语言仍属于相对高资源范畴，对于 Kaixana、Ainu 等极超低资源语言的改进效果未知，需进一步探索数据稀缺场景。
- 采用跨语言转移差距（∆）作为公平性度量可能不够全面——若英语性能大幅提升而其他语言小幅提升，差距仍可能较大，需开发更能反映多语言模型性能均衡性与实用性的新指标。

## 研究启发与可借鉴点
1. **无需标注数据的自蒸馏范式**：通过多数投票生成伪标签 + 置信度筛选教师的方式，可将"内部知识再利用"思想迁移到其他多语言任务的后训练优化中，值得在 NER、NER 等下游任务上验证。
2. **教师-学生动态配对策略**：过滤学生-学生配对以避免噪声传播的设计简洁有效，可推广至其他多模型/多语言蒸馏场景。
3. **语言亲缘性的隐性利用**：实验证明非高资源语言也可成为可靠教师（如波斯语对斯瓦希里语），启示我们在跨语言知识迁移中应重视语言学相似度而非单纯依赖资源量。
4. **知识探测作为分析工具**：GeoMLAMA 被用于验证方法是否真正缩小了知识差距而非表面性能，这种"干预前/后知识对比"的评估思路可用于分析其他多语言改进方法的有效性。

## 关键术语表
**mPLM（Multilingual Pre-trained Language Model）**：在多种语言语料上预训练的深度学习模型（如 XLM-R、mT5），支持跨语言零样本/少样本迁移。
**跨语言转移差距（Cross-lingual Transfer Gap, ∆）**：同一 mPLM 中英语性能与其他语言性能的差值，用于衡量模型的多语言公平性。
**教师语言选择（Teacher Language Selection）**：根据模型对各语言预测的置信度动态识别表现可靠的语言作为知识来源。
**跨语言自蒸馏（Cross-lingual Self-Distillation）**：利用 mPLM 内部教师语言的知识通过 KL 散度一致性损失指导学生语言，无需外部标注。
**GeoMLAMA**：Geo-diverse Commonsense Probing 数据集，用于探测 mPLM 在不同国家和语言的常识知识掌握程度。
**Supergen**：基于 PLM 的文本生成方法，通过标签描述提示自动生成本地化无标注训练数据。
**多数投票伪标签（Majority Vote Pseudo-labeling）**：整合多语言预测结果中得票最多的标签作为训练信号，降低单语言预测误差的影响。

## 可复现要素
- **数据集**：XNLI、PAWS-X、XCOPA、GeoMLAMA 均为公开数据集。
- **代码**：已开源，地址 https://github.com/pkunlp-icler/ALSACE。
- **模型**：XLM-R-base/large、mT5-large（公开预训练权重）。
- **关键超参**：学习率 3e-8，dropout 0.1，batch size 32/语言，蒸馏数据 500-shot；XNLI/PAWS-X/XCOPA 的阈值 θ 分别设为 0.06、0.2、0.2。
- **数据构建**：Supergen（语言模型生成）+ 机器翻译构建无标注平行语料。
- **少资源设置**：XNLI 128-shot、PAWS-X 512-shot 英语标注微调。
