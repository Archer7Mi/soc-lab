# Phase 1: Infrastructure & Foundation

**Objective:** Establish the networking backbone, identity services, and initial sensor layer.

**Status:** ✅ **100% COMPLETE** | **Started:** 2026-04-15 | **Finished:** 2026-05-11

---

## 📋 Build Summary

Phase 1 transforms a blank 16GB Windows 11 host into a functional multi-segment network with:
- **Network isolation:** 5 virtual networks (VMnets) controlled by pfSense
- **Enterprise identity:** Active Directory domain with ~2,500 realistic user/group objects
- **Endpoint visibility:** Sysmon + CrowdSec generating deep Windows telemetry
- **Deception layer:** OpenCanary honeypot for early warning and artifact generation

### Completion Checklist

- ✅ pfSense deployed with 5 segments (Victim, Attacker, Security Tools, Management, External)
- ✅ Active Directory domain (lab.local) promoted and verified
- ✅ BadBlood noise population (~2,500 objects)
- ✅ Domain-joined Windows Workstation (LAB-WS01)
- ✅ OpenCanary honeypot (LAB-CANARY) running and generating JSON logs
- ✅ Sysmon deployed on DC and Workstation (Process, Network, Registry events)
- ✅ CrowdSec behavioral detection enrolled
- ✅ Network connectivity verified across all segments
- ✅ Firewall rules tuned for SIEM ingestion, DNS, and HTTP
- ✅ All static IPs assigned per blueprint

---

## 🛠 Implementation Details

### 1. Networking (pfSense Router)

**Virtual Appliance:** pfSense Community Edition (2.7.0+)  
**Allocation:** 1 GB RAM, 20 GB Disk

**Interfaces Configured:**

| Interface | Type | Segment | IP Address | Purpose |
|:---|:---|:---|:---|:---|
| em0 | WAN | VMnet0 (Bridged) | DHCP | Internet uplink (home router) |
| em1 | LAN | VMnet1 | 10.0.1.1/24 | Victim LAN (DC, WS, Honeypot) |
| em2 | OPT1 | VMnet2 | 10.0.2.1/24 | Attacker LAN (Kali) |
| em3 | OPT2 | VMnet3 | 10.0.3.1/24 | Security Tools (Splunk, Wazuh, TheHive) |
| em4 | OPT3 | VMnet4 | 10.0.4.1/24 | Management (out-of-band admin access) |

**Key Configuration Decisions:**

- **Hardware Offloading Disabled:** Suricata IDS requires direct access to packets; offloading was disabled in System > Advanced > Networking.
- **Suricata IDS:** Enabled on LAN (VMnet1) with ET Open ruleset for reconnaissance/exploitation detection.
- **DHCP Disabled:** All segments use static IPs for predictability and security.

**Decision Rationale:**
- pfSense chosen over Cisco IOS or Vyatta because it's open-source, lightweight, and includes integrated IDS/IPS.
- 5 segments chosen to separate victim, attacker, tools, management, and external traffic flows — critical for realistic SOC scenarios.

---

### 2. Identity Services (Active Directory)

**Virtual Appliance:** Windows Server 2019 (LAB-DC01)  
**Allocation:** 2 GB RAM, 60 GB Disk  
**Network:** VMnet1 (Victim LAN) at 10.0.1.10/24

**Build Steps:**

1. **OS Installation**
   - Clean Windows Server 2019 installation
   - Joined to no domain initially
   - Static IP: 10.0.1.10, Subnet: 255.255.255.0, Gateway: 10.0.1.1, DNS: 127.0.0.1

2. **Domain Promotion**
   ```powershell
   Add-WindowsFeature AD-Domain-Services
   Install-ADDSForest -DomainName lab.local -DomainMode Win2016 -ForestMode Win2016 -Force
   ```
   - Domain: `lab.local`
   - Forest/Domain Functional Level: 2016
   - ForestAdmin password: [Stored securely, not in git]

3. **BadBlood Noise Population**
   ```powershell
   git clone https://github.com/davidprowe/BadBlood
   cd BadBlood
   .\Badblood.ps1 -verbose
   ```
   - Generated ~2,500 users, 500+ groups, 100+ organizational units
   - Purpose: Simulate realistic enterprise noise to test detection algorithms
   - Result: Domain now has authentic "clutter" for SOC analysts to navigate

**Key Configuration Decisions:**

- **Windows Server 2019 (not 2022):** Lower resource footprint on 16GB host.
- **Forest Functional Level 2016:** Balances modern AD features with resource efficiency.
- **BadBlood Deployment:** Populating the domain with noise is critical for realistic log baselines and detection tuning.

---

### 3. Endpoint Visibility (Sysmon & CrowdSec)

**Deployed On:** LAB-DC01 and LAB-WS01

#### Sysmon Installation
```powershell
# Download and install Sysmon (latest)
choco install sysmon -y

# Validate with a custom config (process creation, network connections, registry)
sysmon -i -h md5,imphash,sha1,sha256 -l INFO
```

**Events Enabled:**
- **EID 1:** Process Creation
- **EID 3:** Network Connection
- **EID 11:** File Created
- **EID 7:** Image Loaded (DLLs)

**Validation:** 
```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" | Select-Object -First 10
```
✅ Confirmed firing 50-100 events per minute during idle state.

#### CrowdSec Installation
```powershell
# Install CrowdSec
choco install crowdsec -y

# Enroll with Crowdsec Hub
cscli.exe hub update
cscli.exe console enroll [ENROLLMENT_KEY]

# Enable and start service
systemctl enable crowdsec
systemctl start crowdsec
```

**Validation:**
```powershell
cscli status
cscli decisions list
```
✅ Enrollment successful. Behavioral blocking active.

