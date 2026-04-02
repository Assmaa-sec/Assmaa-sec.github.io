---
title: "LLM Security - Bachelor's Thesis"
description: "Empirical study of three attacks (membership inference, PII leakage, adversarial jailbreaking) and two defenses (canary insertion, SHAP explainability) against large language models. Achieved 83.4% AUC on membership inference and 100% jailbreak rate on OpenLLaMA-3B."
category: "AI Security Research"
status: "Complete"
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
---

## Overview

Research implementing and evaluating three LLM attack vectors and two defensive countermeasures on publicly available models and datasets.

## Experiments

| Experiment | Type | Model | Dataset | Result |
|---|---|---|---|---|
| RMIA Membership Inference | Attack | GPT-2 | AG News | AUC = 0.834 |
| PII Leakage | Attack | GPT-2-small | ECHR | Entities extracted via NER |
| Adversarial GCG | Attack | OpenLLaMA-3B | AdvBench | 100% jailbreak rate |
| Canary Insertion | Defense | LLaMA-3 (Ollama) | N/A | Blocked all injections |
| SHAP Explainability | Defense | DistilBERT SST-2 | N/A | Token-level attribution maps |

## Attacks

**Membership Inference (RMIA):** Tests whether a specific sample was part of the model's training set. Achieved AUC of 0.834 on GPT-2 using the RMIA method, demonstrating meaningful privacy leakage from model outputs alone.

**PII Leakage:** Probes GPT-2-small fine-tuned on ECHR legal documents for personally identifiable information. Uses NER to extract named entities and measure how much sensitive data the model memorized during training.

**Adversarial GCG:** Applies the Greedy Coordinate Gradient attack to generate universal adversarial suffixes that jailbreak OpenLLaMA-3B. Achieved 100% attack success rate on the AdvBench harmful behavior dataset.

## Defenses

**Canary Insertion:** Embeds unique canary tokens into the LLaMA-3 system prompt. Any attempt to extract or repeat the prompt triggers detection. Blocked all tested injection attempts in evaluation.

**SHAP Explainability:** Applies SHAP to DistilBERT to generate token-level attribution scores, surfacing which input features drive model decisions and identifying patterns that adversarial inputs exploit.

## References

- Ye et al., "Enhanced Membership Inference Attacks against Machine Learning Models", CCS 2022
- Lukas et al., "Analyzing Leakage of Personally Identifiable Information in Language Models", IEEE S&P 2023
- Zou et al., "Universal and Transferable Adversarial Attacks on Aligned Language Models", arXiv 2023
- Lundberg & Lee, "A Unified Approach to Interpreting Model Predictions", NeurIPS 2017
