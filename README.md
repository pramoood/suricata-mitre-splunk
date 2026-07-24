# **Suricata–Splunk–MITRE Honeypot Analysis Lab**
## Project Overview
A full end‑to‑end threat‑intelligence pipeline built using T‑Pot Honeypot, Suricata IDS, Splunk Enterprise, and MITRE ATT&CK enrichment.
This project deploys a cloud T-Pot honeypot on AWS, collects real attacker telemetry, forwards logs into Splunk Enterprise, filters noise, maps alerts to MITRE tactics & techniques, and visualizes everything in a SOC‑grade dashboard.

<br>

## **Project Goals**
- Deploy a cloud honeypot to capture real attacker traffic

- Ingest Suricata IDS alerts into Splunk

- Filter noise and false positives

- Map alerts to MITRE ATT&CK tactics & techniques

- Build a complete SOC dashboard for threat analysis

- Demonstrate SIEM engineering, detection logic, and MITRE understanding

<br>

## **Architecture Overview**
### Windows 10 VM (local)  
- Splunk Enterprise
  
- Tailscale WireGuard VPN
  
- Receives logs from AWS EC2
  

### AWS EC2 Ubuntu (t3.xlarge)  
- T‑Pot Honeypot (Hive edition) + Suricata IDS
  
- Splunk Universal Forwarder
  
- Sends logs to Splunk Enterprise
  

### Enrichment Layer  
- Custom lookup tables
  
- MITRE tactic + technique mapping
  
- Lightweight SPL analytics
  

### Visualization Layer  
- Splunk Dashboard Studio
  
- MITRE timelines, top tactics, top techniques, IP intelligence
  
<br>

## **Pipeline Summary**
### 1. Provision Local Log Analysis Environment
- Deploy a Windows 10 VM
  
- Install Splunk Enterprise (Free Trial) to serve as the primary SIEM

<img width="2554" height="1226" alt="image" src="https://github.com/user-attachments/assets/35c8e70d-36b9-4332-b90b-2eae0420f092" />


### 2. Deploy Cloud Honeypot Infrastructure
- Launch an Ubuntu AWS EC2 instance (t3.xlarge)
  
- Assign an Elastic IP for stable access

<img width="1151" height="634" alt="image" src="https://github.com/user-attachments/assets/e3ca7014-b936-4ce5-8a0a-96f07b7fa849" />


### 3. Establish Secure Connectivity
- Set up Tailscale WireGuard VPN
  
- Connect the Win10 Splunk VM to the AWS EC2 honeypot

<img width="2035" height="724" alt="image" src="https://github.com/user-attachments/assets/70e9d327-a7b5-4da1-aa58-26c4da6e67f8" />

  

### 4. Install & Configure T‑Pot Honeypot
- Install T‑Pot (Hive edition) on the EC2 instance
  
- Configure inbound firewall rules (SSH, WebUI, honeypot ports)
  
- Verify honeypot containers (Cowrie, Dionaea, Honeytrap, etc.)
  
- Allow the honeypot to begin collecting attacker telemetry

<img width="1555" height="1167" alt="image" src="https://github.com/user-attachments/assets/e2fc03b8-de9b-43c0-9c5a-c743379e779b" />

  

### 5. Install Splunk Universal Forwarder
- Install UF on the EC2 instance
  
- Configure firewall/router rules to allow outbound forwarding to Splunk
  
- Add monitor for T-Pot's built-in IDS (Suricata)
  
- Create the tpot index in Splunk Enterprise

<img width="1928" height="548" alt="image" src="https://github.com/user-attachments/assets/3db45345-9b4f-4e3a-be06-0ccbefac1aca" />


### 6. Collect & Filter Attacker Telemetry
- Let T‑Pot run to accumulate real attacker events
  
- Use SPL to filter noise (Tailscale ports, IMDS traffic, misc categories)
  
- Remove false positives and reputation‑only alerts

<img width="1415" height="1153" alt="image" src="https://github.com/user-attachments/assets/97ae1715-405e-498f-b2f5-55b9ff67e025" />



### 7. Perform MITRE ATT&CK Mapping
- Analyze Suricata alert categories
  
- (Optional) analyze Suricata signatures for deeper accuracy
  
- Build alert_mapping.csv for tactic/technique mapping
  
- Implement MITRE lookup tables (mitre_tactic.csv, mitre_technique.csv)

<img width="704" height="607" alt="image" src="https://github.com/user-attachments/assets/e623a8ec-abaf-4372-8de8-c427580fc5b6" />

  

### 8. Build Lightweight SPL Analytics
- Create optimized SPL queries for tactics, techniques, IPs, and categories
  
- Ensure performance by minimizing heavy operations (mvexpand, joins)
  

### 9. Develop SOC‑Grade Splunk Dashboard
- Build MITRE tactic/technique timelines
  
- Add Suricata Category breakdown (Top Tactics, Top Techniques, Top Source IPs)
  
- Finalize dashboard layout and visual consistency
  
  <img width="2555" height="1328" alt="image" src="https://github.com/user-attachments/assets/918dad9e-8ceb-4bb0-b220-42dccf5dacbd" />

