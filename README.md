# Cisco CyberOps & Cyber Essentials — Lab Portfolio

This repository documents hands-on labs completed as part of the
**Cisco CyberOps Associate** and **Cisco Networking Essentials** programmes.
Each lab was completed in a simulated or virtualised environment using
Cisco Packet Tracer or the Cisco CyberOps LabVM, with findings documented
independently rather than copied from lab guides.

**Tools used:** Cisco Packet Tracer · Wireshark · Linux CLI (Ubuntu) · VirtualBox

---

## Labs

| # | Lab | Tool | Core Skill |
|---|-----|------|------------|
| 01 | [NetFlow Traffic Observation](lab-01-netflow-observation/) | Packet Tracer | Interpreting flow records, 5-tuple analysis, DNS vs HTTP traffic |
| 02 | [Syslog, AAA, and NetFlow Monitoring](lab-02-syslog-aaa-netflow/) | Packet Tracer | Centralised logging, TACACS+ access auditing, SIEM concepts |
| 03 | [Wireshark: Telnet vs SSH](lab-03-wireshark-telnet-ssh/) | Wireshark · Linux | Packet capture, plaintext credential exposure, encryption verification |
| 06 | [Incident Response: System Information Gathering](lab-06-incident-response-system-info/) | Linux CLI | Volatile data collection, open port triage, log analysis |

> Labs 04 and 05 are in progress and will be added on completion.

---

## Skills Demonstrated

**Network Traffic Analysis**
Reading and interpreting NetFlow records, identifying protocols by port and
IP protocol number, understanding unidirectional flow capture, and
distinguishing DNS, HTTP, ICMP, and UDP traffic in a collector.

**Security Monitoring and Logging**
Configuring and observing Syslog severity levels, correlating AAA
authentication records (TACACS+) with login and logout events, and
understanding why centralised logging is a prerequisite for effective
incident detection.

**Packet Capture and Protocol Analysis**
Using Wireshark to capture traffic on specific interfaces, applying display
filters, and directly observing the difference between plaintext (Telnet)
and encrypted (SSH) protocol behaviour at the packet level — including
credential exposure character-by-character.

**Incident Response — First Response**
Collecting volatile system data (network interfaces, open ports, running
processes, routing table, login history) from a potentially compromised
Linux host before shutdown, in the correct order to preserve the most
time-sensitive evidence first. Identifying anomalous findings including
unexpected listening services, unknown login attempts, and suspicious
routing entries.

---

## Cyber Essentials Control Mapping

| Cyber Essentials Control | Labs |
|--------------------------|------|
| Firewalls | 01 — understanding traffic flows is foundational to firewall rule design |
| Secure Configuration | 03 — direct demonstration of why Telnet must be disabled in favour of SSH |
| User Access Control | 02 — TACACS+ centralised authentication and session auditing |
| Malware Protection | — |
| Patch Management | — |
| Security Monitoring *(Plus)* | 02, 06 — Syslog, AAA logging, and volatile data collection for IR |

---

## Repository Structure

```
cyber-essentials-labs/
├── README.md                              ← this file
├── lab-01-netflow-observation/
├── lab-02-syslog-aaa-netflow/
├── lab-03-wireshark-telnet-ssh/
└── lab-06-incident-response-system-info/
```

Each lab folder contains:
- `README.md` — objectives, findings, analysis, and reflection
- `screenshots/` — evidence from the lab
- `topology/` — Packet Tracer files and network diagrams (where applicable)
- `report/` — exported artefacts such as incident reports (where applicable)

