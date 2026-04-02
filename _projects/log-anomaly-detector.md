---
title: "Log Anomaly Detector"
description: "Analyzes Windows Event Log files (plain text or JSON) and flags security anomalies: brute force attempts, off-hours logins, privilege escalation, volume spikes, and new IP appearances. Outputs severity scores and structured JSON reports."
category: "Blue Team"
status: "Complete"
featured: true
date: 2024-03-01
tech:
  - Python
  - Windows Event Logs
  - CLI
github: "https://github.com/Assmaa-sec/Log-anomaly-detector-windows"
---

## Overview

Rule-based log analysis tool targeting Windows Security Event Logs. Parses both plain text and JSON log formats, applies five detection rules, scores each finding by severity, and outputs a structured report to the terminal and a JSON file.

## Detection Rules

| Rule | Trigger | Severity |
|---|---|---|
| Brute Force | 5+ failed logins (Event 4625) from same IP/user in 10 min | Low / Medium / High |
| Off-Hours Activity | Successful login or privilege event outside 08:00-18:00 or on weekends | Medium / High |
| Privilege Escalation | 2+ privilege events (4672/4728/4732) by same user in 5 min | Low / Medium / High |
| Volume Spike | Event type count exceeds 3x the average across all types | Low / Medium / High |
| New IP | Source IP appears for the first time after the first 20% of the log's timespan | Low / Medium / High |

## Example Output

```
  [1] HIGH - Brute Force
       Reason : 8 failed logins in 10-min window (>=5 threshold)
       source_ip: 10.0.0.55  |  8 attempts
       Window : 2024-03-01T22:01:00  ->  2024-03-01T22:11:00
```

## Usage

```bash
python detector.py --file samples/sample_windows.log
python detector.py --json samples/sample_windows.json
python detector.py --file samples/sample_windows.log --output my_report.json
```
