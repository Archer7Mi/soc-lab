# Network Design: SOC Lab Architecture

**Authoritative Blueprint for Virtual Network Layout**

**Last Updated:** 2026-05-11

---

## 🏗 Network Topology

```
┌─────────────────────────────────────────────────────────────┐
│                    HOME ROUTER / ISP                         │
│                   (VMnet0 — Bridged)                         │
└────────────────────────┬────────────────────────────────────┘
                         │ (WAN — Dynamic IP via DHCP)
                         │
              ┌──────────▼──────────┐
              │   pfSense Firewall   │
              │    em0 (WAN)         │
              │    em1-em4 (LANs)    │
              └──┬──────┬──────┬─────┘
                 │      │      │      ┌────────────┐
                 │      │      │      │ VMnet0     │
                 │      │      │      │ (Bridged)  │
                 │      │      │      └────────────┘
                 │      │      │
    ┌────────────┘      │      └──────────────┐
    │                   │                     │
    ▼                   ▼                     ▼
┌────────────┐    ┌────────────┐        ┌──────────────┐
│  VMnet1    │    │  VMnet2    │        │   VMnet3     │
│ Victim     │    │ Attacker   │        │ Security     │
│ LAN        │    │ LAN        │        │ Tools        │
│10.0.1.0/24│    │10.0.2.0/24 │        │ 10.0.3.0/24  │
└─┬──────────┘    └────────────┘        └──────────────┘
  │
  ├─ LAB-DC01       ├─ LAB-KALI          ├─ LAB-SIEM
  │  10.0.1.10      │  10.0.2.10         │  10.0.3.10
  │                 │                    │  (Splunk)
  ├─ LAB-WS01       │                    ├─ LAB-WAZUH
  │  10.0.1.20      │                    │  10.0.3.20
  │                 │                    │  (XDR)
  └─ LAB-CANARY     │                    ├─ LAB-HIVE
     10.0.1.30      │                    │  10.0.3.30
                    │                    │  (TheHive)
                    │                    ├─ LAB-SHUFFLE
                    │                    │  10.0.3.40
                    │                    │  (SOAR)
                    │                    └─ LAB-ZEEK
                    │                       10.0.3.50
                    │                       (NIDS)
                    │
                    ┌─────────────────────┐
                    │    VMnet4 — MGMT    │
                    │   10.0.4.0/24       │
                    ├─────────────────────┤
                    │ Out-of-band admin   │
                    │ Console access only │
                    └─────────────────────┘
                            │
                            ├─ 10.0.4.1   (pfSense MGMT gateway)
                            ├─ 10.0.4.10  (LAB-DC01 MGMT NIC)
                            ├─ 10.0.4.20  (LAB-WS01 MGMT NIC)
                            ├─ 10.0.4.30  (LAB-SPLUNK MGMT NIC)
                            ├─ 10.0.4.40  (LAB-HIVE MGMT NIC)
                            ├─ 10.0.4.254 (Host Windows 11 MGMT)
```

---

## 📊 IP Addressing Table

### VMnet1 — Victim LAN (10.0.1.0/24)

| Hostname | IP Address | MAC Address | Purpose |
|:---|:---|:---|:---|
| pfSense LAN GW | 10.0.1.1 | (auto) | Network gateway, Suricata IDS |
| LAB-DC01 | 10.0.1.10 | 00:0c:29:da:6f:6b | Domain Controller, Sysmon, CrowdSec |
| LAB-WS01 | 10.0.1.20 | (auto) | Victim Workstation, Sysmon |
| LAB-CANARY | 10.0.1.30 | (auto) | OpenCanary Honeypot |

**Subnet Mask:** 255.255.255.0  
**Gateway:** 10.0.1.1 (pfSense)  
**DNS:** 10.0.1.10 (LAB-DC01)  
**Broadcast:** 10.0.1.255  

