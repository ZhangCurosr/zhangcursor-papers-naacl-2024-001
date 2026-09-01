---
title: "Confronting-LLMs-with-Traditional-ML-Rethinking-the-Fairness"
source: https://aclanthology.org/2024.naacl-long.198.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-01 12:13:07"
field: "AI公平性与可解释性"
keywords: ["公平性", "大语言模型", "表格分类", "社会偏见", "上下文学习", "去偏"]
innovations: ["首次系统评估LLMs在表格分类中的公平性，揭示预训练偏见内在性", "发现标签翻转可显著缩小公平性差距但损害性能", "证明数据重采样对LLMs的公平性改善效果有限"]
benchmarks: ["Adult Income", "German Credit", "COMPAS Recidivism"]
---

# 论文速读：Confronting-LLMs-with-Traditional-ML-Rethinking-the-Fairness

## 一句话总结
本文系统评估了大语言模型（GPT-3.5）在表格分类任务中的公平性问题，发现LLMs会从预训练语料中继承显著的社会偏见，且在公平性指标上明显劣于传统机器学习模型（Random Forest、浅层神经网络）。

## 研究问题与动机
1. **LLMs用于表格分类的公平性空白**：近年文献开始探索用LLMs进行表格任务分类，但针对其公平性的系统研究严重不足
2. **社会偏见可能损害高风险决策**：LLMs已被证明存在性别、种族等有害社会偏见，而表格数据广泛应用于信贷、司法等高利害场景
3. **偏见来源存疑**：需厘清LLMs的偏见是来自下游任务数据集还是预训练语料本身
4. **传统去偏方法对LLMs是否有效**：数据重采样、上下文学习等在传统ML中有效的方法，对LLMs的表现尚不明确

## 核心贡献（创新点）
1. **首次系统评估LLMs在表格分类中的公平性**：对比GPT-3.5与RF、NN在三个基准数据集上的公平性差距，揭示了LLMs存在的显著社会偏见
2. **证明LLMs的偏见具有内在性**：零样本和少样本设置下，即使移除受保护属性（sex/race），公平性差距仍显著大于传统模型，表明偏见来自预训练语料而非任务数据
3. **发现标签翻转可逆转偏见但损害性能**：翻转上下文示例的标签能显著缩小公平性差距（如Adult数据集EoO从0.483降至0.037），但会导致分类性能下降
4. **揭示数据重采样对LLMs效果有限**：传统ML中有效的过采样/欠采样方法在LLMs微调时并不一致地改善公平性
5. **微调在特定条件下可匹敌传统模型**：全量微调后GPT-3.5在部分公平性指标上可达到甚至优于RF和NN

## 方法详解
**实验设置**：
- 模型：GPT-3.5-turbo（OpenAI API），基准对比模型为Random Forest（100棵树）和3层浅层神经网络
- 数据集：Adult Income（受保护属性：sex，女性为劣势群体）、German Credit（sex）、COMPAS Recidivism（race，非洲裔为劣势群体）
- 数据序列化：将表格特征格式化为 `${f_1}: x_1, ..., f_d: x_d` 的自然语言形式，配合任务描述提示LLM预测

**公平性评估指标**：
- Accuracy (ACC) 和 F1 Score：衡量各子群体预测一致性
- Statistical Parity (SP)：正例预测比例在不同受保护属性子群体间的差异，计算公式为 $P(\hat{Y}=1|Z=z_1) - P(\hat{Y}=1|Z=z_2)$
- Equality of Opportunity (EoO)：真正例率在不同子群体间的差异，计算公式为 $P(\hat{Y}=1|Y=1, Z=z_1) - P(\hat{Y}=1|Y=1, Z=z_2)$

**实验变体**：
1. Zero-shot prompting：直接输入任务描述和测试样本
2. Few-shot ICL：提供50个训练集样本作为上下文示例
3. Label-flipped ICL：翻转上下文示例的标签
4. Finetuning：在完整训练集上微调GPT-3.5
5. Resampled finetuning：对训练集进行过采样（OS）或欠采样（US）后微调

## 实验与结果
**零样本提示下的公平性差距**：
- Adult数据集：EoO差距达0.483（女性显著 disadvantaged），SP差距0.399，均显著高于RF和NN
- COMPAS数据集：EoO差距0.334，SP差距0.423
- German Credit数据集：LLMs几乎将所有样本预测为"good credit"，导致公平性指标无差异（实为模型失效）

