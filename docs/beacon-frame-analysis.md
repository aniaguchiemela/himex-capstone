# Beacon Frame Analysis

## Source
Analysis performed on `evidence/logs/baseline_scan-06.cap` in Wireshark, filtered to
isolate beacon frames only:
```
wlan.fc.type_subtype == 0x08
```
Target beacon: BSSID `98:A9:42:45:C3:0F`, SSID `MTN_4G_45C30F`, Channel 11,
RSN: WPA2-PSK-CCMP (Group Cipher: AES/CCM, Pairwise Cipher: AES/CCM, AKM: PSK).

A beacon frame is broadcast continuously and unencrypted by design — any device within
range can read its contents without authenticating or associating first. The findings
below are all things this MiFi reveals to a purely passive observer.

## Finding 1 — Manufacturer identification via OUI
Wireshark auto-resolved the source MAC's OUI (`98:A9:42`) to a manufacturer name,
visible directly in the packet list. The first three octets of any MAC address are
IEEE-registered to a specific vendor, so this lookup requires no active probing —
it's derivable from the beacon alone.

**Risk:** Low on its own, but it's the first data point in a fingerprinting chain —
knowing the exact hardware vendor narrows down which known firmware vulnerabilities,
default credentials, or exploit tooling might apply to this specific device.

## Finding 2 — Regulatory domain / country code leak
```
Tag: Country Information: Country Code CN, Environment All
```
The beacon discloses a regulatory domain of **China (CN)**, not Nigeria, where the
device is actually deployed. This reflects the firmware's original certification
region rather than its current location, and is broadcast unencrypted to anyone
listening.

**Risk:** Low direct risk, but it's a small OSINT data point — it can hint at where a
device's firmware baseline originates, which is relevant when cross-referencing known
CVEs or default configurations tied to a specific regional firmware build.

## Finding 3 — WPS enabled and configured (most actionable finding)
```
Tag: Vendor Specific: Microsoft Corp.: WPS
Type: WPS (0x04)
Wifi Protected Setup State: Configured (0x02)
```
The beacon confirms Wi-Fi Protected Setup (WPS) is active and already configured on
this AP. WPS was designed to simplify pairing via an 8-digit PIN, but its PIN
validation is split into two halves checked separately, which historically made it
vulnerable to brute-force PIN recovery (and, on some chipsets, "Pixie Dust"-style
offline attacks that recover the PIN in seconds rather than hours).

**Risk:** High relative to the other findings. A successfully recovered WPS PIN
grants an attacker the WPA2 passphrase itself, bypassing the need to crack or
otherwise obtain the PSK directly — meaning WPS can function as a weaker side-door
into a network that is otherwise protected by a strong WPA2 passphrase.

**Recommendation:** Disable WPS in the MiFi's admin settings if the option is
available. This is the clearest, lowest-effort hardening action to come out of the
beacon analysis phase.

## Finding 4 — Radio chipset disclosure
```
Tag: Vendor Specific: Realtek Semiconductor Corp.
```
In addition to the OUI-based manufacturer identification, a second vendor tag
explicitly names the wireless radio chipset vendor (Realtek). Combined with Finding 1,
this gives a passive observer a fairly complete hardware profile — device
manufacturer and radio chipset — without any active interaction with the AP.

**Risk:** Low on its own; contributes to the same fingerprinting chain as Finding 1.
Chipset-specific vulnerabilities (e.g. certain Realtek driver-level flaws disclosed
over the years) become a more targeted research angle once the chipset is known.

## Summary
| Finding | Leak type | Risk level | Actionable fix? |
|---|---|---|---|
| Manufacturer OUI | Passive fingerprinting | Low | No (inherent to MAC addressing) |
| Country code (CN) | Passive fingerprinting | Low | No (firmware-level, not configurable) |
| WPS enabled/configured | Weak alternate access path | High | Yes — disable WPS |
| Chipset (Realtek) | Passive fingerprinting | Low | No (inherent to hardware) |

Of the four findings, **WPS is the only one with a direct, actionable fix** and the
only one that materially weakens the network's actual access control rather than
simply revealing metadata about the device.
