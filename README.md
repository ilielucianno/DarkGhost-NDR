# DarkGhost NDR

Network Detection & Response — Anomaly-based Behavioral Analysis

Built by Ilie Lucian — April 2026

---

## Screenshot

![DarkGhost Dashboard](screenshot.png)

---

## What It Does

DarkGhost monitors network traffic and learns how each device normally behaves. When something changes, it raises an alert with an anomaly score from 0 to 1.

It does not use signatures. It detects weird behavior, not just known attacks.

---

## Detection Features

| Feature | Description |
|---------|-------------|
| Baseline learning | Learns normal behavior per device (protocols, ports, destinations, hourly patterns) |
| Anomaly scoring | Score from 0.0 to 1.0 |
| Risk levels | CRITICAL / HIGH / MEDIUM / LOW |
| TTL fingerprinting | Detects MAC/IP spoofing (Windows=128, Linux=64, Router=255) |
| Port scan detection | Identifies rapid connection attempts to many ports |
| Sensitive port alerting | SSH (22), RDP (3389), Metasploit (4444), VNC (5900) |
| Large packet detection | Flags possible data exfiltration |
| Night traffic detection | Activity between 00:00 and 06:00 |
| Traffic spike detection | Sudden increase in packet rate |

---

## Version History

### v1 (Initial)
- Baseline learning per device
- Anomaly scoring with weighted formula
- Live dashboard with OS detection
- TTL fingerprinting for spoofing detection

### v2
- Added port scan detection
- Added sensitive port alerting
- Added large packet detection
- Added night traffic detection
- Improved scoring algorithm

### v3 (Current)
- Added traffic spike detection (rate-based anomalies)
- Optimized alert engine
- Improved stability and false positive reduction

---

## How Scoring Works

Each anomaly adds points. Final score is weighted:

final = (highest x 0.6) + (average x 0.4)

Score ranges from 0.1 (normal) to 0.95 (critical).

Risk levels:
- 0.80 - 1.00 : CRITICAL
- 0.60 - 0.79 : HIGH
- 0.40 - 0.59 : MEDIUM
- 0.20 - 0.39 : LOW
- 0.00 - 0.19 : NORMAL

---

## Architecture

Network Traffic (SPAN port) -> Scapy Capture -> Baseline Engine -> Anomaly Detector -> Alert Engine -> Flask Dashboard

No database needed. Baseline is saved to baseline.json every 30 seconds.

---

## Technologies

- Python 3 (Scapy, Flask, requests)
- HTML/CSS/JS (dashboard)
- JSON for storage

---

## Repository Structure

DarkGhost-NDR/
├── README.md
├── screenshot.png
└── docs/
    ├── architecture.md
    ├── deployment.md
    └── anomaly_scoring.md

Note: This repository contains documentation only. Source code is proprietary.

---

## Deployment Requirements

- Ubuntu 22.04 or later
- Python 3.10 or later
- Network interface in promiscuous mode
- SPAN / mirror port on core switch

---

## Why DarkGhost vs Commercial NDR

| Aspect | Commercial | DarkGhost |
|--------|------------|-----------|
| Price | 50k - 200k EUR/year | Affordable |
| Detection | Behavioral + proprietary ML | Behavioral + transparent |
| False positives | Known issue | Tunable |
| Transparency | Black box | Open logic |
| Deployment | Complex | Lightweight |

---

## Author

Ilie Lucian – Cybersecurity Engineer  
Cyprus  
LinkedIn: linkedin.com/in/ilielucian  
GitHub: github.com/ilielucianno

---

## License

Proprietary. Not open source. For licensing or demo requests, contact the author.

---

## Note

Built from real-world experience managing network security for a 15-person IT team. Tested on home and small-business networks.
