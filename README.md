# SOC Lab

Hands-on SOC practice  alert triage, log analysis, and detection documentation.

## Structure

```
investigations/
  01-ssh-brute-force.md        SSH brute force alert: identification and triage
  02-failed-logins-analysis.md Spike in failed logins: root cause walkthrough
```

## Tools Used
- SIEM (log correlation and alerting)
- Wireshark (packet capture analysis)
- Linux CLI (`grep`, `awk`, `cut`, `journalctl`)

## Methodology
Each investigation follows the same structure:
**Alert  Context  Evidence  Conclusion  Recommendation**
