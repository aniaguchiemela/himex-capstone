# Nmap Scan Findings
**Date:** 2026-07-30
**Tool:** Nmap 7.99
**Target:** 192.168.0.1 (ZTE MiFi — Tozed Kangwei Tech)
**Command:** nmap -sV -O -A 192.168.0.1

## Open Ports

### Port 53 — DNS (dnsmasq 2.80)
- MiFi runs its own DNS resolver for connected clients
- dnsmasq 2.80 has known CVEs (see research section)
- MITRE: T1590.002 (Gather Victim Network Info — DNS)

### Port 80/443 — HTTP/HTTPS (Demo-Webs admin panel)
- Admin panel accessible over HTTP (unencrypted) on port 80
- HTTP redirects to HTTPS (good)
- Security headers present:
  ✓ X-Frame-Options: SAMEORIGIN
  ✓ X-Content-Type-Options: nosniff
  ✓ X-XSS-Protection: 1; mode=block
  ✓ Strict-Transport-Security (HSTS) with includeSubDomains; preload
- MITRE: T1133 (External Remote Services)

## OS Fingerprint
- Detected: Linux kernel 3.2 - 3.16
- Significance: Kernel range dates to 2012–2016
- Embedded firmware — end user cannot update kernel independently
- Any unpatched CVEs from that kernel era remain present
- MITRE: T1592.001 (Gather Victim Host Info)

## MAC Vendor Confirmation
- OUI: 98:A9:42 → Tozed Kangwei Tech
- Independently cross-confirms beacon analysis finding

## Summary Risk Rating
| Finding | Severity | Notes |
|---------|----------|-------|
| dnsmasq 2.80 exposed | Medium | Known CVEs in this version |
| Admin panel on port 80 | Medium | HTTP access before redirect |
| Linux kernel 3.x | High | Long-unpatched embedded base |
| MAC vendor identifiable | Low | Passive recon risk |
| Security headers present | Positive | Good baseline web hardening |