**Traffic Flow:** All outbound traffic from Victim LAN passes through pfSense firewall (em1 interface) to pfSense WAN (em0) for internet or to other segments (OPT1-3).

---

### VMnet2 — Attacker LAN (10.0.2.0/24)

| Hostname | IP Address | MAC Address | Purpose |
|:---|:---|:---|:---|
| pfSense OPT1 GW | 10.0.2.1 | (auto) | Network gateway, Suricata IDS |
| LAB-KALI | 10.0.2.10 | (auto) | Kali Linux, attack simulation |

**Subnet Mask:** 255.255.255.0  
**Gateway:** 10.0.2.1 (pfSense)  
**DNS:** Automatic (pfSense forwarder)  
**Broadcast:** 10.0.2.255  

**Traffic Flow:** LAB-KALI can initiate attacks toward Victim LAN (10.0.1.0/24) and receive return traffic, but is blocked from accessing Management (10.0.4.0/24) by pfSense firewall rules.

**Firewall Rule Applied:**
```
Destination: LAB-KALI
Action: Block
(Victim LAN cannot initiate connections to Attacker LAN)
```

---

### VMnet3 — Security Tools (10.0.3.0/24)

| Hostname | IP Address | MAC Address | Purpose |
|:---|:---|:---|:---|
| pfSense OPT2 GW | 10.0.3.1 | (auto) | Network gateway, Suricata IDS |
| LAB-SIEM | 10.0.3.10 | (auto) | Splunk Enterprise (SIEM) |
| LAB-WAZUH | 10.0.3.20 | (auto) | Wazuh XDR/EDR platform |
| LAB-HIVE | 10.0.3.30 | (auto) | TheHive case management |
| LAB-SHUFFLE | 10.0.3.40 | (auto) | Shuffle SOAR orchestration |
| LAB-ZEEK | 10.0.3.50 | (auto) | Zeek NIDS |

**Subnet Mask:** 255.255.255.0  
**Gateway:** 10.0.3.1 (pfSense)  
**DNS:** Automatic (pfSense forwarder)  
**Broadcast:** 10.0.3.255  

**Traffic Flow:** Tools segment receives inbound logs/telemetry from Victim LAN via:
- Splunk Universal Forwarder on LAB-DC01 (port 9997 / HEC 8088)
- Zeek listening on pfSense mirror port
- Syslog from pfSense Suricata (UDP 514)

**Firewall Rules Applied:**
```
Source: LAN net (10.0.1.0/24)
Destination: 10.0.3.10 (LAB-SIEM)
Protocol: TCP
Port: 9997, 8088 (Splunk)
Action: Pass

Source: LAN net
Destination: 10.0.3.0/24
Protocol: UDP
Port: 514 (Syslog)
Action: Pass
```

---

### VMnet4 — Management (10.0.4.0/24)

| Hostname | IP Address | MAC Address | Purpose |
|:---|:---|:---|:---|
| pfSense MGMT GW | 10.0.4.1 | (auto) | Management network gateway |
| LAB-DC01 (NIC2) | 10.0.4.10 | 00:0c:29:da:6f:75 | DC out-of-band console |
| LAB-WS01 (NIC2) | 10.0.4.20 | (auto) | Workstation out-of-band console |
| LAB-SIEM (NIC2) | 10.0.4.30 | (auto) | Splunk out-of-band console |
| LAB-HIVE (NIC2) | 10.0.4.40 | (auto) | TheHive out-of-band console |
| Host Windows 11 | 10.0.4.254 | (auto) | Hypervisor management access |

**Subnet Mask:** 255.255.255.0  
**Gateway:** None (isolated segment, no routing)  
**DNS:** None (no external queries from this segment)  
**Broadcast:** 10.0.4.255  

**Traffic Flow:** Management segment is **strictly isolated**. Each machine has a secondary NIC on VMnet4 for direct console/SSH access from the hypervisor (10.0.4.254) without transiting production networks.

