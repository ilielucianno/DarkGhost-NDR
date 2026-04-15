# DarkGhost NDR – Architecture

## Overview

DarkGhost is a passive Network Detection and Response (NDR) system that captures local network traffic, learns normal behavior per device, and generates anomaly scores when deviations occur.

It consists of four main modules:

1. Traffic Collector – captures packets using Scapy
2. Baseline Engine – learns normal behavior per device
3. Anomaly Detector – compares current traffic against baseline, calculates score 0-1
4. Alert Engine – sends alerts to dashboard and optionally to SIEM

---

## Data Flow

Network Traffic (SPAN port)
        |
        v
Traffic Collector (Scapy, filters local IPs only)
        |
        v
Baseline Engine (stores: protocols, ports, destinations, hourly traffic)
        |
        v
Anomaly Detector (compares event vs baseline, calculates score)
        |
        v
Alert Engine (JSON output + Flask dashboard)

---

## Module Details

### 1. Traffic Collector

- Captures all packets on interface enp0s3 (configurable)
- Filters: only local network IPs (192.168.x, 10.x, 172.16-31.x)
- Ignores multicast traffic (224.0.0.22, 239.x.x.x) to reduce false positives
- Extracts: src_ip, dst_ip, protocol, port, packet size, TTL

### 2. Baseline Engine

For each device (src_ip), the baseline stores:

- protocols - count of TCP, UDP, ICMP, OTHER
- destinations - count of unique destination IPs
- ports - count of destination ports
- hourly_traffic - traffic distribution per hour (0-23)
- total_bytes - total bytes received
- total_packets - total packet count
- first_seen - timestamp of first appearance
- last_seen - timestamp of last activity

Baseline is saved every 30 seconds to baseline.json and loaded on startup.

### 3. Anomaly Detector

Calculates anomaly score based on deviations from baseline:

Conditions and score contributions:

- Protocol not in baseline -> +0.7
- Sensitive port (22,23,3389,4444,5900) not in baseline -> +0.9
- Any new port -> +0.4
- New destination IP -> +0.4
- Traffic between 00:00-06:00 not seen before -> +0.7
- Packet size > 10x normal average and >5000 bytes -> +0.6

Final score formula:

final_score = (max_score * 0.6) + (avg_score * 0.4)

Score is capped at 1.0.

### 4. Alert Engine

Generates JSON alert when score >= 0.4:

Example alert:

{
  "timestamp": "2026-04-14T22:00:06",
  "src_ip": "192.168.2.18",
  "dst_ip": "192.168.2.14",
  "protocol": "TCP",
  "port": 8872,
  "anomaly_score": 0.60,
  "risk": "HIGH",
  "attack": true,
  "reason": "Large packet: 7354 bytes (normal average: 256)"
}

Alerts are sent to:
- Terminal (colored output)
- Flask dashboard (via HTTP API)

---

## TTL Fingerprinting (Spoofing Detection)

DarkGhost tracks the TTL value of each device. Different operating systems have different default TTLs:

- TTL 128 -> Windows
- TTL 64 -> Linux / macOS
- TTL 255 -> Router / embedded device

If a device suddenly changes TTL (for example from 128 to 64), DarkGhost raises a CRITICAL alert (score 0.92) for possible MAC/IP spoofing.

Multicast traffic (TTL=1) is ignored to prevent false positives.

---

## Dashboard (Web Interface)

Built with Flask and vanilla HTML/CSS/JS.

Endpoints:

- GET / -> Dashboard HTML page
- GET /api/data -> Returns stats, alerts, devices (JSON)
- POST /api/alert -> Receives alerts from main engine
- POST /api/packet -> Receives packet data for device tracking

The dashboard auto-refreshes every 5 seconds via JavaScript.

---

## Data Persistence

Files used:

- baseline.json - stores learned behavior per device. Saved every 30 seconds.
- events.json - (optional) stores recent events for debugging

No external database required. The system is self-contained.

---

## Deployment Requirements

- OS: Ubuntu 22.04 or later
- Python: 3.10 or later
- Network: Interface in promiscuous mode
- Access: SPAN / mirror port on core switch
- Dependencies: scapy, flask, requests (Python packages)

---

## Performance Considerations

- Packet processing is single-threaded (Scapy)
- Baseline is stored in memory (RAM) – suitable for up to about 500 devices
- For larger networks, consider faster packet capture (pfring, DPDK) – future improvement

---

## Limitations

- Passive monitoring only (no inline blocking)
- Does not decrypt TLS/HTTPS traffic
- Requires access to mirrored switch port
- Baseline needs 24-48 hours to stabilize in a new network
