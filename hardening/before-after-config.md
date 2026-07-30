# Hardening — Before/After Configuration

## Context
Based on findings from `docs/beacon-frame-analysis.md`, WPS (Wi-Fi Protected Setup) was
identified as the most actionable hardening target: a weak alternate authentication path
that, if exploited, could grant an attacker the WPA2 PSK without needing to crack the
passphrase directly. This document shows the before/after state of this configuration
change, verified at both the admin UI level and the protocol level (via beacon frame
analysis).

## Finding to remediate
- **Device:** MTN Broadband 4G MiFi ZLT M30S
- **Finding:** WPS enabled and advertising in beacon frames
- **Risk:** Weak brute-force target (8-digit PIN) vs. strong WPA2 passphrase
- **Remediation:** Disable WPS in admin settings
- **Verification method:** re-scan and confirm WPS vendor tag absence in beacons

---

## Before state

**Admin panel screenshot:**
See `evidence/screenshots/Screenshotmifi.png` — WPS Switch radio button set to "Enabled".

**Protocol-level evidence:**
From `docs/beacon-frame-analysis.md` and `evidence/logs/baseline_scan-06.cap`:
```
Tag: Vendor Specific: Microsoft Corp.: WPS
Type: WPS (0x04)
Wifi Protected Setup State: Configured (0x02)
```

The beacon actively advertised WPS capability and that it was in "Configured" state,
meaning a client or attacker could attempt to use the WPS PIN mechanism to join the
network.

---

## Hardening action

**Procedure:**
1. Accessed MiFi admin panel at `192.168.0.1` (default gateway)
2. Navigated to Device Settings → WPS Settings
3. Changed WPS Switch from "Enabled" to "Disabled"
4. Clicked "Apply" to save the change

**Time to implement:** < 2 minutes

---

## After state

**Admin panel screenshot:**
See `evidence/screenshots/Screenshotmifi2.png` — WPS Switch radio button set to "Disabled".

**Protocol-level verification:**
Post-hardening scan captured in `evidence/logs/post_hardening_scan-02.cap` and analyzed
in Wireshark with beacon filter (`wlan.fc.type_subtype == 0x08`).

**Finding:** No WPS vendor tag present in post-hardening beacons. The beacon frame
details (vendor-specific tags, capabilities, etc.) no longer include any Microsoft Corp.
WPS tag or state advertisement.

See `evidence/screenshots/ScreenshotOFWIRESHARKPOST.png` for protocol verification.

---

## Impact

| Aspect | Before | After |
|---|---|---|
| WPS advertised in beacon | Yes | No |
| Alternate weak auth path | Yes (8-digit PIN) | No |
| Admin panel setting | Enabled | Disabled |
| Client impact | None expected (no clients were actively using WPS) | None |

**Practical effect:** An attacker passively observing this MiFi can no longer attempt
a WPS PIN brute-force as an alternate route to network access. The network now relies
solely on the WPA2-PSK handshake and passphrase strength for authentication.

---

## Residual considerations

While WPS is now disabled, the MiFi still operates on WPA2-only (no WPA3 support, per
`docs/research-krack-wpa3.md`). This hardening action removes one known weakness
(WPS) without changing the baseline encryption protocol. The network remains protected
by WPA2-PSK-CCMP, which is adequate for a small/medium enterprise (SME) environment
provided:

- Client devices connecting to this network remain patched (confirmed phone at
  2026-05-01 patch level)
- The passphrase itself is strong (in-scope but not attacked in this assessment)
- Regular AP firmware updates are applied when available

---

## Recommendations for Nigerian SME context

1. **Disable WPS by default on all access points** — WPS convenience rarely justifies
   the PIN-bruteforce vulnerability, especially in environments where devices are
   preconfigured by IT rather than self-onboarded by users.

2. **Document AP configurations as part of IT baseline** — store a copy of the admin
   panel settings (sanitized of passwords) in a shared IT knowledgebase so future
   device replacements can be configured identically.

3. **Enforce client patch management** — since this AP cannot upgrade to WPA3, the
   practical defense is ensuring all connected devices (staff phones, laptops) receive
   OS security patches regularly. This is often the weakest link in SME environments.

4. **Consider a replacement cycle plan** — the ZLT M30S is a solid device but WPA3-capable
   hardware is becoming more affordable. When budget allows, migrating to a WPA3-capable
   AP would eliminate KRACK-class concerns entirely at the protocol level.
