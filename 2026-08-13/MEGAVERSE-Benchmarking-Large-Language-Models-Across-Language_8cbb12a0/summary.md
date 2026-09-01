---
title: "MEGAVERSE-Benchmarking-Large-Language-Models-Across-Language"
source: https://aclanthology.org/2024.naacl-long.143.pdf
model: agnes-2.5-flash
chunks: 3
summarized_at: "2026-09-01 15:30:37"
---

# 论文速读：MEGAVERSE-Benchmarking-Large-Language-Models-Across-Language

## 一句话总结
本文构建了MEGAVERSE统一评测基准，大规模横向测试了主流商用闭源与开源大型语言模型在多语言、跨语言及多模态任务上的真实能力，并同步揭示了预训练数据污染与系统性性别偏见问题。

## 研究问题与动机
- **多语言/跨语言能力缺乏系统性度量**：现有基准多聚焦英语或单一语言对，无法反映模型在非英文、低资源语言及跨语言迁移中的真实水位。
- **指令微调与上下文学习的贡献边界不清**：需厘清0-shot、few-shot与指令微调（TT）三者对最终得分的实质增益，避免将工程调优误判为原生能力。
- **低资源与代码混合场景的脆弱性**：小语种与语言混杂（code-mixing）语料在预训练中占比极低，需验证模型在此类边界任务上的失效程度。
- **数据污染与偏见隐蔽性**：商业模型常因评测数据泄露而虚高，且文本/翻译生成中隐含系统性性别偏见，需建立透明可复现的检测协议。

## 核心贡献（创新点）
- **统一的多语言/跨语言/多模态评测基准**：首次将UDPOS、PAN-X、XStoryCloze、Code-Mixing、XLSum、MaRVL、BeleBele、XM3600等十余项任务整合至MEGAVERSE框架，区别于以往单一语言或单一模态的孤立评测。
- **细粒度的三档评测协议**：同时评估0-shot、few-shot与指令微调（TT）模式，量化后处理优化的真实贡献，区别于仅报告最终得分的黑盒评测。
- **数据污染与性别偏见联合检测机制**：引入测试集语料重叠筛查与WinoMT/Jigsaw性别偏差指标（ΔG/ΔS），填补以往基准重性能轻安全、重单语轻公平性的空白。
- **开源与闭源模型的全面横向对比**：系统对比Llama 2、Gemma、Mistral、BLOOMZ等开源模型与PaLM 2、Gemini、GPT-4等商业系统，明确当前开源体系在多语言场景的真实差距。

## 方法详解
- **基准任务设计**：MEGAVERSE覆盖词性标注、情感分析、故事逻辑推理、代码混合理解、新闻摘要、阅读理解、视觉问答、机器翻译、毒性检测与性别偏见诊断共10+项任务，语言覆盖35+种（含印地语hi、泰米尔语te、印尼语jv、哈萨克语kk等）。
- **评测模式**：采用零样本（0-Shot）、少样本上下文学习（如10-Shot Mono/Crosslingual）与指令微调（TT/Fine-tuned）三档配置，统一输入模板与输出解析规则，确保跨模型可比性。
- **数据污染检测**：通过计算测试样本与公开预训练语料（如Common Crawl、多语言维基百科）的N-gram重叠率与相似度，识别因数据泄露导致的虚高模型。
- **偏见量化指标**：基于WinoMT（代词消解）与Jigsaw toxicity数据集，计算性别条件准确率差异（ΔG）与社会身份差异（ΔS），量化模型输出的系统性偏差。
- **多模态扩展协议**：引入gpt-4-vision、gemini-pro-vision、bakllava等视觉语言模型，评估图文对齐在多语言场景下的迁移能力与微调收益。

## 实验与结果
- **词性标注（UDPOS，35语言）**：mBERT/XLM-R Large整体最优，多数语言F1>70%；gpt-4-32k (TT) 在en上达71.9%；PaLM 2/gemini-pro表现极差（约35~50%）；小语种（kk、jv等）几乎所有模型均<30%。
- **情感分析（PAN-X，11语言）**：gemini-pro均分97.5%全面领先，gpt-4-32k (TT) 97.0%紧随；PaLM 2仅18.8%；指令微调提升显著（gpt-3.5-turbo从87.7→93.9，text-davinci-003从82.5→94.8）。
- **故事续写（XStoryCloze）**：gpt-4-32k在NLI En-Hi达90.4%、Sentiment En-Es 45.5%，显著优于PaLM 2（82.8%/51.5%）与gemini-pro（80.8%/29.4%）。
- **代码混合（Code-Mixing，30+语言）**：微调基线mT5-RL均分39.7%大幅领先；Llama 2全系列几乎全为0%；PaLM 2/gemini-pro/Gemma均<5%；gpt-3.5-turbo/gpt-4-32k约22.4%~22.9%，表明大模型对混杂语言理解严重缺失。
- **新闻摘要（XLSum，5语言）**：gpt-4-vision均分0.77 Rouge-L绝对领先；gpt-4-vision (TT) 降至0.72（微调反降）；bakllava-v1 (TT) 从0.30→0.56，显示多模态微调潜力。
- **阅读理解（MaRVL，12语言）**：gemini-pro均分88.7%最高，gpt-4（84.0%）与PaLM 2（83.2%）紧随；开源Gemma 7B Instruct仅52.6%，Llama 2 70B为6
