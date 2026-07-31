# Wi-Fi Security Assessment — Final Report

## Executive Summary
This report documents a Wi-Fi security assessment of an MTN Broadband 4G MiFi
(ZLT M30S), conducted in a self-contained lab environment as a 3MTT capstone project.
The assessment simulates the kind of small/medium enterprise (SME) wireless setup
common in Nigeria, where a MiFi or similar mobile hotspot often serves as the primary
office network.

Using passive and active reconnaissance techniques, the assessment identified one
significant finding — WPS enabled and exposed via beacon frames — which was
remediated and verified during the engagement. Additional findings around device
fingerprinting and WPA2/WPA3 posture are documented for completeness, along with
recommendations suited to the SME context.

## Objective
To assess the security posture of a real, owned wireless access point using
industry-standard tools, document every finding with reproducible evidence, map
findings to the MITRE ATT&CK framework, and apply/verify at least one concrete
hardening action.

## Scope
Full scope details are in `SCOPE.md`. In summary: all testing was performed against
a device owned by the assessor (the MiFi), using an owned client device (a phone) as
the only test client. No destructive actions, credential recovery attempts, or
third-party network interaction occurred during this assessment.

## Methodology
Full methodology is documented in `docs/methodology.md`. The assessment followed four
phases: passive reconnaissance, active reconnaissance and analysis, vulnerability and
configuration review, and hardening/remediation.

## Target Overview
| Attribute | Value |
|---|---|
| Device | MTN Broadband 4G MiFi ZLT M30S |
| SSID | MTN_4G_45C30F |
| BSSID | 98:A9:42:45:C3:0F |
| Encryption (initial) | WPA2-PSK-CCMP |
| WPS status (initial) | Enabled, Configured |
| Admin interface | 192.168.0.1 |

## Findings

### Finding 1 — Passive reconnaissance exposes device metadata
A purely passive beacon scan (no active interaction with the AP) revealed the SSID,
BSSID, channel, encryption type, manufacturer OUI, radio chipset (Realtek), and
regulatory country code (CN). Full detail in `docs/beacon-frame-analysis.md`.

**Severity:** Low. These are metadata leaks inherent to how 802.11 beacons work, not
misconfigurations. They contribute to device fingerprinting but do not on their own
grant network access.

### Finding 2 — WPS enabled, exposing a weaker authentication path
The AP advertised WPS as enabled and "Configured" in its beacon frames. WPS's 8-digit
PIN validation is a well-documented weaker alternative to attacking the WPA2
passphrase directly. Full detail in `docs/beacon-frame-analysis.md`.

**Severity:** High (relative to other findings). This was the only finding that
represented an actual weakening of access control rather than passive metadata
exposure.

**Status:** Remediated during this assessment — see Hardening section below.

### Finding 3 — WPA2-only, no WPA3 support (hardware limitation)
The device does not support WPA3, confirmed via its admin panel. This is a hardware/
firmware ceiling, not a misconfiguration — there is no setting to change. Full detail
in `docs/research-krack-wpa3.md`.

**Severity:** Low, contingent on client patch status. The test client device was
confirmed patched (2026-05-01 security patch level) and therefore not exposed to
KRACK-class key-reinstallation attacks despite the AP's WPA2-only ceiling.

**Status:** Documented as a residual limitation; mitigated by client-side patch
management rather than an AP-side fix, since no AP-side fix exists for this hardware.

### Finding 4 — WPA2 handshake successfully captured (expected behavior, documented for completeness)
A legitimate WPA2 4-way handshake was captured during a normal client reassociation,
confirmed via `aircrack-ng` (`1 handshake` reported). No password recovery or offline
cracking was attempted, in line with scope. This finding demonstrates that the
handshake is capturable by a nearby passive observer — a precondition for any future
offline brute-force attempt, not an attack in itself.

**Severity:** Informational. Handshake capturability is inherent to how WPA2
authentication works and is not, by itself, a vulnerability — passphrase strength is
what determines real-world risk from a captured handshake, which was out of scope to
test.

### Finding 5 — Nmap service scan reveals outdated embedded Linux kernel and exposed services

