# SIEM Fundamentals Labs

This section of my SOC analyst portfolio contains labs focused on Security Information and Event Management (SIEM)-the centralized platform SOC analysts use to collect, correlate, and act on logs from across an organization's systems.

Each lab is documented in a structured investigation format that mirrors real SOC analyst workflows.

---

## What These Labs Cover

SIEM is the foundation a SOC analyst works out of day to day. These labs build practical experience in:

- Understanding host-centric vs. network-centric log sources
- Recognizing why isolated, uncorrelated logs fail to tell the full story of an incident
- Understanding how SIEM solves centralization, normalization, and correlation problems at scale
- Reading and building detection rules from normalized log fields
- Investigating triggered alerts and classifying them as true or false positive
- Mapping SIEM detections to the MITRE ATT&CK framework
- Understanding where key activity (process execution, RDP sessions, log clearing) actually gets logged

---

## Labs

| Lab | Platform | Focus Area |
| --- | --- | --- |
| [Introduction to SIEM](https://github.com/angeline-infosec/SOC-Analyst-Lab-Portfolio/blob/main/Introduction%20to%20SIEM/Introduction-to-SIEM.md) | TryHackMe | SIEM fundamentals, log sources and ingestion, detection rules, alert investigation |

---

## Platforms & Tools

- TryHackMe - simulated enterprise SOC environment
- Splunk - SIEM platform used for log search, dashboards, and alert investigation
- MITRE ATT&CK Framework - for mapping detections to known attacker techniques

---

## How These Labs Are Documented

Each lab follows a consistent investigation format:

1. Concept breakdown - the SIEM fundamentals the lab is built around
2. Detection rule logic - how raw log fields become a triggered alert
3. Investigation walkthrough - tracing the alert back to the responsible process, user, and host
4. Classification - true positive or false positive, and the reasoning behind it
5. Response action - what was actually done once the alert was confirmed

This format is intentional. It reflects how I approach investigations: understanding why something is a threat, not just that it is one.


---

premium labs:

Junior Security Analyst Intro

Splunk: The Basics

Incident Handling with Splunk

Benign

Investigating with Splunk

Investgating with ELK

ItsyBitsy
