---
title: "Found-in-the-Middle-Permutation-Self-Consistency-Improves-Li"
source: https://aclanthology.org/2024.naacl-long.129.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 20:09:50"
field: "信息检索与大语言模型排序"
keywords: ["listwise ranking", "positional bias", "self-consistency", "Kemeny ranking", "retrieval", "LLM decoding"]
innovations: ["置换自洽性：通过随机打乱输入顺序采样多组排名并用Kemeny聚合消除位置偏差", "理论证明Kemeny聚合在满足一致子集条件下收敛到真实排名", "系统刻画GPT-3.5/GPT-4在passage reranking中的位置偏置模式（反转率热力图）"]
benchmarks: ["MathSort", "WordSort", "GSM8KSort", "TREC-DL19", "TREC-DL20"]
---

# 论文速读：Found-in-the-Middle-Permutation-Self-Consistency-Improves-Li

## 一句话总结
本文提出**置换自洽性（Permutation Self-Consistency, PSC）**解码策略，通过随机打乱输入列表顺序后多次调用黑盒 LLM，再用 Kemeny 排序聚合，有效缓解 listwise ranking 中的位置偏差，在排序和段落重排序任务上显著提升 GPT-3.5/4、LLaMA v2、Mistral 等模型性能。

## 研究问题与动机
- **位置偏差影响 listwise ranking 质量**：LLM 存在"lost in the middle"现象（Liu et al., 2023），对列表中中间部分的条目利用不充分；Wang et al. (2023a) 也发现 prompt 顺序显著影响输出质量。
- **现有方法不足**：现有 LLM 重排序方法（RankGPT、RankVicuna 等）采用滑动窗口逐段处理，窗口策略本身可能引入偏差，且无法消除位置偏置对最终排序结果的影响。
- **自洽性框架的局限**：原自洽性（Self-Consistency, Wang et al., 2023b）依赖温度采样产生多样推理路径，但 listwise ranking 不涉及多路径推理，温度变化对其效果无效（本文实验证实）。
- **缺乏对位置偏差的系统分析**：此前工作虽有关注 prompt 顺序的影响，但未见对 passage-ranking LLM 位置偏置与成对位置关系的系统刻画。

## 核心贡献（创新点）
1. **提出置换自洽性（PSC）解码策略**：通过随机置换输入列表顺序生成多组排序结果，再聚合为与所有输出 Kendall tau 距离最小的中心排名，本质上将"推理路径多样性"替换为"位置偏差多样性"。与自洽性通过温度采样探索不同推理路径的本质不同，PSC 通过干预输入顺序打破位置与输出的关联。
2. **给出理论保证**：证明在满足一定噪声分布假设下，Kemeny 排序可收敛到真实排名，PSC 是真实排名的相容估计量（Proposition 2.1/2.2）。相比 Borda 计数等启发式聚合方法，Kemeny 优化具有最优性理论支撑。
3. **系统性刻画 LLM 位置偏置模式**：在 BM25 检索场景下，分析 GPT-3.5/GPT-4 对不同位置对的重排反转率，揭示 GPT-3.5 对中后部条目关注不足、GPT-4 在末尾位置更易反转的模式，解释 PSC 在不同模型上的增益差异。
4. **全面实验验证**：在 MathSort、WordSort、GSM8KSort、TREC-DL19/DL20 五个数据集和七个模型上验证，PSC 在排序任务平均提升 51%（Kendall tau），在重排序任务上达到新的 SOTA（nDCG@10）。

## 方法详解
**核心流程（两阶段）：**
1. **采样阶段**：给定输入序列 X 和固定指令 prompt s，对 X 进行随机置换 πᵢ（均匀采样自所有 n! 种排列），送入 LLM 得到 m 个输出排名 σ̂ᵢ = h(X[πᵢ]; s)。每个输出因位置不同而产生不同的位置偏差，但错误模式相互独立。
2. **聚合阶段**：采用 Kemeny–Young 最优排序，求解：
   σ̄ = argmin_σ Σᵢ d_κ(σ̂ᵢ, σ)
   其中 d_κ 为 Kendall tau 距离（即成对不一致的数目）。该中心排名最小化与所有采样排名的总距离，从而"边际化"掉各次调用的位置偏置。

