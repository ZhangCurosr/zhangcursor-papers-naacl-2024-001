---
title: "FPT-Feature-Prompt-Tuning-for-Few-shot-Readability-Assessmen"
source: https://aclanthology.org/2024.naacl-long.16.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:16:05"
field: "可读性评估与少样本学习"
keywords: ["readability assessment", "prompt tuning", "few-shot learning", "linguistic features", "soft prompt", "feature fusion"]
innovations: ["将语言学特征通过MultiHeadMLP映射为可训练软提示向量融入prompt输入", "设计基于ListMLE的类别间相似性校准损失以保持特征嵌入前后排序一致性", "提出分类损失与校准损失交替训练策略优化特征嵌入"]
benchmarks: ["ChineseLR", "WeeBit", "Cambridge"]
---

# 论文速读：FPT-Feature-Prompt-Tuning-for-Few-shot-Readability-Assessmen

## 一句话总结
本文提出 Feature Prompt Tuning (FPT)，一种将语言学特征嵌入可训练软提示的 prompt-based tuning 框架，用于少样本可读性评估任务；通过新增的类别间相似性校准损失保持特征原始相似度关系，显著优于现有 prompt 方法和特征融合基线，并在多数场景下超越 GPT-3.5-Turbo-16K。

## 研究问题与动机
1. **现有 prompt 方法缺少语言学知识**：Prompt-based tuning 在多数少样本文本分类任务上表现优异，但在可读性评估（Readability Assessment, RA）任务上性能不足，原因是这类任务高度依赖语言学特征（如词汇难度、句法复杂度等），而传统 prompt 方法无法显式利用此类先验知识。
2. **现有特征融合方法在少样本下不稳定**：先前研究证明融合语言学特征与预训练语言模型（PLM）可获得 RA 的最优结果，但这些方法主要面向大量标注数据的 fine-tuning 场景，在 few-shot 设置下可能表现不稳定甚至损害性能。
3. **RA 任务具有类别间的顺序/相似性结构**：可读性等级之间存在自然的相似度关系（如等级 2 比等级 5 更接近等级 3），而现有方法未对此进行显式建模。

## 核心贡献（创新点）
1. **提出 Feature Prompt Tuning (FPT) 框架**：将提取的语言学特征通过 MultiHeadMLP 映射为可训练的软提示向量，直接嵌入 PLM 输入序列；与以往直接在 [CLS] 表示层融合特征的方式本质不同，FPT 使语言学知识从输入端参与预训练模型的推理过程。
2. **设计类别间相似性校准损失（Similarity Calibration Loss）**：基于 ListMLE 排序损失，对输入特征与嵌入后特征的类别间余弦相似度矩阵的列排序关系进行约束，确保不同等级间的相似度相对顺序不变；区别于普通分类损失仅关注正确类别预测。
3. **交替训练策略**：分步更新 PLM 参数和特征嵌入，避免两类损失梯度相互干扰；相比联合优化，该方法在实验中被证明更稳定有效。
4. **系统性地验证语言学特征在少样本 RA 中的价值**：通过 ablation 和替换实验证明语言学特征对中文 RA 效果尤为显著，且小样本场景下增益更大。

## 方法详解
**特征提取（Feature Extraction）**：英文文本使用 lingfeat 工具包提取 discourse、syntactic、lexical、shallow 四类特征；中文文本使用 zhfeat 工具包提取字符级、词汇级、句子级和段落级特征。输入文本 x 的特征表示为 f_x（α 维向量）。

**特征嵌入（Feature Embedding）**：将 f_x 通过 MultiHeadMLP（l 个输出头）映射为 l 个可训练软提示向量 {v_1, ..., v_l}，拼接硬模板 T 和 [MASK] 后形成完整输入序列 (v_1, ..., v_l, e(T), e([MASK]), e(x)) 送入 PLM。

**分类损失（L_classification）**：与标准 prompt tuning 一致，最小化 cross-entropy 分类损失：
$$\mathcal{L}_{classification} = -\frac{1}{|\mathcal{X}|}\sum_{x \in \mathcal{X}}\log P_M(y|x)$$

**相似性校准损失（L_calibration）**：
- 对每个类别 c_i 的所有样本特征做 average pooling 得到 F'_{c_i}；
- 计算类别间余弦相似度矩阵 M（原始特征）和 M'（嵌入后特征）；
- 以 M 的列排序 Π 为参考，用 ListMLE 损失最小化 M' 对应位置的排序差异：
$$L_{calibration} = -\sum_{k=1}^{n}\log\prod_{i=1}^{n}\frac{\exp(s'_{\pi_{ik}})}{\sum_{j=i}^{n}\exp(s'_{\pi_{jk}})}$$

**交替训练（Algorithm 1）**：每个 epoch 内，先对每个 batch 计算 L_classification 更新 PLM 参数 θ 和特征嵌入 f，再以整个数据集计算 L_calibration 更新 f，交替进行。

## 实验与结果
**数据集**：ChineseLR（中文，5 个难度等级，共 4160 篇）、WeeBit（英语，5 级，共 3125 篇）、Cambridge（英语，5 级，共 300 篇）。

**评估指标**：Accuracy，每 k-shot 采样 4 个训练集×4 次重复取均值。

**关键结果**：

