# Simulations

This folder contains all SOC simulation reports conducted in the home lab. Each simulation targets a specific attack category, documents the full execution and detection chain, and includes packet captures, Wazuh alerts, and Wireshark analysis as evidence.

This is an ongoing project - new simulations are added regularly as complexity and scope increase.

---

## Index

| # | Title | Category | Key Finding | Status |
|---|-------|----------|-------------|--------|
| 01 | [Network Reconnaissance via Nmap](01-nmap-recon/report.md) | Reconnaissance | Windows Firewall silently drops all recon; Wazuh and Sysmon have zero visibility — network-level IDS required | Complete |
| 02 | [RDP Brute Force Attack & Detection](02-rdp-brute-force/report.md) | Credential Access | Detection is speed-dependent — slow brute force evades correlation entirely; NLA bypassed via NTLM fallback; full compromise chain captured | Complete |
| 03 | [Privilege Escalation via Always Install Elevated](03-priv-esc/report.md) | Privilege Escalation | Standard user escalated to NT AUTHORITY\SYSTEM via crafted MSI payload; default Wazuh config completely blind to the attack — Sysmon ingestion and FIM registry monitoring required to surface high-severity alerts | Complete |

---

## Sim01 — Network Reconnaissance via Nmap

A network reconnaissance simulation against a Windows 10 endpoint across two phases — Windows Defender Firewall enabled and disabled. Eight Nmap scan variants were executed (TCP Connect, SYN, UDP, Aggressive, in both firewall states), with full packet captures analyzed in Wireshark. The central finding was a complete detection gap: neither Wazuh nor Sysmon generated any alerts for network-level scanning activity, regardless of firewall state. With the firewall disabled, the aggressive scan extracted hostname, workgroup, MAC address, SMB configuration, and open ports — demonstrating how much an attacker can learn from reconnaissance alone.

---

## Sim02 — RDP Brute Force Attack & Detection

A five-scenario credential-access simulation covering SSH and RDP brute force against two Windows 10 hosts. Scenarios tested detection at slow and fast attack speeds, a full compromise chain (password cracked, interactive RDP session established), and the protective impact of Network Level Authentication. Key findings include: Wazuh's correlation rule is entirely speed-dependent and blind to slow attacks; NLA is bypassed by Hydra's NTLM fallback path; Wazuh FIM captured anomalous registry churn as a supplementary brute force signal; and OpenSSH on Windows produces empty source-IP fields that prevent correlation entirely. The successful compromise scenario captured the full kill chain from brute force through interactive desktop access.

---

## Sim03 — Privilege Escalation via Always Install Elevated

A privilege escalation simulation targeting the Always Install Elevated GPO misconfiguration on a Windows 10 endpoint. A crafted MSI payload was generated and executed as a standard user, escalating to NT AUTHORITY\SYSTEM. The default Wazuh configuration produced no alerts — the entire attack was invisible. Detection was engineered by adding Sysmon ingestion and targeted FIM registry monitoring, which surfaced high-severity alerts (rule 92213, 61618) tied to the MSI installation and registry modifications. The simulation demonstrates how default SIEM configurations can have critical blind spots for common privilege escalation techniques, and how targeted tuning closes those gaps.
