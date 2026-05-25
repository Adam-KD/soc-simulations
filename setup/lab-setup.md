# Lab Setup Documentation
 
**Last Updated:** 2026-05-19
**Author:** Adam Kadmany
 
---
 
## Overview
 
A controlled virtual lab environment built for SOC simulation and practice.
The lab simulates real-world attack and defense scenarios using four virtual machines in a segmented network topology, with all traffic routed through a central Ubuntu gateway for centralized monitoring and inspection.
 
---
 
## Virtual Machines
 
| Machine | OS | Role | IP Address |
|---------|-----|------|------------|
| Kali | Debian (x64) | Attacker | 192.168.2.10 |
| Ubuntu Server | Ubuntu (x64) | Gateway + Wazuh SIEM Server | 192.168.1.1 / 192.168.2.1 |
| Windows 10 Home | Windows 10 Home (x64) | Victim/Endpoint | 192.168.1.10 |
| Windows 10 Pro | Windows 10 Pro (x64) | Victim/Endpoint | 192.168.1.20 |
 
The two Windows hosts coexist intentionally — Home and Pro differ in features (notably, Pro supports RDP server while Home does not), which allows different attack surfaces to be exercised against equivalent baselines.
 
---
 
## Network Architecture
 
```
Kali Linux (192.168.2.10)
        ↕
Ubuntu Gateway (192.168.2.1 | 192.168.1.1)  ← All traffic inspected here
        ↕
   ┌────┴────┐
   ↓         ↓
Win10 Home   Win10 Pro
(192.168.1.10) (192.168.1.20)
```
 
### Network Configuration
 
| Segment | Network | Machines |
|---------|---------|----------|
| Lab1 | 192.168.1.0/24 | Ubuntu (192.168.1.1) ↔ Win10 Home (192.168.1.10) ↔ Win10 Pro (192.168.1.20) |
| Lab2 | 192.168.2.0/24 | Ubuntu (192.168.2.1) ↔ Kali (192.168.2.10) |
 
- **Hypervisor:** Oracle VirtualBox
- **Network Type:** Internal Network (segmented)
- **Internet Access:** NAT adapter on all VMs
- **Kali and Windows hosts are on separate subnets** — communication only possible through Ubuntu gateway
- All lab traffic isolated from host network ✅
### Ubuntu Gateway Configuration
 
- IP Forwarding enabled (`net.ipv4.ip_forward=1`)
- iptables forwarding rules persistent via `iptables-persistent`
- Routes traffic between Lab1 and Lab2 segments
---
 
## Security Monitoring Stack
 
### Wazuh SIEM
 
- **Server:** Ubuntu (192.168.1.1 / 192.168.2.1)
- **Agents:**
  - Windows 10 Home (192.168.1.10)
  - Windows 10 Pro (192.168.1.20)
- **Dashboard:** https://localhost (from Ubuntu) or https://192.168.1.1 (from either Windows host)
### Sysmon
 
- Installed on both Windows hosts (Home and Pro)
- Configuration: SwiftOnSecurity sysmon-config
- Logs forwarded to Wazuh via `ossec.conf`
- Captures process creation, file activity, and network connections
---
 
## Tools Installed
 
| Tool | Machine | Purpose |
|------|---------|---------|
| Wazuh Manager | Ubuntu | SIEM server and log aggregation |
| Wazuh Agent | Windows 10 Home, Windows 10 Pro | Endpoint log forwarding |
| Sysmon | Windows 10 Home, Windows 10 Pro | Advanced Windows event logging |
| TCPDump | Ubuntu | Command-line network packet capture |
| Wireshark | Ubuntu | Visual network traffic analysis |
| VS Code | Ubuntu | Configuration file editing |
| Nmap | Kali | Network scanning and reconnaissance |
| Hydra | Kali | Brute force authentication testing |
| OpenSSH Server | Windows 10 Home | SSH service for SSH-based attack scenarios |
| RDP (Remote Desktop) | Windows 10 Pro | Native RDP service for RDP-based attack scenarios |
 
---
 
## Known Behaviors
 
- Windows hosts do not respond to ICMP ping by default — use `nmap -Pn` to scan.
- Kali cannot directly reach the Windows hosts — all traffic routes through Ubuntu.
- Ubuntu sees all traffic between Kali and the Windows hosts (double packets per request — incoming and forwarded).
- Wazuh dashboard accessible via `https://192.168.1.1` from any Windows host browser.
### Detection Gaps Identified
 
- **OpenSSH on Windows — empty `ipAddress` field:**
  When OpenSSH Server on Windows logs authentication failures, the `ipAddress` field in the resulting Windows event is empty. As a result, Wazuh rule **60204** (*Multiple Windows Logon Failures*), which correlates failed logins by source IP within a time window, does not fire on SSH brute force attempts against Windows. Individual failure events (rule 60122) still log, but the multi-failure correlation never triggers. This is a known limitation to account for in any SSH brute force scenario targeting Windows hosts.
---
 
## Snapshots
 
| VM | Snapshot Name | Description |
|----|--------------|-------------|
| Kali | Clean - Segmented Network | Clean baseline after network reconfiguration |
| Ubuntu | Clean - Gateway + Wazuh Server + Segmented Network | Clean baseline after gateway configuration |
| Windows 10 Home | Clean - Segmented Network + Wazuh + Sysmon | Clean baseline after network reconfiguration |
| Windows 10 Pro | Clean - Pre-Sim02 (Wazuh + Sysmon + RDP enabled) | Clean baseline prior to Simulation 02 (RDP brute force) |
 
---
 
*Lab built as part of SOC home lab practice and portfolio documentation*
*Network redesigned on 2026-05-05 to simulate real SOC segmented network topology*
*Windows 10 Pro host added on 2026-05-19 to support RDP-based attack scenarios*
