<img width="701" height="410" alt="image" src="https://github.com/user-attachments/assets/c4b919fd-8324-4259-99cf-e2e1bc65e94f" />

---


Quiet as the Grave
Scenario overview
TryHatMe Studios is a small, fast-rising game house with a reputation bigger than its headcount, riding the buzz around its most anticipated release yet, NeuralNexus, a platformer about a newly departed soul working through the afterlife's paperwork, one numbered ticket at a time. For a studio this size, the unreleased build and the commercial plans around it are effectively the whole business.
That is exactly what makes it a target. Financially motivated crews that once chased banks and payment processors have started turning toward creative studios, where a single leaked project can be resold or held to ransom. So when game developer Sophia Reyes opened what looked like a routine project-review file one afternoon, nobody thought twice about it.
In the days that followed, the security team began to wonder whether that quiet afternoon had been the opening move of something larger, something reaching from Sophia's workstation toward the studio's most closely held work. It would not be the first time something unwelcome had tried to move into the studio. The question for this hunt is how far it went, and whether anything got out.
Scenario objectives
Establish how the actor first gained access to the environment and how that foothold was kept alive over time.
Trace the credentials the actor used to operate, and how it moved from one system to the next.
Determine how far the actor's reach extended, including any attempts against higher-value systems such as domain infrastructure, and the broader reconnaissance it carried out along the way.
Determine what data the actor was after, how it was collected, and whether any of it ultimately left the environment.
60 mins
+1100 xps
hard

Step 1 of 3
Review threat intel and hypothesis
The Threat Intel hub contains IOCs, attack timelines, analyst notes, and a working hypothesis to assess. Study the intel closely to understand the full context and spot any gaps. Your task is to prove or disprove the hypothesis through investigation.
You don’t need to decide the hypothesis yet, think of it as a guiding framework for your SIEM and VM investigation. Stay open: follow the evidence, whether it supports or challenges this initial theory.

Step 2 of 3
Investigate and reconstruct the attack chain
Use the SIEM and virtual machine to trace the attack. Correlate logs, analyze forensic data, and map each step the adversary took. Build your attack chain to understand how and why it happened.
A predefined timeline outlines the key stages of the intrusion. Use it to populate each phase of your attack chain. Fill in every step to complete the end-to-end picture of the breach.

Step 3 of 3
Review report and conclude
Submit your attack chain and review the AI-generated report, which details the timeline from initial foothold to full compromise. Then, state whether you accept or refute the hypothesis. Hit Submit to finish.
A ready-to-use narrative template awaits, simply fill in each section to tell the full story of the breach and record whether you accept or refute the hypothesis.

---

Quiet as the Grave
Briefing
TryHatMe Studios is a small, fast-rising game house with a reputation bigger than its headcount, riding the buzz around its most anticipated release yet, NeuralNexus, a platformer about a newly departed soul working through the afterlife's paperwork, one numbered ticket at a time. For a studio this size, the unreleased build and the commercial plans around it are effectively the whole business.

That is exactly what makes it a target. Financially motivated crews that once chased banks and payment processors have started turning toward creative studios, where a single leaked project can be resold or held to ransom. So when game developer Sophia Reyes opened what looked like a routine project-review file one afternoon, nobody thought twice about it.

In the days that followed, the security team began to wonder whether that quiet afternoon had been the opening move of something larger, something reaching from Sophia's workstation toward the studio's most closely held work. It would not be the first time something unwelcome had tried to move into the studio. The question for this hunt is how far it went, and whether anything got out.

Hypothesis
Consistent with Pinstripe Moth's reported tradecraft, TryHatMe Studios may have been compromised through a lure delivered to one of its developers, giving the actor a foothold from which it used valid credentials and legitimate Windows tooling to move through the environment toward the studio's unreleased NeuralNexus work. This hunt sets out to confirm whether that activity is present, how far through the network the actor reached, and whether any of that data was taken.

Objectives
Establish how the actor first gained access to the environment and how that foothold was kept alive over time.
Trace the credentials the actor used to operate, and how it moved from one system to the next.
Determine how far the actor's reach extended, including any attempts against higher-value systems such as domain infrastructure, and the broader reconnaissance it carried out along the way.
Determine what data the actor was after, how it was collected, and whether any of it ultimately left the environment.










