# Findings Mapped to MITRE ATT&CK

## Source
This mapping is built entirely from findings confirmed during this assessment against
the MiFi target (`98:A9:42:45:C3:0F`, SSID `MTN_4G_45C30F`). Each row links a real,
evidenced finding to the ATT&CK technique it most closely represents — no theoretical
or unconfirmed findings are included beyond what's explicitly flagged as such.

| # | Finding | Evidence | ATT&CK Tactic | Technique | ID |
|---|---|---|---|---|---|
| 1 | Passive beacon scan revealed SSID, BSSID, channel, encryption type without any active interaction | `evidence/logs/baseline_scan-06.cap` | Reconnaissance | Wireless Sniffing | T1040 |
| 2 | Beacon frames leaked manufacturer (OUI), regulatory country code (CN), and radio chipset (Realtek) to a passive observer | `docs/beacon-frame-analysis.md` | Reconnaissance | Gather Victim Host Information | T1592 |
| 3 | WPA2 4-way handshake captured while phone reassociated to the AP | `evidence/logs/handshake_capture-01.cap`, confirmed via `aircrack-ng` (1 handshake) | Credential Access | Steal or Forge Authentication Certificates / Network Sniffing | T1557.002 |
| 4 | WPS enabled and in "Configured" state on the target AP | `docs/beacon-frame-analysis.md`, Vendor Specific: Microsoft Corp. WPS tag | Credential Access | Brute Force (weak alternate authentication path) | T1110 |
| 5 | Client device confirmed patched (2026-05-01); AP confirmed WPA2-only with no WPA3 support | `docs/research-krack-wpa3.md` | Credential Access (theoretical, not exploited) | Adversary-in-the-Middle | T1557 |
|  6 | Bounded 5-frame deauthentication forced reassociation and fresh handshake capture on the tester's own client device | `evidence/logs/deauth_capture-01.cap`, confirmed via `aircrack-ng` (1 handshake), screenshot `deauth_capture_result.png` | Impact / Credential Access | Network Denial of Service (client-scoped) enabling Adversary-in-the-Middle handshake capture | T1498 / T1557 |

## Notes on mapping decisions

**Row 1 vs Row 2:** these are split deliberately. Row 1 covers the mechanical act of
passively observing 802.11 traffic (the technique used), while Row 2 covers what
specific information was extracted from that traffic (the outcome). Both are real,
evidenced findings from the same baseline scan.

**Row 3:** capturing a handshake is not equivalent to cracking the passphrase — no
password recovery was attempted in this assessment, in line with `SCOPE.md`. This row
represents the technique of intercepting the authentication exchange itself, which is
the necessary precondition for an offline brute-force attempt, not the attempt itself.

**Row 4:** WPS is mapped to Brute Force because its 8-digit PIN, split into two halves
during validation, is the well-documented weak point — an attacker targeting WPS is
attempting a lower-effort brute-force path to network access rather than attacking the
WPA2 passphrase directly.

**Row 5:** explicitly marked theoretical. No adversary-in-the-middle position was
established or attempted during this assessment. This row exists to document the
technique class that KRACK-style vulnerabilities fall under, alongside the finding
that this specific environment (patched client, WPA2-only AP) was assessed for — not
exploited for — that exposure.

## Deauthentication finding (completed)

A bounded, 5-frame deauthentication burst was sent against the tester's own client
device only, using `aireplay-ng --deauth 5`. This forced a brief disconnect and
automatic reconnection, during which a fresh WPA2 handshake was captured and verified
via `aircrack-ng` (`1 handshake`). This demonstrates the practical technique behind
Row 5 (Adversary-in-the-Middle / handshake interception): an attacker capable of
sending deauth frames toward a target client can force a handshake to occur on
command, rather than waiting passively for one to happen naturally.

An earlier attempt at this test failed due to a copy-paste error (a placeholder MAC
address was used instead of the actual target MAC), which caused an unbounded,
continuous deauth flood against unintended clients for approximately 3 minutes before
being manually stopped. That capture was discarded and is not included in this
repository's evidence. The successful, bounded 5-frame test described above is the one
documented and evidenced here.
