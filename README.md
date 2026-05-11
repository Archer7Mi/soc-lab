# SOC Home Lab — Michael Ted

**A portfolio-grade Security Operations Center (SOC) built on a single physical machine, simulating enterprise-scale threat detection, investigation, and response.**

---

## 🎯 Overview

This lab demonstrates practical SOC analyst skills across the full detection and response lifecycle — from infrastructure deployment and log ingestion, through attack simulation and detection engineering, to formal incident reporting.

The environment simulates a small enterprise network with:
- **Active Directory domain** (lab.local) populated with realistic noise via BadBlood
- **Domain-joined workstations** generating Windows telemetry (Sysmon)
- **Network firewall** (pfSense) providing segmentation and IDS (Suricata)
- **Deception layer** (OpenCanary honeypot) for early warning
- **SIEM stack** (Splunk) for log aggregation and detection
- **Incident management** (TheHive) for case workflow

All findings are documented as professional deliverables, mapped to the **MITRE ATT&CK framework**.

---

## 🏗 Architecture

```
[ INTERNET ]
     │
[ VMnet0 — Bridged ]
     │
┌────▼─────────────────────────────────────────┐
│          pfSense Router/Firewall              │
│  (Suricata IDS, 5 virtual segments)           │
└─┬──────────┬──────────┬──────────┬────────────┘
  │          │          │          │
  ├─ VMnet1  ├─ VMnet2  ├─ VMnet3  └─ VMnet4
  │          │          │            (MGMT)
  │          │          │
  │ Victim   │ Attacker │ Security
  │ LAN      │ LAN      │ Tools
  │          │          │
  │          │          ├─ Splunk 10.0.3.10
  │          │          ├─ Wazuh 10.0.3.20
  │          │          ├─ TheHive 10.0.3.30
  │          │          ├─ Shuffle 10.0.3.40
  │          │          └─ Zeek 10.0.3.50
  │          │
  │          └─ Kali 10.0.2.10
  │
  ├─ LAB-DC01 10.0.1.10 (Domain Controller)
  ├─ LAB-WS01 10.0.1.20 (Workstation)
  └─ LAB-CANARY 10.0.1.30 (Honeypot)
```

---

## 📊 Phase Status

| Phase | Name | Status | Deliverable |
|-------|------|--------|-------------|
| 1 | **Infrastructure** | ✅ COMPLETE | pfSense, AD, Sensors |
| 2 | **Visibility** | 🔄 IN PROGRESS | Splunk, log ingestion |
| 3 | **Offensive Simulation** | ⏳ PLANNED | Attack scenarios |
| 4 | **Response & Automation** | ⏳ PLANNED | Incident cases |
| 5 | **Detection Engineering** | ⏳ PLANNED | SPL/Sigma rules |
| 6 | **Deliverable** | ⏳ PLANNED | Formal IR report |

---

## 🔬 Investigation Projects (10 Total)

| # | Project | Status | Methodology | Evidence |
|---|---------|--------|-------------|----------|
| 1 | SSH Brute Force | Ready | Hydra + OpenCanary logs | SPL query, Sigma rule |
| 2 | Port Scanning | Ready | Nmap reconnaissance | Zeek detection |
| 3 | Reverse Shell | Ready | Metasploit listener | Sysmon EID 3 & 1 |
| 4 | End-to-End Investigation | Planned | Full attack chain | Correlated evidence |
| 5 | Custom Detection Script | Planned | Python/Bash automation | Script + test results |
| 6 | Beaconing Detection | Planned | Zeek/RITA analysis | Traffic timeline |
| 7 | Exploitation Analysis | Planned | Vulnerability assessment | Gap analysis |
| 8 | Web Attack Detection | Planned | DVWA SQLi/XSS | HTTP logs + alerts |
| 9 | Baseline vs Deviation | Planned | Statistical analysis | Anomaly scores |
| 10 | Detection Tuning | Planned | Iterative FP reduction | Rule tuning logs |

---

## 🛠 Tool Stack

| Category | Tool | Purpose |
|----------|------|---------|
| **Firewall** | pfSense | Network segmentation, IDS via Suricata |
| **Identity** | Active Directory + BadBlood | Domain infrastructure, realistic noise |
| **Endpoint Visibility** | Sysmon | Process, network, registry telemetry |
| **Endpoint Protection** | CrowdSec | Behavioral blocking, threat intel |
| **Honeypot** | OpenCanary | Deception layer, early warning |
| **Network Analysis** | Zeek | Connection logs, DNS, HTTP, SSL metadata |
| **SIEM** | Splunk Free | Log aggregation, detection, dashboarding |
| **XDR** | Wazuh | FIM, response automation |
| **Case Management** | TheHive + Cortex | Incident triage, enrichment |
| **SOAR** | Shuffle | Workflow automation, response |
| **Attack Simulation** | Kali, Metasploit, Caldera | Controlled attack generation |
| **Detection Engineering** | Sigma + Git | Detection-as-code, version control |