Active scanning of the admin interface (192.168.0.1) identified three open TCP ports:
- **Port 53** — dnsmasq 2.80 (DNS resolver for connected clients)
- **Port 80/443** — "Demo-Webs" HTTP/HTTPS admin panel
- **OS fingerprint** — Linux kernel 3.2–3.16 (2012–2016 era)

dnsmasq 2.80 is affected by the DNSpooq vulnerability cluster
(CVE-2020-25681 through CVE-2020-25687), a set of heap overflow and
DNS cache poisoning flaws patched in dnsmasq 2.83. This device
runs 2.80 and cannot be user-upgraded.

The embedded Linux kernel version is consistent with long-unsupported
firmware — kernel 3.16 reached end-of-life and no longer receives
security patches. This is common for IoT/embedded network hardware
but represents an unresolvable residual risk for this device class.

**Positive finding:** The admin panel returns strong HTTP security headers
(HSTS, X-Frame-Options, X-Content-Type-Options, X-XSS-Protection),
indicating reasonable baseline web hardening by the manufacturer.

**Severity:** Medium (dnsmasq CVEs), High (kernel EOL — unpatchable)
**Status:** Documented as residual hardware limitation. No user-side fix available.
Full detail in `docs/nmap-scan-findings.md`.

## MITRE ATT&CK Mapping
Full mapping table in `attck-mapping/findings-to-attck.md`. Summary:

| Finding | Tactic | Technique |
|---|---|---|
| Passive beacon scan | Reconnaissance | Wireless Sniffing (T1040) |
| Metadata leakage (OUI, country, chipset) | Reconnaissance | Gather Victim Host Information (T1592) |
| Handshake capture | Credential Access | Steal or Forge Auth Certificates (T1557.002) |
| WPS enabled | Credential Access | Brute Force (T1110) |
| KRACK exposure (theoretical) | Credential Access | Adversary-in-the-Middle (T1557) |
| dnsmasq 2.80 / kernel 3.x EOL | Reconnaissance | Gather Victim Host Info (T1592.001) |
## Hardening Applied
Full before/after documentation in `hardening/before-after-config.md`. Summary:

- **Action taken:** Disabled WPS via the MiFi's admin panel
- **Verification:** Post-hardening beacon scan confirmed the WPS vendor tag no longer
  appears in beacon frames — the change was verified at the protocol level, not just
  the admin UI
- **Result:** The weaker WPS authentication path is eliminated; the network now relies
  solely on WPA2-PSK-CCMP for access control

## Recommendations for SME Deployments
1. **Disable WPS by default** on any AP deployed in an SME environment — the
   convenience it offers rarely outweighs the brute-force exposure it introduces.
2. **Track client device patch levels**, since this class of AP cannot be upgraded
   past WPA2 — the practical defense against KRACK-class issues sits with the client
   fleet, not the AP.
3. **Document AP configuration baselines** so that hardening actions (like disabling
   WPS) survive device resets or replacements, rather than being a one-time,
   undocumented fix.
4. **Budget for WPA3-capable hardware** over time. WPA3 removes KRACK-class exposure
   structurally rather than relying on ongoing client patch discipline.
5. **Replace aging hardware with actively maintained firmware** — devices
   running end-of-life embedded Linux kernels (3.x era) cannot receive
   security patches regardless of admin panel settings. For SMEs handling
   sensitive data, hardware refresh cycles should account for firmware
   support windows, not just physical failure rates.

## Limitations
- A non-destructive deauthentication capture was planned but deferred due to testing
  being conducted in a shared network environment (a hub with multiple unrelated
  Wi-Fi networks present) where isolating the target safely could not be guaranteed
  at the time. This phase remains a candidate for future work in a controlled,
  isolated environment.
- Passphrase strength was not tested as part of this assessment; it remains a
  residual consideration for a complete security posture but was out of scope here.

## Conclusion
This assessment demonstrates a complete, evidence-backed Wi-Fi security review cycle
on a real device: reconnaissance, handshake capture, vulnerability research, and a
verified hardening action. The most actionable finding — WPS exposure — was
identified, remediated, and confirmed fixed at the protocol level within the scope of
this engagement, providing a concrete, reproducible example of the assessment
methodology applied end-to-end.
