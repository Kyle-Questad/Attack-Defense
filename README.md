# 🛡️ Attack & Defense SOC Lab

**Created by:** Adam Snyder & Kyle Questad  
**Institution:** Grand Canyon University  
**Project Type:** Self-Paced Cybersecurity Lab

---

## 📋 Project Overview

This project involved designing and implementing a comprehensive virtualized SOC (Security Operations Center) lab environment consisting of six virtual machines connected through a custom-configured LAN to simulate a realistic enterprise network.

On the **defensive side**, we deployed:
- **Wazuh** — centralized SIEM for log aggregation, analysis, and alerting
- **Snort 3** — network-based intrusion detection system (IDS/IPS)
- **Suricata** — network intrusion detection system (NIDS)
- **Custom Snort rules** — tailored to our environment for meaningful alert categorization

On the **offensive side**, we configured a dedicated Kali Linux attacker VM to perform a range of attack techniques including network scanning, brute force, DoS, and Man-in-the-Middle attacks — all detected and correlated within the Wazuh SIEM dashboard.

---

## 🖧 Network Topology

| VM | Hostname | OS | IP Address | Role |
|----|----------|----|------------|------|
| Host 1 | KaliATTACK | Kali Linux | 192.168.100.20 | Attacker |
| Host 1 | WindowsAgent | Windows 10 | 192.168.100.12 | Agent / Victim |
| Host 2 | UbuSIEM | Ubuntu Desktop | 192.168.100.11 | Wazuh SIEM |
| Host 2 | snortubuntu1 | Ubuntu Server | 192.168.100.30 | Snort IDS Server |
| Host 2 | KaliAgent1 | Kali Linux | 192.168.100.10 | Agent / Victim |

All VMs are connected on a private LAN (`192.168.100.0/24`) with a bridged network adapter for LAN communication and a NAT adapter for internet access.

---

## 🔵 Blue Team — Defensive Setup

### Wazuh SIEM
- Installed Wazuh (indexer, manager, and dashboard) on the Ubuntu SIEM VM
- Deployed Wazuh agents on all victim VMs (Kali Agent, Windows Agent, Snort Server)
- Created custom `snort3_rules.xml` and `snort3_decoders.xml` to parse and categorize Snort 3 alerts by attack type and severity
- All alerts successfully forwarded to and displayed in the Wazuh dashboard

### Snort 3 (IDS/IPS)
- Compiled and installed Snort 3 from source on Ubuntu Server VM
- Configured `snort.lua` with community rules (8,496 rules loaded)
- Enabled `alert_fast` output logging to `/var/log/snort/alert_fast.txt`
- Integrated with Wazuh agent using `snort-fast` log format
- Configured as a systemd service (`snort3.service`) for auto-start on boot

### Suricata (NIDS)
- Installed Suricata on the Kali Agent VM
- Configured `suricata.yaml` with correct HOME_NET subnet and network interface
- Created custom ICMP detection rule for lab environment
- Integrated with Wazuh using `eve.json` log format and JSON decoder

---

## 🔴 Red Team — Attack Simulation

All attacks were launched from the attacker VM (`192.168.100.20`) against the victim VM (`192.168.100.10`) and detected by **Snort 3** or **Suricata**, with alerts correlated in the **Wazuh SIEM** dashboard.

### 1. 🔍 Network Reconnaissance (Nmap)
- **Tool:** Nmap
- **Commands:**
  ```bash
  nmap -sS 192.168.100.10       # SYN port scan
  nmap -O 192.168.100.10        # OS fingerprinting
  nmap -sV 192.168.100.10       # Service version scan
  ```
- **Result:** Discovered open ports — FTP (21), SSH (22), HTTP (80) — and identified target OS as Linux 4.15-5.19

### 2. 🔑 SSH Brute Force (Hydra)
- **Tool:** Hydra + rockyou.txt wordlist
- **Command:**
  ```bash
  hydra -l root -P /usr/share/wordlists/rockyou.txt 192.168.100.10 ssh -t 4 -V
  ```
- **Result:** 14,344,399 password attempts launched against SSH service on port 22, detected by Suricata as invalid SSH attempts

### 3. 💥 SYN Flood DoS (hping3)
- **Tool:** hping3
- **Command:**
  ```bash
  hping3 -S --flood -V -p 80 192.168.100.10
  ```
- **Result:** 343,551 packets transmitted to port 80, causing 100% packet loss on the target, triggering over 10,000 alerts in Wazuh

### 4. 🕵️ ARP Spoofing (arpspoof)
- **Tool:** arpspoof
- **Command:**
  ```bash
  arpspoof -i eth1 -t 192.168.100.10 192.168.100.1
  ```
- **Result:** Victim's ARP cache poisoned, redirecting network traffic through the attacker VM — Man-in-the-Middle (MitM) attack successfully executed and detected by Snort

### 5. 🌐 Nikto Web Scan
- **Tool:** Nikto
- **Command:**
  ```bash
  nikto -h 192.168.100.10
  ```
- **Result:** Identified Apache 2.4.66 server with multiple misconfigurations — missing X-Frame-Options and X-Content-Type headers, exposed HTTP methods, and CVE-2003-1418 (information disclosure / XSS vulnerability)

### 6. 📁 FTP Brute Force (Hydra)
- **Tool:** Hydra + rockyou.txt wordlist
- **Command:**
  ```bash
  hydra -l root -P /usr/share/wordlists/rockyou.txt 192.168.100.10 ftp -t 4 -V
  ```
- **Result:** Brute force credential attack against FTP service on port 21, detected and alerted by Suricata in Wazuh

---

## ⚠️ Problems Encountered

- **Snort 3 Configuration:** Snort was compiled from source meaning default config paths didn't exist and the systemd service file required multiple fixes including changing `Type=forking` to `Type=simple` and correcting the network interface before it would run reliably
- **Wazuh Agent Connectivity:** The Wazuh agent repeatedly failed to connect to the manager whenever the Wazuh Manager VM was powered off, requiring manual restarts
- **Custom Decoder Placement:** Snort alerts were not appearing on the dashboard because the custom decoder and rules files were created on the agent VM instead of the Wazuh Manager VM where all log parsing takes place
- **Regex Syntax Errors:** The initial prematch regex in the Snort decoder caused repeated syntax errors preventing the Wazuh Manager from starting, requiring simplification to match Snort 3's alert log format

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| Wazuh | SIEM — log aggregation, alerting, dashboard |
| Snort 3 | Network IDS/IPS |
| Suricata | Network IDS (NIDS) |
| Nmap | Network reconnaissance |
| Hydra | Brute force attacks |
| hping3 | SYN flood DoS |
| arpspoof | ARP spoofing / MitM |
| Nikto | Web vulnerability scanning |
| VirtualBox | VM hypervisor |
| Kali Linux | Attacker and agent OS |
| Ubuntu | SIEM and Snort server OS |
| Windows 10 | Agent OS |

---

## 📚 Resources

- [Suricata Documentation](https://docs.suricata.io/)
- [Wazuh Documentation](https://documentation.wazuh.com/current/user-manual/index.html)
- [Snort 3 Documentation](https://www.snort.org/documents)
- [Kali Linux Docs](https://www.kali.org/docs/)