**段落重排序适配**：因 LLM 上下文长度限制，RankGPT/RankVicuna 等方法对 top-k 列表采用滑动窗口从尾到头遍历。PSC 对每个窗口独立应用置换聚合。

**理论性质**：
- Proposition 2.1：若每个噪声排名与真排名存在一个非空一致子集且其余元素为随机排列，则当 m→∞ 时，Kemeny 排序以概率 1 收敛到真排名（Hoeffding 不等式证明）。
- Proposition 2.2：即使噪声分布固定非随机，只要任意置换输入下均满足上述一致子集条件，PSC 仍是相容估计量。

## 实验与结果
**数据集与基线：**
- **排序任务**：MathSort（10 个算术表达式排序）、WordSort（10 个单词字母序）、GSM8KSort（GSM8K 句子还原），各 100 例，用 Kendall tau 评估。
- **重排序任务**：TREC-DL19/DL20（MS MARCO 语料），top-20/top-100 由 BM25/SPLADE++ ED 检索，用 nDCG@10 评估。
- **模型**：Mistral-7B、Zephyr_β-7B、LLaMA v2 (7B/13B/70B)、GPT-3.5 Turbo、GPT-4；基线包括 RankGPT、RankVicuna、PRP-Best、MonoT5、RankT5、RankLLaMA 等。

**主要结果：**
- 排序任务（Table 3）：PSC 在所有 3 数据集×7 模型组合上持续超越常规推理，平均提升 51%（Kendall tau）。LLaMA₂-7B/13B/70B 分别提升 157%/28%/12%，Mistral 提升 42%，Zephyr 提升 106%，GPT-3.5 提升 3–18%，GPT-4 提升 2–7%。增益与原始质量负相关（r = −0.72）。
- 重排序任务（Table 4）：PSC 在 13/16 组合上提升。关键结果：Single (GPT-4) on DL19 达 76.87（+1.28 over RankGPT-GPT4）；Single (GPT-4) on DL20 达 78.52（+3.79 over RankVicuna）；RankGPT (GPT-4) on DL19 达 75.66，DL20 达 71.00。
- 超参分析（Figure 5）：m=5 可达 m=20 平均 67% 的收益，超过 5–10 后边际收益急剧下降；采样温度对质量几乎无影响（ρ = −0.078）。
- 聚合方法对比（Figure 6）：Kemeny 在 8/10 比较中显著优于 RRF（p<0.05），RRF 仅达 Kemeny 增益的 93.5%。

## 相关工作脉络
1. **Self-Consistency（Wang et al., 2023b）**：通过温度采样+多数投票聚合 CoT 推理路径。PSC 借用其 shuffle–aggregate 范式，但针对 listwise ranking 替换为置换采样和 Kemeny 聚合，且不依赖温度参数。
2. **RankGPT（Sun et al., 2023）/ RankVicuna（Pradeep et al., 2023）**：用 LLM 做零样本 listwise 重排序，采用滑动窗口策略。PSC 可无缝叠加于这些方法之上提升其输出质量。
3. **PRP（Qin et al., 2023）**：pairwise 提示排序方法，需 20–200 次串行调用。PSC 完全并行且仅需 5–20 次调用，计算效率更高。
4. **Contrast-Consistent Ranking（Stoehr et al., 2023）**：训练 order-agnostic probe 检测排名一致性，但评估时泄露答案方向，不适用于标准评估场景。
5. **Positional Bias 研究（Liu et al., 2023; Wang et al., 2023a）**：前者发现"lost in the middle"现象，后者发现 prompt 顺序影响 QA/eval 质量。本文首次系统刻画 passage-ranking 中位置偏置与成对位置的关系。
6. **Bootstrapping/Borda Count 聚合（Hou et al., 2023）**：启发式求和排名法。本文证明 Kemeny 在理论上更优，实验上 PSC 显著优于 Borda 计数（Table 9）。

