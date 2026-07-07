# SIEM + IDS Detection Pipeline — Splunk Enterprise & Snort

**End-to-end attack detection lab: custom IDS rules, real-time SIEM correlation, and a full incident response runbook — 100% detection rate across 4 simulated attack types.**

> Built in a fully isolated VMware lab (host-only network) for educational purposes as part of my Incident Response coursework (Algonquin College, Cyber Security Analysis program). No external systems were involved.

---

## Architecture

Three-VM environment on an isolated subnet:

| Role | OS | Software |
|---|---|---|
| Web Server (target + IDS host) | Windows Server 2016 | IIS 10, Snort 2.9.20, Splunk Universal Forwarder |
| SIEM Server | Windows Server 2016 | Splunk Enterprise 9.x |
| Attacker | Kali Linux | Hydra v9.6, Nmap 7.98 |

**Detection data flow:**

```
Kali attack → Web Server NIC → Snort IDS (packet inspection)
    → alert.ids log → Splunk Universal Forwarder (TCP 9997)
    → Splunk index (csa271_logs) → Real-time SPL alerts → Triggered Alerts dashboard
```

All VM clocks were NTP-synchronized (`w32tm /resync /force`) — accurate timestamps are non-negotiable for cross-source event correlation and incident timeline reconstruction.

## What I built

**1. Custom Snort detection rules (`local.rules`)**
Wrote signature rules keyed to TCP flag behavior rather than tool names:
- **SYN scan** — alerts on TCP packets with only the SYN flag set (`flags:S`), the hallmark of half-open scanning
- **Xmas scan** — alerts on the FIN+PSH+URG flag combination
- **UDP scan** — alerts on UDP datagrams sweeping the protected subnet
- **TCP connect scan** — detected via Snort's stream preprocessor recognizing completed-handshake port sweeps

**2. Log forwarding pipeline**
Configured the Splunk Universal Forwarder (`inputs.conf` / `outputs.conf`) on the web server to tail Snort's alert log, IIS HTTP/FTP logs, Windows Security events, and Windows Firewall logs into a dedicated Splunk index.

**3. Five real-time SPL correlation alerts**
Saved searches correlating Windows EventCodes (failed logon bursts), FTP status-code anomalies (`sc_status=530` repetition → brute force), and Snort sourcetype events — each firing as a High-severity triggered alert in real time.

**4. Attack simulation & validation**
Executed four attack types from Kali — FTP brute force (Hydra + rockyou.txt), SYN scan (`nmap -sS`), TCP connect scan (`nmap -sT`), UDP scan (`nmap -sU`) — achieving **100% detection: 18 high-severity alerts, zero missed attacks**. The scans also surfaced a real finding: **WinRM (port 5985) exposed** on the target, flagged for hardening.

**5. Incident Response Runbook**
Authored a full runbook covering five incident types (HTTP brute force, FTP brute force, SYN/TCP/UDP scans), each with:
- Detection logic and manual SPL investigation queries
- **Detect → Contain → Eradicate → Recover → Post-Incident** procedures
- PowerShell containment commands per attack type
- Escalation & notification matrix and an evidence preservation checklist

## Skills demonstrated

`Splunk (SPL)` · `Snort IDS` · `Detection engineering` · `Log forwarding & indexing` · `Windows Server 2016` · `IIS` · `Nmap` · `Hydra` · `Incident response (NIST lifecycle)` · `PowerShell` · `Network traffic analysis`

## What I'd do differently in production

- Deploy Snort 3.x (or Suricata) with community rulesets layered under custom rules
- Ship logs over TLS and add a heavy forwarder tier for parsing at scale
- Tune thresholds against baseline traffic to manage false-positive rates before alerting
- Add SOAR-style automated containment instead of manual PowerShell response

---

*Full configuration report and runbook available on request (sanitized).*

