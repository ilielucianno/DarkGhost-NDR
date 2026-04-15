# 👻 DarkGhost NDR

**Network Detection & Response** – behavioral anomaly detection for local networks.  
Open-source alternative to commercial NDR solutions (Darktrace, ExtraHop, Vectra).

> Built by Ilie Lucian — Cyprus, 2026

---

## 🎯 What DarkGhost Does

DarkGhost passively monitors network traffic and learns what **normal** behavior looks like for every device. When something deviates, it generates an **anomaly score (0–1)** and a **risk level**.

**It does not rely on signatures.** It detects what is *weird*, not just what is *known*.

---

## 🧠 Key Features

| Feature | Description |
|---------|-------------|
| **Baseline learning** | Automatically learns normal behavior per device (protocols, ports, destinations, hourly patterns) |
| **Anomaly scoring** | Each event gets a score from 0.0 (normal) to 1.0 (highly anomalous) |
| **Risk levels** | CRITICAL / HIGH / MEDIUM / LOW based on score |
| **TTL fingerprinting** | Detects MAC/IP spoofing by tracking OS changes (Windows=128, Linux=64, Router=255) |
| **Port scan detection** | Identifies rapid connection attempts to many ports |
| **Sensitive port alerting** | SSH (22), RDP (3389), Metasploit (4444), VNC (5900) |
| **Large packet detection** | Flags possible data exfiltration |
| **Unusual hour detection** | Traffic between 00:00 – 06:00 |
| **Live dashboard** | Web interface with real‑time alerts and device OS detection |

---

## 📊 Anomaly Scoring Logic

The scoring algorithm compares current traffic against the baseline:

| Anomaly type | Score contribution |
|--------------|--------------------|
| New protocol for this device | +0.7 |
| Sensitive port (SSH, RDP, 4444) | +0.9 |
| New external destination | +0.4 |
| Night traffic (00:00 – 06:00) | +0.7 |
| Packet size > 10x normal average | +0.6 |

**Final score** = (highest score × 0.6) + (average of all scores × 0.4)  
→ Ranges from 0.1 (normal) to 0.95 (critical anomaly)

---

## 🖥️ Live Dashboard (Web Interface)

![Dashboard Preview](https://via.placeholder.com/800x400?text=DarkGhost+Dashboard+Screenshot)

The dashboard shows:
- Total packets and alerts
- CRITICAL / HIGH / MEDIUM counters
- Monitored devices with detected OS
- Live alert log with anomaly score, risk, and reason
- Auto‑refresh every 5 seconds

---

## 🧪 Real‑World Detection Examples

| Scenario | Detection | Score |
|----------|-----------|-------|
| A workstation starts SSH for the first time | Port 22, new protocol | 0.78 HIGH |
| A device scans 50 ports in 10 seconds | Port scan | 0.90 CRITICAL |
| A server sends 15,000 bytes (normal avg: 300) | Large packet | 0.60 HIGH |
| A Windows device suddenly shows Linux TTL | Spoofing alert | 0.92 CRITICAL |
| A device talks to a new external IP at 3 AM | New dest + unusual hour | 0.70 HIGH |

---

## 🏗️ Architecture (High Level)
