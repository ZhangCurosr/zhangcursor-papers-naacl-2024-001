---
title: "Multi-Scale-Prompt-Memory-Augmented-Model-for-Black-Box-Scen"
source: https://aclanthology.org/2024.naacl-long.98.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 20:11:06"
field: "黑盒小样本文本分类"
keywords: ["black-box few-shot", "multi-scale prompt", "memory-augmented model", "non-parametric scoring", "text classification", "in-context learning"]
innovations: ["提出多尺度实例级与类级双粒度记忆库，在无参数条件下实现高效的 few-shot 文本分类", "利用 KL 散度驱动的非参数评分模块融合多尺度多粒度知识，避免传统黑盒优化的高 API 成本"]
benchmarks: ["SST-2", "MPQA", "CR", "MR", "TREC", "RTE", "SUBJ", "AGNews", "CB", "MRPC", "DBPedia"]
---

# 论文速读：Multi-Scale Prompt Memory-Augmented Model for Black-Box Scenarios

## 一句话总结
论文提出了 MuSKPrompt，一种面向黑盒场景的多尺度知识提示记忆增强模型，通过实例级与类级多尺度知识存储与评分模块，实现仅需不到 10 次 API 调用即可获得与小样本微调竞争力相当且更稳定的文本分类性能。

## 研究问题与动机
- **黑盒场景下的高效小样本分类**：工业界 LLM 多以 API 形式提供，无法访问内部参数与梯度，如何在零训练参数条件下实现高效的 few-shot 文本分类是当前重要需求。
- **现有黑盒优化方法 API 调用成本过高**：BBT/BBTv2（CMA 进化）、RLPrompt/TEMPERA（强化学习）等方法需要上千至上万次 LLM 调用搜索最优 prompt，训练效率低、成本高，且容易过拟合验证集。
- **ICL 示例选择不稳定**：In-context learning 性能高度依赖上下文中演示样本的选择与排列，导致预测结果不稳定，且存在标签偏差（label bias）问题。
- **记忆模型在黑盒 few-shot 分类中尚未被充分探索**：kNN Prompting 等基于记忆的推理方法虽有效，但缺少对不同尺度知识的系统利用，难以充分挖掘深层与浅层语义表征。

## 核心贡献（创新点）
- **提出多尺度知识记忆增强架构 MuSKPrompt**：通过实例级与类级双粒度知识组织方式，结合多尺度 prompt，使 LLM 在无需梯度的条件下仍能获取丰富的任务知识。与 BBT/RLPrompt 等依赖黑盒优化的方法本质不同。
- **设计多尺度上下文示例构造策略**：以 $c=2^{m-1}$（$m\in\{1,\dots,M\}$）的规模从每类采样多样化的样例构造不同尺度的 prompt，使模型在不同层级上分别抽取浅层示例级知识与深层类级知识。与仅用单一长度提示的方法形成鲜明区别。
- **构建兼具实例级与类级存储的非参数化记忆库**：实例级 key 存储各训练样本在给定 prompt 下的输出分布 $k_i$；类级 key 为同类别内所有 $k_i$ 的均值，兼顾样本特异性与类别共性。相比 kNN Prompting 仅保留单尺度实例特征，信息维度更丰富。
- **提出 KL 散度驱动的轻量级评分模块**：在推理阶段通过 $D_{KL}(p_{test} \| k_i)$ 与 $D_{KL}(p_{test} \| k_j^{cls})$ 分别计算实例得分与类别得分，并通过加权平均与多尺度均值融合，无需额外学习任何参数。
- **零训练参数、少于 10 次 API 调用的高效推理**：每个实例在构建记忆库时仅需一次前向调用，推理阶段不依赖验证集或搜索循环，显著降低计算开销并保持可解释性与非参数特性。

## 方法详解
- **问题设定**：在 $K$-shot 设置下，从训练集 $\mathcal{D}_{train}=\{(x_i,y_i)\}_{i=1}^{K \times |\mathcal{V}|}$ 中抽取少数样本，给定测试样本 $x_{test}$ 与模板变换 $\pi(\cdot)$，在冻结的 LLM $\mathcal{L}$ 上进行分类，输出词汇分布 $p(v|x_{test})=\mathcal{L}(v|\mathcal{P},\pi(x_{test},*))$。
- **单尺度记忆库构建**：在每个尺度 $c$ 下，由每类选取 $c$ 个上下文样例拼接成 prompt $\mathcal{P}_c$。将每个训练样本 $x_i$ 与该 prompt 输入 LLM 得到实例级表示：
  $k_i=p(v|\mathcal{P}_c,x_i)$
  同时计算每类 $j$ 的类级表示：
  $k_j^{cls}=\frac{\sum_i k_i \cdot \mathbb{1}(y_i=j)}{\sum_i \mathbb{1}(y_i=j)}$
