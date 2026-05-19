# Simulation Report — [Title Here]
 
**Date:** [YYYY-MM-DD]                      
**Attacker Machine:** [e.g. Kali Linux (192.168.2.10)]               
**Victim Machine:** [e.g. Windows 10 Pro (192.168.1.20)]                     
**Objective:** [What is this simulation trying to demonstrate or test?]                             
**Environment State:** [Relevant configuration — e.g. Firewall ON/OFF, NLA ON/OFF, service version. For multi-phase or multi-scenario sims, describe the variants at a high level here and break them out in Section 3.]                          
**Evidence:** [Pointer to pcap files and screenshot folder — e.g. `pcaps/`, `screenshots/`]
 
---
 
## 1. Attack Overview
 
[2–4 sentences: attack type, attacker's goal, lab positioning, and what makes this simulation worth running. Avoid step-by-step detail here — that belongs in Section 3.]
 
---
 
## 2. Tools Used
 
| Tool | Purpose |
|------|---------|
| [Tool name] | [What it was used for in this simulation] |
 
---
 
## 3. Attack Steps (Attacker Side)
 
<!--
If the simulation has a single linear flow, use flat numbered steps (Step 1, Step 2, ...).
If it has multiple scenarios or phases (e.g. Firewall ON/OFF, multiple attack speeds, NLA ON/OFF),
group steps under each scenario as shown below. Either pattern is valid — pick what fits.
-->
 
### Scenario / Phase 1 — [Name]
 
**Setup:** [Conditions for this scenario — what's different about it]
 
#### Step 1 — [Action]
 
[What was done and why]
 
```bash
[command]
```
 
**Result:** [Outcome — what the tool reported]
 
![Screenshot description](screenshots/filename.png)
 
#### Step 2 — [Action]
 
...
 
---
 
### Scenario / Phase 2 — [Name]
 
...
 
---
 
## 4. Victim Side Observations
 
[What was visible on the target host — system behavior, service responses, user-facing artifacts, endpoint logs (Event Viewer, Sysmon events). If the attack was silent at the OS level, state that explicitly — it is a finding.]
 
![Screenshot description](screenshots/filename.png)
 
---
 
## 5. Network Traffic Analysis
 
[Packet-level observations from TCPDump/Wireshark — packet counts, timing, sizes, flags, retransmissions, scan/attack signatures, comparative tables across scenarios if applicable. Focus on *what the traffic looked like*, not what it meant for services — that goes in Section 6.]
 
| Scenario | Packets | Duration | Key Finding |
|----------|---------|----------|-------------|
| | | | |
 
![Screenshot description](screenshots/filename.png)
 
---
 
## 6. Protocol / Service Investigation
 
[Deeper investigation of the protocol(s) or service(s) at the heart of this simulation — e.g. SMB negotiation, RDP handshake details, SSH version banners, DNS query patterns. Include script outputs (Nmap NSE, Hydra, etc.), service-specific configuration findings, and any version/dialect information.
 
If this simulation did not involve a specific application-layer protocol worth investigating, state that and keep the section short.]
 
![Screenshot description](screenshots/filename.png)
 
---
 
## 7. Detection Analysis
 
[What the full detection stack caught or missed:
 
- **Wazuh SIEM:** Which rules fired, at which levels, with which alert counts. Link to specific rule IDs.
- **Sysmon:** Which Event IDs were generated (or notably absent).
- **Windows Event Log:** Native events of interest (e.g. 4624, 4625, 4648).
- **Network layer:** What would have been visible to an IDS/IPS if one were deployed.
If detection failed or was incomplete, explain *why* — this is often the most valuable finding in a simulation.]
 
![Screenshot description](screenshots/filename.png)
 
---
 
## 8. Critical Findings
 
| Finding | Severity | Details |
|---------|----------|---------|
| [Finding] | Critical / High / Medium / Low | [Concrete details — what, where, why it matters] |
 
---
 
## 9. MITRE ATT&CK Mapping
 
| Technique | ID | Description |
|-----------|-----|-------------|
| [Technique name] | [Txxxx[.xxx]] | [How it applies to this simulation specifically — not a generic definition] |
 
---
 
## 10. Analysis
 
[What does this simulation reveal? Why is this attack significant? What would a real attacker do next from this position? What does the success/failure of detection say about the defensive posture? This is the "so what" section — connect the technical findings to operational meaning.]
 
---
 
## 11. Mitigation & Recommendations
 
| Finding | Recommendation | Priority |
|---------|---------------|----------|
| [Finding from Section 8] | [Specific, actionable remediation — config path, GPO setting, tool to deploy, rule to tune] | High / Medium / Low |
 
---
 
## 12. Lessons Learned
 
[Personal takeaways from running this simulation — what worked, what didn't, what was surprising, what to do differently next time. Honest reflection, not marketing.]
 
---
 
*Report generated as part of SOC Home Lab practice*
*Lab environment: see [Lab Setup Documentation](../../setup/lab-setup.md)*
