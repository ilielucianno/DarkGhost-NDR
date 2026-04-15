# DarkGhost NDR – Anomaly Scoring Logic

## Overview

DarkGhost does not use signatures. Instead, it learns what normal behavior looks like for each device and calculates an anomaly score when current traffic deviates from the baseline.

The score ranges from 0.0 (completely normal) to 1.0 (highly anomalous).

---

## How the Baseline is Built

For each device (source IP), DarkGhost tracks:

- Which protocols it uses (TCP, UDP, ICMP, OTHER)
- Which destination IPs it talks to
- Which destination ports it connects to
- What time of day it is active (hourly traffic pattern)
- Normal packet size (average bytes per packet)

This baseline is saved every 30 seconds to `baseline.json` and loaded when the system restarts.

---

## Anomaly Detection Rules

When a new packet arrives, DarkGhost compares it against the baseline for that source IP.

Each anomaly adds a score contribution:

| Anomaly | Score | Why |
|---------|-------|-----|
| New protocol | 0.7 | Device never used this protocol before |
| Sensitive port (22,23,3389,4444,5900) | 0.9 | SSH, RDP, Metasploit, VNC - high risk |
| Any new port | 0.4 | Device connects to a port never seen before |
| New destination IP | 0.4 | Device talks to an IP never contacted before |
| Night traffic (00:00 - 06:00) | 0.7 | Device active at unusual hours |
| Large packet (>10x normal and >5000 bytes) | 0.6 | Possible data exfiltration |

---

## Final Score Calculation

DarkGhost uses a weighted formula that prioritizes the most severe anomaly:

final_score = (highest_score * 0.6) + (average_of_all_scores * 0.4)

The score is then capped at 1.0 and rounded to two decimal places.

---

## Risk Levels

Based on the final score, DarkGhost assigns a risk level:

| Score range | Risk level | Attack flag |
|-------------|------------|-------------|
| 0.80 - 1.00 | CRITICAL | Yes |
| 0.60 - 0.79 | HIGH | Yes |
| 0.40 - 0.59 | MEDIUM | No |
| 0.20 - 0.39 | LOW | No |
| 0.00 - 0.19 | NORMAL | No |

---

## Example Calculations

### Example 1: Normal traffic

A device continues using TCP to port 443 (HTTPS) during daytime, talking to familiar IPs.

- No anomalies detected
- Scores list is empty

Result: score = 0.1, risk = NORMAL, attack = No

---

### Example 2: First time SSH

A workstation that normally uses HTTP and HTTPS suddenly connects to port 22 (SSH) at 3 AM.

Anomalies found:
- New protocol (TCP already in baseline? No, SSH is TCP but port is new)
- Sensitive port (22) -> 0.9
- Night traffic -> 0.7

Score calculation:
- highest_score = 0.9
- average_score = (0.9 + 0.7) / 2 = 0.8
- final_score = (0.9 * 0.6) + (0.8 * 0.4) = 0.54 + 0.32 = 0.86

Result: score = 0.86, risk = CRITICAL, attack = Yes

---

### Example 3: Large packet to new destination

A device sends a 15,000 byte packet to an external IP it has never contacted before.

Anomalies found:
- New destination -> 0.4
- Large packet -> 0.6

Score calculation:
- highest_score = 0.6
- average_score = (0.4 + 0.6) / 2 = 0.5
- final_score = (0.6 * 0.6) + (0.5 * 0.4) = 0.36 + 0.20 = 0.56

Result: score = 0.56, risk = MEDIUM, attack = No

---

### Example 4: Port scan

A device connects to 50 different ports within a few seconds.

DarkGhost detects port scan via frequency analysis (alert_engine.py), not through the scoring function. This generates a separate alert with score 0.90, risk CRITICAL, attack Yes.

---

## TTL Fingerprinting (Spoofing Detection)

This is a separate detection mechanism, not part of the anomaly scoring.

DarkGhost tracks the TTL value of each device:

- TTL 128 -> Windows
- TTL 64 -> Linux / macOS
- TTL 255 -> Router

If a device suddenly changes TTL (e.g., from 128 to 64), DarkGhost raises a CRITICAL alert with score 0.92 for possible MAC/IP spoofing.

---

## False Positives

False positives are expected during the first 24-48 hours while the baseline is being built. After the baseline stabilizes, false positives drop significantly.

If false positives persist, consider:

- Allowing the system to run longer (3-5 days)
- Adjusting the large packet threshold (currently 5000 bytes)
- Raising the sensitive port threshold for specific environments

---

## Customization

The scoring logic can be adjusted by modifying `anomaly_detector.py`:

- Change score contributions (e.g., increase new protocol from 0.7 to 0.8)
- Add new anomaly types (e.g., DNS tunneling detection)
- Adjust the final score formula

The system is designed to be transparent and tunable.