**Purpose:** Out-of-band access ensures lab can be recovered if production network is compromised or misconfigured.

**Firewall Rules Applied:**
```
Destination: 10.0.4.0/24 (Management)
Action: Block (from all other segments)
```

---

### VMnet0 — Bridged (WAN)

| Interface | IP Address | Purpose |
|:---|:---|:---|
| pfSense em0 (WAN) | DHCP from ISP | Internet uplink |
| Host Windows 11 | ISP-assigned | Lab connectivity to internet (for software downloads) |

**Traffic Flow:** Only the pfSense WAN interface and the host Windows 11 are bridged to the physical network. All internal lab traffic remains isolated.

---

## 🔗 Data Flow Diagram

### Scenario 1: Normal Operations (Splunk Log Ingestion)

```
LAB-DC01 (Sysmon events)
    ↓
    └─→ Splunk Universal Forwarder (TCP 9997)
            ↓
            LAB-SIEM (Splunk HEC Listener)
                ↓
                Splunk Index: main, sysmon
                    ↓
                    SPL Query (Detection logic)
                        ↓
                        Alert → TheHive (webhook)
```

### Scenario 2: Attack Simulation

```
LAB-KALI (Nmap, Hydra, Metasploit)
    ↓
    └─→ Victim LAN (10.0.1.0/24)
            ↓
            pfSense (Suricata IDS)
                ├─→ Alert Log → Syslog to LAB-SIEM (UDP 514)
                ├─→ Pass/Block Decision per firewall rules
                │
            LAB-DC01 / LAB-WS01
                ├─→ Sysmon (Process, Network, Registry events)
                │   └─→ LAB-SIEM (via Universal Forwarder)
                │
            LAB-CANARY (OpenCanary)
                ├─→ JSON logs (SSH, HTTP, FTP, RDP attempts)
                    └─→ LAB-SIEM (Splunk Input)
```

### Scenario 3: Incident Response

```
Alert Triggered (Splunk)
    ↓
    Webhook to TheHive
        ↓
        Case Created (analyst starts triage)
            ↓
            Evidence Collection (logs, artifacts)
                ↓
                Cortex Enrichment (IP reputation, file hashes)
                    ↓
                    Shuffle (SOAR) Workflow Automation
                        ├─→ Isolate endpoint (future)
                        ├─→ Send notification
                        └─→ Close case
```

---

## 🔐 Firewall Rule Matrix

| Source | Destination | Port(s) | Protocol | Action | Purpose |
|:---|:---|:---|:---|:---|:---|
| LAN (10.0.1.0/24) | pfSense LAN (10.0.1.1) | 53 | UDP | PASS | DNS resolution |
| LAN | Any | 53 | UDP | PASS | DNS resolution |
| LAN | Any | 80, 443 | TCP | PASS | HTTP/HTTPS web traffic |
| LAN | Any | 123 | UDP | PASS | NTP time sync |
| LAN | 10.0.3.10 | 9997 | TCP | PASS | Splunk log forwarding |
| LAN | 10.0.3.10 | 8088 | TCP | PASS | Splunk HEC |
| LAN | 10.0.3.0/24 | 514 | UDP | PASS | Syslog (Suricata) |
| LAN | 10.0.4.0/24 | Any | Any | BLOCK | Block access to management |
| LAN | 10.0.2.0/24 | Any | Any | BLOCK | Block access to attacker LAN |
| OPT1 (10.0.2.0/24) | LAN (10.0.1.0/24) | Any | Any | PASS | Allow attacks for simulation |
| OPT1 | 10.0.4.0/24 | Any | Any | BLOCK | Block attacker to management |
| OPT2 (10.0.3.0/24) | LAN | Any | Any | PASS | Tools can query victim logs |
| WAN | LAN | Any | Any | BLOCK | Block external inbound |
| Any | Any | * | ICMP | PASS | Troubleshooting (ping) |

**Last "Any" Rule:** Default DROP (implicit).

---