- **vs. Prompt-based Baselines**（Table 2）：FPT 在所有 k-shot（1/2/4/8/16）和所有数据集上均最优。2-shot 下，ChineseLR 相对 Soft Prompt 提升 **43.9%**（相对增益）/ 绝对提升 14.10 个百分点；Weebit 提升 5.50%。
- **vs. Feature Fusion Methods**（Table 3）：2-shot 下，FPT 超越此前最优方法 PF，分别在 ChineseLR、Weebit、Cambridge 上提升 **43.19%**、**11.55%**、**11.66** 个百分点。
- **vs. LLM**（Table 5）：110M 参数的 FPT 在英语数据集上全面超过 gpt-3.5-turbo-16k；中文任务因 LLM 上下文长度限制无法进行 1/2-shot 实验。

**Ablation**（Table 4）：去除 Similarity Calibration（-SC）导致性能下降（ChineseLR 2-shot：46.24→40.97）；同时去除 Feature Prompt（-SC and FP）性能骤降（ChineseLR 2-shot 降至 25.45）。

## 相关工作脉络
1. **Prompt-based Tuning（Shin et al., 2020; Lester et al., 2021; Liu et al., 2021）**：本文在其基础上引入语言学特征作为软提示来源，解决 RA 任务中传统 prompt 缺少领域知识的问题。
2. **Projecting Feature (PF)（Li et al., 2022）**：此前 RA 最优特征融合方法，通过将特征投影到 [CLS] 表示空间并与之融合；FPT 与之的本质区别在于特征从输入端进入 prompt 而非输出端融合。
3. **T5/BART-based Prompt Learning for RA（Lee & Lee, 2023）**：将 RA 视为 seq2seq 生成任务并优化 hard prompt，但任务定义（生成式 vs. 判别式）和训练策略（多数据集联合训练）不同，本文认为难以直接对比。
4. **Neural + Linguistic Feature Fusion（Lee et al., 2021; Qiu et al., 2021）**：证明特征融合在满标注场景有效，但少样本下不稳定；本文首次系统验证特征嵌入 prompt 在少样本下的优势。
5. **LLM Few-shot Prompting（Brown et al., 2020）**：本文在 Table 5 中与 gpt-3.5-turbo-16k 对比，表明专门设计的 prompt tuning 在小样本特定领域任务上仍可超越通用 LLM。

## 局限性与未来方向
1. **长文本处理能力受限**：基于 BERT 的 MLM 架构对长文本支持有限，而中文 RA 数据通常为长篇文章（平均长度超千 token），这是实际应用的瓶颈。
2. **模型可解释性不足**：黑箱性质使得分类决策难以追溯，不利于教学和教育应用场景中对结果的合理解释。
3. **语言学特征质量依赖性强**：对于形态丰富的语言或低资源语言，高质量特征提取无法保证，限制了方法的可迁移性。
4. **中文数据集单一**：仅使用一个中文 RA 数据集（ChineseLR），缺乏更多中文benchmark 验证泛化性。

## 研究启发与可借鉴点
1. **特征→软提示的映射范式**：将领域特征（非语言学特征）通过 MLP 映射为可训练软提示，可直接迁移至其他需要结合手征特征的少样本分类任务（如情感分析、疾病诊断等）。
2. **相似性校准损失的设计思路**：ListMLE 排序损失用于保持输入特征到嵌入空间的顺序一致性，可推广至任何具有天然类别相似度结构的有序分类任务（如等级评估、程度判断）。
3. **交替训练策略**：分类损失与辅助正则损失分步更新，避免梯度冲突，适用于多目标协同优化的 prompt tuning 场景。
4. **小规模数据集上的 LLM 对比基线**：与 GPT-3.5 的 few-shot 对比为小模型专用方法提供了说服力，启示后续研究应考虑与 LLM in-context learning 的系统对比。

## 关键术语表
**Readability Assessment (RA)**：自动评估文本阅读难度的任务，通常将文本归类为若干难度等级。
**Prompt-based Tuning**：通过将下游分类任务重构为完形填空（cloze）形式，在预训练语言模型前插入可学习的 prompt 进行高效微调的技术。
**Soft Prompt**：用连续可训练向量替代硬提示模板中的离散 token，无需人工设计模板的 prompt 方法。
**Feature Prompt**：将由输入文本提取的领域特征经神经网络映射后作为可训练软提示向量融入 prompt 的机制。
**Similarity Calibration**：通过排序损失约束嵌入前后类别间特征相似度矩阵的列排序保持一致性的正则化手段。
**ListMLE**：一种 listwise 排序学习损失函数，通过对排列概率的负对数似然最小化来维持排序关系。
**ChineseLR**：基于中国义务教育课程标准编制的中文可读性评估数据集，包含 5 个难度等级的 4160 篇课文。
**WeeBit**：基于 Weekly Reader 语料扩展的英语可读性评估基准数据集，广泛用于 RA 研究。

## 可复现要素
- **数据集**：ChineseLR、WeeBit、Cambridge（均引用自公开论文，数据可获取）
- **代码/权重**：论文声明将公开代码（"We will make our code public available"）
- **PLM 骨干**：bert-base-uncased（英文）、bert-base-chinese（中文）
- **优化器**：AdamW，weight_decay=0.01，warmup_ratio=0.05
- **学习率/Epoch/Batch**：lr=1e-5，30 epochs，batch_size=8
- **特征提取工具包**：lingfeat（英文，CC-BY-SA-4.0）、zhfeat（中文）
- **Few-shot 设置**：k=1,2,4,8,16，每 k 值采样 4 个训练集×4 次重复
- **硬件**：4× NVIDIA GeForce RTX 3090
