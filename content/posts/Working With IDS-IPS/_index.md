---
title: "Working With IDS IPS"
date: 2026-04-17T14:33:33+02:00
draft: false
toc: false
images:
tags:
  - untagged
---

# Introduction To IDS/IPS

- ***Network security monitoring (NSM)*** relies on `Intrusion Detection Systems (IDS)` and `Intrusion Prevention Systems (IPS)` to:
  - Identify potential Threats.
  - Mitigate their impact.
- An `Intrusion Detection System (IDS)`:
  - Monitors network or system activity for malicious behavior and policy violations.
  - Reporting to a management station to provide network visibility.
- While an `IDS` alerts to intrusions, it does not prevent them.

- The IDS operates in two main modes:
  - Signature-Based Detection.
  - Anomaly-Based Detection.
- In `signature-based detection`:
  - The IDS recognizes bad patterns, such as malware signatures and previously identified attack patterns.
  - However, this method is limited to only known threats.
- For this reason, `anomaly-based detection` is implemented:
  - Which establishes a baseline of normal behavior and sends an alert when it detects behavior deviating from this baseline.
  - It's a more proactive approach but is susceptible to false positives, hence why both methods are used to balance each other out.

- An `IPS`, positioned behind the firewall, actively prevents detected threats, providing an additional layer of protection by not only alerting to intrusions but also actively stops them from entering.
- An `IPS` also operates in a similar mode to IDS, offering both `signature-based` and `anomaly-based detection`.
- Once a potential threat is detected, it takes actions such as:
  - Dropping malicious packets.
  - Blocking network traffic.
  - Resetting the connection.
- The goal is to interrupt or halt activities that are deemed dangerous to our network or against our policy.

- IDS and IPS are typically deployed on the internal network side, behind the firewall, to:
  - Maximize visibility of relevant traffic.
  - Effectively protect resources.

- ***Intrusion Detection Systems (IDS)***:
  - Passively monitor network traffic for threats and generate alerts.
  - Positioned behind the firewall, they analyze traffic that has bypassed initial defenses, focusing on complex threats.
- ***Intrusion Prevention Systems (IPS)***:
  - Actively block detected threats.
  - Deployed inline, typically directly behind the firewall, IPS do both monitor and block malicious traffic.

- Intrusion Detection/Prevention Systems (IDS/IPS) deployment depends on network requirements and traffic monitoring needs.
- These systems can be implemented at the host level (HIDS/HIPS) to monitor individual host traffic for suspicious activity.
- Placement of these systems is integral to a defense-in-depth strategy.
- The specific architecture will vary based on:
  - Network characteristics/nature.
  - Data sensitivity.
  - The threat landscape.

## IDS/IPS Updates

- Moreover, to ensure these systems perform at their best, consistently:
  - Update them with the latest threat signatures.
  - Fine-tune their anomaly detection algorithms.
- This requires a diligent, ongoing effort from our security team, but it's absolutely essential given the continually evolving threat landscape.

- Security Information and Event Management (SIEM) systems are also crucial.
- By aggregating logs from IDS/IPS and other network devices, SIEMs:
  - Correlate events and use advanced analytics to detect complex attacks.
  - Providing a unified security view for rapid threat response.

---

# Suricata

- [Suricata](suricata.md/).

---

# Snort

- [Snort](snort.md/).

---

# Zeek

- [Zeek](zeek.md/).

---
