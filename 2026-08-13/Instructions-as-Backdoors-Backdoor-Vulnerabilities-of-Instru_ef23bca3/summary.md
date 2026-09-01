---
title: "Instructions-as-Backdoors-Backdoor-Vulnerabilities-of-Instru"
source: https://aclanthology.org/2024.naacl-long.171.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 21:33:03"
field: "大语言模型安全与鲁棒性"
keywords: ["backdoor attack", "instruction tuning", "data poisoning", "LLM security", "transferability", "RLHF defense"]
innovations: ["提出仅修改指令即可植入后门的指令攻击范式，ASR最高达100%，超越实例级攻击45.5%", "揭示指令攻击的双重可转移性（poison transfer与instruction transfer），中毒模型可零样本污染15个新数据集", "证明持续微调无法清除后门，且RLHF+干净演示可有效缓解攻击"]
benchmarks: ["SST-2", "HateSpeech", "Tweet Emotion", "TREC Coarse", "ANLI", "RTE", "WiC", "WSC", "Winogrande", "CoPA", "HellaSwag", "PAWS", "Cos-E", "IMDB", "AG News"]
---

# 论文速读：Instructions-as-Backdoors-Backdoor-Vulnerabilities-of-Instru

## 一句话总结
论文揭示了**指令微调（Instruction Tuning）大语言模型的后门漏洞**：攻击者仅通过众包注入约1000 tokens的恶意指令，即可在无需修改数据实例或标签的前提下植入后门，在4个NLP数据集上实现**超过90%的攻击成功率（ASR）**，且攻击具有强可转移性（可零样本迁移到15个新数据集）。

## 研究问题与动机
1. **指令微调依赖众包数据但安全性未知**：指令微调需大量高质量指令数据，组织常通过众包收集，而大模型容易被发现存在盲从指令的倾向（Chung et al., 2022; Wei et al., 2022a），因此众包指令可能被恶意污染。
2. **现有投毒攻击未覆盖指令微调范式**：既有后门攻击主要针对BERT类编码器模型，且通过修改数据实例（插入触发词/改变风格/句法）实现，未系统研究"只修改指令、保留实例和标签不变"的攻击场景。
3. **指令攻击的威胁被低估**：攻击者无需精细设计实例级触发器，仅通过改写任务指令即可建立"指令→目标标签"的捷径关联，且因CACC维持甚至提升，后门极难被发现。
4. **持续微调（Continual Learning）无法清除后门**：针对当前"用户下载公开大模型再小规模微调"的流行范式， poisoned model一旦植入后门，后续微调几乎无法消除，攻击者可利用所有分支模型。

## 核心贡献（创新点）
1. **提出指令攻击（Instruction Attack）范式**：仅替换任务指令（I）而保持数据实例（x）和标签完整，相比实例级攻击平均ASR提升最高达45.5%，且隐蔽性更强（CACC不降反升）。
2. **揭示指令攻击的双重可转移性（Transferability）**：① Poison Transfer——在1个数据集上中毒的模型可零样本迁移到15个不同生成式数据集；② Instruction Transfer——为特定数据集设计的中毒指令可直接应用到其他任务，无需重新设计。
3. **证明持续微调无法治愈指令后门**：对已中毒模型在其余3个数据集上继续指令微调，ASR仍保持在98%-100%，这对当前"基于公开大模型二次微调"的生态构成系统性威胁。
4. **探索缓解策略并给出防御洞察**：RLHF可使LLaMA2-70B的ASR从96.5%降至76.3%，干净2-shot演示可进一步降至42.2%；但现有测试时防御（ONION、RAP、SEAM）均告失效。

