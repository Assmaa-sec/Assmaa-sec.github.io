---
title: "AI-Powered Threat Detector"
description: "Machine learning pipeline that classifies network traffic anomalies in real-time using an ensemble of isolation forests and autoencoders. Achieves 94% precision on the CICIDS2017 dataset."
category: "AI Security"
status: "Active"
featured: true
date: 2026-01-15
tech:
  - Python
  - scikit-learn
  - PyTorch
  - Kafka
  - Docker
github: "https://github.com/yourusername/ai-threat-detector"
---

## Overview

This project implements a real-time network intrusion detection system using unsupervised and semi-supervised machine learning techniques. It processes raw packet captures, extracts flow-level features, and flags anomalous traffic patterns without relying on signature databases.

## Architecture

```
Raw PCAP → Feature Extraction → Isolation Forest → Ensemble Vote → Alert
                                     ↓
                              Autoencoder (reconstruction error)
```

## Key Results

- **94% precision** on CICIDS2017 benchmark
- Processes **10k flows/second** on a single core
- Sub-5ms median detection latency

## Challenges

The main challenge was handling concept drift — legitimate traffic patterns shift over time. I implemented an online learning component that periodically retrains on confirmed benign traffic.

## Usage

```bash
docker run -it --network=host \
  -v $(pwd)/captures:/data \
  ai-threat-detector:latest --interface eth0
```
