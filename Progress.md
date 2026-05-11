# 📊 SOC Lab Build: Executive Progress Report

**Analyst:** Michael Ted  
**Project Identity:** Portfolio-grade SOC on 16GB RAM  
**Last Updated:** 2026-05-11  
**Current Phase:** Phase 2 (Detection & Visibility)

---

## 🏗 Phase 1: Infrastructure (Foundation)

**Status:** 🟢 **100% COMPLETE**

The "Nervous System" of the lab is now fully functional. All networks are routed, identity services are live, and the initial sensor layer is firing.

### ✅ Completion Criteria Met

| Task | Status | Technical Details |
|:---|:---|:---|
| **pfSense Core** | ✅ COMPLETE | 5 segments assigned (VMnet0-4). LAN Gateway at 10.0.1.1. |
| **Active Directory** | ✅ COMPLETE | Domain `lab.local` promoted and verified. BadBlood population ~2,500 objects. |
| **Connectivity** | ✅ VERIFIED | ICMP successful from 10.0.1.10 to 10.0.1.1 and 8.8.8.8. All segments routable. |
| **Firewall Rules** | ✅ COMPLETE | DNS (53), HTTP/S (80/443), SIEM (9997), Syslog (514) pass rules active. |
| **Honeypot** | ✅ LIVE | LAB-CANARY active at 10.0.1.30. JSON logs verified firing. |
| **Sysmon** | ✅ LIVE | Deployed on LAB-DC01 and LAB-WS01. Event IDs 1, 3, 11 confirmed. |
| **CrowdSec** | ✅ LIVE | Enrollment complete on LAB-DC01. Hub update successful. Behavioral blocking active. |

### 🛠 Technical Milestone Ledger (Chronological)

- **VM Deployment:** Created pfSense, AD DC, Windows Workstation, and OpenCanary.
- **Network Mapping:** Resolved VMware host adapter conflicts by remapping host IPs to .254, ensuring pfSense remains the authoritative .1 gateway.
- **Firewall Logic Audit:** Utilized `pfctl -d` to isolate connectivity failures to the packet filter, then implemented granular "Allow" rules per blueprint.
- **Management NIC Stabilization:** Corrected Ethernet1 (Management) on LAB-DC01 from DHCP (APIPA 169.254.x.x) to static 10.0.4.10.
- **BadBlood Noise Floor:** Populated domain with 2,500+ users and groups to simulate realistic enterprise logs.
- **Sensor Verification:** Confirmed Sysmon firing on all Windows machines. CrowdSec enrollment successful after DNS forwarder fixes.

---

## 👁 Phase 2: Detection & Visibility

**Status:** 🔄 **IN PROGRESS**

### 🎯 Current Objective: LAB-SIEM Deployment

Sensors are actively firing. The next phase activates the "Recording Station" to ingest and correlate all telemetry.

| Component | Target | Status | Purpose |
|:---|:---|:---|:---|
| **LAB-SIEM (Splunk)** | 10.0.3.10 | 🔄 QUEUED | Log aggregation, detection logic, dashboarding |
| **Splunk UF** | LAB-DC01 | ⏳ PENDING | Forward Sysmon, Windows Security, CrowdSec |
| **Splunk UF** | LAB-WS01 | ⏳ PENDING | Forward Sysmon, endpoint logs |
| **Zeek** | 10.0.3.50 | ⏳ PLANNED | Network flow analysis, DNS, HTTP metadata |
| **Wazuh Agent** | LAB-DC01 | ⏳ PENDING | XDR, file integrity monitoring |

**Hardware Allocation:**
- Splunk: 4GB RAM (optimized for the 16GB host limit)
- Zeek: 1GB RAM
- Wazuh: 512MB RAM

---

## 🔬 Investigation Roadmap (Methodology Ready)

Your 10 projects are fully mapped and awaiting telemetry ingestion:

### Ready to Execute (Phase 3+)
1. **SSH Brute Force** — OpenCanary JSON logs → Splunk ingestion → Detection rule
2. **Port Scanning** — Suricata/Zeek logs → Port count anomaly detection
3. **Reverse Shell** — Sysmon EID 1/3 → Network connection anomaly detection
4. **End-to-End Investigation** — Full attack chain correlation and case triage
5. **Custom Detection Script** — Python automation for log parsing and enrichment

### Planned
6. Beaconing Traffic Detection
7. Exploitation Visibility Analysis
8. Web Attack Detection (DVWA)
9. Network Baseline vs. Attack Deviation
10. Detection Rule Tuning & False Positive Reduction

---

## 💻 Resource Utilization (16GB RAM)

