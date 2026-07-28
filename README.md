# Wi-Fi Security Assessment — Capstone Project

## Overview
An end-to-end Wi-Fi security assessment conducted in a controlled lab environment,
simulating a small/medium enterprise (SME) wireless network in a Nigerian context.
This project demonstrates the full assessment lifecycle: reconnaissance, vulnerability
analysis, exploitation demonstration (non-destructive), and hardening recommendations —
mapped against the MITRE ATT&CK framework.

## Objectives
- Identify common Wi-Fi misconfigurations found in SME environments (default passwords,
  no channel optimization, absent guest networks, weak encryption)
- Demonstrate reconnaissance and analysis techniques using industry-standard tools
- Map findings to MITRE ATT&CK techniques to connect technical findings to adversary behavior
- Provide concrete, evidence-backed hardening recommendations (before/after configs)

## Tools Used
- Kali Linux (airmon-ng, airodump-ng, aircrack-ng suite)
- Wireshark
- OWASP ZAP
- GNOME Boxes / Cisco Packet Tracer (lab topology)

## Project Structure
- `docs/` — methodology and final report
- `lab-topology/` — network diagram and VM setup
- `evidence/` — PCAPs, screenshots, and logs supporting every finding
- `attck-mapping/` — findings mapped to MITRE ATT&CK techniques
- `hardening/` — before/after AP configuration and remediation steps
- `demo/` — link to walkthrough video

## Scope
See [SCOPE.md](./SCOPE.md) for full scope and lab environment disclaimer.

## Author
Aniagu Chiemela — 3MTT Capstone Project
