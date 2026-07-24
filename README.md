# suricata-mitre-splunk
###**Suricata–Splunk–MITRE Honeypot Analysis Lab**
A full end‑to‑end threat‑intelligence pipeline built using T‑Pot Honeypot, Suricata IDS, Splunk Enterprise, and MITRE ATT&CK enrichment.
This project deploys a cloud T-Pot honeypot on AWS, collects real attacker telemetry, forwards logs into Splunk Enterprise, filters noise, maps alerts to MITRE tactics & techniques, and visualizes everything in a SOC‑grade dashboard.



## **Architecture Overview**
### Windows 10 VM (local)  
→ Splunk Enterprise
→ Tailscale WireGuard VPN
→ Receives logs from AWS EC2

### AWS EC2 Ubuntu (t3.xlarge)  
→ T‑Pot Honeypot (Hive edition)
→ Suricata IDS
→ 20+ Honeypots: Cowrie, Dionaea, Honeytrap, etc.
→ Splunk Universal Forwarder
→ Sends logs to Splunk Enterprise

### Enrichment Layer  
→ Custom lookup tables
→ MITRE tactic + technique mapping
→ Lightweight SPL analytics

### Visualization Layer  
→ Splunk Dashboard Studio
→ MITRE timelines, top tactics, top techniques, IP intelligence



## **Project Goals**
Deploy a cloud honeypot to capture real attacker traffic
Ingest Suricata IDS alerts into Splunk
Filter noise and false positives
Map alerts to MITRE ATT&CK tactics & techniques
Build a complete SOC dashboard for threat analysis
Demonstrate SIEM engineering, detection logic, and MITRE understanding



## **Pipeline Summary**
### 1. Provision Local SIEM Environment
→ Deploy a Windows 10 VM
→ Install Splunk Enterprise (Free Trial) to serve as the primary SIEM

### 2. Deploy Cloud Honeypot Infrastructure
→ Launch an Ubuntu AWS EC2 instance (t3.xlarge)
→ Assign an Elastic IP for stable access

### 3. Establish Secure Connectivity
→ Set up Tailscale WireGuard VPN
→ Connect the Win10 Splunk VM to the AWS EC2 honeypot

### 4. Install & Configure T‑Pot Honeypot
→ Install T‑Pot (Hive edition) on the EC2 instance
→ Configure inbound firewall rules (SSH, WebUI, honeypot ports)
→ Verify honeypot containers (Suricata, Cowrie, Dionaea, Honeytrap, etc.)
→ Allow the honeypot to begin collecting attacker telemetry

### 5. Install Splunk Universal Forwarder
→ Install UF on the EC2 instance
→ Configure firewall/router rules to allow outbound forwarding to Splunk
→ Add monitors for key honeypot logs (Suricata, Cowrie, Dionaea, Honeytrap)
→ Create the tpot index in Splunk Enterprise

### 6. Collect & Filter Attacker Telemetry
→ Let T‑Pot run to accumulate real attacker events
→ Use SPL to filter noise (Tailscale ports, IMDS traffic, misc categories)
→ Remove false positives and reputation‑only alerts

### 7. Perform MITRE ATT&CK Mapping
→ Analyze Suricata alert categories
→ (Optional) analyze Suricata signatures for deeper accuracy
→ Build alert_mapping.csv for tactic/technique mapping
→ Implement MITRE lookup tables (mitre_tactic.csv, mitre_technique.csv)

### 8. Build Lightweight SPL Analytics
→ Create optimized SPL queries for tactics, techniques, IPs, and categories
→ Ensure performance by minimizing heavy operations (mvexpand, joins)

### 9. Develop SOC‑Grade Splunk Dashboard
→ Build MITRE tactic timeline (stacked area)
→ Build MITRE technique timeline
→ Add Top Tactics, Top Techniques, Top Source IPs
→ Add Suricata Category breakdown
→ Finalize dashboard layout and visual consistency
