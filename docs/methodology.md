# Methodology

This assessment follows a four-phase methodology, mirroring standard wireless
penetration testing practice while emphasizing evidence capture and ATT&CK mapping
at every step.

## Phase 1 — Passive Reconnaissance
**Goal:** Gather information about the target AP without sending any packets to it.

- Enable monitor mode: `airmon-ng start <interface>`
- Run `airodump-ng` to passively capture beacon frames, SSID, BSSID, channel,
  encryption type, and connected client MACs
- Analyze beacon frame contents in Wireshark to identify information leakage
  (vendor info, hidden SSID reveal attempts, supported rates)

**ATT&CK mapping:** Reconnaissance (TA0043), Discovery (TA0007)

**Evidence captured:** Raw `airodump-ng` output, initial PCAP, screenshots of
scan results → saved to `evidence/logs/` and `evidence/pcaps/`

## Phase 2 — Active Reconnaissance & Analysis
**Goal:** Interact with the target to extract deeper information.

- Capture WPA/WPA2 handshake (client-triggered or via non-destructive
  deauth-and-capture)
- Analyze handshake for weak/default PSK exposure risk
- Test for KRACK vulnerability applicability given the AP's protocol version
- Review DNS traffic for exfiltration-style patterns (mirroring SOC-streak DNS
  telemetry work)

**ATT&CK mapping:** Credential Access (TA0006), Exfiltration (TA0010)

**Evidence captured:** Handshake PCAP, Wireshark filter screenshots, deauth
capture (non-destructive) → saved to `evidence/pcaps/` and `evidence/screenshots/`

## Phase 3 — Vulnerability & Configuration Review
**Goal:** Identify concrete misconfigurations against known best practices.

- Compare current AP config against vendor hardening guide
- Assess encryption protocol (WPA2 vs WPA3), channel width, and beacon rate
- Identify SME-specific risks: default admin credentials, absent guest network,
  no client isolation

**ATT&CK mapping:** Initial Access (TA0001)

## Phase 4 — Hardening & Remediation
**Goal:** Apply and document fixes, with before/after evidence.

- Apply configuration changes (WPA3 where supported, strong PSK, channel
  optimization, guest network segregation)
- Re-run Phase 1–2 scans post-hardening to confirm improvement
- Document before/after configs and scan results side-by-side

**Evidence captured:** Before/after config screenshots and scan comparisons →
saved to `hardening/before-after-config.md`

## Reproducibility
Every command run, tool version, and timestamp is logged alongside its evidence
file, so each finding can be independently verified or reproduced.
