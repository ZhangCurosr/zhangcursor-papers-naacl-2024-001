---
title: "Embodied-Executable-Policy-Learning-with-Language-based-Scen"
source: https://aclanthology.org/2024.naacl-long.105.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:14:17"
field: "具身智能与语言模型结合的策略学习"
keywords: ["具身智能", "大语言模型", "模仿学习", "强化学习", "视觉-语言导航", "机器人策略学习", "VirtualHome"]
innovations: ["提出纯视觉输入的两阶段 SUM+APM 范式，将图像描述与可执行程序生成统一于 LLM 框架内", "同时支持 IL（交叉熵）和 RL（REINFORCE）两种微调策略，RL 可在稀疏环境反馈下继续优化策略", "构建了 VirtualHome 三视角大规模图像-动作配对数据集并开源代码，系统评估了多种 SUM/APM 组合"]
benchmarks: ["VirtualHome", "VirtualHome (7个布局环境)", "Li et al. (2022b) baseline"]
---

# 论文速读：Embodied-Executable-Policy-Learning-with-Language-based-Scen

## 一句话总结
本文提出了一种基于纯视觉输入的机器人学习范式，通过场景理解模块（SUM）将图像观测转化为自然语言描述，再由动作预测模块（APM，基于大语言模型）生成可执行的机器人动作计划；通过在 VirtualHome 环境中的实验，该方法在使用专家数据进行模仿学习微调后，平均执行率可达 79.0%，显著优于已有基线。

## 研究问题与动机
- **视觉观测与文本动作的模态鸿沟**：预训练 LLM 依赖文本输入，而机器人从视觉（图像）观测出发，如何将图像观测有效转化为 LLM 可理解的语言表示是关键挑战。
- **领域分布偏移**：预训练模型的训练分布与机器人学习任务之间存在显著分布差异，直接应用效果差，需要有效的微调策略来弥合差距。
- **现有方法依赖规则化转换或纯文本输入**：如 Liang et al. (2022) 使用规则化感知 API 将图像转为文本，缺乏可学习性；Li et al. (2022b) 使用 Oracle 级别的文本描述作为输入，而非真实视觉观测。
- **缺乏从稀疏环境反馈中进化的能力**：专家演示数据有限，如何让模型在仅有稀疏奖励信号的情况下继续学习是重要问题。

## 核心贡献（创新点）
- **提出端到端的视觉→语言→动作的具身学习范式**：将多模态视觉编码器与大语言模型相结合，以纯视觉观测为输入直接生成可执行程序，而非依赖外部规则转换或文本指令。
- **设计两阶段微调策略（IL + RL）**：先对 SUM 进行模仿学习微调以适配目标域场景，再对 APM 分别采用交叉熵损失（IL）和 REINFORCE（RL）进行微调，后者可容忍非专家数据的稀疏反馈。
- **构建并开源了 VirtualHome 视觉-动作配对数据集**：收集了三种视角（AUTO、FIRST_PERSON、FRONT_PERSON）共约 8 万张图像-文本对，用于 SUM 和 APM 的微调，代码已开源。
- **系统评估了多种 SUM/APM 组合在 7 个 VirtualHome 环境中的表现**：表明 OFA + BART 组合在两种微调策略下均取得最优且最稳定的执行率。

## 方法详解
- **整体架构分为两个模块**：场景理解模块（SUM）将视觉观测编码为语言描述；动作预测模块（APM）以 SUM 的输出为输入，生成可执行的机器人动作序列（形如 `[ACTION] <object> (id)`）。
- **SUM 选用预训练图像描述模型**（OFA_Large、BLIP、GRIT），去掉了 "a picture of" 等提示词以避免模型仅描述场景而不推导动作，微调 7 个 epoch。
- **APM 选用 encoder-decoder 架构的 LLM**（BERT、RoBERTa、BART），通过最后隐藏层输出 token 后解码为可执行程序。
- **IL 微调（交叉熵损失）**：
  $$L_{XE}(\theta) = -\sum_{i=1}^{n} \log P(y_i | y_{1:i-1}, X)$$
  在词级别优化，以专家轨迹的 ground-truth 动作序列为监督信号。
- **RL 微调（REINFORCE）**：将 LLM 视为策略，在完整轨迹上采样，用序列级奖励函数（执行成功与否）计算梯度：
  $$\nabla_\theta L(\theta) = -\frac{1}{k}\sum_{i=1}^{k}\left((r(w^i) - b)\nabla_\theta \log P(w^i)\right)$$
  其中 $b$ 为 baseline（greedy 解码或 beam 候选的平均奖励），通常需先进行 CE 预训练再引入 RL 微调以稳定策略。

## 实验与结果
- **环境**：VirtualHome，7 个不同的房屋布局环境。
- **数据集**：约 8 万张图像-文本对（AUTO 26,600 / FIRST_PERSON 26,607 / FRONT_PERSON 26,608）。
- **评测指标**：BLEU、ROUGE-L、METEOR、CIDEr、SPICE，以及核心指标——执行率（Execution Rate，即模拟器成功执行输出动作的概率）。
- **最强结果（IL 微调）**：OFA + BART 在 7 个环境上的平均执行率为 **79.0% ± 1.91%**，BLEU-1 为 59.5，显著高于 Li et al. (2022b) 的基线方法（见 Figure 2）。
- **RL 微调最强结果**：OFA + BART 平均执行率 **57.2% ± 2.43%**，仍优于 Li et al. (2022b) 的 RL 结果（53.7%，见 Table 5）。
- **泛化到未见任务（Novel Tasks）**：IL 微调下达到 44.8%，RL 微调下达到 33.7%，均超过 Li et al. (2022b)（27.8%）。
- **消融结论**：OFA 在 IL 下最优，BLIP 在 RL 下略优但方差大（约 4 倍）；BART 因 denoising autoencoder 结构最适合语言→程序的任务；视角方面 FIRST_PERSON 因缺乏显式动作呈现导致 SUM 生成质量较差。

