# SOC Simulations — Home Lab Portfolio
 
**Author:** Adam Kadmany                                        
**Last Updated:** 2026-05-25
 
A personal cybersecurity home lab built to simulate real-world SOC scenarios, practice threat detection, and document findings in a structured and professional manner.
 
This is an ongoing project — simulations are added continuously as the lab evolves in complexity and scope.
 
---
 
## Lab Architecture
 
```
Kali Linux (192.168.2.10)
        ↕
Ubuntu Gateway (192.168.2.1 | 192.168.1.1)  ← Centralized traffic monitoring
        ↕
   ┌────┴────┐
   ↓         ↓
Win10 Home   Win10 Pro
(192.168.1.10) (192.168.1.20)
```
 
| Machine | Role | IP |
|---------|------|----|
| Kali Linux | Attacker | 192.168.2.10 |
| Ubuntu | Gateway + Wazuh SIEM Server | 192.168.1.1 / 192.168.2.1 |
| Windows 10 Home | Victim/Endpoint | 192.168.1.10 |
| Windows 10 Pro | Victim/Endpoint | 192.168.1.20 |
 
For full lab setup details, see [Lab Setup Documentation](setup/lab-setup.md).
 
---
 
## Simulations
 
| # | Title | Category | Status |
|---|-------|----------|--------|
| 01 | [Network Reconnaissance via Nmap](simulations/01-nmap-recon/report.md) | Reconnaissance | ✅ Complete |
| 02 | [RDP Brute Force Attack & Detection](simulations/02-rdp-brute-force/report.md) | Credential Access | ✅ Complete |
 
---
 
## Skills Demonstrated
 
### Infrastructure & Monitoring
- Network segmentation and gateway-based routing (VirtualBox + iptables)
- SIEM deployment and agent configuration (Wazuh)
- Endpoint telemetry configuration (Sysmon with SwiftOnSecurity config)
### Offensive Tradecraft (Simulation)
- Network reconnaissance using Nmap (TCP Connect, SYN, UDP, Aggressive scans)
- Brute force credential attacks against RDP and SSH (Hydra, ncrack)
- Attack adaptation and tooling substitution under operational constraints
- Attack execution against segmented targets
### Analysis & Detection
- Packet capture and traffic analysis (TCPDump, Wireshark)
- Protocol-level investigation (TCP, UDP, ICMP, SMB/NBSS, DCERPC, RDP/TLS)
- Authentication event correlation analysis and rule threshold testing
- Kill chain documentation from initial access through interactive compromise
- Detection gap identification (speed-dependent correlation, NTLM fallback, SSH source-IP gap)
- Windows FIM registry analysis as a supplementary detection signal
- Detection gap analysis across endpoint vs network monitoring layers
- Comparative testing methodology (controlled variables across scenarios)
### Reporting & Frameworks
- MITRE ATT&CK technique mapping
- Multi-scenario comparative simulation design and documentation
- Vulnerability identification and mitigation recommendations
- Structured SOC incident report writing
---
 
## Tools & Technologies
 
| Tool | Purpose |
|------|---------|
| VirtualBox | Hypervisor |
| Wazuh | SIEM — log aggregation and alerting |
| Sysmon | Windows endpoint monitoring |
| Nmap | Network scanning and reconnaissance |
| Hydra | Brute force authentication testing |
| ncrack | Alternative brute force tool (RDP) |
| rdesktop | RDP client for post-compromise session and NLA testing |
| TCPDump | Network packet capture |
| Wireshark | Visual network traffic analysis |
| Kali Linux | Attack simulation |
 
---
 
*Built as part of a personal cybersecurity learning journey and SOC practice portfolio*
