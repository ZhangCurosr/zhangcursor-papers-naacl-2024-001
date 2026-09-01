---
title: "Text-Diffusion-Model-with-Encoder-Decoder-Transformers-for-S"
source: https://aclanthology.org/2024.naacl-long.2.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 21:34:25"
field: "文本生成与扩散模型"
keywords: ["diffusion model", "sequence-to-sequence generation", "self-conditioning", "adaptive noise schedule", "text generation", "encoder-decoder transformer"]
innovations: ["提出Encoder-Decoder架构的序列到序列扩散模型SeqDiffuSeq", "引入自条件技术提升生成质量", "设计token级自适应噪声调度平衡去噪难度"]
benchmarks: ["QQP", "Wiki-Auto", "Quasar-T", "CCD", "IWSLT14", "WMT14"]
---

# 论文速读：Text-Diffusion-Model-with-Encoder-Decoder-Transformers-for-S

## 一句话总结
论文提出 **SeqDiffuSeq**，一种基于连续扩散模型与 Encoder-Decoder Transformer 的序列到序列文本生成方法，通过**自条件技术**和**自适应 token 级噪声调度**显著提升生成质量与推理效率。

---

## 研究问题与动机
1. 扩散模型在图像、音频等领域已取得突破，但文本是离散分类数据，直接将连续扩散模型迁移到自然语言生成具有挑战性。
2. 现有文本扩散工作（如 DiffusionLM、D3PM）仅聚焦于无条件或可控文本生成，尚未解决序列到序列（Sequence-to-Sequence）生成任务。
3. 已有的 Seq2Seq 扩散模型 DiffuSeq 采用 **Encoder-only** 架构，在推理过程中需要重复计算输入序列，导致效率低下。
4. 固定噪声调度（如 sqrt schedule）无法平衡不同 token 位置在不同时间步的去噪难度，限制了生成质量。

---

## 核心贡献（创新点）
1. **提出 SeqDiffuSeq 框架**：将 DiffusionLM 的连续扩散框架扩展到序列到序列设置，采用 Encoder-Decoder Transformer 架构建模去噪函数，输入序列仅需一次前向计算。
2. **引入自条件技术（Self-Conditioning）**：将前一步的去噪预测结果与当前噪声序列在嵌入维度拼接，使模型能够基于历史预测进行渐进式细化，而非从零开始生成。
3. **设计自适应噪声调度（Adaptive Noise Schedule）**：在 token 级别学习噪声调度曲线，使去噪难度随时间步线性增加，并为不同位置 token 分配差异化噪声水平。
4. **实验验证全面性**：在 5 个 Seq2Seq 生成任务（改写、文本简化、问题生成、对话、机器翻译）上验证了方法的优越性，并证明自条件与自适应噪声调度具有互补性。

---

## 方法详解
### 1. 扩散过程建模
- **前向过程**：将输出序列 $w_y$ 通过嵌入函数 $g_\phi$ 映射为连续向量 $z_0 \in \mathbb{R}^{n \times d}$，然后在 $T$ 个时间步逐步添加高斯噪声，最终得到近似标准正态分布的 $z_T$。
- **反向过程**：使用参数化去噪分布 $p_\theta(z_{t-1}|z_t, w_x)$ 从噪声中逐步恢复目标序列嵌入，最终通过最近邻舍入映射回词表生成离散序列。

### 2. 训练目标
简化后的损失函数为：
$$
\mathcal{L}_{simple} = \mathbb{E}\left[\sum_{t=2}^{T} \|z_\theta^0(z_t, w_x, t) - z_0\|^2 + \|\tilde{\mu}(z_T, z_0)\|^2 + \|z_\theta^0(z_1, w_x, 1) - g_\phi(w_y)\|^2 - \log\tilde{p}_\phi(w_y|z_0)\right]
$$
其中 $z_\theta^0$ 是由 Encoder-Decoder Transformer 实现的去噪函数。

### 3. 自条件技术
- 在时间步 $t$，将前一步预测 $\hat{z}_0^t = z_\theta^0(z_{t+1}, w_x, t+1)$ 与当前噪声 $z_t$ 在嵌入维度拼接，输入维度变为 $n \times 2d$。
- 训练时以 50% 概率使用零输入训练，另一半概率使用前向估计的 $\hat{z}_0^t$ 作为条件，且不反向传播通过该估计。

### 4. 自适应噪声调度
- 定义 token 级别训练损失：$\mathcal{L}_t^i = \mathbb{E}[\|z_\theta^0(z_t, \hat{z}_0^t, w_x, t)^i - z_0^i\|^2]$
- 通过线性插值拟合映射 $M_i: \mathcal{L}^i \rightarrow \bar{\alpha}^i$，使噪声水平与训练损失对齐。
- 每隔 20,000 步更新一次调度，使用粗粒度下采样（K=20）平滑损失曲线。
- 实验发现较早位置的 token 噪声更低，呈现从左到右的生成偏好。

---

## 实验与结果
### 数据集
- **QQP**：改写生成（144,715 训练样本）
- **Wiki-Auto**：文本简化（677,751 训练样本）
- **Quasar-T**：问题生成（116,953 训练样本）
- **CCD**：对话生成（3,382,137 训练样本）
- **IWSLT14 / WMT14**：德语-英语机器翻译

### 评估指标
BLEU、BERTScore、dist-1（词汇多样性）、SacreBLEU（翻译任务）