---

## 📁 Repository Structure

```
soc-lab-michael-ted/
├── README.md                          ← You are here
├── Progress.md                        ← Executive status dashboard
│
├── architecture/
│   ├── network-design.md              ← IP addressing, VMnet layout, data flows
│   └── topology-diagram.png           ← Visual architecture
│
├── phases/
│   ├── phase-1-infrastructure/
│   │   ├── README.md                  ← Build decisions, issues, resolution
│   │   └── screenshots/               ← pfSense, AD, Sysmon evidence
│   ├── phase-2-visibility/
│   ├── phase-3-offensive-simulation/
│   ├── phase-4-response-automation/
│   ├── phase-5-detection-engineering/
│   └── phase-6-deliverable/
│
├── projects/                          ← Each investigation as a case study
│   ├── 01-ssh-brute-force/
│   │   ├── README.md                  ← Executive summary
│   │   ├── report.pdf                 ← Formal investigation report
│   │   ├── splunk-query.spl           ← Detection logic
│   │   ├── sigma-rule.yml             ← Portable detection rule
│   │   └── screenshots/               ← Evidence artifacts
│   ├── 02-port-scanning/
│   └── [03-10 following same pattern]
│
├── detections/                        ← Consolidated rules for reuse
│   ├── sigma/                         ← Sigma rules library
│   └── splunk/                        ← SPL queries library
│
├── runbooks/                          ← Investigation procedures
│   ├── ssh-brute-force.md
│   ├── port-scan.md
│   └── reverse-shell.md
│
└── docs/                              ← Primary deliverables
    └── incident-report-IR-2025-001.pdf
```

---

## 🚀 Getting Started

### Prerequisites
- VMware Workstation Pro / Hyper-V
- 16GB RAM minimum
- 100GB disk space
- Kali Linux, Ubuntu Server ISOs

### To Follow This Build
1. Start with [Phase 1: Infrastructure](phases/phase-1-infrastructure/README.md)
2. Review [Network Design](architecture/network-design.md) for IP addressing
3. Check [Progress.md](Progress.md) for current status and blockers
4. Follow commit history to see build decisions and troubleshooting

---

## 📋 Skills Demonstrated

✅ **SIEM & Log Aggregation**
- Splunk deployment, configuration, query writing (SPL)
- Log ingestion pipeline design (Universal Forwarders, HEC, Syslog)
- Alert tuning and baseline establishment

✅ **Network Security**
- Firewall rule design and segmentation
- Network traffic analysis (Zeek, Suricata, Wireshark)
- IDS/IPS tuning and signature management

✅ **Endpoint Security**
- Windows telemetry collection (Sysmon)
- Process analysis and behavioral detection
- Host-based intrusion detection (CrowdSec, Wazuh)

✅ **Detection Engineering**
- Sigma rule authorship (portable, vendor-agnostic)
- SPL query optimization
- False positive tuning and alert refinement

✅ **Incident Response**
- Case management workflows (TheHive)
- Evidence correlation and timeline analysis
- MITRE ATT&CK mapping and attribution

✅ **Attack Simulation**
- Metasploit exploitation
- Hydra brute-forcing
- Atomic Red Team and Caldera scenario execution

✅ **Defensive Coding**
- Python/Bash automation scripts
- Detection-as-code (Sigma + CI/CD)
- Log parsing and correlation logic

---

## 💾 Commit Convention

All commits follow a disciplined pattern:

```
[phase-1] Add pfSense VM — WAN/LAN interfaces configured
[phase-1] Deploy AD DC — Badblood populated, domain lab.local live
[project-01] Add SSH brute-force Sigma rule
[project-01] Add Splunk SPL query for auth failure detection
[detection] Tune SSH rule — reduce false positive threshold from 5 to 10
```

This ensures the repo history becomes **evidence of methodology**, not just results.

---

## 📄 Key Deliverable

**Incident Report: IR-2025-001**  
A professional investigation of a multi-stage attack against the lab environment, complete with:
- Executive summary (C-level context)
- Technical analysis (forensic detail)
- MITRE ATT&CK mapping
- Timeline and evidence correlation
- Remediation recommendations

---

## 👤 Author

**Michael Ted**  
SOC Analyst | Security Operations | GRC  
[LinkedIn](https://linkedin.com/in/archer7mi) | [GitHub](https://github.com/Archer7Mi)

---

**Last Updated:** 2026-05-11  
**Lab Status:** Phase 2 (Visibility) — In Progress  
**Next Milestone:** LAB-SIEM (Splunk) deployment at 10.0.3.10
