---
title: Releasing Qradar101 - SIEM Case Investigation
classes: wide
categories:
  - Blueteaming
tags:
  - SIEM
---

![Qradar 101 Challenge](/imgs/IBM Qradar.png)

I have released a challenge based on Qradar SIEM consisting of 24 questions. The challange is hosted at Cyberdefenders platform. I tried as much as possible to be realistic in the scenario and mimic a real world incident. Almost all the questions can be solved by digging into sysmon logs.

<!-- more -->

### Dataset

- Sysmon - swift on security configuration
- Powershell logging
- Windows Eventlog
- Suricata IDS
- Zeek logs (conn, HTTP)


### The scenario

A financial company was compromised, and they are looking for a security analyst to help them investigate the incident. The company suspects that an insider helped the attacker get into the network, but they have no evidence.  

The initial analysis performed by the company's team showed that many systems were compromised. Also, alerts indicate the use of well known malicious tools in the network. As a SOC analyst, you are assigned to investigate the incident using QRadar SIEM and reconstruct the events carried out by the attacker.

[Qradar101 - SIEM Case Investigation](https://cyberdefenders.org/labs/39)