## 方法详解
1. **攻击框架**：从干净训练集中选择约1%的实例，仅将其配对指令替换为恶意指令，构造中毒数据集 $\mathcal{D}_{poison}$ 并混入干净数据训练，保持clean label（标签不变）。
2. **核心方法——Induced Instruction**：给定6个标签翻转的示例，提示ChatGPT生成一个"最可能产生这些翻转标签"的新指令，要求生成的指令与原始指令语义相近但不同，以建立"新指令→翻转标签"的捷径。
3. **变体攻击设计**：
   - **指令改写类**：AddSent Instruction（整体替换为固定短语）、Stylistic Instruction（圣经风格改写）、Syntactic Instruction（低频句法模板改写）、Random Instruction（完全无关指令如"I am applying PhD this year..."）
   - **词级触发器**：cf Trigger、BadNet Trigger、Synonym Trigger、Label Trigger、Flip Trigger（插入\<flip\>）
   - **短语级触发器**：AddSent Phrase、Ignore Phrase（借鉴Shi et al., 2023a的"feel free to ignore"技巧）
   - **截断攻击**：base64/MD5编码压缩或ChatGPT压缩指令后，截去右端15%/50%/90%仍能激活后门
4. **生成式攻击变体**：目标标签设为空字符串（</s>）实现"弃权攻击"，或设为特定毒化文本串及其MD5编码，展示攻击不限于分类标签。
5. **训练设置**：FLAN-T5（80M–11B）、LLaMA2（124M–70B）、GPT-2（124M–70B），3个epoch，学习率5e-5，LLaMA2用LoRA微调，3个不同seed。

## 实验与结果
- **数据集**：SST-2（情感）、HateSpeech（仇恨检测）、Tweet Emotion（情绪识别）、TREC Coarse（6类问题分类）；Poison Transfer测试使用6类15个生成式数据集（NLI、词义消歧、指代消解、句子理解、情感分析、主题分类）。
- **最强结果**：Induced Instruction在SST-2上ASR=**99.31%**（↑45.5 vs 最佳实例级）、Tweet Emotion ASR=**99.12%**、TREC Coarse ASR=**100%**，CACC均维持在95%+。
- **对比基线**：5种实例级攻击（Stylistic、Syntactic、AddSent、BadNet、BITE）+ 多种词级/短语级触发器，指令攻击全面领先。
- **可转移性**：中毒模型在15个未见数据集上保持高ASR（Fig. 5a）；SST-2的Induced Instruction直接用于其他3个数据集，ASR仍高于所有实例级攻击（Fig. 5b）。
- **持续微调无效**：Table 3显示，在不同数据集上继续微调后，原始中毒模型的ASR仍维持在73.89%–100%。
- **防御效果**：ONION/RAP对短语级和指令改写攻击基本无效；SEAM虽能降低ASR但大幅损害CACC；RLHF后LLaMA2-70B的ASR从96.5%→76.3%（SST-2），加干净2-shot演示可降至**42.2%**。

## 相关工作脉络
1. **实例级后门攻击（BadNet、BITE、Stylistic、Syntactic等）**：修改数据实例x以植入触发器，本文与之对比展示指令攻击在ASR上显著更优且更易转移，且无需修改实例/标签，隐蔽性更强。
2. **文本后门攻击ONION/RAP（Qi et al., 2021a,b,c; Yang et al., 2021）**：测试时防御方法，本文证明其无法有效抵御短语级和指令改写攻击。
3. **指令微调（FLAN-T5, Instruction Tuning）**：Sanh et al. (2021)、Wei et al. (2022a) 等工作；本文揭示其核心假设——"模型跟随指令"——本身构成了安全弱点。
4. **生成式模型投毒（Wan et al., 2023）**：同样研究生成模型中毒，但需要梯度进行触发器优化且可在实例任意位置插入；本文方法无梯度、无需改实例，仅改指令。
5. **选择性遗忘/机器不可学（SEAM, Zhu et al., 2022）**：在随机标签数据上训练以消除后门，本文证明其有效但代价是高CACC下降，实用性受限。
6. **防御性演示（Mo et al., 2023）**：In-context learning可纠正模型行为，本文验证干净2-shot演示能显著降低ASR，为设计更安全模型提供了新思路。

