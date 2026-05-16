# EDR Investigation Labs

This section of my SOC analyst portfolio contains hands-on labs focused on Endpoint Detection and Response (EDR)-the process of monitoring endpoint activity, triaging alerts, and investigating suspicious behavior in enterprise environments.

Each lab is documented in a structured investigation format that mirrors real SOC analyst workflows.

---

## What These Labs Cover

EDR is a core skill in modern SOC work. These labs build practical experience in:

- Reading and interpreting endpoint telemetry (process execution, file changes, network connections, registry activity)
- Tracing parent-child process relationships to reconstruct attack chains
- Triaging alerts and distinguishing malicious activity from false positives
- Mapping findings to the MITRE ATT&CK framework
- Validating suspicious indicators using threat intelligence
- Understanding EDR response capabilities - isolation, quarantine, and remote access
  
---

# Labs

| Lab | Platform | Focus Area |
|---|---|---|
| [Introduction to EDR](./TryHackMe-Introduction-to-EDR) | TryHackMe | EDR fundamentals, process tree analysis, multi-alert triage |

    

---

## Platforms & Tools

* TryHackMe — simulated enterprise SOC environment

* CrowdStrike Falcon — EDR interface used for alert triage and endpoint investigation

* MITRE ATT&CK Framework — for mapping detections to known attacker techniques

## How These Labs Are Documented

Each lab follows a consistent investigation format:

1. Detection summary - what triggered the alert
2. Process chain analysis - visualizing the parent-child relationships
3. Why it's suspicious - the reasoning behind the assessment, not just the conclusion
4. MITRE ATT&CK mapping - tactic and technique identification
5. Analyst verdict - malicious, benign, or requires further investigation

This format is intentional. It reflects how I approach investigations: understanding why something is a threat, not just that it is one.

---
