# Simulation Report — Network Reconnaissance via Nmap

**Date:** 2026-05-03  
**Attacker Machine:** Kali Linux (192.168.0.30)  
**Victim Machine:** Windows 10 (192.168.0.20)  
**Objective:** Minimally simulating an attacker's initial reconnaissance step via Nmap scans, highlighting the importance of Windows Defender Firewall as a first line of defense, and the need for network traffic analysis capabilities through IDS/IPS solutions rather than relying solely on a SIEM for network-level threat detection.

---

## 1. Attack Overview

In this simulation, an attacker positioned on the same internal network segment performed network reconnaissance against a Windows 10 endpoint using Nmap. The simulation was conducted in two phases — first with Windows Defender Firewall enabled, then with it disabled — to evaluate the impact of the firewall on both attack surface exposure and SIEM detection capability.

---

## 2. Tools Used

| Tool | Purpose |
|------|---------|
| Nmap | Conduct network reconnaissance and port scanning to simulate an attacker's initial discovery phase |
| Wazuh SIEM | Aggregate endpoint logs and evaluate detection capability against network reconnaissance activity |
| Sysmon | Installed for endpoint monitoring, however incoming network connections were not captured due to Sysmon's design limitations for this scenario |
| Windows Defender Firewall | Evaluated as a first line of defense — tested in both enabled and disabled states to measure its impact on attack surface and detection visibility |

---

## 3. Attack Steps (Kali)

### Step 1 — Host Discovery
Before scanning, confirmed the target was reachable on the network.

```bash
nmap -sn 192.168.0.20
```

**Result:** Target did not respond to ICMP ping requests. Windows Defender Firewall was blocking ICMP echo requests, however the host was confirmed alive based on prior network configuration verification.

---

### Step 2 — TCP Connect Scan (Firewall ON)

```bash
nmap -sT -v 192.168.0.20
```

**Result:** All 1000 ports returned as ignored state. Firewall successfully blocked all connection attempts. Approximately 1 packet sent and received.

---

### Step 3 — SYN Stealth Scan (Firewall ON)

```bash
nmap -sS -v 192.168.0.20
```

**Result:** Identical result to TCP Connect scan — all ports ignored. Notable difference: 2001 packets sent due to Nmap's default retransmission behavior of 2 attempts per port plus 1 host discovery packet, yet still fully blocked by the firewall.

---

### Step 4 — UDP Scan (Firewall ON)

```bash
nmap -sU -v 192.168.0.20
```

**Result:** All ports ignored. 2091 packets sent, 4 received — likely ICMP port unreachable responses. No open UDP ports identified.

---

### Step 5 — Aggressive Scan (Firewall ON)

```bash
nmap -A -v 192.168.0.20
```

**Result:** All ports ignored. 2049 packets sent, 1 received. Additional scan intensity provided no advantage against an active firewall.

---

### Step 6 — TCP Connect Scan (Firewall OFF)

```bash
nmap -sT -v 192.168.0.20
```

**Result:** 997 ports closed, 3 open — **135 (RPC), 139 (NetBIOS), 445 (SMB).** Disabling the firewall immediately revealed the real attack surface of the target.

![TCP Scan Results - Firewall OFF](screenshots/tcp-scan-firewall-off.png)

---

### Step 7 — SYN Stealth Scan (Firewall OFF)

```bash
nmap -sS -v 192.168.0.20
```

**Result:** Same open ports identified as TCP Connect scan — 135, 139, 445. Stealth scan produced identical findings confirming consistent attack surface exposure.

---

### Step 8 — UDP Scan (Firewall OFF)

```bash
nmap -sU -v 192.168.0.20
```

**Result:** Eight UDP ports identified. Nmap automatically increased probe delay from 0ms to 400ms due to high packet drop rates, indicating Windows rate-limits ICMP port unreachable responses — naturally slowing UDP reconnaissance regardless of firewall state.