## 相关工作脉络
- **Li et al. (2022b)**：使用 Oracle 级别文本描述 + BERT 进行 IL/RL 微调，本文与其相比使用了真实视觉输入并增加了 SUM 模块，性能全面超越。
- **Liang et al. (2022) Code as Policies**：将 LLM 用于生成机器人控制代码，但依赖文本指令输入；本文从纯视觉出发， bridging modality gap。
- **Zeng et al. (2022) Socratic Models**：结合零样本多模态推理，侧重于推理而非生成可执行程序；本文直接输出 simulator 可执行的 program。
- **Li et al. (2019) VLN-BERT**：结合图像描述和规划模型，但以纯语言指令为输入；本文输入为纯视觉观测，更具实用性。
- **Xiao et al. (2022)**：结合语言指令和视觉图像微调 VLM，但生成的是高级自然语言指令而非可执行程序；本文直接生成 VirtualHome 可执行的 program 格式。
- **Huang et al. (2022b) Inner Monologue**：使用 LLM 进行具身推理和规划，依赖文本场景描述；本文通过 SUM 自动从图像生成描述，无需人工标注。

## 局限性与未来方向
- **仅涉及高层抽象动作，未考虑低层运动控制**（如关节电机控制），限制了在复杂动态环境中的整体有效性。
- **长尾动作（long-tailed actions）学习不足**：数据集中频繁出现的动作效果好，罕见动作的执行政策难以学好。
- **跨平台泛化受限**：模型在 VirtualHome 上微调，不同模拟器平台间差异大，泛化性不足。
- **首人称视角（FIRST_PERSON）下 SUM 生成质量下降**：相机视角限制了对动作的显式捕捉，影响了后续 APM 的表现。
- **未来方向**：扩展到更数据/参数高效的泛化任务；学习通用低层控制器以适应不同形态的具身代理。

## 研究启发与可借鉴点
- **"视觉→语言→动作"的桥接范式可迁移到其他具身平台**：SUM+APM 的两阶段架构不依赖特定模拟器，只需适配目标平台的动作语料库即可复用。
- **IL 与 RL 分阶段微调的策略设计**：先用 CE 损失在专家数据上预训练稳定策略，再引入 REINFORCE 利用稀疏奖励进一步精调，这一流程可直接借鉴于其他 LLM 驱动的策略学习任务。
- **去除 "a picture of" 类提示词以避免场景描述偏向**：这一简单但有效的工程技巧对图像描述任务中需要推导行动信息的场景有参考价值。
- **将 BART 用作 action generation 的 APM**：其 denoising autoencoder 架构适合从噪声化的自然语言描述到结构化程序的映射，这一选择思路可推广到代码生成类机器人任务。
- **虚拟仿真环境（VirtualHome）作为 LLM 策略学习的可复现 benchmark**：为研究团队提供了一个标准评测平台，便于横向对比。

## 关键术语表
- **SUM（Scene Understanding Module）**：场景理解模块，将机器人视觉观测（图像）转换为自然语言描述的多模态组件，是连接视觉与语言动作的桥梁。
- **APM（Action Prediction Module）**：动作预测模块，基于预训练大语言模型，将 SUM 输出的语言描述解码为机器人可执行的程序序列。
- **Execution Rate（执行率）**：核心评估指标，指 APM 输出的动作计划在整个轨迹中被 VirtualHome 模拟器成功执行的概率。
- **Imitation Learning（IL）**：模仿学习，利用专家演示数据通过交叉熵损失对 APM 进行监督微调的策略。
- **REINFORCE**：一种策略梯度强化学习算法，在本文中被用于利用稀疏环境奖励对 APM 进行序列级微调。
- **VirtualHome**：一个多智能体虚拟家庭活动仿真平台，提供超过 3000 项家务活动的可执行程序和高保真视觉观测。
- **BLEU / ROUGE-L / CIDEr / SPICE**：自然语言生成任务的标准自动评测指标，用于衡量 APM 输出与专家动作描述的文本相似度。
- **Denoising Autoencoder**：BART 模型的核心架构，通过噪声输入重建原始序列，适用于从自然语言到结构化程序的任务。

## 可复现要素
- **数据集**：基于 VirtualHome v0.1.0 构建，约 8 万张图像-文本对，三种视角（AUTO / FIRST_PERSON / FRONT_PERSON）；**论文未声明数据集完全公开**，但提供了代码链接。
- **代码**：已开源，GitHub 地址为 https://github.com/Jason-Qiu/Embodied_Policy_Learning。
- **关键超参**：SUM 和 APM 微调均为 7 个 epoch；学习率搜索范围为 [1e-4, 1e-5, 1e-7]；Batch Size 搜索范围为 [4, 8, 16, 32, 64]；Dropout 为 [0.1, 0.2, 0.3]；每个环境运行 10 个随机种子。
- **GPU/硬件资源**：论文未明确提及。