## 局限性与未来方向
- **计算成本**：需多次 LLM 调用（建议 5–20 次），对商业 API 产生较高费用（估算 GPT 结果复现约 $100–200）。但所有调用完全并行，实际延迟增加不超过 25%。
- **闭源模型黑盒性**：GPT-3.5/4 内部机制未知，偏差模式分析依赖外部观测而非模型内省。
- **自动化评估局限**：Kendall tau 和 nDCG@10 不能完全捕捉人类偏好，在实际搜索引擎/推荐系统中的效果尚待验证。
- **未来方向**：将 PSC 改造为可微分形式以支持训练时应用（如 RankVicuna 的微调阶段）；扩展至 LLM 评估、人工反馈标注等其他 list-oriented 任务；与提示工程（如 Pezeshkpour & Hruschka, 2023）互补使用。

## 研究启发与可借鉴点
1. **位置偏差解耦的思路**：通过随机置换输入顺序将位置偏置转化为独立噪声源，再用聚合消除——此范式可迁移到任何受顺序影响的 LLM 输出任务（如序列标注、多文档摘要）。
2. **Kemeny 排序作为聚合器的优越性**：相比多数投票或 Borda 计数，Kemeny 优化 Kendall tau 距离具有理论最优性保障，且在 8/10 比较中显著优于 RRF，值得在排名聚合场景优先考虑。
3. **温度参数对 listwise 任务无效**：暗示 listwise ranking 不涉及"多路径推理"，未来设计自洽类方法时应根据任务特性选择多样性来源（如本题的输入置换 vs. 原方法的温度采样）。
4. **位置偏置可视化分析方法**：通过计算不同输入位置对的"反转率"热力图，可直观诊断模型的位置敏感模式，该分析框架可用于其他位置敏感任务（如 long-context QA）的模型诊断。
5. **m=5 即可获得 m=20 的 67% 收益**：表明在实际应用中可用更少调用次数换取显著性价比提升，对部署约束场景有直接参考价值。

## 关键术语表
**Permutation Self-Consistency (PSC)**：通过随机置换输入列表顺序多次调用 LLM 获取多组排序，再用 Kemeny 聚合得到去偏的中心排名。

**Kemeny–Young Ranking**：最小化与所有输入排名 Kendall tau 距离之和的最优中心排名，计算上为 NP-hard 但有高效近似算法。

**Kendall tau 距离**：两个排名之间成对不一致（discordant pair）的数量，取值范围 0 到 n(n−1)/2。

**Positional Bias / Lost in the Middle**：LLM 对输入序列不同位置的处理能力不均，中间部分信息利用效率下降的现象。

**Listwise Ranking**：将待排序项作为整体输入，由模型一次性输出完整排序结果，区别于 pointwise/pairwise 方法。

**Concordant Subset**：两个排名中保持相同相对顺序的元素子集，PSC 理论保证要求每次采样至少存在一个非空一致子集。

**Reversion（反转）**：在位置偏置分析中，指 LLM 输出中某对元素相对顺序与其输入位置顺序相反的情况。

**Reciprocal Rank Fusion (RRF)**：信息检索中经典的排名融合方法，按 Σ 1/(k+rankᵢ) 排序，本文作为 Kemeny 的对比基线。

## 可复现要素
- **数据集**：自行构造的 MathSort、WordSort、GSM8KSort（各 100 例）；TREC-DL19/DL20（公开基准）。
- **代码**：已开源，https://github.com/castorini/perm-sc
- **模型**：开放模型（Mistral-7B、Zephyr_β-7B、LLaMA v2 7B/13B/70B、RankVicuna）本地推理；GPT-3.5 Turbo (0613) 和 GPT-4 通过 Azure API 调用。
- **关键超参**：采样次数 m=20（敏感性分析显示 m=5 即可达主要增益）；采样温度 T=0（无效）；Kemeny 聚合；重排序 top-k=20 或 100。
- **实现细节**：使用 FlashAttention v2 和 BF16 加速；环境为 Ubuntu 22.04 + 2×A6000 GPU + 256GB RAM；PyTorch 2.1.0 + Transformers 4.36.1 + PuLP 2.7.0。
