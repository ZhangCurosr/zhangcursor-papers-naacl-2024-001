---
title: "Do-Localization-Methods-Actually-Localize-Memorized-Data-in"
source: https://aclanthology.org/2024.naacl-long.176.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 15:25:48"
field: "大语言模型可解释性"
keywords: ["LLM可解释性", "记忆定位", "网络剪枝", "机器遗忘", "知识神经元", "模型编辑"]
innovations: ["提出INJ Benchmark通过主动注入创建ground truth直接评估定位", "Adapt网络剪枝方法用于定位且表现最优", "双基准验证揭示定位方法与擦除能力的关系"]
benchmarks: ["INJ Benchmark", "DEL Benchmark"]
---

# 论文速读：Do Localization Methods Actually Localize Memorized Data in LLMs? A Tale of Two Benchmarks

## 一句话总结
本文首次系统性地直接评估了LLM中记忆数据定位方法的有效性，提出了INJ和DEL两个互补基准测试，发现基于网络剪枝的方法（如HARD CONCRETE）表现最佳，但所有方法均难以精确定位单一序列而不误伤相关序列。

## 研究问题与动机
1. **核心问题缺失**：现有定位研究多通过下游任务成功间接证明定位有效性，未直接评估定位本身是否成功
2. **知识编辑≠定位**：Hase et al. (2023)指出编辑成功与定位无关，但定位方法的有效性仍无标准评估框架
3. **Ground Truth缺失**：自然预训练后的记忆数据其"真实位置"未知，难以直接验证定位准确性
4. **应用需求迫切**：成功的定位可为机器遗忘（machine unlearning）提供基础，如删除敏感信息

## 核心贡献（创新点）
1. **提出INJ Benchmark**：通过主动注入新信息到指定权重，创建可直接验证定位准确性的ground truth
2. **提出DEL Benchmark**：基于knockout思想，通过drop out定位神经元测量对记忆序列的影响
3. **首次系统对比5种定位方法**：包括知识神经元方法、梯度方法、暴力搜索和剪枝自适应方法
4. **发现剪枝方法优于归因方法**：HARD CONCRETE在两个基准上均表现最佳，Recall@0.5%超过80%
5. **揭示定位的固有局限性**：所有方法都会混淆相关类别的记忆序列，暗示共享神经元机制

## 方法详解

### INJ Benchmark设计
- **数据**：ECBD-2021数据集的156条新实体定义句（所有LLM预训练于2021年前）
- **注入过程**：
  1. 对每条序列x_i，随机采样r%的FFN权重向量{v_i^l} across all L layers
  2. 仅fine-tune这些权重，其余参数冻结
  3. 使用语言建模损失训练至loss < 0.05
  4. 每条序列使用不同的权重集合φ_i，训练独立模型θ̃_i
- **评估指标**：Recall@k% = |Γ_i ∩ Γ̂_i| / |Γ_i|，其中Γ_i为ground truth注入神经元集合，Γ̂_i为方法预测的神经元集合

### DEL Benchmark设计
- **数据收集**：
  - Pythia模型：从Pile-dedupe预训练批次中筛选505条记忆序列（Accuracy>90%, Levenshtein距离<20）
  - GPT2-XL：手动搜索105条记忆序列（引语、代码、URL等类别）
- **评估方式**：drop out top-k%定位神经元，测量：
  - ∆ Self-Acc：目标序列准确率变化（越大越好）
  - ∆ Self-Dist：目标序列Levenshtein距离变化（越大越好）
  - ∆ Neg-Acc/Neg-Dist：负例序列变化（越小越好）
  - ∆ Rand-PPL：随机序列perplexity变化（越小越好）

### 五种定位方法

**1. ACTIVATIONS (Geva et al., 2022)**
- 将神经元激活h_i^l视为记忆系数
- 公式：A^l(i) = (1/T) Σ_t |h_{i,t}^l| · ||v_i^l||
- 计算高效，仅需一次forward pass

**2. Integrated Gradients (IG) (Dai et al., 2022)**
- 累积从零向量到原始隐状态的梯度路径
- 公式：IG_i(z) = z_i ∫₀¹ (∂P(αz)/∂z_i) dα
- 需要多次forward pass（文中设20步）

**3. ZERO-OUT**
- 暴力留一法：逐个drop out神经元，以memorization loss变化作为归因分数
- 公式：A^l(i) = ℓ^{mem}_{θ\nₜᵢˡ}(x) - ℓ^{mem}_θ(x)
- 计算成本最高（8.5小时/序列 on Pythia-6.9B）

**4. SLIMMING (adapted from Liu et al., 2017)**
- 从网络剪枝 Adapt，学习稀疏mask m^l ∈ ℝ^{d₂}
- 目标函数：min_{m^l} ℓ^{mem}_θ(x) + λΣ_l||m^l||₁
- L1正则化鼓励稀疏性，但mask值多在0-1之间非二元