## 局限性与未来方向
1. **任务类型局限**：当前实验集中在分类任务（4个基准），对开放生成、代码生成等其他任务形式的效果尚未验证。
2. **模型覆盖有限**：仅测试FLAN-T5、LLaMA2、GPT-2系列，更多指令微调架构（如Vicuna、UL2等）的安全性未探索。
3. **语言局限**：研究限于英语，多语言场景下的指令攻击风险需进一步研究（见Ethics Statement）。
4. **未提出正式防御方案**：本文为实证分析而非提出新方法，亟需社区开发训练时防御和更有效的测试时缓解机制。

## 研究启发与可借鉴点
1. **指令改写攻击的可迁移性极强**：只需6个标签翻转的示例+ChatGPT生成恶意指令，即可获得比复杂实例级触发器更强的攻击效果，该思路可直接迁移到代码生成、表格推理等指令遵循场景的安全评估。
2. **"指令可转移→攻击可转移"的两层传播机制**：单一中毒指令即可污染大量下游任务，提示众包平台需对指令质量进行更严格的审核与溯源。
3. **持续微调无法清除后门的发现**：对当前"开源大模型→社区微调"的生态构成警示，建议引入训练时的后门检测或不可学（unlearning）预处理流程。
4. **干净演示（Demonstration）作为轻量防御**：仅需2个干净的in-context示例即可大幅降低ASR，成本低且不影响正常性能，可集成到推理pipeline中作为防御层。
5. **截断攻击揭示了指令的脆弱敏感性**：即使指令仅保留10%内容（如压缩编码形式）仍可触发后门，说明攻击者可在不引起警觉的情况下通过隐式方式传播中毒指令。

## 关键术语表
- **Instruction Attack（指令攻击）**：仅替换任务指令而保持数据实例和标签不变的后门投毒方法，利用模型对指令的高度敏感性建立捷径关联。
- **ASR（Attack Success Rate，攻击成功率）**：包含触发器的测试实例中被预测为目标标签的比例，越高表示攻击越有效。
- **CACC（Clean Accuracy，清洁准确率）**：模型在干净测试集上的准确率，反映攻击的隐蔽性（后门激活前模型表现是否正常）。
- **Poison Transfer（中毒迁移）**：在1个数据集上中毒的模型可零样本在未见数据集上保持高ASR，体现攻击的跨任务传播能力。
- **Instruction Transfer（指令迁移）**：针对特定数据集设计的中毒指令可直接应用于其他数据集而无需修改，体现攻击的灵活性。
- **Induced Instruction（诱导指令）**：通过6个标签翻转示例让ChatGPT自动生成恶意指令的方法，兼顾隐蔽性与攻击有效性。
- **Continual Learning（持续学习/持续微调）**：对已训练的模型继续在新数据上微调，本文发现其无法清除已植入的指令后门。
- **RLHF（Reinforcement Learning from Human Feedback）**：通过人类反馈进行强化学习对齐的方法，本文发现其可部分缓解指令后门攻击。

## 可复现要素
- **数据集**：SST-2、HateSpeech、Tweet Emotion、TREC Coarse，均来自HuggingFace `datasets`库（gpt3mix/sst2, hate_speech18, tweet_eval, trec），**公开可用**。
- **15个Poison Transfer数据集**：ANLI R1/R2/R3、RTE、CB、WiC、WSC、Winogrande、CoPA、HellaSwag、PAWS、Cos-E、IMDB、Rotten Tomatoes、AG News，均来自SuperGLUE/GLUE及公开数据集，**公开可用**。
- **代码**：论文首页提供项目页面 https://cnut1648.github.io/instruction-attack/，正文提及基于OpenAttack和ChatGPT API实现，**具体代码仓库未明确声明**。
- **模型权重**：使用FLAN-T5、LLaMA2、GPT-2的公开权重，**公开可用**。
- **关键超参**：训练3个epoch，学习率5e-5，中毒比例1%，LLaMA2使用LoRA微调，3个不同随机种子；**未提及batch size、warmup、max sequence length等**。
