# Task 2 - New User Creation Monitoring

### Objective
Detect when a new local user account is created on Windows, which can indicate persistence.

### How I Simulated It
1. On Windows Agent (DESKTOP-R1UESBFB), I created a new user: net user testuser Password123! /add
2. This action generated a Windows Security Event ID 4720

### Wazuh Detection
Wazuh detected it in real-time. Rule: 4720 - A user account was created.
Dashboard shows user: testuser, created by: Respect

### Screenshot Evidence
(Screenshot will be added next)

### SOC Analyst Analysis
This is important for SOC because attackers create accounts to maintain access. If we see a new user we didn't expect, we must investigate and disable it.

### Mitre ATT&CK
T1136.001 - Create Account: Local Account
