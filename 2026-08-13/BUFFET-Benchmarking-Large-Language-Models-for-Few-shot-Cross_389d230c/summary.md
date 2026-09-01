---
title: "BUFFET-Benchmarking-Large-Language-Models-for-Few-shot-Cross"
source: https://aclanthology.org/2024.naacl-long.100.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:12:08"
field: "跨语言自然语言处理"
keywords: ["少样本迁移", "跨语言迁移", "上下文学习", "多语言大语言模型", "基准评测", "指令微调"]
innovations: ["提出 BUFFET 统一基准，将 15 个任务与 54 种语言统一为固定 k-shot text-to-text 格式进行少样本跨语言迁移评测", "揭示强大多语言 LLM 的 ICL 在少样本跨语言设置下显著落后于小参数微调模型，尤其低资源语言", "发现指令微调模型（如 mT0）在少样本 ICL 设置下出现性能退化，可能因过度拟合零样本指令训练"]
benchmarks: ["BUFFET", "BUFFET-Light", "XTREME", "XTREME-R", "MEGA"]
---

# 论文速读：BUFFET-Benchmarking-Large-Language-Models-for-Few-shot-Cross-lingual-Transfer

## 一句话总结
本文提出了 BUFFET，一个统一格式的少样本跨语言迁移基准，涵盖 15 个任务与 54 种语言；实验发现强大多语言 LLM（如 ChatGPT）在少样本上下文学习中仍显著落后于更小的 mT5-base 微调模型，尤其在低资源语言上，且指令微调模型对少样本示例的使用存在明显退化现象。

## 研究问题与动机
- **核心问题**：现有少样本学习研究主要集中在英语，缺少对多语言环境下"少样本跨语言迁移"的公平、统一评测框架。
- **现有不足 1**：已有基准（XTREME、XTREME-R、XGLUE 等）聚焦零样本迁移，缺少在相同设置下比较微调与上下文学习（ICL）的方法；且在少样本设置下性能方差较大，不利于公平对比。
- **现有不足 2**：多语言基准多关注高/中资源语言，或使用翻译数据，存在翻译偏差与标注质量问题，缺乏面向低资源语言的原始标注数据集。
- **关键研究问题**：（RQ1）ICL 与微调在少样本跨语言迁移中是否可比？（RQ2）不同方法在不同任务和语言上的表现如何？（RQ3）演示样本与指令的选择如何影响迁移效果？

## 核心贡献（创新点）
- **提出 BUFFET 统一基准**：将 15 个多样任务统一为 text-to-text 格式，提供固定的 k-shot 示例集，相比 CrossFit 等更强调跨语言少样本公平比较。
- **覆盖 54 种语言与多样化任务类型**：包含分类、生成、抽取和结构化预测，36/54 为非印欧语言，并纳入土著语言与非洲语言等低资源语言，弥补以往基准的语言覆盖缺陷。
- **提供固定 k-shot 演示以避免方差**：每个任务提供三组不同的 k-shot 训练/验证集，取平均结果，缓解 Zhao et al. (2021) 指出的少样本性能高方差问题。
- **系统性评估六种 LLM 与多种迁移策略**：在统一框架下对比了 mT5-base、BLOOM、BLOOMZ、mT0、ChatGPT 以及 Llama/Mistral 等英文中心模型的微调与 ICL 表现，揭示了若干关键发现（详见实验）。
- ** BUFFET-Light 子集**：在保持任务与语言多样性的同时减少 66% 评估设置，为资源受限场景提供高效替代方案。

## 方法详解
- **统一文本到文本格式**：所有任务按 SuperNaturalInstructions/PromptSource 的指令模板统一为 input-output 形式，模型直接生成答案，无需任务特定修改。
- **指令与翻译**：使用英文指令，并通过 NLLB 机器翻译为 54 种目标语言，另手动翻译 5 种语言；每个任务从原始训练集随机采样三组 k-shot 示例（不同任务 k 值不同，见 Table 1）。
- **六种迁移方法**（Table 2）：
  - **微调类**：TARGET FT（仅在目标语言 k-shot 上微调）、ENGLISH FT（仅在英语全量数据上微调后零样本迁移）、ENG.+TGT. FT（先英语微调再目标语言 k-shot 微调）。
  - **ICL 类**：ENGLISH ICL（英文指令+目标语言 k-shot 演示）、TARGET ICL（目标语言指令+目标语言 k-shot 演示）、Z-EICL（仅英文指令，零样本）。
