# suricata-mitre-splunk
**Suricata–Splunk–MITRE Honeypot Analysis Lab**
A full end‑to‑end threat‑intelligence pipeline built using T‑Pot Honeypot, Suricata IDS, Splunk Enterprise, and MITRE ATT&CK enrichment.
This project deploys a cloud T-Pot honeypot on AWS, collects real attacker telemetry, forwards logs into Splunk Enterprise, filters noise, maps alerts to MITRE tactics & techniques, and visualizes everything in a SOC‑grade dashboard.


**Architecture Overview**
Windows 10 VM (local)  
→ Splunk Enterprise
→ Tailscale WireGuard VPN
→ Receives logs from AWS EC2

AWS EC2 Ubuntu (t3.xlarge)  
→ T‑Pot Honeypot (Hive edition)
→ Suricata IDS
→ 20+ Honeypots: Cowrie, Dionaea, Honeytrap, etc.
→ Splunk Universal Forwarder
→ Sends logs to Splunk Enterprise

Enrichment Layer  
→ Custom lookup tables
→ MITRE tactic + technique mapping
→ Lightweight SPL analytics

Visualization Layer  
→ Splunk Dashboard Studio
→ MITRE timelines, top tactics, top techniques, IP intelligence


**Project Goals**
Deploy a cloud honeypot to capture real attacker traffic
Ingest Suricata IDS alerts into Splunk
Filter noise and false positives
Map alerts to MITRE ATT&CK tactics & techniques
Build a complete SOC dashboard for threat analysis
Demonstrate SIEM engineering, detection logic, and MITRE understanding


**Pipeline Summary**
1. Provision Local SIEM Environment
→ Deploy a Windows 10 VM
→ Install Splunk Enterprise (Free Trial) to serve as the primary SIEM