| Port | Service | State |
|------|---------|-------|
| 137 | NetBIOS Name Service | Open |
| 138 | NetBIOS Datagram | Open\|Filtered |
| 500 | ISAKMP | Open\|Filtered |
| 1900 | UPnP | Open\|Filtered |
| 4500 | NAT-T-IKE | Open\|Filtered |
| 5050 | MMCC | Open\|Filtered |
| 5353 | Zeroconf/mDNS | Open\|Filtered |
| 5355 | LLMNR | Open\|Filtered |

![UDP Scan Results - Firewall OFF](screenshots/udp-scan-firewall-off.png)

---

### Step 9 — Aggressive Scan (Firewall OFF)

```bash
nmap -A -v 192.168.0.20
```

**Result:** Revealed detailed system intelligence including hostname, workgroup, MAC address, SMB configuration and security posture.

![Aggressive Scan Results](screenshots/aggressive-scan-firewall-off.png)

---

## 4. SMB Investigation

Following the discovery of port 445, a deeper investigation into SMB configuration and vulnerability was conducted.

### SMB Protocol Enumeration

```bash
nmap -Pn -p 445 --script smb-protocols 192.168.0.20
```

**Result:** SMB protocol enumeration revealed that despite SMBv1 being enabled as a Windows feature, it is not advertised or negotiated by the system. Only SMBv2 and SMBv3 dialects were active:

| Dialect | Version |
|---------|---------|
| 2:0:2 | SMBv2 |
| 2:1:0 | SMBv2.1 |
| 3:0:0 | SMBv3 |
| 3:0:2 | SMBv3.0.2 |
| 3:1:1 | SMBv3.1.1 |

---

### EternalBlue Vulnerability Check (MS17-010)

```bash
nmap -Pn -p 445 --script smb-vuln-ms17-010 192.168.0.20
```

**Result:** The vulnerability check was inconclusive. Without an active SMBv1 dialect being negotiated, the EternalBlue exploit has no attack surface — explaining the failed result. This highlights the importance of protocol-level verification over feature-level checks alone.

---

### SMB Security Configuration

```bash
nmap -Pn -p 445 --script smb2-security-mode 192.168.0.20
```

**Finding:** Message signing enabled but not required.

**Implication:** SMB traffic is not mandatorily authenticated, leaving the target potentially vulnerable to SMB Relay attacks — where an attacker intercepts and relays SMB authentication to gain unauthorized access without needing credentials.

![SMB Investigation Results](screenshots/smb-investigation.png)

---

## 5. Detection Analysis

### Wazuh SIEM

**Firewall ON:** No scanning-related alerts generated. Wazuh received no relevant logs from the Windows agent during any of the scans.

**Firewall OFF:** No change in detection capability. Wazuh still generated no alerts related to the scanning activity regardless of firewall state.

**Root Cause:** Network-level reconnaissance does not generate endpoint logs that Wazuh is designed to capture in its default configuration.

### Sysmon

Sysmon was active and logging process and file activity (Event IDs 1, 11, 13) throughout the simulation. However, incoming network connection attempts from an external source are not captured by Sysmon by design — it monitors outbound connections initiated by Windows processes, not inbound reconnaissance traffic.

### Detection Gap Identified

Both Wazuh and Sysmon failed to detect any scanning activity throughout the entire simulation. This is a fundamental architectural limitation — endpoint monitoring tools are not designed to detect network-level reconnaissance. Windows Defender Firewall successfully prevented access when enabled, however it is a prevention tool, not a detection one — it provides no visibility or alerting capability.

This reveals a critical detection gap: network reconnaissance requires dedicated network-level inspection tools such as an IDS/IPS solution capable of analyzing raw network packets and alerting on scanning behavior.

---

## 6. Analysis & Findings

### Attack Surface

Through reconnaissance alone, the attacker successfully profiled the target without any authentication or exploitation. The following intelligence was gathered:

- **Hostname:** WINDOWS10
- **Workgroup:** WORKGROUP
- **MAC Address:** 08:00:27:aa:45:df
- **Open TCP Ports:** 135 (RPC), 139 (NetBIOS), 445 (SMB)
- **Open UDP Ports:** 137, 138, 500, 1900, 4500, 5050, 5353, 5355
- **SMB Version:** SMBv2/3 active, SMBv1 not negotiated
- **SMB Security:** Message signing not required — SMB Relay possible
- **LLMNR Active:** Vulnerable to LLMNR poisoning

This level of intelligence is sufficient to plan a targeted follow-up attack.

### Firewall Impact

Windows Defender Firewall proved highly effective at reducing attack surface. With the firewall enabled, all 1000 scanned ports appeared in ignored state, revealing nothing to the attacker. With the firewall disabled, the true attack surface was immediately exposed. This demonstrates that a properly configured firewall is a critical first line of defense, significantly limiting reconnaissance capability.

### Critical Findings

| Finding | Severity | Details |
|---------|----------|---------|
| SMB Port 445 Open | High | Exposes file sharing service, historically exploited |
| SMB Signing Not Required | High | Enables SMB Relay attacks without credentials |
| LLMNR Active (UDP 5355) | Medium | Vulnerable to LLMNR poisoning via Responder |
| NetBIOS Active (137/139) | Medium | Leaks hostname, workgroup and system information |
| SMBv1 Enabled | Low | Not actively negotiated but should be disabled |
| No Network IDS | High | Network reconnaissance goes completely undetected |

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Network Service Discovery | T1046 | Nmap port scanning against target |
| System Network Configuration Discovery | T1016 | Hostname and workgroup enumeration via NBstat |
| SMB/Windows Admin Shares | T1021.002 | SMB exposure on port 445 |
| LLMNR/NBT-NS Poisoning | T1557.001 | LLMNR active and exploitable |

---

## 7. Mitigation & Recommendations

| Finding | Recommendation | Priority |
|---------|---------------|----------|
| SMB Signing Not Required | Enforce SMB signing via Group Policy: `Computer Configuration → Windows Settings → Security Settings → Local Policies → Security Options → "Microsoft network client: Digitally sign communications (always)" → Enabled` | High |
| LLMNR Active | Disable LLMNR via Group Policy: `Computer Configuration → Administrative Templates → Network → DNS Client → "Turn off multicast name resolution" → Enabled` | High |
| NetBIOS Active | Disable NetBIOS over TCP/IP on all network adapters unless legacy systems require it | Medium |
| SMBv1 Enabled | Disable SMBv1 completely: `Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol` | Medium |
| No Network IDS | Deploy Suricata or Snort to inspect raw network packets and alert on scanning behavior in real time | High |
| SIEM Detection Gap | Integrate network IDS logs into Wazuh for centralized alerting — Suricata integrates natively with Wazuh | High |

---

## 8. Lessons Learned

**1. Windows Defender Firewall is stronger than expected by default.**
Windows Defender Firewall proved surprisingly effective as a default defense. Being deeply integrated into Windows and operating at the network layer, it successfully neutralized all reconnaissance attempts without any additional configuration — making it a critical protection layer even for non-technical users.

**2. SIEMs require significant engineering to be effective.**
SIEM solutions like Wazuh require significant tuning and engineering to balance false positives, noise, and true positives. Equally important are false negatives — critical events that go undetected. This balance varies heavily depending on the organization, environment, and systems being monitored.

**3. Real SOC work requires continuous testing and tuning.**
In a real SOC environment, all potential reconnaissance techniques should be tested against the SIEM, with detection gaps documented and alerts continuously tuned to improve coverage while minimizing noise.

**4. This is just the beginning.**
This simulation motivates further exploration of advanced reconnaissance techniques, higher-level Nmap scripts, and additional tools used in real-world attacks — progressively building complexity in future simulations.

---

*Report generated as part of SOC Home Lab practice*
*Lab environment: VirtualBox — Internal Network "Lab" (192.168.0.0/24)*