- **评估指标**：分类任务用 accuracy，QA/NER 用 F1，生成任务用 ROUGE/BLEU；每个任务取三类 k-shot 的平均值，再做宏平均，最终汇总为 Avg. class score 与 Avg. gen score。

## 实验与结果
- **数据集与基线**：BUFFET-Full 含 15 个数据集/54 种语言；BUFFET-Light 选取 31 种语言做快速评估。模型包括 mT5-base（580M）、BLOOM-7B、BLOOMZ-7B、mT0-xxl（13B）、ChatGPT（gpt-3.5-turbo-0301），以及 Llama1/2/Mistral 等英文中心模型。
- **主要结果**（Table 3，BUFFET-Light）：
  - **微调模型普遍优于大模型 ICL**：mT5 ENG.+TGT. FT 在多项判别任务（如 PAWS-X 77.8%、AMAZON 91.0%）上显著超过 BLOOM/BLOOMZ/mT0 的 ENGLISH ICL；ChatGPT ENGLISH ICL 在 NLI（54.5%）和 XCOPA（76.7%）上表现突出，但仍低于微调模型在 PAWS-X（68.6% vs 77.8%）和 NER（45.4% vs 47.8%）上的结果。
  - **低资源语言差距显著**：图 2 显示 mT5 ENGLISH FT 和 ChatGPT ENGLISH ICL 在高资源语言上表现良好，但在 Aymara 等低资源语言上 NLI 仅略高于随机基线（~33%）；而 TARGET FT 可带来大幅提升（如 Hausa 上 MASAKHANER +30%）。
  - **英文中心模型高资源表现强但低资源掉点严重**：Mistral 7B 在非指令微调模型中综合最佳，但仍在 AMERICASNLI、INDIC SENTIMENT 等任务上大幅落后。
  - **指令微调模型的少样本退化**：mT0 在 ENGLISH ICL 下比 Z-EICL 表现更差（如 XNLI 32.6% vs 48.5%），说明其在少样本设置下未能有效利用演示样本，可能因训练时过度拟合零样本指令格式。
  - **生成任务上大模型占优**：在 TYDIQA-QG 和 XCOPA 等生成/常识推理任务上，ChatGPT 和 mT0 的 ICL 超过 mT5 微调，尤其是当英语源数据规模有限时。
  - **k 值敏感性**：图 4 显示 ICL 对 k-shot 选择比微调更敏感（如 AMAZON REVIEW 上 BLOOM ENGLISH ICL 标准差 2.2，mT5 ENG.+TGT. FT 仅 0.2）；约 49.7% 的情况下 BLOOM 和 BLOOMZ 的最优 k-shot 集不同。
  - **模型缩放**：BLOOM 560M→1B→7B，ICL 性能随规模显著提升（附录 Figure 12），证实 ICL 跨语言迁移可受益于 scale。
  - **手动生成评估**：Telugu 上 BLOOMZ 语法正确率仅 56.0%、质量 40%，Llama2 为 0%，说明生成任务在低资源语言上仍是巨大挑战。

## 相关工作脉络
- **XTREME/XTREME-R/XGLUE**：聚焦零样本跨语言迁移评测，主要评估微调模型在英语训练后的零样本泛化；BUFFET 首次系统化对比微调与 ICL 在少样本场景下的跨语言迁移，并强调低资源语言覆盖。
- **CrossFit**：英文中心的少样本多任务基准，缺少多语言和跨语言维度；BUFFET 将其扩展为 54 种语言、15 类任务的统一少样本跨语言基准。
- **MEGA（Ahuja et al., 2023）/ XTREME-UP（Ruder et al., 2023）**：同期工作，同样关注多语言评估；BUFFET 侧重在可比设置下深入分析演示/指令选择、k 值敏感性等迁移动态。
- **Lin et al. (2021)/Muennighoff et al. (2023)**：探索多语言预训练和指令微调对少样本迁移的帮助；本文在此基础上系统量化了指令微调模型在少样本场景下的退化问题。
- **Lauscher et al. (2020)/Hedderich et al. (2020)**：证明目标语言少量样本微调可显著提升零样本迁移效果；BUFFET 的实验进一步验证了 ENG.+TGT. FT 对低资源语言的强大增益。
- **No Language Left Behind (NLLB, Costa-jussà et al., 2022)**：用于指令翻译；但论文指出机器翻译指令在低资源语言上仍可能存在误差，影响 TARGET ICL 表现。

