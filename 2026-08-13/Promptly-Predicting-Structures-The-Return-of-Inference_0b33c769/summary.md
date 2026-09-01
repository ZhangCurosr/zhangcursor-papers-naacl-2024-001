---
title: "Promptly-Predicting-Structures-The-Return-of-Inference"
source: https://aclanthology.org/2024.naacl-long.7.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 20:12:41"
field: "结构化自然语言处理"
keywords: ["structured prediction", "prompt-based methods", "zero-shot", "semantic role labeling", "coreference resolution", "inference"]
innovations: ["提出将LLM作为评分器结合传统组合推理的统一框架实现零样本结构化预测", "SRL任务中引入图最短路径推理消除span重叠约束", "共指解析中利用ILP传递性约束保证全局一致性"]
benchmarks: ["QA-SRL wiki", "QA-SRL 2.0", "ECB+", "OntoNotes 5.0", "GENIA"]
---

# 论文速读：Promptly Predicting Structures: The Return of Inference

## 一句话总结
本文提出一种结合提示学习与组合推理的结构化预测框架，通过将任务分解为多个局部决策问题并利用结构约束进行全局优化，实现零样本/少样本下的一致性结构化输出，在语义角色标注和共指解析任务上均获得性能提升与零不一致率。

## 研究问题与动机
1. **结构化预测的标注瓶颈**：语义角色标注（SRL）、共指解析等任务要求预测相互约束的多标签组合，标注成本极高，尤其在低资源领域更加严峻。
2. **现有提示方法忽略结构约束**：已有Prompt-based结构化预测工作（如 Liu et al., 2023）仅独立预测各组件，未建模标签间依赖关系，导致输出存在重叠、违反传递性等结构性不一致问题。
3. **零/少样本结构化预测的空白**：传统方法依赖大量标注数据学习联合概率分布，而基于Prompt的方法缺乏对组合推理的有效利用，难以在零/少样本场景下生成合法结构。

## 核心贡献（创新点）
1. **提出"提示+推理"统一框架**：将LLM用作评分器而非直接生成器，通过推理算法在候选结构中搜索全局最优解；与已有工作的本质区别在于不再依赖微调或手工规则，而是利用LLM的输出分数驱动传统结构化推理。
2. **在SRL任务中引入基于图的最短路径推理**：将候选词 spans 构建为有向图，利用 Täckström et al. (2015) 的 K 最短路径算法（Yen算法）求解无重叠约束；区别于此前仅聚合局部答案的工作，本文显式建模 span 间互斥性。
3. **在共指解析任务中引入 All-Link 整数线性规划**：将传递性约束编码为 ILP，结合 LLM 生成的 co-reference 链接得分求解全局最优聚类；不同于近期并发工作 Rajaby Faghihi & Kordjamshidi (2024) 使用多种归一化函数融合多模型输出，本文使用单一 LLM 评分。
4. **系统验证了推理带来的双重收益**：不仅实现零不一致率，且在多数据集上获得稳定性能提升；揭示小模型+约束可匹敌更大规模无约束模型的效果。

## 方法详解

### 整体框架
将结构化预测任务分解为 $n$ 个局部子任务，每个子任务对应一个问题 $q_i$，LLM 对每个问题独立生成 top-$n$ 个候选答案及得分 $f(t_r, i)$，再经推理算法在全局约束下选择最优结构。

### 语义角色标注（SRL）推理
- **问题构造**：对谓词 $p$ 的每个语义角色类型生成一个 QA 问题（如"Who gave something?"）。
- **图构建**：对句子 token 序列构建有向图，节点 $v_j$ 表示 token $w_{j-1}$ 与 $w_j$ 之间的边界；每个候选 span $t_r$ 对应一条从起始节点到结束节点的边。
- **边权重**：$s_{r,i} = f(t_1, i) - f(t_r, i)$，最高分 span 权重为 0。
- **K 最短路径**：使用 Yen 算法求 top-K 条最短路径，从中选取恰好包含每个角色一条 span 且路径长度最小的路径作为最终结构。

### 共指解析推理
- **问题构造**：对每对 mention $(m_i, m_j)$ 生成 Yes/No 问题"Does $m_i$ refer to $m_j$?"。
- **链接得分**：$s_{i,j} = f_{yes}(i,j) - f_{no}(i,j)$。
- **All-Link ILP**：
$$\max_{y}\sum_{i,j}y_{i,j}s_{i,j} \quad \text{s.t.} \quad y_{i,k} \geq y_{i,j}+y_{j,k}-1, \forall i,j,k; \quad y_{i,j}\in\{0,1\}$$
约束确保传递闭包成立，使用 Gurobi 求解器求解。

### 少样本扩展
通过在 prompt 前添加 in-context 示例实现 few-shot，推理框架保持不变。

## 实验与结果

### 数据集
- **SRL**：QA-SRL_wiki（约10.8K QA pairs）、QA-SRL 2.0（265K QA pairs）
- **共指解析**：ECB+、OntoNotes 5.0 English、GENIA（低资源生物医学领域，作者提供新划分的 train/dev/test split）

### 主要模型
T5-3B、Flan-T5-XL（3B）、Macaw-3B、GPT-4

### SRL 结果（QA-SRL 2.0）
| 系统 | Head_q(↑) | Head_s(↑) | ρ(↓) |
|---|---|---|---|
| T5-3B | 68.57 | 46.35 | 39.06 |
| T5-3B + constraints | 68.87 | **52.31** | **0** |
| Flan-T5-XL^itr + constraints | 71.97 | **55.41** | **0** |
| GPT-4 | 86.78 | 76.89 | 7.18 |

