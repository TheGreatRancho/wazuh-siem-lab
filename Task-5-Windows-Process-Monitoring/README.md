# Task 5 - Windows Process Monitoring (Sysmon + Wazuh)

## Objective
Verify Sysmon process creation / termination events are collected locally and forwarded to Wazuh for detection.

## Lab Setup
- Endpoint: DESKTOP-R1UESBF (Agent 001, 192.168.18.4)
- Manager: wazuh-server (192.168.18.62)
- Sysmon installed with config added to Wazuh agent
- Wazuh Agent: WazuhSvc - Running

## Test Performed
Controlled process test in PowerShell (Admin):

notepad.exe; Start-Sleep 2; Stop-Process -Name notepad -Force

Test completed: 2026-11-08 12:33:19
Second test: net user via net1.exe for discovery activity.

## Local Verification (Sysmon Operational Log)
Microsoft-Windows-Sysmon/Operational
- 2026/11/08 12:27:37 - Id 1 - Process Create
- 2026/11/08 12:27:21 - Id 1 - Process Create
- 2026/11/08 12:27:06 - Id 5 - Process terminated

Proves Sysmon is logging process lifecycle locally.

## Wazuh Verification (Threat Hunting > Events)
Filter: data.win.eventdata.image: *notepad*
Result: 15 hits

Example alerts:
- Rule 92031 (Level 3) - Discovery activity executed
- Rule 92066 (Level 4) - Binary in suspicious location launched

Document Details for net user event:
- agent.name: DESKTOP-R1UESBF
- agent.ip: 192.168.18.4
- data.win.eventdata.commandLine: C:\Windows\system32\net1 user
- data.win.eventdata.image: C:\Windows\SysWOW64\net1.exe
- data.win.eventdata.company: Microsoft Corporation
- data.win.eventdata.hashes: SHA256=FE9C3CB1D90A1F489DAA71357AC78C1CBAC9FE8FE415AD441F70FCF8CAE98944A831
- data.win.eventdata.integrityLevel: System
- Index: wazuh-alerts-4.x-2026.11.08

## Investigation / Analyst Analysis
1. Triage: Process Create (Sysmon 1) + Wazuh rule 92031 = expected for notepad.exe and net1.exe test.
2. Indicators: Full command line, image path, hash, integrity level, parent, user, and timestamp all present - high fidelity telemetry.
3. Context: net user / net1.exe is a common Discovery technique. Alone it is low severity, but combined with other recon commands it signals attacker enumeration.
4. Verdict: TRUE POSITIVE - Expected lab test activity. Detection pipeline works end-to-end: Endpoint -> Sysmon -> Wazuh Agent -> Wazuh Manager -> Dashboard.

## MITRE ATT&CK
- T1057 - Process Discovery
- T1087.001 - Account Discovery: Local Account (net user)
- T1036 - Masquerading (if binary in suspicious location - Rule 92066)

## Recommendation
- Alert on net.exe, net1.exe, whoami.exe, qwinsta.exe, nltest.exe when run by non-admin / unusual parent (e.g. Word, Outlook, browser).
- Keep Sysmon Event 1, 5, 11, 3 logging enabled.
- Baseline normal notepad.exe, SecEdit.exe activity to reduce false positives.

## Evidence
- 01-sysmon-local-events.jpg - Sysmon Operational log showing Event 1 and 5
- 02-notepad-test.jpg - PowerShell test execution
- 03-wazuh-notepad-15-hits.jpg - Wazuh Threat Hunting, 15 hits for notepad
- 04-wazuh-net1-document-details.jpg - Document Details for net user event

## Conclusion
Windows Process Monitoring is WORKING. Sysmon events are successfully forwarded to Wazuh and trigger detection rules.
