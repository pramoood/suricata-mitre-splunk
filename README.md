# **Suricata–Splunk–MITRE Honeypot Analysis Lab**
## Project Overview
A cloud honeypot pipeline that captures attacker telemetry, forwards Suricata IDS alerts into Splunk, enriches them with MITRE ATT&CK, and visualizes the results in a SOC dashboard.

## **Project Goals**
- Capture real attacker traffic using [T‑Pot Honeypot]((https://github.com/telekom-security/tpotce))
- Forward [Suricata IDS](https://suricata.io/) alerts to [Splunk Enterprise](https://www.splunk.com/en_us/products/splunk-enterprise.html)
- Analyze raw IDS logs to identify meaningful attacker behavior
- Filter noise and false positives using SPL queries
- Map IDS alerts to [MITRE ATT&CK](https://attack.mitre.org/) tactics and techniques
- Build a SOC dashboard for threat analysis
- Demonstrate SIEM engineering through log analysis

## **Architecture Overview**
<img width="2292" height="704" alt="image" src="https://github.com/user-attachments/assets/ba3e3c53-2b49-4bd7-98e1-10e6ad1e865a" />

### Windows 10 VM (local, Hyper-V)  
- Splunk Enterprise (indexer + search head)
- Receives forwarded logs from AWS EC2 over port `9997`
  

### AWS EC2 Ubuntu (t3.xlarge)  
- [T-Pot Honeypot (Hive edition) running Suricata IDS + 20+ honeypot daemons (Cowrie, Dionaea, Honeytrap, etc.)](https://github.com/telekom-security/tpotce)
- Splunk Universal Forwarder shipping logs to the SIEM
- Elastic IP for a stable public-facing address
- Security group scoped so admin ports (SSH, web UI) are IP-restricted while honeypot bait ports are open to the internet
- SSH access restricted to trusted IPs and requires the EC2 private key for access


### Connectivity
- [Tailscale WireGuard VPN](https://tailscale.com/) linking the EC2 honeypot and the local Splunk instance, avoiding a fully public data path between the two

### Enrichment Layer  
- Custom CSV lookup tables mapping Suricata alert categories to MITRE tactics/techniques and human-readable names
- SPL-based noise filtering
  

### Visualization Layer  
- Splunk Dashboard Studio
- Splunk dashboard with MITRE tactic/technique breakdowns, alert category timelines, and top source IPs

## **Pipeline Summary**
### 1. Provision Local Log Analysis Environment
- Deploy a Windows 10 VM
- Install Splunk Enterprise (Free Trial) to serve as the primary SIEM


### 2. Deploy Cloud Honeypot Infrastructure
- Launch an Ubuntu AWS EC2 instance (`t3.xlarge`, initially `t3.large` — later upgraded, see Lessons Learned)
- Assign an Elastic IP for stable access
- Restrict admin ports (`SSH 64295`, web `UI 64297`) to a single IP; leave bait ports open to the internet

  <img width="1151" height="634" alt="image" src="https://github.com/user-attachments/assets/e3ca7014-b936-4ce5-8a0a-96f07b7fa849" />


### 3. Establish Secure Connectivity
- Set up Tailscale WireGuard VPN
- Connect the Win10 Splunk VM to the AWS EC2 honeypot
  
  <img width="2035" height="724" alt="image" src="https://github.com/user-attachments/assets/70e9d327-a7b5-4da1-aa58-26c4da6e67f8" />

  

### 4. Install & Configure [T‑Pot Honeypot](https://github.com/telekom-security/tpotce)
- [Install T‑Pot (Hive edition)](https://github.com/telekom-security/tpotce) on the EC2 instance
- Configure inbound firewall rules (SSH, WebUI, Splunk UF ports)
- Verify honeypot containers (Cowrie, Dionaea, Honeytrap, etc.)
- Allow the honeypot to begin collecting attacker telemetry

  

### 5. Install Splunk Universal Forwarder
- Install UF on the EC2 instance
- Configure UF to forward to the Windows Splunk instance on port `9997`
- Monitor T-Pot's Suricata IDS: suricata/log/eve.json — alerts, scans, exploits, signatures
- Create the tpot index in Splunk Enterprise

  <img width="1928" height="548" alt="image" src="https://github.com/user-attachments/assets/3db45345-9b4f-4e3a-be06-0ccbefac1aca" />


### 6. Collect Attacker Telemetry
- Let T‑Pot run to accumulate real attacker events (I used a 24‑hour window)
- Extend the runtime if you want a larger dataset


### 7. SPL Queries for Log Analyzing / Filtering
- Analyze Suricata alert categories
  - (Optional) analyze Suricata signatures for deeper accuracy
- Use SPL to filter noise (Tailscale ports, AWS IMDS traffic, misc categories)
- Remove false positives and reputation‑only alerts
- Create optimized SPL queries for chart creation



### 8. Perform MITRE ATT&CK Mapping Using Lookup Tables with Remaining Alert Cateogories
- Build `alert_mapping.csv` for tactic/technique mapping
- Implement MITRE lookup tables (`mitre_tactic.csv`, `mitre_technique.csv`)


  *For this project, an LLM was used to generate the initial mapping, followed by manual review to make sure the assignments were reasonable. With a larger dataset, more attack categories will naturally appear, and the mapping can be expanded to cover them. There’s also room to take this further by mapping at the `alert.signature` level instead of the `alert.category` level. Signature‑level mapping is more precise and would produce richer MITRE attribution, but it requires researching each signature’s real‑world behavior, which is a solid next step for a future iteration.*

    | Category                         | Tactic(s)                                   | Technique(s)                                           |
    |----------------------------------|----------------------------------------------|---------------------------------------------------------|
    | A Network Trojan was detected    | TA0011 (Command and Control)                 | T1071 (Application Layer Protocol), T1105 (Ingress Tool Transfer) |
    | Attempted Admin Priv Gain        | TA0006 (Credential Access)                   | T1110 (Brute Force)                                     |
    | Attempted DoS                    | TA0040 (Impact)                              | T1499 (Endpoint Denial of Service)                      |
    | Attempted Info Leak              | TA0043 (Reconnaissance)                      | T1592 (Gather Victim Host Information)                  |
    | Attempted User Priv Gain         | TA0006 (Credential Access)                   | T1110 (Brute Force)                                     |
    | Decode RPC Query                 | TA0043 (Reconnaissance)                      | T1046 (Network Service Scanning)                        |
    | Detection of DoS                 | TA0040 (Impact)                              | T1499 (Endpoint Denial of Service)                      |
    | Detection of Network Scan        | TA0043 (Reconnaissance)                      | T1046 (Network Service Scanning)                        |
    | Successful Admin Priv Gain       | TA0004 (Privilege Escalation), TA0006 (Credential Access) | T1078 (Valid Accounts)                     |
    | Web Application Attack           | TA0001 (Initial Access)                      | T1190 (Exploit Public-Facing Application)               |



    <img width="1415" height="1153" alt="image" src="https://github.com/user-attachments/assets/97ae1715-405e-498f-b2f5-55b9ff67e025" />



### 9. Develop SOC‑Grade Splunk Dashboard
- Build MITRE tactic/technique timelines
- Add Suricata Category breakdown (Top Tactics, Top Techniques, Top Source IPs)
  
  <img width="2555" height="1328" alt="image" src="https://github.com/user-attachments/assets/918dad9e-8ceb-4bb0-b220-42dccf5dacbd" />


## Filtering Out Noise: Why 100,000+ Events Became ~4,000

Raw event volume is not the same as attacker activity. A big portion of what an IDS logs is routine infrastructure traffic rather than real adversary behavior. As I worked through the dataset, several sources of noise and false positives showed up that needed to be filtered out before any MITRE mapping would be meaningful:

| Source of noise | Cause | Fix |
|---|---|---|
| ~15,000+ "Network Trojan" alerts on port 41641 | Tailscale's default WireGuard peer-to-peer port, misclassified as malicious traffic | Exclude `src_port=41641` / `dest_port=41641` |
| ~10,000 alerts to `169.254.169.254` | AWS's link-local Instance Metadata Service (IMDS) — not real internet traffic | Exclude `dest_ip=169.254.169.254` |
| Alerts under "Misc Attack" | Entirely IP-reputation flags from a threat-intel feed (CINS Army list) — no actual exploit or malware behavior | Exclude `alert.category="Misc Attack"` |
| Alerts under "Misc Activity," "Not Suspicious Traffic," "Generic Protocol Command Decode" | Categories too broad/ambiguous to reliably tie to a MITRE technique without per-signature analysis | *Excluded for this iteration. Flagged as a future refinement as next step of the project!* |



**Result:** The dataset went from roughly 100,000 raw Suricata events to about 45,000 alert‑type events, and then down to around 4,000 meaningful alerts after filtering out noise and false positives. Those 4,000 alerts, covering 15 attack categories (and 109 distinct attack signatures), which were mapped to the MITRE ATT&CK framework.



## MITRE ATT&CK Mapping

Alert categories were mapped to MITRE tactics and techniques, with an LLM helping generate the initial pass. The final implementation uses three chained Splunk lookup tables:

- [`alert_mapping.csv`](https://github.com/pramoood/suricata-mitre-splunk/blob/main/splunk/lookups/alert_mapping.csv) — maps Suricata `alert.category` to MITRE tactic ID(s) and technique ID(s)
- [`mitre_tactic.csv`](https://github.com/pramoood/suricata-mitre-splunk/blob/main/splunk/lookups/mitre_tactic.csv) — maps tactic ID to tactic name
- [`mitre_technique.csv`](https://github.com/pramoood/suricata-mitre-splunk/blob/main/splunk/lookups/mitre_technique.csv) — maps technique ID to technique name

A single SPL query ties all three lookups together, expands multi‑value tactic and technique fields with `mvexpand`, and produces a clean `tactic_name` and `technique_name` pair for every event. This is what drives the heatmap and timeline panels in the dashboard.

**Known limitation:** the mapping was done at the `alert.category` level, which is fairly broad (around 15 values). Mapping at the `alert.signature` level would be much more accurate because signatures are far more specific (around 109 values), but doing that properly requires researching each signature’s real‑world behavior. *That’s a solid next step, It was just outside the scope of this iteration of the project!*


## SPL Queries
 
A few of the key queries used throughout the project, from initial exploration through to the final MITRE-mapped heatmap dataset.
 
**Baseline alert breakdown**, before any filtering:
```
index=tpot source="/home/ubuntu/tpotce/data/suricata/log/eve.json" event_type=alert
| chart count by alert.category
```
 
**Filtering out Tailscale's WireGuard port**, which was showing up as thousands of false "Network Trojan" alerts:
```
index=tpot source="/home/ubuntu/tpotce/data/suricata/log/eve.json" event_type=alert
src_port!=41641 dest_port!=41641
| chart count by alert.category
```
 
**Breaking down a noisy category by signature**, to confirm it was reputation-list noise rather than real attack behavior:
```
index=tpot source="/home/ubuntu/tpotce/data/suricata/log/eve.json" event_type=alert
src_port!=41641 dest_port!=41641 alert.category="Misc attack"
| chart count by alert.signature
```
 
**Fully filtered baseline**, exluding Tailscale traffic, the AWS metadata endpoint, and low‑value categories. This is the dataset used for MITRE mapping:
```
index=tpot source="/home/ubuntu/tpotce/data/suricata/log/eve.json" event_type=alert
src_port!=41641 dest_port!=41641 dest_ip!=169.254.169.254
(alert.category!="Misc Attack" NOT "Misc Activity" NOT "Not Suspicious Traffic" NOT "Generic Protocol Command Decode")
```
 
**Full MITRE enrichment query**, chaining all lookup tables and expanding multi‑value tactic and technique fields so each technique gets its own row. This powers the heatmap and timeline panels:
```
index=tpot source="/home/ubuntu/tpotce/data/suricata/log/eve.json" event_type=alert
src_port!=41641 dest_port!=41641 dest_ip!=169.254.169.254
(alert.category!="Misc Attack" NOT "Misc Activity" NOT "Not Suspicious Traffic" NOT "Generic Protocol Command Decode")
| lookup alert_mapping.csv alert.category as alert.category OUTPUT tactic technique
| eval tactic=split(tactic,"|"), technique=split(technique,"|")
| mvexpand tactic
| mvexpand technique
| lookup mitre_tactic.csv tactic_id as tactic OUTPUT tactic_name
| lookup mitre_technique.csv technique_id as technique OUTPUT technique_name
```
 
**Aggregating into the heatmap dataset**, added to the end of the query above:
```
| stats count by tactic_name technique_name
```


## Troubleshooting Notes

A few real incidents worth documenting, since diagnosing them was as instructive as the pipeline itself:

1. **Biggest hurdle: ISP CGNAT prevented any direct connection between my home network and the EC2 instance, so I had to switch to Tailscale.** The plan was to connect the home Splunk instance and the AWS honeypot directly over the internet via router port forwarding, firewall rules, etc. It didn't work, and took close to two days to diagnose: I even made a [LinkedIn post](https://www.linkedin.com/feed/update/urn:li:activity:7485780634255163393/) about it.

   
      I learned that my ISP uses Carrier‑Grade NAT, which means my router’s “public” IP is actually a shared upstream address. Because of that, none of my port‑forwarding rules were reachable from the internet. The fix was to stop trying to use a public path and instead link both machines through a Tailscale WireGuard VPN. Once the tunnel was up, the Splunk Forwarder connection on port 9997 worked immediately.

<br>

2. **EC2 became unreachable under attack load.** When I opened the honeypot’s TCP and UDP ports to the internet, the instance became reachable but quickly stopped responding, including SSH. Restarting the VM and briefly closing the honeypot ports made it clear that the attack traffic was overwhelming the `t3.large`. Upgrading to a `t3.xlarge` with 16 GB of RAM solved the issue. T‑Pot lists 8 GB as the minimum, but real internet‑facing traffic needed more headroom.

    See below with `t3.xlarge` (16GB RAM) was running the T-Pot, which was using over 8GB.

    <img width="873" height="91" alt="image" src="https://github.com/user-attachments/assets/5fb8ad9c-c74c-49b8-83ab-d749e638d541" />

<br>

3. **Tailscale silently broke DNS.** After installing Tailscale, `docker compose` on the EC2 instance began failing to pull images, and `curl` couldn't resolve any public hostname. Tailscale replaced the instance’s DNS resolver with one that couldn’t reach public domains, which caused external lookups to fail.

      <img width="1202" height="122" alt="image" src="https://github.com/user-attachments/assets/80153dbd-eb55-4311-8466-04c17003545b" />


   Fix: `sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'` to force Google's public DNS.

      <img width="999" height="165" alt="image" src="https://github.com/user-attachments/assets/fec0d955-7bee-4db9-8098-31af00668d6c" />



## If I Did This Again / Possible Extensions

- Start with at least 16 GB of RAM. T‑Pot’s 8 GB minimum works on paper, but real internet‑facing traffic needs more headroom
- Set up Tailscale from the beginning to avoid CGNAT issues that block direct connections
- Map MITRE techniques using the `alert.signature` field instead of `alert.category` for much more accurate MITRE mapping
- Add GeoIP and ASN enrichment for source IPs so Splunk can show attacker locations and network ownership
- Use an LLM API to automatically generate MITRE technique mappings and extra context for each alert

## Repository Contents

- [`alert_mapping.csv`](https://github.com/pramoood/suricata-mitre-splunk/blob/main/splunk/lookups/alert_mapping.csv), [`mitre_tactic.csv`](https://github.com/pramoood/suricata-mitre-splunk/blob/main/splunk/lookups/mitre_tactic.csv), [`mitre_technique.csv`](https://github.com/pramoood/suricata-mitre-splunk/blob/main/splunk/lookups/mitre_technique.csv) - Splunk lookup tables used for MITRE enrichment
- [`mitre dashboard.json`](https://github.com/pramoood/suricata-mitre-splunk/blob/main/splunk/dashboard/mitre%20dashboard.json) - exported Splunk Dashboard Studio configuration
- `README.md` - You're reading it now :)