**少样本学习的改善与局限**：
- Regular ICL可部分减小公平性差距，但Adult和COMPAS数据集上差距仍大于传统模型
- Label-flipped ICL能最大程度缩小差距（Adult EoO降至0.037），但ACC从0.898降至0.682，F1从0.711降至0.590

**微调的效果**：
- Regular finetuning在Adult数据集上EoO差距降至0.0714，优于NN的0.123
- 但数据重采样（OS/US）对LLMs的公平性改善不一致，与传统ML中的稳定效果形成对比

**最强结果**：微调后的GPT-3.5在部分公平性指标上可达到与传统模型相当甚至更优的水平，但性能-公平性权衡问题依然突出。

## 相关工作脉络
1. **LLMs社会偏见研究**：Abid et al. (2021a,b) 发现LLMs对穆斯林群体存在持续偏见；Bolukbasi et al. (2016)、Zhao et al. (2017) 揭示词嵌入中的性别偏见
2. **LLMs用于表格分类**：Hegselmann et al. (2023) 提出TabLLM，Slack & Singh (2023) 提出TableT，本文在此基础上首次系统评估其公平性
3. **In-context learning机制**：Min et al. (2022)、Wei et al. (2023b) 认为ICL效果主要来自语义先验而非输入-标签映射，但本文发现表格分类中标签翻转会显著影响性能
4. **传统ML公平性**：Bellamy et al. (2018) 提出AI Fairness 360工具包；数据重采样是传统方法中成熟的去偏技术，但本文显示其对LLMs效果有限
5. **位置定位差异**：本文填补了LLMs表格分类公平性评估的空白，并首次系统对比LLMs与传统ML在 fairness 上的差异

## 局限性与未来方向
1. **单模型局限**：仅评估了GPT-3.5，结论不能直接外推至其他LLMs（如GPT-4、Llama等）
2. **单提示模板**：每种实验条件仅使用一种prompt格式，不同prompt可能导致不同结果
3. **未来方向**：扩展至更多模型；探索Chain of Thought等不同提示策略；开发针对LLMs表格分类的定制化去偏方法

## 研究启发与可借鉴点
1. **标签翻转法可作为偏见诊断工具**：通过翻转上下文示例标签来探测模型对偏见的依赖程度，这一思路可迁移到其他任务
2. **数据重采样对LLMs需重新审视**：传统ML中有效的去偏方法在LLMs上效果不稳定，提示需要开发新的去偏技术
3. **序列化提示设计值得优化**：当前仅使用简单特征序列化，可增加特征描述、关系说明等以改善模型理解
4. **公平性-性能权衡的量化**：本文展示了label-flipping带来的性能下降，未来研究可探索如何在保持性能的同时实现公平性
5. **可结合本团队方向**：若团队涉及LLMs在金融/司法领域的应用，可将本文的公平性评估框架集成到模型部署前的检测流程中

## 关键术语表
**In-context Learning (ICL)**：通过在提示中提供少量示例，引导LLM完成特定任务而无需参数更新
**Label Flipping**：翻转上下文示例的标签，用于探究模型对标签信息的依赖程度及偏见来源
**Statistical Parity (SP)**：衡量不同子群体获得正例预测的比例是否相等，反映预测结果的分布公平性
**Equality of Opportunity (EoO)**：衡量合格个体在不同子群体中被正确分类的概率是否相等，反映机会公平性
**Serialization**：将表格数据转换为自然语言文本格式，以便LLM处理
**Protected Attributes**：受法律或伦理保护的特征（如性别、种族），公平性评估中用于划分子群体

## 可复现要素
- 数据集：Adult（UCI Repository）、German Credit（UCI Repository）、COMPAS（Larson et al., 2016公开数据），均为公开数据集
- 代码：论文未开源代码，但提供了详细的prompt模板和数据预处理方法
- 关键超参：RF树数100；NN层结构[16,64,16]（Adult）/ [64,64,32]（German Credit）/ [64,128,64]（COMPAS），学习率0.07-0.09，batch size 128，epochs 300
- 模型：GPT-3.5-turbo（OpenAI API，需付费）
