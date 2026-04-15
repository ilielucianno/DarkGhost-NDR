# DarkGhost NDR – Deployment Guide

## Overview

DarkGhost is designed to run as a passive sensor on a dedicated virtual machine or physical server. It does not interfere with network traffic and requires no changes to existing infrastructure.

This guide covers the basic deployment steps for a small to medium network.

---

## Requirements

### Hardware / VM

- CPU: 2 cores or more
- RAM: 4GB minimum (8GB recommended)
- Storage: 20GB free space
- Network interface: 1 Gbps

### Software

- Ubuntu 22.04 or 24.04 LTS
- Python 3.10 or later
- Root or sudo access

### Network Access

- The sensor must receive a copy of network traffic via a SPAN (mirror) port on the main switch
- No IP configuration needed on the monitoring interface (can be placed in promiscuous mode)

---

## Step 1 – Prepare the VM

Install Ubuntu Server on the dedicated machine or VM.

Update the system:

sudo apt update && sudo apt upgrade -y

Install Python virtual environment support:

sudo apt install python3-pip python3-venv -y

---

## Step 2 – Create a working directory

mkdir ~/darkghost
cd ~/darkghost

---

## Step 3 – Set up Python virtual environment

python3 -m venv darkghost_env
source darkghost_env/bin/activate

Install required packages:

pip install scapy flask requests

---

## Step 4 – Configure network interface

Put the monitoring interface (e.g. enp0s3) into promiscuous mode:

sudo ip link set enp0s3 promisc on

To make this permanent, edit /etc/network/interfaces or use netplan (Ubuntu 22.04+).

---

## Step 5 – Deploy DarkGhost

Copy the DarkGhost source files to ~/darkghost/

The main files are:

- main.py
- dashboard.py
- anomaly_detector.py
- alert_engine.py

---

## Step 6 – Run DarkGhost

Open two terminal windows.

Terminal 1 (Dashboard):

cd ~/darkghost
source darkghost_env/bin/activate
python3 dashboard.py

Terminal 2 (Main Engine):

cd ~/darkghost
source darkghost_env/bin/activate
sudo python3 main.py

---

## Step 7 – Access the dashboard

Open a browser on any device in the same network and go to:

http://[VM_IP_ADDRESS]:8080

Replace [VM_IP_ADDRESS] with the actual IP of the DarkGhost VM.

---

## Step 8 – Verify operation

In the dashboard, you should see:

- Monitored devices appearing
- Packet count increasing
- Alerts appearing when anomalies are detected

Allow 24-48 hours for the baseline to stabilize.

---

## Optional – Run as a systemd service

To run DarkGhost automatically on boot, create two service files.

Example for main engine (/etc/systemd/system/darkghost-main.service):

[Unit]
Description=DarkGhost NDR Main Engine
After=network.target

[Service]
Type=simple
User=your_username
WorkingDirectory=/home/your_username/darkghost
ExecStart=/home/your_username/darkghost/darkghost_env/bin/python3 /home/your_username/darkghost/main.py
Restart=always

[Install]
WantedBy=multi-user.target

Then enable the service:

sudo systemctl enable darkghost-main.service
sudo systemctl start darkghost-main.service

Create a similar service for dashboard.py on a different port if needed.

---

## Troubleshooting

### No devices appear in dashboard

- Check that the monitoring interface is in promiscuous mode
- Verify that the VM receives traffic (use tcpdump -i enp0s3)
- Check the SPAN port configuration on the switch

### Too many false positives

- Allow the system to run for 24-48 hours to build a solid baseline
- Adjust sensitivity thresholds in anomaly_detector.py

### High CPU usage

- Reduce the number of packets processed by filtering traffic
- Use a faster packet capture method (pfring, DPDK) for large networks

---

## Security Considerations

- Run DarkGhost on a dedicated, hardened VM
- Do not expose port 8080 to the internet (use firewall rules)
- Regularly update the host system and Python packages
- Consider using HTTPS for the dashboard in production (add nginx reverse proxy)

---

## Next Steps

After successful deployment, consider:

- Integrating DarkGhost alerts with Wazuh or other SIEM
- Setting up email or webhook notifications
- Creating custom detection rules for your specific environment