- 约束后 Head_s 从 46.35→**52.31**（+5.96），Flan-T5-XL^itr 达 **55.41**（+6.12），不一致率降至 0。
- GPT-4 无约束时仍有 7.18% 不一致率。

### 共指解析结果（最强）
| 数据集 | 系统 | F1(↑) | CoNLL(↑) | ρ(↓) |
|---|---|---|---|---|
| ECB+ | Flan-T5-XL + All-Link | **66.06** | **65.09** | 0 |
| OntoNotes | Flan-T5-XL + All-Link | 51.33 | 48.52 | 0 |
| GENIA | Flan-T5-XL + All-Link | **65.36** | **57.47** | 0 |

- Flan-T5-XL + All-Link 在 ECB+ 上 F1=66.06，相比无约束基线提升显著，且 ρ=0。

### 关键结论
- 约束模型在所有情况下将不一致率降至 **0%**。
- 小模型+约束的性能可匹敌更大模型的无约束版本。
- 少样本实验中，SRL_wiki 上5-shot约束系统超过零样本T5-3B约束。

## 相关工作脉络
1. **Liu et al. (2023)**：独立使用Prompt预测结构各组件，忽略交互——本文的核心对比基线，本文通过推理补充缺失的全局约束。
2. **Blevins et al. (2023)**：自回归方式生成 PoS/NER 标签——适用于可展平的序列结构，本文处理更复杂的 span 重叠约束和传递性约束。
3. **Täckström et al. (2015)**：传统 SRL 图搜索推理——本文沿用其框架但将训练好的分数替换为 LLM 的零样本打分。
4. **Chang et al. (2011) All-Link**：共指解析 ILP 推理——本文沿用其传递性约束建模，但用 LLM 替代传统 linker 得分。
5. **Rajaby Faghihi & Kordjamshidi (2024)**：并发工作同样在推理阶段强制一致性——但使用多种归一化函数融合多模型输出，本文使用单 LLM 评分。
6. **Pyatkin et al. (2020, 2023) QA-SRL**：将 SRL 转化为 QA 格式——本文在其 QA 形式基础上增加推理模块。

## 局限性与未来方向
1. **依赖手工/预定义问题生成**：框架假设任务可分解为独立子问题，问题模板需人工设计，未探索自动问题生成。
2. **标签偏差（Label Bias）问题**：LLM 可能对特定回答（如"Yes"）存在先验偏好，影响得分校准；论文仅在共指实验上粗略调查了 Macaw vs T5 的校准差异，未系统研究。
3. **大文档场景的上下文限制**：共指实验中仅使用含 mention pair 的句子作为上下文而非完整文档；虽然分析显示不同上下文风格结果相近，但未充分利用长窗口模型（如 LongFormer）。
4. **迭代提示在共指中的挑战**：由于传递性约束涉及三元组，难以在 prompt 中自然表达，限制了 iterative prompting 在共指任务上的应用。
5. **计算开销**：LLM 生成为主要耗时，推理步骤虽运行在 CPU 上（见 Appendix E），但在大数据集上生成时间较长。

## 研究启发与可借鉴点
1. **"LLM作为评分器+传统推理"范式**：无需微调即可将成熟的结构化推理算法复用于 LLM，可迁移至其他结构化任务（如依存句法分析、事件抽取）。
2. **约束编码为图/ILP**：SRL 的图建模和共指的 ILP 传递性约束提供了两种经典约束编码范式，可直接复用到 span-overlap、chain/transitivity 等场景。
3. **K 最短路径后处理策略**：Yen 算法返回 K 条路径后再筛选"完整结构"，避免了贪心搜索的次优问题，这一策略适用于任何需要全局最优组合的任务。
4. **零不一致率可作为新评测维度**：论文引入的 inconsistency percent (ρ) 指标值得在结构化预测评测中推广，尤其对零样本方法至关重要。
5. **小模型+约束 ≈ 大模型无约束**：Figure 4/5 的发现表明在资源受限场景下，设计合理的结构约束比单纯扩大模型规模更有效。

## 关键术语表
- **Structured Prediction**：预测相互依赖、受结构约束的多个标签的组合输出任务。
- **Prompt-based Methods**：通过自然语言提示引导 LLM 进行零/少样本预测的方法。
- **QA-SRL**：将语义角色标注转化为问答形式的数据集和标注范式。
- **Unary Potential**：因子里图中单个变量的局部概率，本文中等同于 LLM 对候选答案的打分。
- **All-Link Inference**：通过整数线性规划在共指解析中求解满足传递性约束的全局最优链接方案。
- **Inconsistency Percent (ρ)**：预测结构中违反任务约束的比例，本文中的核心一致性指标。
- **Iterative Prompting**：依次逐题提问并在后续 prompt 中累积之前答案的提示策略。

## 可复现要素
- **数据集**：QA-SRL_wiki、QA-SRL 2.0、ECB+、OntoNotes 5.0 均为公开数据集；GENIA 作者声明将发布自定义划分 splits，**论文未提供代码/仓库链接**。
- **模型**：T5-3B、Flan-T5-XL、Macaw-3B 均为开源模型；GPT-4 通过 API 调用（花费 $96.31）。
- **推理实现**：SRL 使用 Täckström et al. (2015) 算法 + Yen K-shortest path；共指使用 Gurobi 求解器（附录 D.2 给出 prompt 模板和 ILP 公式）。
- **关键超参**：beam size=20（每个问题生成 top-20 spans）；K=20（Yen 算法返回 top-20 路径）；few-shot 实验使用 3 个随机种子（42, 20, 1984）取平均。