### 主要结果
| 任务 | 模型 | BLEU / SacreBLEU | 关键对比 |
|------|------|------------------|----------|
| QQP | SeqDiffuSeq | 23.28 | 优于 DiffuSeq（18.47），接近 DiffuSeq w/ MBR=10（24.13） |
| Wiki-Auto | SeqDiffuSeq | 37.09 | 超越 DiffuSeq（29.89）及 GPT-2-large FT |
| Quasar-T | SeqDiffuSeq | 17.20 | 超越 DiffuSeq（15.84） |
| IWSLT14 EN-DE | SeqDiffuSeq | 21.96 | 超越 CMLM iter=1（14.36） |
| WMT14 EN-DE | SeqDiffuSeq | 19.16 | 与 CDCD（19.30）相当 |

- **MBR 解码提升**：SeqDiffuSeq w/ MBR=10 在 QQP 达到 24.34 BLEU，在 Wiki-Auto 达到 37.12。
- **推理加速**：相比 DiffuSeq，SeqDiffuSeq 在单张 V100 上推理速度提升 **3.56 倍**（89s vs 317s）。

### 消融实验
| 模型 | Avg. ∆BLEU |
|------|------------|
| 完整 SeqDiffuSeq | - |
| 移除自适应噪声调度 | -2.29 |
| 移除自条件 | -1.34 |
| 同时移除两者 | -5.71 |

---

## 相关工作脉络
1. **DiffusionLM**（Li et al., 2022）：首次将连续扩散模型应用于词嵌入空间，本文在其基础上扩展至 Seq2Seq 设置。
2. **DiffuSeq**（Gong et al., 2022）：首个 Seq2Seq 扩散模型，但采用 Encoder-only 架构，推理效率受限；本文使用 Encoder-Decoder 提升效率。
3. **D3PM / Multinomial Diffusion**（Hoogeboom et al., 2021; Austin et al., 2021）：直接在离散 token 空间定义扩散过程；本文采用连续嵌入空间避免离散梯度问题。
4. **Bit-Diffusion**（Chen et al., 2022）：将离散数据编码为二进制位并应用自条件；本文将其适配到连续嵌入空间的 Seq2Seq 场景。
5. **CDCD**（Dieleman et al., 2022）：并发工作，使用连续扩散做语言建模和翻译；本文聚焦自适应 token 级噪声调度而非全局 learned schedule。
6. **Levenshtein Transformer / CMLM**：非自回归基线；本文在翻译任务上超越 CMLM iter=1，证明扩散模型在非自回归框架下的潜力。

---

## 局限性与未来方向
1. **计算成本高**：扩散模型仍需数千次迭代（T=2000），尽管相比 DiffuSeq 已加速 3.56 倍，但相比 AR 模型仍有差距。
2. **生成多样性下降**：自条件与自适应噪声调度提升了质量，但以牺牲多样性为代价，MBR 解码带来的边际增益有限。
3. **CCD 对话任务表现不佳**：所有模型在该任务上 BLEU 均较低，可能源于任务特性或数据质量。
4. **未来方向**：减少反向过程迭代步数（参考 DDIM 等快速采样方法）、探索提升多样性的采样策略。

---

## 研究启发与可借鉴点
1. **Encoder-Decoder 架构用于扩散模型**：在 Seq2Seq 任务中，仅对输入编码一次可显著降低推理开销，该设计可直接迁移到其他扩散生成任务。
2. **自条件技术在文本扩散中的有效性**：将历史预测作为额外输入可缓解信息丢失问题，值得在图像/音频扩散模型中探索。
3. **自适应噪声调度的通用性**：基于训练损失动态调整噪声水平的思路可推广至其他离散/连续扩散模型，平衡各时间步的学习难度。
4. **MBR 解码与多样性的权衡分析**：本文揭示了质量-多样性 trade-off 现象，为后续研究提供了评估视角。
5. **Token 级位置感知调度**：不同位置 token 分配不同噪声曲线的发现，启示未来可设计位置感知的扩散调度策略。

---

## 关键术语表
**Diffusion Model**：通过前向加噪和反向去噪两个过程学习数据分布的生成模型。
**Self-Conditioning**：将前一步预测结果作为当前步骤额外输入的技术，用于保留历史信息。
**Adaptive Noise Schedule**：根据训练损失动态调整各时间步噪声水平的策略，使去噪难度均匀分布。
**MBR（Minimum Bayes Risk）解码**：从多个候选输出中选择期望风险最小的序列作为最终预测。
**Encoder-Decoder Transformer**：同时包含编码器和解码器的注意力网络架构，广泛用于 Seq2Seq 任务。
**SacreBLEU**：机器翻译评估指标，提供标准化且可复现的 BLEU 计算。
**Dist-1**：衡量生成文本词汇多样性的指标，计算 unigram 的种类数与总词数之比。

---

## 可复现要素
- **数据集**：全部公开可用（QQP、Wiki-Auto、Quasar-T、CCD、IWSLT14、WMT14）
- **代码/权重**：论文未提及开源
- **关键超参**：
  - 最大扩散步数 $T = 2000$
  - 学习率 $1e-4$，warm-up 10000 步，线性衰减
  - 自适应噪声调度更新间隔：20000 步
  - 粗粒度下采样参数 $K = 20$
  - 编码器/解码器层数：6 层
  - 隐藏维度：翻译任务 512，其他任务 768
  - Embedding 维度：128