**5. HARD CONCRETE (adapted from Louizos et al., 2018)**
- 改进SLIMMING，使用binary concrete distribution使mask值近似二元
- 公式：m̂_i^l = σ((1/β)(log(u_i/(1-u_i)) + log(m_i^l)))
- 通过重参数化实现可微优化
- 目标函数：min_{m^l} ℓ^{mem}_θ(x) + λΣ_lR(m̄^l)，其中R为L0正则化

## 实验与结果

### INJ Benchmark结果（Table 2）
- **最佳方法**：HARD CONCRETE在4个模型上Recall@0.5%均超过76%（Pythia-6.9B达76.4%，GPT2-124M达87.4%）
- **排名一致**：HARD CONCRETE > SLIMMING > ZERO-OUT ≈ IG > ACTIVATIONS > RANDOM
- **低注入比例更易定位**：ratio=0.1%时所有方法recall更高（信息更集中）
- **显著性检验**：HARD CONCRETE vs IG在24项测试中有23项p<10⁻¹⁰，差异极显著

### DEL Benchmark结果（Table 3）
- **擦除能力**：HARD CONCRETE最有效，Pythia-6.9B上drop 0.5%神经元使目标序列forget 57.7%
- **保留负例能力**：IG表现最好，对负例和随机序列影响最小
- **难以兼顾**：擦除目标序列会同时损伤相关序列记忆（如不同引语、地址）
- **跨层分布**：记忆分布在多层而非单层，多層drop out比单層高效得多（Figure 3）

### 计算效率（Table 6, RTX A6000）
| 方法 | 单序列耗时 |
|------|-----------|
| ACTIVATIONS | ~0.3秒 |
| SLIMMING | ~12秒 |
| HARD CONCRETE | ~1分钟 |
| IG | ~43分钟 |
| ZERO-OUT | ~8.5小时 |

## 相关工作脉络
1. **知识神经元 (Dai et al., 2022)**：识别存储事实知识的FFN神经元，使用IG定位
2. **Transformer FFN作为记忆 (Geva et al., 2021, 2022)**：提出FFN是LLM记忆存储的主要位置
3. **模型编辑 (Meng et al., 2022; Mitchell et al., 2022)**：通过定位更新参数实现知识修改，但编辑成功≠定位成功
4. **网络剪枝 (Han et al., 2016; Liu et al., 2017)**：通过学习稀疏子网络压缩模型，本文Adapt用于定位
5. **Knockout分析 (Olsson et al., 2022; Geva et al., 2023)**：通过移除组件观察行为变化，本文以此为评估思路
6. **机器遗忘 (Cao & Yang, 2015; Bourtoule et al., 2021)**：定位是遗忘的基础，本文评估为后续工作提供依据
7. **归因方法比较 (Hase et al., 2023)**：指出编辑成功与定位不相关，本文直接评估定位本身

## 局限性与未来方向
1. **仅研究FFN神经元**：假设FFN是主要记忆位置，未考虑attention层等其他组件
2. **Dropout评估的局限**：仅用zero ablation，未探索mean ablation、path patching等其他方法
3. **负例定义简化**：将所有其他记忆序列视为负例，未考虑语义重叠或共现关系
4. **共享神经元问题**：相关序列可能共享神经元，完美定位可能不可行
5. **未扩展到大模型**：仅在GPT2和Pythia小规模模型上评估

## 研究启发与可借鉴点
1. **Benchmark设计思路**：INJ Benchmark的"主动创建ground truth"策略可迁移至其他可解释性评估任务
2. **剪枝方法的Adapt**：将网络剪枝技术Adapt为定位方法是新颖且有效的思路
3. **双基准互补验证**：INJ提供可控评估，DEL反映真实场景，两者一致性增强结论可信度
4. **显著性检验**：大规模t-test分析支持方法差异的统计显著性，值得借鉴
5. **跨层vs单层分析**：揭示记忆分布式存储特性，对理解LLM内部机制有启发

## 关键术语表
- **Localization**：识别模型中负责特定行为的组件（如神经元）
- **Memorized Sequence**：LLM近乎逐字重现的预训练序列（前缀+后缀）
- **Neuron Dropout**：测试时置零特定神经元，排除其对输出的贡献
- **Recall@k%**：定位方法预测的top-k%神经元中，ground truth神经元的比例
- **Injection Ratio**：用于记忆注入的权重向量占比（1%或0.1%）
- **Binary Concrete Distribution**：产生近似二元mask值的连续松弛分布
- **Levenshtein Distance**：字符级别序列编辑距离，衡量记忆保真度
- **Knowledge Neuron**：存储特定关系事实的小集FFN神经元

## 可复现要素
- **数据集**：ECBD-2021（公开）、Pile-dedupe（公开）、手动收集序列（见附录Table 12-13）
- **模型**：GPT2 124M、GPT2-XL 1.5B、Pythia-deduped 2.8B/6.9B（公开）
- **代码**：依赖EleutherAI/knowledge-neurons仓库（开源）
- **关键超参**：
  - IG步数：20
  - HARD CONCRETE温度β、初始化值m（需调优）
  - SLIMMING/HARD CONCRETE的λ（平衡mem-loss与稀疏损失）
  - Dropout比例：0.1%、0.5%
