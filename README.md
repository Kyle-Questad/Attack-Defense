# 🛡️ Attack & Defense SOC Lab

**Created by:** Adam Snyder & Kyle Questad
**Institution:** Grand Canyon University
**Project Type:** Self-Paced Cybersecurity Lab

---

## 📋 Project Overview

In this project, we designed and implemented a comprehensive virtualized lab environment consisting of six virtual machines connected through a custom-configured LAN to simulate a realistic enterprise network. The lab incorporated a variety of operating systems including Linux, Ubuntu, and Windows to reflect the diversity of systems commonly found in real-world infrastructures.

On the **defensive side**, we deployed:
- **Wazuh** — centralized SIEM for log aggregation, analysis, and alerting
- **Snort 3** — network-based intrusion detection system (IDS/IPS)
- **Suricata** — network intrusion detection system (NIDS)
- **Custom Snort rules** — tailored to our environment for meaningful alert categorization

On the **offensive side**, we configured a dedicated Kali Linux attacker VM to perform a range of offensive techniques including network scanning, brute force, DoS, and Man-in-the-Middle attacks — all detected and correlated within the Wazuh SIEM dashboard.

---

## 🖧 Network Topology

| Hostname | OS | IP Address | Role |
|----------|----|------------|------|
| KaliATTACK | Kali Linux | 192.168.100.20 | Attacker VM |
| UbuSIEM | Ubuntu Desktop | 192.168.100.11 | Wazuh SIEM |
| snortubuntu1 | Ubuntu Server | 192.168.100.30 | Snort IDS Server |
| KaliAgent1 | Kali Linux | 192.168.100.10 | Agent / Victim |
| WindowsAgent | Windows 10 | 192.168.100.12 | Agent / Victim |

All VMs are connected on a private LAN (`192.168.100.0/24`) with a bridged network adapter for LAN communication and a NAT adapter for internet access.

---

## 📁 Repository Structure

```
Attack-Defense-SOC-Lab/
├── README.md                        ← Project overview (this file)
├── docs/
│   ├── 1-network-setup.md           ← VM and LAN configuration
│   ├── 2-blue-team.md               ← Wazuh, Snort, Suricata setup
│   ├── 3-red-team.md                ← Attack walkthrough
│   └── 4-problems-encountered.md   ← Issues and fixes
├── rules/
│   ├── snort3_rules.xml             ← Custom Snort detection rules
│   └── snort3_decoders.xml          ← Custom Wazuh Snort decoder
└── screenshots/
    ├── blue-team/                   ← Defensive tool screenshots
    └── red-team/                    ← Attack screenshots
```

---

## 🔵 Blue Team Summary

We deployed Wazuh as our centralized SIEM to aggregate, correlate, and analyze security events, providing real-time visibility into system and network activity. We integrated both Snort 3 and Suricata as IDS solutions, and developed a comprehensive custom Snort ruleset tailored to our lab environment, improving alert categorization and log readability within Wazuh.

→ [Full Blue Team Walkthrough](docs/2-blue-team.md)

---

## 🔴 Red Team Summary

All attacks were launched from the attacker VM (`192.168.100.20`) against the victim VM (`192.168.100.10`) and detected by **Snort 3** or **Suricata**, with alerts correlated in the **Wazuh SIEM** dashboard.

| Attack | Tool | Detection |
|--------|------|-----------|
| Network Reconnaissance | Nmap | Snort 3 |
| SSH Brute Force | Hydra | Suricata |
| SYN Flood DoS | hping3 | Suricata |
| ARP Spoofing | arpspoof | Snort 3 |
| Web Vulnerability Scan | Nikto | Snort 3 |
| FTP Brute Force | Hydra | Suricata |

→ [Full Red Team Walkthrough](docs/3-red-team.md)

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| Wazuh | SIEM — log aggregation, alerting, dashboard |
| Snort 3 | Network IDS/IPS |
| Suricata | Network NIDS |
| Nmap | Network reconnaissance |
| Hydra | Brute force attacks |
| hping3 | SYN flood DoS |
| arpspoof | ARP spoofing / MitM |
| Nikto | Web vulnerability scanning |
| VirtualBox | VM hypervisor |

---

## 📚 Resources

- [Suricata Documentation](https://docs.suricata.io/)
- [Wazuh Documentation](https://documentation.wazuh.com/current/user-manual/index.html)
- [Snort 3 Documentation](https://www.snort.org/documents)
- [Kali Linux Docs](https://www.kali.org/docs/)
