# Investigation: Spike in Failed Login Attempts

**Date:** 2026-02-03  
**Severity:** Low  
**Status:** Closed  benign explanation found

---

## 1. Alert

SIEM triggered: `Unusual volume of failed authentication events on workstation WS-014`  
Count: 34 failed events between 08:00 and 08:07 UTC.

---

## 2. Context

Host: `WS-014` (Windows 10 workstation)  
User involved: `jmwangi`  
Time: Morning login window

---

## 3. Evidence

### Windows Event Log (Event ID 4625  Failed Logon)
```
Event ID: 4625
Account Name: jmwangi
Failure Reason: Unknown username or bad password
Logon Type: 2 (Interactive)
Source: WS-014
Count: 34 events over 7 minutes
```

### Follow-up: Event ID 4624 (Successful Logon)
```
Event ID: 4624
Account Name: jmwangi
Logon Type: 2
Time: 08:07:43 UTC
```

---

## 4. Conclusion

User `jmwangi` had forgotten their password after returning from leave. The spike is entirely from manual typing attempts followed by a successful login after a password reset. No malicious activity.

---

## 5. Recommendation

- No action needed on the security side
- Ticket raised with IT helpdesk to remind user of self-service password reset portal
- Adjust SIEM alert threshold for interactive logon failures (Type 2) to reduce noise: suggest 15+ failures before alerting, or add time-of-day context
