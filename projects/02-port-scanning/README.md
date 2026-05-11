# Project 02: Port Scanning Detection

**Status:** Methodology Ready | Detection Ready | Investigation Pending

---

## 📋 Executive Summary

Demonstrates detection of reconnaissance activity (horizontal port scanning) using:
- **Attack Vector:** Nmap SYN scan
- **Target:** Victim LAN (10.0.1.0/24)
- **Detection Method:** Zeek + Splunk correlation
- **Expected Output:** Alert + MITRE ATT&CK mapping (T1046 - Network Service Scanning)

---

## 🎯 Learning Objectives

- ✅ Understand network flow analysis (Zeek)
- ✅ Detect reconnaissance patterns (port count anomaly)
- ✅ Write Splunk queries against Zeek conn.log
- ✅ Baseline normal network behavior
- ✅ Distinguish reconnaissance from legitimate network management

---

## 🔍 Detection Logic

### Splunk Query

```spl
index=zeek sourcetype=zeek_conn
| stats dc(id.resp_p) as port_count by id.orig_h
| where port_count > 20
| eval severity="HIGH", scan_type="horizontal"
```

**Logic:** Identify any source IP touching more than 20 unique destination ports (horizontal scan indicator).

---

## 📁 Artifacts (To Be Populated)

- [ ] `report.pdf` — Investigation report
- [ ] `splunk-query.spl` — Detection query
- [ ] `sigma-rule.yml` — Portable rule
- [ ] `screenshots/` — Evidence

---

**Next Step:** Deploy Zeek and execute Nmap scan simulation.
