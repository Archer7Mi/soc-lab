# Investigation: SSH Brute Force Alert

**Date:** 2026-01-15  
**Severity:** Medium  
**Status:** Resolved

---

## 1. Alert

SIEM triggered: `Multiple failed SSH authentication attempts from single source IP`  
Threshold: 10 failed attempts within 60 seconds.

---

## 2. Context

Source IP: `192.168.1.45`  
Target: `ubuntu-server-01` (192.168.10.5)  
Timeframe: 03:12  03:14 UTC  
Protocol: SSH (TCP/22)

---

## 3. Evidence

### Log snippet (auth.log)
```
Jan 15 03:12:04 ubuntu-server-01 sshd[3821]: Failed password for root from 192.168.1.45 port 51230 ssh2
Jan 15 03:12:06 ubuntu-server-01 sshd[3821]: Failed password for root from 192.168.1.45 port 51231 ssh2
Jan 15 03:12:08 ubuntu-server-01 sshd[3822]: Failed password for admin from 192.168.1.45 port 51232 ssh2
Jan 15 03:12:09 ubuntu-server-01 sshd[3822]: Failed password for admin from 192.168.1.45 port 51233 ssh2
Jan 15 03:12:11 ubuntu-server-01 sshd[3823]: Failed password for ubuntu from 192.168.1.45 port 51234 ssh2
```

Attempted usernames: `root`, `admin`, `ubuntu`, `test`, `user`  
Total attempts in window: 47  
Successful logins: **0**

### Parsing the log with grep
```bash
grep "Failed password" /var/log/auth.log | grep "192.168.1.45" | wc -l
# Output: 47

grep "Accepted password" /var/log/auth.log | grep "192.168.1.45"
# Output: (none)
```

### Wireshark observation
Packet capture on eth0 confirmed rapid sequential TCP SYN packets to port 22 from `192.168.1.45`, consistent with automated tooling (sub-second timing between attempts).

---

## 4. Conclusion

This is a **dictionary/automated brute force attack** targeting common usernames. No successful authentication occurred. The source IP appears to be an internal host  possible compromised machine or misconfigured script.

---

## 5. Recommendation

- Block `192.168.1.45` at the firewall pending investigation of the source machine
- Disable SSH password authentication; enforce key-based auth only
- Enable `fail2ban` on the target server to auto-ban IPs after N failures
- Review what process on `192.168.1.45` initiated the scan
