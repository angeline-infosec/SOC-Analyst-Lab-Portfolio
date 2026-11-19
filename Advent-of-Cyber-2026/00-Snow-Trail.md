# Snow Trail: Advent of Cyber 2026 Warm-Up

**Platform:** ![TryHackMe](https://img.shields.io/badge/TryHackMe-1abc9c?style=flat-square) | **Event:** Advent of Cyber 2026 


## Scenario

Wareville is empty and the TBFC offices have been breached. A Digital Assistant sends five evidence tasks by email, and each one has to be processed on the right device. A correct choice yields one clue about the attacker: when the attack started, where it was geolocated, the IP address, an alias, and a device serial number.

The room is a guided introduction to the interface. It also teaches a habit that carries over to real work where we match the tool to the evidence before starting to process anything.


## Approach

| Task | Evidence | Device used | Why |
|------|----------|-------------|-----|
| 1 | Corrupted 4K surveillance video | Workstation | Heavy processing that runs for a long time |
| 2 | Office network log | Server | Always on, and it holds the network records |
| 3 | Damaged computer in the meeting room | Laptop | Portable, battery-powered, and can be connected to a machine that cannot be moved |
| 4 | Scrambled image | Desktop | Needs processing, but not at video scale |
| 5 | QR code sticker | Phone | Has a camera |


<img width="1190" height="807" alt="image" src="https://github.com/user-attachments/assets/eb9ca886-1659-406a-87e6-b6d61d473656" />

### Task 1: Repair 4K footage

I first sent the video to the laptop, and the repair failed. The error explained that laptops are built to be portable, not for heavy processing. The server also rejected it, since servers store and serve data and are not meant for reviewing footage. The workstation completed the repair, and I used the footage to establish the time of the breach.

### Task 2: Search the network log

The email said the visitor record lives on the machine that manages the office network and has to stay on day and night to catch every connection. That pointed to the server. I ran the provided search command against the network log, filtering on the breach time from Task 1, and it returned the IP address the attacker connected from.

This is the first place the clues chain together. The timestamp from the video is what made it possible to narrow the log to a single entry.
<img width="952" height="546" alt="image" src="https://github.com/user-attachments/assets/3408b837-ece3-4cb1-9500-27a5cf4e272b" />


### Task 3: Recover browser profile

The damaged computer could not be moved, so the email told me to bring a battery-powered portable machine to it. I connected the laptop and ran the recovery tool, which detected a damaged SSD. After recovery I opened the recovered profile file and found the attacker's alias.

### Task 4: Restore evidence image

I sent the scrambled image to the workstation first, but that was more than the job needed. The laptop failed as well, and the desktop rebuilt the image successfully. Inspecting the restored image gave the location where it was taken.

### Task 5: Scan QR code

I scanned the QR code from the sticker on the case file using the phone camera, which gave me the device serial number.


## The Twist

After the last clue, the Digital Assistant warned that the email I had just opened was sent by the attacker. The attacker had taken control of most devices and could still reach anything on the network. 

I had to find and disconnect the three IoT devices in the office, which were the printer, the smart speaker, and the thermostat. The assistant stressed the difference between IoT devices, which do one specific job and connect over Wi-Fi, and embedded devices that don't need the office network.


## Analyst Notes

These are my own reflections and are not stated by the room.

- **Sender address.** In the Task 5 screenshot, the sender reads `d1g1tal.assistant@tbfc.com`, with digits in place of letters, while the earlier emails came from `digital.assistant@tbfc.com`. This is a good example of a lookalike sender. A Tier 1 analyst should check sender addresses character by character whenever an email asks for an action.
- **Trusted channel.** The attacker's email arrived through a channel I already trusted, which is what made it believable. The same mechanism makes business email compromise effective.
- **Order of containment.** Containment came last in the room. In a real incident, I would expect to contain earlier. 
- **Evidence handling.** After some digging, it seems that in real forensics, work is normally done on verified copies of the evidence, with hashes recorded to preserve integrity. The room skips this step..

---

## Takeaways

- Choosing the device is part of the investigation. A wrong choice costs time and in a real case it could also damage evidence.
- Clues from one source narrow the search in another, as the video timestamp did for the network log.
- A message that arrives through a trusted channel still needs its sender verified.
- IoT devices are an easy foothold on a network, so they belong in any containment list.
