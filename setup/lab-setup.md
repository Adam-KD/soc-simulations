\# Initial Lab Setup Documentation



\*\*Date:\*\* 2026-05-03

\*\*Author:\*\* Adam Kadmany



\---



\## Overview

A controlled virtual lab environment built for SOC simulation and practice.

The lab simulates real-world attack and defense scenarios using three virtual machines.



\---



\## Virtual Machines



| Machine | OS | Role | IP Address |

|---------|-----|------|------------|

| Windows 10 | Victim/Endpoint | 192.168.0.20 |

| Kali Linux | Attacker | 192.168.0.30 |

| Ubuntu Server - 22.04 LTS | Wazuh SIEM Server | 192.168.0.10 |



\---



\## Network Configuration

\- \*\*Hypervisor:\*\* Oracle VirtualBox

\- \*\*Network Type:\*\* Internal Network

\- \*\*Network Name:\*\* Lab

\- \*\*Subnet:\*\* 192.168.0.0/24

\- All VMs isolated from host network



\---



\## Security Monitoring Stack



\### Wazuh SIEM

\- \*\*Server:\*\* Ubuntu 22.04 LTS (192.168.0.10)

\- \*\*Agent:\*\* Windows 10 (192.168.0.20)

\- \*\*Dashboard:\*\* https://192.168.0.10



\### Sysmon

\- Installed on Windows 10

\- Configuration: SwiftOnSecurity sysmon-config

\- Logs forwarded to Wazuh via ossec.conf



\---



\## Tools Installed



| Tool | Machine | Purpose |

|------|---------|---------|

| Wazuh Manager | Ubuntu | SIEM server and log aggregation |

| Wazuh Agent | Windows 10 | Endpoint log forwarding |

| Sysmon | Windows 10 | Advanced Windows event logging |

| Nmap | Kali | Network scanning and reconnaissance |





\*Lab built as part of SOC home lab practice and portfolio documentation\*

