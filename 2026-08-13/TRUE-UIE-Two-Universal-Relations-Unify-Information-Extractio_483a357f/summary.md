---
title: "TRUE-UIE-Two-Universal-Relations-Unify-Information-Extractio"
source: https://aclanthology.org/2024.naacl-long.103.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 21:34:28"
field: "信息抽取与结构化预测"
keywords: ["Universal Information Extraction", "Token Pair Linking", "IS Relation", "NEXT Relation", "Structure Language Prompt", "Semi-Matrix BiLSTM", "Zero-shot Transfer"]
innovations: ["用IS和NEXT两个通用关系统一所有IE任务的学习目标", "结构语言提示将schema信息前置到prompt中", "Semi-Matrix BiLSTM显式建模span内部token序列依赖"]
benchmarks: ["CoNLL03", "ACE04", "ACE05", "CoNLL04", "NYT", "SciERC", "SemEval-14/15/16", "CADeC", "SAOKE", "CASIE", "GENIA"]
---

# 论文速读：TRUE-UIE: Two Universal Relations Unify Information Extraction Tasks

## 一句话总结
TRUE-UIE 提出仅用 **IS** 和 **NEXT** 两个通用关系，将所有信息提取任务统一为一个共同学习目标，在 16 个数据集、7 类 IE 任务上达到 SOTA，并在零样本/少样本迁移中显著优于现有通用 IE 模型。

---

## 研究问题与动机
1. **现有 UIE 方法缺乏统一学习目标**：生成式方法（如 UIE）为不同任务生成差异化的结构化语言，链接式方法（如 USM）中同一关系符号在不同任务中承担不同角色，导致知识无法充分共享。
2. **关系定义不一致制约跨任务泛化**：USM 中黄色虚线与实线箭头定义相同但用途不同（NER vs RE），绿色/蓝色关系仅在 RE 中训练，NER 任务完全未接触。
3. **复杂 IE 任务处理能力不足**：不连续 NER、重叠关系抽取、开放信息提取等任务在现有主流链接式方法中难以有效建模。
4. **span 内部序列依赖被忽视**：现有 span 提取模型通常仅关注 span 的首尾 token，未充分利用 span 内部 token 的序列依赖信息。

---

## 核心贡献（创新点）
1. **提出 IS 与 NEXT 两个通用关系统一所有 IE 任务**：IS 负责 span 与类型占位符的对齐（类型识别），NEXT 负责连接同一知识实例中的相邻 span（分组），所有任务共享同一套关系定义。
2. **设计结构语言提示（Structure Language Prompt）**：将 schema 的结构化信息融入 prompt（如 `<subject type> <relation type> <object type>`），替代 USM 单独枚举 entity/relation 类型的方式，使 prompt 更接近自然语言表达式。
3. **在 span 特征中显式建模 token 序列依赖**：通过 Semi-Matrix BiLSTM 高效嵌入 span 内部全部 token 的序列信息，弥补现有模型仅关注首尾 token 的不足。
4. **统一的类不平衡损失函数**：针对 IS 关系远多于 NEXT 关系的类别不平衡问题，沿用 ZLPR loss 进行优化。

---

## 方法详解

### 整体架构
将 **结构语言提示**（含 schema 信息）与 **输入文本** 拼接后送入编码器（BERT/RoBERTa/XLM-R），获取隐藏状态后经过两个全连接层分别得到 begin 特征 $h_j^b$ 和 end 特征 $h_j^e$，再并行送入 **Semi-Matrix BiLSTM**（span 评分）和 **乘法注意力模块**（关系评分）。

### 通用链接方案
- **IS 关系**：将文本 span 与 prompt 中的类型占位符对齐，完成 span 类型识别。
- **NEXT 关系**：连接同一知识实例中当前 span 与其后续 span（如三元组中的主体→客体、事件触发词→论元序列），完成分组。

### 各任务 prompt 设计
- **关系抽取**：`<subject type> <relation type> <object type>`，主体与客体 span 通过 NEXT 连接，且分别 IS 到 subject type 和 object type，共同 relation type 处完成类型判定。
- **情感抽取**：`aspect <polarity>`，aspect span 与 opinion span 经 NEXT 连接，各自 IS 到 aspect 和 polarity 占位符。
- **事件抽取**：`<event type>: [argument role1, argument role2, ...]`，trigger 也视为论元；沿 NEXT 路径从边界到边界输出各事件实例，自然解决论元重叠。
- **嵌套/不连续 NER**：对链接到实体类型的长 span，检查其内部是否存在由更短 span 经 NEXT 连接的连续路径——有则输出为不连续实体，无则输出为连续实体；嵌套时长短 span 同时保留。
- **开放信息提取**：role 成对识别（`<role1> <role2>`），两个 span 经 NEXT 连接且分别与附近 role 占位符 IS 对齐，避免单一 role 链接导致的角色重叠。