| Active VM | RAM Allocated | Status | Role |
|:---|:---|:---|:---|
| **pfSense** | 1GB | Active | Gateway, Firewall, IDS |
| **LAB-DC01** | 2GB | Active | Domain Controller, Sysmon, CrowdSec |
| **LAB-WS01** | 2GB | Active | Victim Workstation, Sysmon |
| **LAB-CANARY** | 512MB | Active | Honeypot (OpenCanary) |
| **Current Total** | **~5.5GB** | **Active** | **~10.5GB Remaining** |

**Allocation Plan (Phase 2+):**
- LAB-SIEM (Splunk): +4GB
- LAB-ZEEK: +1GB
- LAB-WAZUH: +512MB
- **Projected Total: ~11.5GB** ✅ Within 16GB limit

---

## 🚧 Current Blockers & Resolutions

### ❌ Resolved: Connectivity Blackout (Virtual Interface Mapping)
**Issue:** LAB-DC01 could not ping 10.0.1.1 despite correct static IP configuration.  
**Root Cause:** VMware virtual adapter assignment was mismatched. Ethernet0 (Victim LAN) was logically configured for 10.0.1.0/24 but physically plugged into the wrong VMnet.  
**Resolution:** Verified MAC addresses against VMware settings. Confirmed Ethernet0 on VMnet1 and Ethernet1 on VMnet4. Restored connectivity immediately.

### ✅ Resolved: Firewall Over-Restriction
**Issue:** Even with correct routing, ping to external DNS (8.8.8.8) failed.  
**Root Cause:** pfSense default-drop policy blocked ICMP and DNS queries.  
**Resolution:** Implemented granular "Allow" rules per blueprint: DNS (53), HTTP/S (80/443), ICMP, and SIEM (9997).

### ✅ Resolved: DNS Resolution Timeouts
**Issue:** CrowdSec `hub update` failed with DNS I/O timeout on LAB-DC01.  
**Root Cause:** Windows DNS service was configured to query itself (127.0.0.1) without forwarders to pfSense or external DNS.  
**Resolution:** Added DNS forwarders: 10.0.1.1 (pfSense) and 8.8.8.8 (Google). Flushed DNS cache. Hub update now succeeds.

---

## 📈 Key Metrics

| Metric | Value |
|:---|:---|
| **Infrastructure Uptime** | 24/7 (Active) |
| **Domain Objects** | ~2,500 (BadBlood) |
| **Network Segments** | 5 (VMnet1-4 + VMnet0) |
| **Firewall Rules** | 8 active pass rules |
| **Sensors Deployed** | 3 (Sysmon, CrowdSec, OpenCanary) |
| **Sensor Log Rate** | ~50-100 events/minute (idle) |
| **Time to Build Phase 1** | ~8 hours (including troubleshooting) |

---

## 🎯 Next Steps (Priority Order)

1. **Deploy LAB-SIEM (Splunk)**
   - Ubuntu Server 22.04 LTS
   - Splunk Enterprise (trial license)
   - HEC listener on port 8088
   - Web UI on port 8000

2. **Install Universal Forwarders**
   - Deploy to LAB-DC01
   - Deploy to LAB-WS01
   - Configure inputs.conf for Sysmon, Windows Security, CrowdSec

3. **Verify Telemetry Flow**
   - Test connectivity to Splunk HEC (port 8088)
   - Ingest first batch of Sysmon logs
   - Build initial dashboard

4. **Execute Project 1 (SSH Brute Force)**
   - Run Hydra against OpenCanary
   - Capture logs in Splunk
   - Write and test SPL query
   - Create Sigma rule
   - Document findings in formal report

---

## 📅 Phase 2 ETA

- **LAB-SIEM Installation:** 1-2 hours
- **UF Deployment:** 30 minutes
- **Telemetry Verification:** 30 minutes
- **Project 1 Execution:** 2 hours
- **Phase 2 Complete:** ~4-5 hours from now

**Estimated Phase 2 Completion:** 2026-05-11 (EOD)

---

## 📝 Notes

- All VMs are configured with static IPs per the authoritative `network-design.md` blueprint.
- The host machine's VMware adapter has been remapped to 10.0.4.254 to avoid conflicts.
- Firewall logs are available at `pfSense > Firewall > Logs` for audit and tuning.
- All credentials are stored securely (not in git).
- Lab is entirely self-contained within VMware virtual networks; no external traffic exposure.

---

**Analyst Signature:** Michael Ted  
**Last Verified:** 2026-05-11 16:15 UTC  
**Next Review:** Upon Phase 2 milestone (Splunk live)
