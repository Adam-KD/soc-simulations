# Lab Setup Documentation
**Last Updated:** 2026-05-05
**Author:** Adam Kadmany

---

## Overview
A controlled virtual lab environment built for SOC simulation and practice.
The lab simulates real-world attack and defense scenarios using three virtual machines in a segmented network topology, with all traffic routed through a central Ubuntu gateway for centralized monitoring and inspection.

---

## Virtual Machines

| Windows 10 (64-bit)          | Victim/Endpoint             | 192.168.1.10
| Kali Linux - Debian (64-bit) | Attacker                    | 192.168.2.10
| Ubuntu (64-bit)              | Gateway + Wazuh SIEM Server | 192.168.1.1 / 192.168.2.1

---

## Network Architecture

```
Kali Linux (192.168.2.10)
        ↕
Ubuntu Gateway (192.168.2.1 | 192.168.1.1)  ← All traffic inspected here
        ↕
Windows 10 (192.168.1.10)
```

### Network Configuration

| Segment | Network | Machines |
|---------|---------|---------|
| Lab1 | 192.168.1.0/24 | Ubuntu (192.168.1.1) ↔ Windows (192.168.1.10) |
| Lab2 | 192.168.2.0/24 | Ubuntu (192.168.2.1) ↔ Kali (192.168.2.10) |

- **Hypervisor:** Oracle VirtualBox
- **Network Type:** Internal Network (segmented)
- **Internet Access:** NAT adapter on all VMs
- **Kali and Windows are on separate subnets** — communication only possible through Ubuntu gateway
- All lab traffic isolated from host network ✅

### Ubuntu Gateway Configuration
- IP Forwarding enabled (`net.ipv4.ip_forward=1`)
- iptables forwarding rules persistent via `iptables-persistent`
- Routes traffic between Lab1 and Lab2 segments

---

## Security Monitoring Stack

### Wazuh SIEM
- **Server:** Ubuntu (192.168.1.1 / 192.168.2.1)
- **Agent:** Windows 10 (192.168.1.10)
- **Dashboard:** https://localhost (from Ubuntu) or https://192.168.1.1 (from Windows)

### Sysmon
- Installed on Windows 10
- Configuration: SwiftOnSecurity sysmon-config
- Logs forwarded to Wazuh via ossec.conf
- Captures process creation, file activity and network connections

---

## Tools Installed

| Tool | Machine | Purpose |
|------|---------|---------|
| Wazuh Manager | Ubuntu | SIEM server and log aggregation |
| Wazuh Agent | Windows 10 | Endpoint log forwarding |
| Sysmon | Windows 10 | Advanced Windows event logging |
| Nmap | Kali | Network scanning and reconnaissance |
| TCPDump | Ubuntu | Command-line network packet capture |
| Wireshark | Ubuntu | Visual network traffic analysis |
| VS Code | Ubuntu | Configuration file editing |

---

## Known Behaviors
- Windows 10 does not respond to ICMP ping — use `nmap -Pn` to scan
- Kali cannot directly reach Windows — all traffic routes through Ubuntu
- Ubuntu sees all traffic between Kali and Windows (double packets per request — incoming and forwarded)
- Wazuh dashboard accessible via `https://192.168.1.1` from Windows browser

---

## Snapshots

| VM | Snapshot Name | Description |
|----|--------------|-------------|
| Windows 10 | Clean - Segmented Network + Wazuh + Sysmon | Clean baseline after network reconfiguration |
| Ubuntu | Clean - Gateway + Wazuh Server + Segmented Network | Clean baseline after gateway configuration |
| Kali | Clean - Segmented Network | Clean baseline after network reconfiguration |

---

*Lab built as part of SOC home lab practice and portfolio documentation*
*Network redesigned on 2026-05-05 to simulate real SOC segmented network topology*
