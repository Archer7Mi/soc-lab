# Project 01: SSH Brute Force Detection

**Status:** Methodology Ready | Detection Ready | Investigation Pending

---

## 📋 Executive Summary

Demonstrates the detection of repeated failed SSH authentication attempts against a honeypot (OpenCanary) using:
- **Attack Vector:** Hydra brute-force attack
- **Target:** OpenCanary SSH service (10.0.1.30:22)
- **Detection Method:** SPL query + Sigma rule
- **Expected Output:** Alert + MITRE ATT&CK mapping (T1110 - Brute Force)

---

## 🎯 Learning Objectives

- ✅ Understand how authentication logs translate to detectable patterns
- ✅ Write Splunk SPL queries for threshold-based detection
- ✅ Create portable Sigma rules for vendor-agnostic distribution
- ✅ Establish baseline vs. attack anomaly
- ✅ Document findings in formal incident report

---

## 🔍 Detection Logic

### Splunk Query

```spl
index=opencanary sourcetype=opencanary_json logtype=2000
| stats count by src_host
| where count > 5
| eval severity="HIGH", attack_type="brute_force"
```

**Logic:** Identify any source IP attempting more than 5 failed authentications within the ingestion window. Adjust threshold based on baseline.

### Sigma Rule

```yaml
title: SSH Brute Force Attack
logsource:
  product: opencanary
  service: ssh
detection:
  selection:
    logtype: 2000
    event: 'Attempted SSH login'
  timeframe: 1m
  condition: selection | count(src_host) > 5
severity: high
tags:
  - attack.credential_access
  - attack.t1110_brute_force
```

---

## 🛠 Execution Plan

**Step 1: Run Attack (On LAB-KALI)**
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt -t 4 -f ssh://10.0.1.30 -vV
```

**Step 2: Monitor in Splunk**
- Wait for logs to ingest (3-5 seconds via HEC)
- Run SPL query above
- Verify alert triggers

**Step 3: Document Findings**
- Screenshot the alert
- Collect logs (src_host, timestamp, logtype, event)
- Map to MITRE ATT&CK (T1110)
- Write formal report

---

## 📁 Artifacts (To Be Populated)

- [ ] `report.pdf` — Formal investigation report
- [ ] `splunk-query.spl` — Detection query
- [ ] `sigma-rule.yml` — Portable detection rule
- [ ] `screenshots/` — Evidence images

---

**Next Step:** Execute attack simulation and collect telemetry.