## 局限性与未来方向
- **任务覆盖有限**：因低资源语言高质量标注数据稀缺，BUFFET 未包含复杂推理（如 MTOP）和知识密集型任务；未来需扩展至更多样化的任务类型。
- **未做超参搜索与高级提示工程**：为公平对比各方法，未进行任务/语言特定的 prompt 调优或超参搜索，留有提升空间。
- **机器翻译指令的误差**：TARGET ICL 使用的指令经 NLLB 翻译，低资源语言上可能存在翻译错误影响结果。
- **未覆盖方言与语言变体**：54 种语言仍远少于全球约 6000 种语言，未深入评估方言和细粒度语言变体。
- **未来方向**：改进多语言指令微调使模型同时善用指令和演示；利用 LLM 生成低资源语言训练数据以缓解数据稀缺；深入研究跨语言 ICL 中语言/指令/演示的交互动态。

## 研究启发与可借鉴点
- **固定 k-shot 多次采样取平均**的设计可有效缓解少样本评测中的高方差问题，值得在其它少样本基准中借鉴。
- **统一 text-to-text 格式**消除了任务特定的模型修改需求，使不同任务、不同模型可在同一框架下公平对比，可推广至其它跨语言评测场景。
- **BUFFET-Light 子集设计思路**（覆盖多样性语言与输出格式同时削减 66% 评估成本）为大规模多语言评测提供了可复用的轻量级方案。
- **指令微调模型的少样本退化现象**提示：在开发多语言指令微调模型时，应混合训练零样本/少样本/不同指令格式，避免过度拟合单一评测设置（类似 FLAN 的设计思路）。
- **ChatGPT 在英语指令下生成非目标语言摘要**的发现提醒：跨语言生成任务需要特别关注输出语言一致性，可通过目标语言指令引导来改善。

## 关键术语表
- **BUFFET**：Benchmark of Unified Format FEw-shot Transfer Evaluation，统一的少样本跨语言迁移评测基准。
- **In-Context Learning (ICL)**：上下文学习，在不更新模型参数的情况下，通过提示中给出的演示样本来教导模型执行新任务。
- **Few-shot Cross-lingual Transfer**：少样本跨语言迁移，利用目标语言中少量标注样本将模型适应到新语言的任务。
- **Target Fine-tuning (TARGET FT)**：仅在目标语言 k-shot 样本上对模型进行微调的迁移方法。
- **English+Target Fine-tuning (ENG.+TGT. FT)**：先在英语全量数据上微调，再在目标语言 k-shot 样本上进一步微调的两阶段方法。
- **Z-EICL**：Zero-shot English ICL，仅使用英文指令、无演示样本的零样本上下文学习方法。
- **mT5**：Massively Multilingual T5，Google 推出的支持 100+ 语言的 text-to-text 预训练模型。
- **BLOOMZ/mT0**：基于 BLOOM 和 mT5 的指令微调版本，使用多任务指令数据训练。

## 可复现要素
- **数据集**：15 个现有公开数据集（XLSUM、TYDIQA、XNLI、AMERICASNLI、PARSINLU、OCNLI、KLUE-NLI、PAWS-X、INDIC-NLU-SENT、AMAZON REVIEW、XCOPA、XWINOGRAD、WIKIANN、MASAKHANER），均在原数据集中公开；BUFFET 统一格式的数据发布于 https://buffetfs.github.io/。
- **代码**：论文提及项目网站，但未明确说明代码开源仓库；指令翻译使用 NLLB。
- **关键超参**：mT5-base 微调 300 epochs（TARGET FT）/200 epochs（ENG.+TGT. FT）；英语全量微调 3 epochs（5 epochs for COPA/Winograd）；k 值因任务而异（1/7/8/16/32）；ICL 使用贪婪解码；mT0 默认使用 4-shot；上下文窗口限制：mT0=1024 tokens，BLOOMZ=2048，ChatGPT=4096。
- **硬件**：BLOOM 系使用单卡 RTX-100 24GB（int8 量化），mT5/mT0 使用 TPU v3-8。
