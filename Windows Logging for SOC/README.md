# Windows Logging Labs

This section of my SOC analyst portfolio contains labs focused on Windows logging - the endpoint log sources SOC analysts read every day to triage alerts, hunt for threats, and reconstruct what happened during an incident.

Each lab is documented in a structured investigation format that mirrors real SOC analyst workflows.

## What These Labs Cover

Windows logs are where most endpoint investigations start. These labs build practical experience in:

- Finding and reading Windows event logs (`.evtx`) with Event Viewer
- Identifying authentication activity (4624 / 4625) and spotting a brute force attack followed by a successful RDP login
- Tracking user account creation and privileged group changes (4720 / 4732) to catch backdoor accounts
- Using Sysmon to monitor process creation, file, registry, DNS, and network activity
- Reading a Sysmon Event ID 1 field by field and building a process tree
- Correlating events across log sources with the Logon ID and ProcessId
- Tracking PowerShell activity through the command history file
- Mapping findings to the MITRE ATT&CK framework
- Understanding where key activity (logins, new accounts, process execution, PowerShell commands) actually gets logged

## Labs

| Lab | Platform | Focus Area |
|-----|----------|------------|
| [Windows Logging for SOC](https://github.com/angeline-infosec/SOC-Analyst-Lab-Portfolio/blob/main/Windows%20Logging%20for%20SOC/Windows-Logging-for-SOC.md) | TryHackMe | Windows event logs, Security log analysis, Sysmon, PowerShell history, event correlation |

## Platforms & Tools

- **TryHackMe** - guided lab environment with a Windows host (THM-PC)
- **Windows Event Viewer** - reading, filtering, and searching event logs
- **Sysmon (Sysinternals)** - detailed process, file, registry, DNS, and network logging
- **PowerShell history files** - recovering commands typed in a PowerShell terminal
- **MITRE ATT&CK Framework** - for mapping findings to known attacker techniques
- **Nmap and Remmina** - connecting to the lab host over RDP from a Linux VM

## How These Labs Are Documented

Each lab follows a consistent investigation format:

1. **Concept breakdown** - the log sources and event IDs the task is built around
2. **Investigation walkthrough** - the filters used, the fields read, and the evidence found, step by step
3. **Attack chain reconstruction** - connecting separate events into one timeline using the Logon ID and ProcessId
4. **MITRE ATT&CK mapping** - each finding tied to a known technique
5. **Detection improvements** - what alerts or logging would catch the activity earlier
6. **Interview Q&A** - questions a SOC interviewer might ask about the topic

This format is intentional. It reflects how I approach investigations: understanding why something is a threat, not just that it is one.