### 序列依赖建模（Semi-Matrix BiLSTM）
对 $n$ 个 token 构建两个 $n \times n$ 矩阵 $B$ 和 $E$（每行重复向量），前向 LSTM 编码 $E$ 的上三角区，后向 LSTM 编码 $B$ 的下三角区，得到 $B'$ 和 $E'$；$S = B'^T + E'$ 的上三角区即为 span $(i,j)$ 的序列特征，包含从 $i$ 到 $j$ 及从 $j$ 到 $i$ 的双向序列信息。Span 评分：$s_{i,j}^p = W_s \cdot S_{i,j} + b_s$。

### 关系评分
采用乘法注意力融合 span 边界特征：$s_{i,j}^* = h_i^* \cdot h_j^{*T}$，其中 $* \in \{b, e\}$。

### 损失函数（类不平衡）
$$L = \sum_{t \in T} \log\left(1 + \sum_{(i,j) \in t^+} e^{-s_{(i,j)}^*}\right) + \log\left(1 + \sum_{(i,j) \in t^-} e^{s_{(i,j)}^*}\right)$$

---

## 实验与结果

### 数据集与任务
覆盖 **7 类 IE 任务**、**16 个公开基准数据集**：
- 平面 NER：CoNLL03、ACE04、ACE05-Ent、GENIA
- 关系抽取：CoNLL04、NYT、SciERC、ACE05-Rel
- 事件抽取：ACE05-EvtT/EvtA、CASIE_T/A
- 情感抽取：SemEval-14/15/16-res、Cadec/Cadec_D
- 不连续 NER：Cadec_D
- 开放信息提取：SAOKE

预训练数据：$D_{task}$（Ontonotes 60K）、$D_{dist}$（远距离监督 356K）、$D_{ind}$（MRQA 等阅读理解 195K）。

### 主要结果（ supervised）
TRUE-UIE 在全部 16 个数据集上均达到 SOTA 或接近 SOTA：
- CoNLL03：TRUE† **94.13**（USM† 93.16）
- ACE04：TRUE" **89.91**（P-NER 88.72）
- ACE05-Ent：TRUE† **90.10**（USM† 87.14）
- CoNLL04：TRUE" **78.94**（USMu 78.84）
- NYT：TRUE" **94.83**（USM† 94.07）
- ACE05-EvtT：TRUE" **76.42**（PFN⁺ 73.44）
- Cadec：TRUE" **73.83**（W²NER 73.21）
- SemEval-16-res：TRUE" **78.83**（USM† 78.25）
- SAOKE：TRUE" **47.11**（TRUE* 46.51，多任务微调后较单任务提升 +0.6，体现跨语言泛化）

### 零样本迁移
- NER 零样本：在 4 种预训练配置下均显著优于 USM，$D_{dist}$ 和 $D_{task,ind,dist}$ 配置下平均提升分别达 **+2.9** 和 **+3.2** 个百分点。
- RE 零样本：TRUE-UIE（374M 参数）以 **27.13** F1 超越 GPT-3（175B，18.10）和 DEEPSTRUCT（10B，25.80），略高于 USM（356M，25.95）。

### 少样本迁移
- TRUE-UIE 在前 5 个数据集上平均提升 **6.29**（vs UIE）和 **1.17**（vs USM）。
- 后 3 个复杂任务（Genia、Cadec_D、SAOKE）较未预训练的 TRUE* 平均提升 **14.46**，证明预训练知识可有效迁移至新任务。

### 消融实验
- 移除序列依赖（Seq Dep）：所有任务均有显著下降，验证其必要性。
- 移除结构语言提示与双关系设计（w/o SLP & TUR）：RE 和 EVT 下降明显（Rel: 68.91→66.48，Evt-T: 73.12→71.97），NER 基本不变（因 prompt/linking 风格与 USM 相似）。

---

