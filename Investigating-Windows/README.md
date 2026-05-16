# Investigating Windows Lab

This section of my SOC analyst portfolio contains hands-on labs focused on Windows incident response and forensic investigation- the process of examining a compromised Windows environment, reconstructing attacker activity, and extracting actionable indicators of compromise (IOCs).


Each lab is documented in a structured investigation format that mirrors real-world SOC and DFIR analyst workflows.

---

## What These Labs Cover

Windows endpoint forensics is a foundational skill in SOC and incident response work. These labs build practical experience in:

- Analyzing Windows Event Logs to trace logon activity, privilege escalation, and account changes
  
- Identifying persistence mechanisms such as malicious scheduled tasks and registry run keys
  
- Investigating credential dumping activity and the tools used to carry it out

- Detecting lateral movement and unauthorized privilege escalation
  
- Extracting network indicators including C2 IPs, backdoor ports, and DNS manipulation
  
- Uncovering web shell deployments and signs of initial access through exposed services
  
- Mapping findings to the MITRE ATT&CK framework to contextualize attacker behavior

---

# Labs


| Lab | Platform | Focus Area |
|---|---|---|
| [Investigating Windows](https://tryhackme.com/room/investigatingwindows) | TryHackMe | Windows forensics, Event Log analysis, persistence, credential dumping, C2 detection |



---

## Platforms & Tools

- TryHackMe - simulated compromised Windows Server environments
  
- Windows Event Viewer - Security and System log analysis (Event IDs 4624, 4672, etc.)
  
- Task Scheduler - persistence mechanism identification
  
- Registry Editor (regedit) - startup entry and run key investigation
  
- Windows Firewall - inbound rule and suspicious port detection
  
- Command Prompt - local group enumeration, user account auditing, network state inspection
  
- MITRE ATT&CK Framework - for mapping detections to known attacker techniques

---

## How These Labs Are Documented

Each lab follows a consistent investigation format:

- Detection summary - what initially indicated a compromise
  
- Attack chain reconstruction - tracing attacker activity from initial access through impact
  
- Tool and artifact analysis - identifying malicious files, scripts, and executables left behind
  
- IOC extraction - cataloging indicators across network, host, and persistence categories
  
- MITRE ATT&CK mapping - tactic and technique identification for each finding
  
- Analyst verdict - a reasoned conclusion grounded in evidence, not just observation
  

This format is intentional. It reflects how I approach investigations: understanding why something is malicious and how it fits into a broader attack chain.

---

## Key Concepts Practiced


| Concept              | Example from Labs                                                                 |
|----------------------|-------------------------------------------------------------------------------------|
| Log Analysis         | Filtering Event ID 4624 / 4672 for logon and privilege events                      |
| Persistence Detection| Scheduled task executing nc.ps1 as a PowerShell backdoor                            |
| Credential Access    | Mimikatz (mim.exe) used for password dumping                                        |
| C2 Communication     | External server at 76.32.97.132 identified via hosts file                          |
| Defense Evasion      | DNS poisoning via modified hosts file entries                                       |
| Initial Access       | .jsp web shell uploaded to IIS wwwroot directory                                    |
| Discovery            | Registry run keys revealing startup connections to attacker infrastructure        |
  
