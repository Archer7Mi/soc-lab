# Project 03: Reverse Shell Detection

**Status:** Methodology Ready | Detection Ready | Investigation Pending

---

## 📋 Executive Summary

Demonstrates detection of post-exploitation activity (reverse shell establishment) using:
- **Attack Vector:** Metasploit reverse shell listener
- **Target:** LAB-WS01 compromised via Meterpreter
- **Detection Method:** Sysmon + Network-based anomaly
- **Expected Output:** Alert + MITRE ATT&CK mapping (T1059 - Command and Scripting Interpreter)

---

## 🎯 Learning Objectives

- ✅ Understand process behavior anomalies (parent-child relationships)
- ✅ Detect outbound network connections to non-standard ports
- ✅ Correlate process creation (Sysmon EID 1) with network connections (EID 3)
- ✅ Write detection queries for suspicious ports (4444, 9001, 1337, etc.)
- ✅ Map to MITRE ATT&CK techniques

---

## 🔍 Detection Logic

### Splunk Query (Sysmon-based)

```spl
index=sysmon EventCode=3 (DestinationPort=4444 OR DestinationPort=9001 OR DestinationPort=1337 OR DestinationPort=5555)
| table _time, Image, DestinationIp, DestinationPort, DestinationHostname
| eval severity="CRITICAL"
```

**Logic:** Flag any process establishing outbound connections to known Metasploit listener ports.

---

## 📁 Artifacts (To Be Populated)

- [ ] `report.pdf` — Investigation report
- [ ] `splunk-query.spl` — Detection query
- [ ] `sigma-rule.yml` — Portable rule
- [ ] `screenshots/` — Meterpreter session, network connection evidence

---

**Next Step:** Execute Metasploit exploitation and capture reverse shell session.