- **多尺度记忆库扩展**：令尺度维度为 $m$，对应各尺度的示例数为 $c\in\{2^0,2^1,\dots,2^{m-1}\}$，则第 $t$ 个尺度的实例表示为 $k_{m,i}=p(v|\mathcal{P}_c,x_i)$，存储为 $(k_{m,i},y_i)$ 键值对。多尺度将知识库的知识容量提升约 $m$ 倍。
- **非参数化评分模块**：
  - 实例级 KL 散度：$D_i^{ins}=D_{KL}(p_{test}\|k_i)=\sum_v p_{test}(v)\log\frac{p_{test}(v)}{p(v)}$
  - 实例得分（Top-k 归一化倒数）：$S_j^{ins}=\frac{\sum_{i\in \text{Top}^k(D^{ins})} 1}{\sum_{i\in \text{Top}^k(D^{ins})} D_i^{ins}}$，其中 $y_i=j$
  - 类级得分：$S_j^{cls}=\frac{1}{D_{KL}(p_{test}\|k_j^{cls})}$
  - 实例与类级融合（$\lambda$ 为类级权重）：
    $S=(1-\lambda)\frac{S^{ins}}{\|S^{ins}\|_1}+\lambda\frac{S^{cls}}{\|S^{cls}\|_1}$
  - 多尺度加权平均：$S=\sum_{i=0}^m d_i\cdot S_i$，默认均匀平均，最后取 $\arg\max_{y_i} S$ 作为预测类别。

## 实验与结果
- **数据集**：7 个主要数据集（SST-2、MPQA、CR、MR、TREC、RTE、SUBJ），外加 AGNews、CB、MRPC、DBPedia 四个扩展数据集；主要实验为 16-shot 设置。
- **基线方法**：Fine-tuning（GPT2-XL 全参数微调）、BBT、BBTv2、RLPrompt、ICL、Noisy Channel、TreePrompt、kNN Prompting。
- **主干模型**：GPT2-XL（1.5B 参数），额外扩展验证 OPT-2.7B 与 OPT-6.7B。
- **主要结果（16-shot，七数据集 AVG）**：
  - **MuSKPrompt AVG=79.5**，超越所有基线。
  - 超越 kNN Prompting（AVG=74.5）约 **5.0 个百分点**；超越 BBTv2（AVG=74.4）约 5.1 个百分点；超越 Fine-tuning（AVG=70.6）约 **8.9 个百分点**。
  - 单项最佳：SST-2 达到 **90.2**（标准差仅 0.8），显著优于 kNN Prompting 的 88.8 与 BBTv2 的 89.5。
  - 在 SUBJ、RTE、TREC 三数据集上略低于 Fine-tuning。
- **API 调用次数**：每个实例在构建记忆库时仅需 1 次前向，整体调用次数 **<10**，远低于 BBT/BBTv2/RLPrompt 的千次级调用。
- **效率分析**：在 SST-2 上仅用 Fine-tuning 3.1% 的数据量即可达到相近性能；在 CR 上使用 64 个样本接近 Fine-tuning 最优。
- **模型规模效应**：随着 GPT-2 从 124M 增至 1.5B，SST-2 从 59.1 升至 90.2；在 OPT-6.7B 上 SST-2 达 94.1，仍优于同规模 kNN Prompting。

## 相关工作脉络
- **BBT / BBTv2（Sun et al., 2022）**：基于 CMA 进化算法的黑盒连续 prompt 优化，需大量 API 调用；MuSKPrompt 完全不需要优化循环，以非参数记忆方式替代搜索过程。
- **RLPrompt / TEMPERA（Deng et al., 2022; Zhang et al., 2022a）**：采用强化学习在测试时优化离散 prompt；MuSKPrompt 避免了策略梯度/软 Q-learning 的高方差与高计算代价，同时保留可解释性。
- **kNN Prompting（Xu et al., 2022a）**：首个在黑盒场景引入非参数记忆模块的文本分类方法；MuSKPrompt 在其基础上进一步引入类级知识、多尺度尺度构造与 KL 评分策略，并在七数据集平均上提升约 5 个百分点。
- **PromptBoosting（Hou et al., 2023）**：通过 T5 生成 prompt pool 并 AdaBoost 集成，需要更大模型支撑；MuSKPrompt 直接利用 Few-shot 样本与前向分布，不依赖辅助生成模型。
- **Noisy Channel / ICL（Min et al., 2022a）**：基于噪声信道公式校准标签偏差的 ICL 变体；MuSKPrompt 通过记忆库中的实例与类级分布显式建模，不依赖条件概率反转。
- **TreePrompt（Singh et al., 2023）**：利用决策树进行零样本/少样本任务适配；MuSKPrompt 属于检索/记忆型方法，两者思路不同但均为无参数方案。

