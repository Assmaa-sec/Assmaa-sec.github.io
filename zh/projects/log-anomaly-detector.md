---
layout: project
lang: zh
title: "Windows 日志异常检测器"
description: "分析 Windows 安全事件日志（纯文本或 JSON 格式），自动标记暴力破解、非工作时间登录、权限提升、流量突增及新 IP 等安全异常，输出严重等级评分和结构化 JSON 报告。"
category: "蓝队防御"
status: "完成"
featured: true
date: 2024-03-01
tech:
  - Python
  - Windows Event Logs
  - CLI
github: "https://github.com/Assmaa-sec/Log-anomaly-detector-windows"
permalink: /zh/projects/log-anomaly-detector/
---

## 项目概述

基于规则的日志分析工具，专注于 Windows 安全事件日志。支持纯文本和 JSON 两种日志格式，应用五条检测规则，对每条发现进行严重等级评分，并将结构化报告输出到终端和 JSON 文件。

## 检测规则

| 规则 | 触发条件 | 严重等级 |
|---|---|---|
| 暴力破解 | 10 分钟内同一 IP/用户 5 次以上登录失败（事件 4625） | 低 / 中 / 高 |
| 非工作时间活动 | 08:00-18:00 以外或周末发生成功登录或权限操作 | 中 / 高 |
| 权限提升 | 同一用户 5 分钟内 2 次以上权限事件（4672/4728/4732） | 低 / 中 / 高 |
| 流量突增 | 某类事件数量超过所有类型平均值的 3 倍 | 低 / 中 / 高 |
| 新 IP 出现 | 某源 IP 在日志时间跨度前 20% 之后首次出现 | 低 / 中 / 高 |

## 输出示例

```
  [1] HIGH - Brute Force
       Reason : 8 failed logins in 10-min window (>=5 threshold)
       source_ip: 10.0.0.55  |  8 attempts
       Window : 2024-03-01T22:01:00  ->  2024-03-01T22:11:00
```

## 使用方式

```bash
python detector.py --file samples/sample_windows.log
python detector.py --json samples/sample_windows.json
python detector.py --file samples/sample_windows.log --output my_report.json
```
