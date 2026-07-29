# Research — KRACK Vulnerability & WPA3 Upgrade Path

## Context from this assessment

The target AP in this lab (MiFi, SSID `MTN_4G_45C30F`, BSSID `98:A9:42:45:C3:0F`)
was confirmed running **WPA2** during the baseline scan (`evidence/logs/baseline_scan-06.cap`).
A genuine WPA2 4-way handshake was captured during the handshake phase
(`evidence/logs/handshake_capture-01.cap`), confirmed via:

```
aircrack-ng evidence/logs/handshake_capture-01.cap
→ 98:A9:42:45:C3:0F  MTN_4G_45C30F  WPA (1 handshake)
```

This handshake is the exact mechanism KRACK targets, which makes this MiFi a
relevant real-world case to reason about, even though no exploitation was performed.

## What KRACK actually attacks

KRACK (Key Reinstallation Attack) does not break WPA2's cryptography — it manipulates
the 4-way handshake itself. During the handshake, the AP sends a cryptographic nonce
that the client uses to install a session encryption key. KRACK tricks the client into
reinstalling an already-used key by replaying handshake message 3, causing nonce reuse.
On WPA2-CCMP (AES) — which is what this MiFi uses, based on the `WPA2 C` / CCMP flag
seen in the airodump-ng scan output — nonce reuse can allow decryption of affected
packets, not full compromise of the passphrase itself.

## Why this is a client-side issue, not just an AP issue

The critical detail for this assessment: KRACK is fixed by patching the **client**,
not the access point. The AP retransmitting handshake message 3 is normal, correct
behavior — the vulnerability lives in how the client responds to that retransmission.
This means:

- Patching the MiFi's firmware does not, by itself, protect any device connecting to it
- Every client device (phones, laptops) that connects to this MiFi needs its own
  OS-level security patch to be protected
- A fully patched client connecting to an unpatched AP is still safe; an unpatched
  client connecting to a patched AP is still vulnerable

## Applicability to this lab

The phone used as the client in this assessment (`5E:EE:FC:DC:AF:21`, seen negotiating
the EAPOL/handshake exchange in the capture) would need to be checked directly for its
OS patch level to determine real exposure:

- **Android**: check Settings → Security → Security patch level. Anything with a patch
  date from November 2017 onward addresses KRACK.
- **iOS**: iOS 11.1 and later addressed KRACK.

**Finding:** The test phone's security patch level is dated **2026-05-01** — well
after the November 2017 KRACK patch cycle. The client device used in this assessment
is confirmed patched and not exposed to KRACK-style key reinstallation regardless of
the AP's own patch status.

## Why this matters more for SME environments than headline severity suggests

Real-world KRACK exploitation requires physical proximity to the target and a
technically capable attacker performing a man-in-the-middle position — it is not a
remote, drive-by threat. This lowers the practical urgency for a single well-maintained
device. However, the risk model changes in a Nigerian SME context specifically because:

- Staff often use older, hand-me-down, or budget Android devices that stop receiving
  security patches years before they're retired from use
- IT asset management (tracking which devices are patched) is frequently informal or
  absent in small offices
- The MiFi itself, once purchased, is rarely revisited for firmware updates unless it
  stops working

The finding for this assessment is therefore not "this MiFi is vulnerable to KRACK" —
it is **"any unpatched or unmanaged client device connecting to this network remains
exposed regardless of the AP's own patch status,"** which is a more actionable and
honest way to frame it for a hardening recommendation.

## WPA3 as a mitigation path

WPA3, introduced in 2018, replaces the 4-way handshake's PSK exchange with
Simultaneous Authentication of Equals (SAE), which is not vulnerable to the same
key-reinstallation manipulation KRACK relies on. This makes WPA3 a structural fix
rather than a per-device patch.

**Finding:** This MiFi does not support WPA3 — it offers WPA2 only, confirmed via its
admin panel. This is a hardware/firmware limitation rather than a misconfiguration:
there is no setting to enable, because the capability isn't present on this device.

This becomes a documented limitation for `hardening/before-after-config.md` rather
than a direct fix. Since the structural mitigation (WPA3) isn't available on this
hardware, the practical hardening recommendation shifts to defense-in-depth on the
client side: keeping connected devices patched, using a strong non-default PSK, and
treating device patch management as an ongoing SME IT practice rather than a one-time
setting.

## Summary finding (to carry into ATT&CK mapping)

- **Technique relevance:** Adversary-in-the-Middle (T1557) — KRACK's mechanism fits
  this technique category, as it relies on positioning between client and AP
- **Current exposure:** Not exploited or tested in this assessment (out of scope).
  Test client device confirmed patched (2026-05-01); MiFi AP confirmed WPA2-only,
  no WPA3 support available on this hardware
- **Recommendation:** Since WPA3 is not available as a structural fix on this device,
  the priority control is enforced client-side patch management across all devices
  connecting to this network — the AP hardware itself cannot be upgraded past WPA2
