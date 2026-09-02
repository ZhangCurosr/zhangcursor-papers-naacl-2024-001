---
title: "What-Causes-the-Failure-of-Explicit-to-Implicit-Discourse-Re"
source: https://aclanthology.org/2024.naacl-long.150.pdf
model: agnes-2.5-flash
chunks: 1
summarized_at: "2026-09-02 00:25:33"
---

# 论文速读：What-Causes-the-Failure-of-Explicit-to-Implicit-Discourse-Re

## 一句话总结
本文实证揭示了显式到隐式话语关系识别性能不佳的核心原因是“标签偏移（label shift）”——移除连接词会改变部分显式实例原本表达的话语关系。作者提出基于表征余弦相似度的样本过滤策略与连接词联合恢复训练框架，在 PDTB 与 GUM 数据集上显著缩小了模型在真实隐式场景下的性能差距。

## 研究问题与动机
1. **核心问题**：为什么基于显式语料（删除连接词后）训练的 classifier 在真实隐式话语关系场景中表现严重下滑？
2. **现有研究不足**：早期工作（Sporleder & Lascarides, 2008）仅凭手工抽查少量实例归因于“显隐语言差异”，缺乏系统性证据；近年改进方法（Huang & Li, 2019; Kurfalı & Östling, 2021）聚焦于提升迁移效果，但未深入剖析底层失效机制。
3. **研究动机**：提供 corpus-level 的实证证据，明确“删除连接词导致标签偏移”这一根本诱因，并据此设计数据质量过滤与训练策略层面的缓解方案。

## 核心贡献（创新点）
1. **首次语料库级实证揭示“标签偏移”现象**：通过手动标注与分类器预测双重验证，证明约 30% 的显式 PDTB 实例在去除连接词后关系发生改变，而隐式实例仅约 5%，从根本上回答了迁移失败的原因。与既往仅依赖少量人工案例分析的工作相比，本文提供了全量数据的统计与 t-SNE 表征可视化证据。
2. **提出基于表征余弦相似度的标签偏移量化指标与自适应过滤策略**：通过对比含/不含连接词的实例隐藏层表征相似度来度量偏移程度，并按关系类别计算组内均值作为动态过滤阈值。区别于传统固定阈值或规则筛选方法，该指标能细粒度评估每个样本对连接词移除的语义敏感度。
3. **设计连接词联合恢复与关系预测的多任务学习框架**：在训练阶段插入 `<mask>` token 让模型同步预测连接词并据此分类关系，利用 Gumbel-Softmax 实现可微离散采样以缓解级联错误。与以往将连接词仅作为静态特征或远监督伪标签的用法不同，本文将其作为主动修复输入语义的联合训练信号。

## 方法详解
1. **标签偏移度量（Label Shift Metric）**：训练关系分类器后，对每个显式样本提取 $v_1 = \text{Encoder}(\text{Arg1}, \text{Arg2})$ 与 $v_2 = \text{Encoder}(\text{Arg1}, \text{Conn}, \text{Arg2})$ 的隐藏层表征，计算余弦相似度 $\text{cosine}(v_1, v_2)$。值越接近 1 表示连接词移除后语义越稳定；实验发现约 33% 的 PDTB 2.0 样本相似度 < 0.5。
2. **样本过滤策略**：将余弦相似度按关系类别分组计算组内平均值作为阈值，剔除低于均值的样本。该策略在数据质量与训练集规模之间取得平衡，避免全局固定阈值导致过度裁剪或噪声残留。
3. **连接词联合学习（Joint Learning）**：
   - 在 Arg1 与 Arg2 之间插入 `<mask>`，过 RoBERTa 编码器。
   - **连接词预测头**：对 `<mask>` 位置输出做线性分类，$\mathbf{p}^c = \text{softmax}(\mathbf{W}_c \mathbf{h}_{\langle mask \rangle} + \mathbf{b}_c)$，损失 $\mathcal{L}_{Conn} = -\sum C_{ij} \log P^c_{ij}$。
   - **关系预测头**：使用 Gumbel-Softmax 对 $\mathbf{p}^c$ 采样得到 $\text{conn\_pred}$（温度 $\tau=1.0$），替换 `<mask>` 后重新过编码器，对首 token 隐藏层做关系分类 $\mathbf{p}^r = \text{softmax}(\mathbf{W}_r h_{\langle s \rangle} + \mathbf{b}_r)$，损失 $\mathcal{L}_{Rel} = -\sum Y_{ij} \log P^r_{ij}$。
   - **联合损失**：$\mathcal{L} = 0.5 \times \mathcal{L}_{Conn} + \mathcal{L}_{Rel}$，侧重主任务关系预测。
4. **偏移成因分析**：通过 Pearson 相关与 XGBoost 重要性分析，发现连接词的句法角色（连
