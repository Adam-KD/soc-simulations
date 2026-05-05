# SOC Simulations — Home Lab Portfolio

**Author:** Adam Kadmany  
**Last Updated:** 2026-05-05

A personal cybersecurity home lab built to simulate real-world SOC scenarios, practice threat detection, and document findings in a structured and professional manner.

---

## Lab Architecture

```
Kali Linux (192.168.2.10)
        ↕
Ubuntu Gateway (192.168.2.1 | 192.168.1.1)  ← Centralized traffic monitoring
        ↕
Windows 10 (192.168.1.10)
```

| Machine | Role | IP |
|---------|------|----|
| Windows 10 | Victim/Endpoint | 192.168.1.10 |
| Kali Linux | Attacker | 192.168.2.10 |
| Ubuntu | Gateway + Wazuh SIEM Server | 192.168.1.1 / 192.168.2.1 |

For full lab setup details see [Lab Setup Documentation](setup/lab-setup.md)

---

## Simulations

| # | Title | Category | Status |
|---|-------|----------|--------|
| 01 | [Network Reconnaissance via Nmap](simulations/01-nmap-recon/report.md) | Reconnaissance | ✅ Complete |

---

## Skills Demonstrated

- Network segmentation and routing configuration
- SIEM deployment, configuration and log analysis (Wazuh)
- Endpoint monitoring (Sysmon)
- Network traffic capture and analysis (TCPDump, Wireshark)
- Attack simulation and documentation
- Vulnerability identification and remediation
- MITRE ATT&CK framework mapping
- Structured incident report writing

---

## Tools & Technologies

| Tool | Purpose |
|------|---------|
| VirtualBox | Hypervisor |
| Wazuh | SIEM — log aggregation and alerting |
| Sysmon | Windows endpoint monitoring |
| Nmap | Network scanning and reconnaissance |
| TCPDump | Network packet capture |
| Wireshark | Visual network traffic analysis |
| Kali Linux | Attack simulation |

---

## Repository Structure

```
soc-simulations/
├── README.md
├── setup/
│   └── lab-setup.md
├── templates/
│   └── simulation-report-template.md
└── simulations/
    └── 01-nmap-recon/
        ├── report.md
        └── screenshots/
```

---

*Built as part of a personal cybersecurity learning journey and SOC practice portfolio*