**Troubleshooting Note:**
- **Issue Encountered:** DNS I/O timeout during `hub update` → Root cause: DC's DNS forwarders were not configured.
- **Resolution:** Added pfSense (10.0.1.1) and Google DNS (8.8.8.8) as forwarders in DNS Manager. Re-ran `hub update` successfully.

---

### 4. Deception Layer (OpenCanary Honeypot)

**Virtual Appliance:** Ubuntu Server 22.04 LTS (LAB-CANARY)  
**Allocation:** 512 MB RAM, 20 GB Disk  
**Network:** VMnet1 (Victim LAN) at 10.0.1.30/24

**Services Enabled:**
- SSH (22)
- HTTP (80)
- FTP (21)
- SMB (445)
- RDP (3389)

**Installation:**
```bash
sudo apt-get update
sudo apt-get install python3-pip python3-venv -y

git clone https://github.com/thinkst/opencanary.git
cd opencanary
pip3 install -r requirements.txt
python3 -m opencanary

# Configure /etc/opencanary/opencanary.conf
# - Set alert output to JSON for Splunk ingestion
# - Enable all services listed above
```

**JSON Log Format:**
```json
{
  "timestamp": "2026-05-11T10:30:15.123456+00:00",
  "type": "SSH",
  "src_host": "10.0.1.20",
  "src_port": 54321,
  "dst_host": "10.0.1.30",
  "dst_port": 22,
  "logtype": 2000,
  "event": "Attempted SSH login"
}
```

**Validation:**
```bash
sudo systemctl status opencanary
tail -f /var/log/opencanary/opencanary.log
```
✅ SSH, HTTP, and FTP services verified firing logs on connection attempts.

---

### 5. Domain-Joined Workstation (LAB-WS01)

**Virtual Appliance:** Windows 10 (LAB-WS01)  
**Allocation:** 2 GB RAM, 60 GB Disk  
**Network:** VMnet1 (Victim LAN) at 10.0.1.20/24

**Configuration:**
1. Static IP: 10.0.1.20, Gateway: 10.0.1.1, DNS: 10.0.1.10 (DC)
2. Domain join: `lab\Administrator` credentials
3. Sysmon deployment (same config as DC)
4. Windows Firewall: Disabled for attack simulation purposes

**Validation:**
```powershell
Get-ADComputer -Filter "Name -eq 'LAB-WS01'" | Select-Object -ExpandProperty Name
```
✅ Workstation confirmed joined to lab.local domain.

---

## 🔧 Issues Encountered & Resolutions

### Issue 1: Virtual Interface Mapping Confusion
**Symptom:** LAB-DC01 could not ping 10.0.1.1 despite correct static IP configuration.  
**Root Cause:** VMware had Ethernet0 (configured for 10.0.1.0/24) physically assigned to VMnet4 instead of VMnet1.  
**Time to Diagnosis:** 45 minutes (used MAC address matching to identify the swap).  
**Resolution:**
```
VMware VM Settings:
  Network Adapter 1: Custom VMnet1 (Victim LAN)
  Network Adapter 2: Custom VMnet4 (Management)
```
**Lesson Learned:** Always verify MAC addresses when debugging virtual interface issues; logical and physical assignments can diverge in VMware.

### Issue 2: Firewall Over-Restriction
**Symptom:** CrowdSec `hub update` failed with DNS timeout.  
**Root Cause:** pfSense default-drop policy blocked all outbound traffic except established connections.  
**Time to Diagnosis:** 20 minutes (identified via `pfctl -d` test disabling firewall temporarily).  
**Resolution:** Implemented granular "Allow" rules:
```
Pass Rule 1: Protocol=TCP/UDP, Source=LAN net, Port=53 (DNS)
Pass Rule 2: Protocol=TCP, Source=LAN net, Port=80,443 (HTTP/S)
Pass Rule 3: Protocol=ICMP, Source=LAN net (Troubleshooting)
```
**Lesson Learned:** Explicit "Allow" rules are more secure than broad "Allow Any" but require careful planning to avoid operational friction.

### Issue 3: DNS Resolver Initialization
**Symptom:** LAB-DC01 DNS service was misconfigured post-promotion.  
**Root Cause:** Windows DNS service was set to query only itself (127.0.0.1) without forwarders.  
**Time to Diagnosis:** 15 minutes (validated via `nslookup` tests).  
**Resolution:**
```
DNS Manager > LAB-DC01 > Properties > Forwarders:
  Add: 10.0.1.1 (pfSense)
  Add: 8.8.8.8 (Google Public DNS)
```
**Lesson Learned:** Post-promotion DNS configuration is a common gotcha; always add forwarders immediately after domain controller promotion.

---

## 📊 Final State Summary

| Component | Status | Metric |
|:---|:---|:---|
| **pfSense Router** | ✅ Live | 5 segments, 8 firewall rules, Suricata IDS active |
| **AD Domain** | ✅ Live | lab.local, ~2,500 objects, 2 computers |
| **Sysmon Sensors** | ✅ Live | 50-100 events/min (idle), firing on DC and WS |
| **CrowdSec** | ✅ Live | Behavioral blocking active, enrolled |
| **OpenCanary** | ✅ Live | 5 services running, JSON logs generated |
| **Network Connectivity** | ✅ Verified | All segments routable; external DNS accessible |

---

## 🎯 Transition to Phase 2

**Next Objective:** Deploy LAB-SIEM (Splunk) at 10.0.3.10 to ingest telemetry.

**Readiness:** ✅ All sensors are firing and ready to forward logs.

**Estimated Time:** 1-2 hours (Splunk download, install, UF deployment).

---

**Phase 1 Completed By:** Michael Ted  
**Completion Date:** 2026-05-11  
**Total Build Time:** ~8 hours (including troubleshooting)