## 相关工作脉络
1. **UIE（Lu et al., 2022）**：生成式 UIE，使用结构化提取语言（SELang）统一表达各任务。本文认为其各任务生成结构不一致（如 NER 无嵌套括号、RE/Event 有多层嵌套），学习目标不统一。
2. **USM（Lou et al., 2023）**：链接式 UIE，提出三种统一 token 链接操作。本文指出其关系定义在不同任务中用途不同（同符号不同含义），且部分关系仅在特定任务中训练，知识共享受限。
3. **UTC-IE（Yan et al., 2023）**：将 IE 任务分解为 token 对分类（start-start、end-end、start-end）。本文认为其仍未实现统一学习目标。
4. **UniEX（Ping et al., 2023）**：统一抽取框架，将任务分解为联合 span 检测、分类和关联。本文同样指出其目标未完全统一。
5. **TPLinker（Wang et al., 2020）**：早期 token pair linking 方法，本文在其基础上引入序列依赖建模和通用关系统一设计。
6. **REBEL（Huguet Cabot & Navigli, 2021）**：端到端生成式关系抽取，本文在开放 IE 实验中作为对比基线。

---

## 局限性与未来方向
1. **结构语言提示在重复类型场景下可能引发混淆**：当同一 entity type（如 "people"）在不同 triplet scheme 中重复出现时，模型可能过度泛化，导致高召回低精确率（Appendix D）。
2. **解决方案已部分提出**：采用 attention mask 策略限制模型仅关注 scheme 组内的文本（如图 4 所示），但需进一步探索是否引入额外复杂度。
3. **未来方向**：可进一步验证该方法在其他复杂 NLP 结构化预测任务中的适用性，以及探索更多通用关系设计的组合。

---

## 研究启发与可借鉴点
1. **"两个通用关系统一所有任务"的设计范式**：用最少抽象关系覆盖最多任务，避免了生成式方法的结构膨胀和链接式方法的关系碎片化，思路可迁移至问答、摘要等其他结构化预测任务。
2. **Semi-Matrix BiLSTM 序列依赖建模**：显式建模 span 内部全 token 序列信息，相比仅用首尾 token 的 approach 带来稳定提升，可复用于任何 span-based 抽取任务。
3. **结构语言提示将 schema 知识前置到 prompt**：减轻模型学习负担，促进跨任务知识迁移，可与其他大模型提示工程方法结合。
4. **跨语言零样本泛化发现**：英语多任务微调后在中文 SAOKE 上仍有提升（+0.6），暗示通用关系设计有助于跨语言迁移，值得在更多语种对中验证。
5. **类不平衡损失在链接式任务中的适配**：IS 关系远多于 NEXT 关系的不平衡现象在各类 IE 任务中普遍存在，该 loss 设计可直接复用。

---

## 关键术语表
**TRUE-UIE**：Two Universal Relations Unify Information Extraction，本文提出的通用信息提取框架。
**IS 关系**：将文本 span 与 prompt 中类型占位符对齐的通用关系，用于 span 类型识别。
**NEXT 关系**：连接同一知识实例中当前 span 与后续 span 的通用关系，用于分组结构化知识。
**Structure Language Prompt（SLP）**：将 schema 结构化信息嵌入 prompt 的设计（如 `<subject type> <relation type> <object type>`），替代传统单独枚举实体/关系类型的方式。
**Semi-Matrix BiLSTM**：通过在 $n \times n$ 矩阵的上/下三角区分别应用前向/后向 LSTM，高效编码 span 内全部 token 的序列依赖信息。
**Token Pair Linking**：将 IE 任务转化为 token 对分类/链接问题的方法，本文在此基础上扩展为通用关系统一范式。
**ZLPR Loss**：Class Imbalance Loss，用于处理 IS 与 NEXT 关系数量不平衡的训练目标。
**Multi-task Fine-tuning（USM†/TRUE†）**：在多任务（除重叠数据集外）上联合微调的模型版本。

---

## 可复现要素
- **数据集**：16 个公开 benchmark 数据集（ACE04/05、CoNLL03/04、GENIA、Cadec、NYT、SciERC、CASIE、SemEval-14/15/16、SAOKE），均为公开研究数据。
- **预训练数据**：$D_{task}$（Ontonotes 60K）、$D_{dist}$（REBEL 300K）、$D_{ind}$（HotpotQA/Natural Questions/NewsQA/SQuAD/TriviaQA 195K）、中文 SAOKE 对应 Wikidata-Wikipedia 远距离监督数据。
- **代码/权重**：论文未明确声明开源仓库，需查阅 ACL Anthology 页面确认（论文未提及）。
- **关键超参**：预训练学习率 $2 \times 10^{-5}$，batch size 96，epochs 5；微调学习率 $\{1,2,3,4,5\} \times 10^{-5}$，batch size $\{8,12,16,32,64,96\}$；Adam 优化器；3 次随机种子取平均。
- **模型 backbone**：英文任务用 RoBERTa-large，SAOKE 用 XLM-RoBERTa-large；参数量约 374M。

---
