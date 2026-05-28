# Introduction to SIEM Lab


This lab introduces the fundamentals of Security Information and Event Management (SIEM) systems, the core platform used by analysts in security operations centers (SOCs) to detect, investigate, and respond to security incidents.


## What this room covered

* How different devices generate logs (host-centric vs. network-centric)
* Why centralised log collection and normalisation are critical
* How SIEM correlates logs across sources to surface attack patterns
* How detection rules work and real-time alerting is triggered
* How to investigate and classify alerts as true or false positives

The lab culminates in a hands-on incident investigation where you identify a cryptominer (cudominer.exe) on a compromised workstation and take appropriate response action.

## Lab

[THM-Introduction-to-SIEM]() — Full writeup covering:

* SIEM fundamentals and architecture
* Log source types and ingestion methods
* Detection rule mechanics with real examples
* Step-by-step incident investigation and findings
* Key analyst takeaways
  

## Key Findingd

| Finding           | Value                 |
| ----------------- | --------------------- |
| Malicious Process | `cudominer.exe`       |
| Responsible User  | `chris`               |
| Affected Host     | `HR_02`               |
| Rule Match Term   | `miner`               |
| Alert Type        | `True Positive`       |
| Completion Flag   | `THM{000_SIEM_INTRO}` |


## Why This Matters

-> SIEM is foundational. Every SOC analyst must understand log collection, normalisation, and correlation.
-> Rules aren't magic. Understanding how detection rules match on field-value pairs helps you write better detection logic and tune out false positives.
-> Context is everything. The same suspicious process on different hosts may warrant completely different responses.
-> Incident classification is critical. Distinguishing true positives from false positives is a core competency measured in SOC interviews.

Completed as part of SOC analyst portfolio and training pathway.