## 🔄 Network Segmentation Rationale

### Why 5 Segments?

1. **VMnet1 (Victim LAN):** Isolated "target" network. All honeypots, DCs, and workstations live here. Represents the "company network" being protected.

2. **VMnet2 (Attacker LAN):** Isolated "attacker" network (Kali Linux). Unreachable from victim LAN by default. Used for controlled attack simulations only.

3. **VMnet3 (Security Tools):** Isolated "detection" network. SIEM, XDR, case management, and SOAR live here. Cannot be reached by workstations in Victim LAN except on specific ports (9997, 8088, 514).

4. **VMnet4 (Management):** Out-of-band network for emergency access. No production traffic ever traverses this network. Allows lab recovery if production is compromised.

5. **VMnet0 (WAN):** Bridged to home network for internet connectivity. Only pfSense em0 and host Windows 11 are connected; all internal VMs are isolated.

### Attack Surface Reduction

By default:
- ✅ Victim LAN cannot reach Attacker LAN (no lateral movement opportunities)
- ✅ Attacker LAN cannot reach Management (no "phone home" to hypervisor)
- ✅ External internet cannot reach any internal segment (firewall default-drop)
- ✅ Tools segment is accessible only on specific log ingestion ports (9997, 8088, 514)

---

## 🧪 Verification Checklist

Use these commands to verify the network is correctly configured:

### From LAB-DC01 (Victim LAN)
```powershell
# Verify gateway reachability
ping 10.0.1.1

# Verify external DNS
nslookup google.com 8.8.8.8

# Verify tools segment access (should fail on other ports)
Test-NetConnection -ComputerName 10.0.3.10 -Port 9997 -InformationLevel Quiet

# Verify firewall rules block management access (should fail)
Test-NetConnection -ComputerName 10.0.4.1 -Port 22 -InformationLevel Quiet
```

### From LAB-KALI (Attacker LAN)
```bash
# Verify victim LAN is reachable
ping 10.0.1.10

# Verify tools segment is reachable
ping 10.0.3.10

# Verify management is NOT reachable (should timeout)
ping 10.0.4.1 -c 3

# Verify can establish reverse shell port (should succeed on victim)
nc -lnvp 4444 &
```

### From pfSense Console
```bash
# Verify interface status
ifconfig em1
ifconfig em2
ifconfig em3
ifconfig em4

# Verify firewall rules are loaded
pfctl -sr | grep "pass"

# Verify Suricata is running
service suricata status
```

---

## 📝 DNS Configuration

### DC-hosted DNS (Internal Lookup)
- **Primary:** LAB-DC01 (10.0.1.10)
- **Forwarders:** 10.0.1.1 (pfSense), 8.8.8.8 (Google)
- **Zone:** lab.local (internal, not publicly resolvable)

### pfSense DNS Forwarder (Outbound Queries)
- **Listen On:** 10.0.1.1 (LAN interface)
- **Upstream Servers:** ISP DNS (via DHCP) + 8.8.8.8 (fallback)

### Client DNS Resolution
- **LAB-DC01:** 127.0.0.1 (self-hosted AD-integrated DNS)
- **LAB-WS01:** 10.0.1.10 (DC-hosted DNS)
- **LAB-CANARY:** DHCP from pfSense (10.0.1.1)
- **Tools Segment:** DHCP from pfSense (10.0.3.1)

---

## 🎯 Future Expansion

**Reserved IP Ranges (for Phase 2+):**
- **10.0.1.31 - 10.0.1.254:** Victim LAN dynamic IPs (future workstations, servers)
- **10.0.3.51 - 10.0.3.254:** Tools segment dynamic IPs (future security appliances)
- **10.0.4.50 - 10.0.4.253:** Management segment dynamic IPs (future appliance MGMT NICs)

---

**Last Updated:** 2026-05-11  
**Approved By:** Michael Ted (Lab Architect)  
**Review Schedule:** Upon Phase 2 completion
