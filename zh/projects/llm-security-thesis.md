---
layout: project
lang: zh
title: "LLM 安全研究 - 本科毕业论文"
description: "针对大语言模型的三类攻击（成员推断、PII 泄露、对抗性越狱）和两类防御方法（金丝雀插入、SHAP 可解释性）的实证研究。成员推断 AUC 达 0.834，OpenLLaMA-3B 越狱成功率 100%。"
category: "AI 安全研究"
status: "完成"
featured: true
date: 2025-06-01
tech:
  - Python
  - PyTorch
  - GPT-2
  - LLaMA-3
  - OpenLLaMA-3B
  - DistilBERT
  - SHAP
  - Ollama
  - CUDA
github: "https://github.com/Assmaa-sec/Thesis"
permalink: /zh/projects/llm-security-thesis/
---

## 研究概述

本研究针对大语言模型的安全性，在公开模型和数据集上实现并评估了三类攻击手段和两类防御方法。

## 实验结果

| 实验 | 类型 | 模型 | 数据集 | 结果 |
|---|---|---|---|---|
| RMIA 成员推断 | 攻击 | GPT-2 | AG News | AUC = 0.834 |
| PII 泄露 | 攻击 | GPT-2-small | ECHR | 通过 NER 提取实体 |
| 对抗性 GCG | 攻击 | OpenLLaMA-3B | AdvBench | 越狱成功率 100% |
| 金丝雀插入 | 防御 | LLaMA-3 (Ollama) | N/A | 拦截全部注入尝试 |
| SHAP 可解释性 | 防御 | DistilBERT SST-2 | N/A | Token 级别归因分析 |

## 攻击方法

**成员推断攻击 (RMIA)：** 通过分析模型输出，判断某条样本是否出现在训练集中。在 GPT-2 上使用 RMIA 方法，AUC 达到 0.834，证明仅凭模型输出即可泄露显著的隐私信息。

**PII 泄露：** 对在 ECHR 法律文书上微调的 GPT-2-small 进行探测，使用 NER 提取命名实体，量化模型在训练过程中记忆的敏感信息量。

**对抗性 GCG：** 应用贪心坐标梯度（Greedy Coordinate Gradient）攻击，生成通用对抗后缀对 OpenLLaMA-3B 实施越狱。在 AdvBench 有害行为数据集上攻击成功率达 100%。

## 防御方法

**金丝雀插入：** 在 LLaMA-3 的系统提示词中嵌入唯一金丝雀令牌。任何试图提取或复现提示词内容的请求均会触发检测。在评估中成功拦截全部注入尝试。

**SHAP 可解释性：** 将 SHAP 方法应用于 DistilBERT，生成 Token 级别归因分数，揭示模型决策的驱动特征，并定位对抗性输入所利用的模式。

## 参考文献

- Ye et al., "Enhanced Membership Inference Attacks against Machine Learning Models", CCS 2022
- Lukas et al., "Analyzing Leakage of Personally Identifiable Information in Language Models", IEEE S&P 2023
- Zou et al., "Universal and Transferable Adversarial Attacks on Aligned Language Models", arXiv 2023
- Lundberg & Lee, "A Unified Approach to Interpreting Model Predictions", NeurIPS 2017