## 局限性与未来方向
- 在自然语言推理等更具挑战的任务（如 RTE、TREC）上仍落后于全参数 Fine-tuning，原因可能是缺乏梯度优化或 GPT-2-XL 本身推理能力受限。
- 当前评分模块未引入任何可学习参数，未来可探索在评分模块中进行轻量参数学习以提升上限。
- 实验主要集中在文本分类任务，尚未验证在 chain-of-thought、生成任务以及命名实体识别等任务上的有效性。
- prompt 模板由人工设计，依赖直觉；未来的层次化 prompt 或自动 prompt 设计有望进一步提升知识提取质量。
- 受限于 GPT-2-XL 最大序列长度（1024 tokens），多尺度维度的增加在部分数据集上被压缩至 $m=3$，限制了大规模多尺度的探索。

## 研究启发与可借鉴点
- **实例级 + 类级双粒度记忆组织**值得迁移：在其它黑盒/零样本任务中，可同时维护样本分布与类分布，提升鲁棒性。
- **多尺度上下文示例构造策略**具有通用性：以指数增长的示例数量分层构造 prompt，能够有效缓解 few-shot 下知识容量不足的问题，可推广至NER、机器翻译等任务。
- **KL 散度替代余弦相似度作为非参数评分**：在分布对齐任务中，KL 更能反映语言模型输出差异，值得在检索增强生成、知识编辑等场景中尝试。
- **API 调用次数与数据利用效率**的评测指标设计可作为后续研究的对比基线，为黑盒场景下的资源敏感型应用提供明确参考。
- 本文强调“无验证集”的设计思路，与当前常见的 search-based 黑盒方法形成对照，可作为轻量级、可部署方案的设计原则。

## 关键术语表
- **Black-box scenario（黑盒场景）**：指无法访问 LLM 内部参数与梯度、仅能通过 API 前向查询获取输出的应用场景。
- **Few-shot text classification（小样本文本分类）**：仅利用每类极少（如 16 个）训练样本完成文本分类任务。
- **Multi-scale prompt（多尺度提示）**：通过从每类选取不同数量的上下文示例构造的 prompt，形成浅层与深层知识的双粒度表示。
- **Instance-level knowledge（实例级知识）**：记忆库中针对每个训练样本存储的输出分布 $k_i$，刻画样本特有的语义表示。
- **Class-level knowledge（类级知识）**：同一类别下多个实例分布的均值 $k_j^{cls}$，反映类别层面的通用语义特征。
- **KL divergence scoring（KL 散度评分）**：利用 $D_{KL}(p_{test}\|k)$ 衡量测试样本分布与记忆库中已知分布的差异并用于分类。
- **Non-parametric memory model（非参数记忆模型）**：不引入任何可训练参数的记忆结构，仅通过检索与分布匹配完成推理。
- **API call（API 调用次数）**：黑盒场景下衡量 LLM 推理效率的核心指标，本文方法控制在每次推理前少于 10 次。

## 可复现要素
- **数据集**：SST-2、MPQA、CR、MR、TREC、RTE、SUBJ、AGNews、CB、MRPC、DBPedia；多为公开数据集，论文采用与 kNN Prompting 相同的 16-shot 划分与 5 个随机种子。
- **代码/权重**：论文未提及开源代码与权重（仅引用了官方实现用于基线复现）。
- **关键超参**：$\lambda=0.5$（类级权重）；默认尺度维度 $m=4$（部分数据集 $m=3$）；各尺度示例数 $c=2^{m-1}$；Top-k 与比例未在正文明确声明，需参考附录。
- **模型与硬件**：GPT2-XL（1.5B）为主干；单卡 NVIDIA RTX A6000；优化器/学习率等细节参见附录 C。
