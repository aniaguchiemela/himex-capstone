# Scope — Wi-Fi Security Assessment Capstone

## Environment
This assessment is conducted entirely within a **controlled, isolated lab environment**
using virtual machines (Virtual Machine Manager / Cisco Packet Tracer). No production networks,
third-party infrastructure, or real-world access points are targeted at any point.

- **Target:** Simulated "office AP" (VM)
- **Attacker system:** Kali Linux (VM)
- **Network isolation:** Lab VMs are isolated from the host's production network and
  the internet during active testing phases

## In Scope
- Passive and active wireless reconnaissance (airmon-ng, airodump-ng)
- Traffic analysis via Wireshark (beacon frames, handshake capture, DNS traffic)
- Known vulnerability review (e.g. KRACK) as applied to the simulated AP configuration
- WPA2/WPA3 configuration review and upgrade path documentation
- Non-destructive deauthentication capture (packet analysis only — no live client
  disruption)
- Hardening recommendations with before/after configuration evidence
- MITRE ATT&CK mapping of all findings

## Out of Scope
- Any real-world, production, or third-party wireless network
- Denial-of-service or destructive attacks against any live system
- Credential exfiltration against real user accounts
- Any activity outside the isolated lab environment

## Purpose
This project is an educational capstone for the 3MTT program, intended to demonstrate
practical Wi-Fi security assessment methodology applicable to Nigerian SME environments.
All techniques are documented for defensive and educational purposes only.

## Disclaimer
All testing was performed in a self-contained lab with no connection to production
systems or third-party assets. This documentation is not an invitation or instruction
to test networks without explicit authorization.
