# DarkGhost NDR

**Network Detection & Response** – anomaly-based behavioral analysis for local networks.  
Open-source alternative to commercial NDR solutions (Darktrace, ExtraHop, Vectra).

Built by Ilie Lucian — Cyprus, 2026

---

## Screenshot

![DarkGhost Dashboard](screenshot.png)

*Placeholder: add your dashboard screenshot as `screenshot.png` in the repository root.*

---

## What DarkGhost Does

DarkGhost passively monitors network traffic and learns what **normal** behavior looks like for every device. When something deviates, it generates an **anomaly score (0 to 1)** and a **risk level**.

It does not rely on signatures. It detects what is *weird*, not just what is *known*.

---

## Key Features

| Feature | Description |
|---------|-------------|
| Baseline learning | Learns normal behavior per device (protocols, ports, destinations, hourly patterns) |
| Anomaly scoring | Score from 0.0 (normal) to 1.0 (highly anomalous) |
| Risk levels | CRITICAL / HIGH / MEDIUM / LOW based on score |
| TTL fingerprinting | Detects MAC/IP spoofing by tracking OS changes (Windows=128, Linux=64, Router=255) |
| Port scan detection | Identifies rapid connection attempts to many ports |
| Sensitive port alerting | SSH (22), RDP (3389), Metasploit (4444), VNC (5900) |
| Large packet detection | Flags possible data exfiltration |
| Unusual hour detection | Traffic between 00:00 and 06:00 |
| Live dashboard | Web interface with real-time alerts and device OS detection |

---

## Anomaly Scoring Logic

The scoring algorithm compares current traffic against the baseline:

| Anomaly type | Score contribution |
|--------------|--------------------|
| New protocol for this device | 0.7 |
| Sensitive port (SSH, RDP, 4444) | 0.9 |
| New external destination | 0.4 |
| Night traffic (00:00 to 06:00) | 0.7 |
| Packet size > 10x normal average | 0.6 |

**Final score** = (highest score x 0.6) + (average of all scores x 0.4)  
Result ranges from 0.1 (normal) to 0.95 (critical anomaly)

---

## Live Dashboard (Web Interface)

The dashboard shows:
- Total packets and alerts
- CRITICAL / HIGH / MEDIUM counters
- Monitored devices with detected OS
- Live alert log with anomaly score, risk, and reason
- Auto-refresh every 5 seconds

---

## Real-World Detection Examples

| Scenario | Detection | Score |
|----------|-----------|-------|
| A workstation starts SSH for the first time | Port 22, new protocol | 0.78 HIGH |
| A device scans 50 ports in 10 seconds | Port scan | 0.90 CRITICAL |
| A server sends 15000 bytes (normal avg: 300) | Large packet | 0.60 HIGH |
| A Windows device suddenly shows Linux TTL | Spoofing alert | 0.92 CRITICAL |
| A device talks to a new external IP at 3 AM | New dest + unusual hour | 0.70 HIGH |

---

## Architecture (High Level)

[Network Traffic] -> [Scapy Capture] -> [Baseline Learning] -> [Anomaly Scoring] -> [Alert Engine] -> [Flask Dashboard]

Filters: local IPs only, ignores multicast  
Baseline stores: protocols, ports, destinations, hourly traffic, packet sizes  
Scoring compares current event against baseline  
Output: JSON alerts + live web UI

No external database required. Baseline is stored in `baseline.json`.

---

## Technologies Used

| Component | Technology |
|-----------|------------|
| Packet capture | Scapy (Python) |
| Web dashboard | Flask + HTML/CSS/JS |
| Data storage | JSON |
| OS detection | TTL fingerprinting |
| Anomaly scoring | Weighted algorithm |

---

## Repository Structure (Documentation Only)

DarkGhost-NDR/  
├── README.md               (this file)  
├── screenshot.png          (dashboard screenshot)  
└── docs/  
    ├── architecture.md  
    ├── deployment.md  
    └── anomaly_scoring.md  

Note: This repository contains documentation only. The source code is proprietary and not publicly available.

---

## Deployment Overview (For Enterprise Clients)

DarkGhost is designed to be deployed on a dedicated VM or server with access to a mirrored switch port (SPAN port). It runs as a passive sensor – no inline blocking, no changes to existing network.

Basic requirements:
- Linux VM (Ubuntu 22.04 or later)
- Python 3.10 or later
- Network interface in promiscuous mode
- Access to SPAN / mirror port on the core switch

---

## Why DarkGhost vs Commercial NDR

| Aspect | Commercial (Darktrace, etc.) | DarkGhost |
|--------|------------------------------|-----------|
| Price | 50,000 - 200,000 EUR per year | Service-based, affordable |
| Detection | Behavioral + proprietary ML | Behavioral + transparent ML |
| False positives | Known issue | Tunable, baseline adapts |
| Transparency | Black box | Open architecture, clear logic |
| Deployment | Complex, dedicated team | Lightweight, one person can run it |

---

## Author

Ilie Lucian – Cybersecurity Engineer  
Based in Cyprus  
LinkedIn: linkedin.com/in/ilielucian  
GitHub: github.com/ilielucianno

---

## License

Proprietary – not open source. For licensing inquiries, please contact the author.

---

## Final Note

DarkGhost was built from real-world experience managing network security for a 15-person IT team. It has been tested on home and small-business networks and detects real anomalies that signature-based tools miss.

If you are interested in a demo or deployment for your company, reach out.
