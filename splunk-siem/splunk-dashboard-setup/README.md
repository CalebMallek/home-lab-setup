# Splunk SOC Lab Dashboard & Alert Setup

This guide documents the full process of setting up **Windows event collection, dashboards, and alerts** in Splunk for a SOC lab environment.

---

## 1. Prerequisites

* **Virtual Machines:**

  * Windows Endpoint VM
  * Kali Attacker VM 
  * Admin / Splunk VM
  * Splunk-siem ubuntu server VM
* **Network:**

  * Properly segmented network allowing Splunk VM to receive Windows logs
* **Splunk Enterprise / Dashboard Studio installed on Admin VM**
* **Windows Event Forwarding configured** to send logs to Splunk

---

## 2. Windows Log Collection

**Key Event IDs Monitored:**

| Event ID | Description              |
| -------- | ------------------------ |
| 4624     | Successful logon         |
| 4625     | Failed logon             |
| 4672     | Privileged account logon |
| 4688     | Process creation         |
| 5140     | Network share access     |

**Configuration Steps:**

1. Ensure Windows VM is forwarding Security logs to Splunk.
2. Verify Splunk receives logs:

```spl
index=windows_logs
```

3. Troubleshoot connectivity and firewall issues (UDP/514, Splunk port open).

---

## 3. Dashboard Setup (Dashboard Studio)

**Panels Created:**

1. **Failed Logins (4625)** – raw table

```spl
index=windows_logs EventCode=4625 | table _time, Account_Name, host, src, Message | sort -_time
```

2. **Top 10 Failed Login Accounts** – bar chart

```spl
index=windows_logs EventCode=4625 | top Account_Name limit=10
```

3. **Privileged Logons (4672)** – table

```spl
index=windows_logs EventCode=4672 | table _time, Account_Name, host, src, Message | sort -_time
```

4. **Process Creation (4688)** – table

```spl
index=windows_logs EventCode=4688
| eval Suspicious=if(match(New_Process_Name,"(?i)powershell\.exe|cmd\.exe|mimikatz\.exe"),"Yes","No")
| table _time, Account_Name, host, New_Process_Name, Command_Line, Suspicious
| sort -_time
```

5. **Network Share Access (5140)** – table

```spl
index=windows_logs EventCode=5140 | table _time, Account_Name, host, Share_Name, src, Message | sort -_time
```

**Optional:** create a **“Suspicious Processes Only” panel** using a `where Suspicious="Yes"` filter.

---

## 4. Alert Setup

### 4.1 Failed Login Threshold (4625)

**Search:**

```spl
index=windows_logs EventCode=4625
| stats count by Account_Name
| where count > 5
```

**Alert Configuration:**

* Schedule: Cron (`* * * * *`) – every minute for lab testing
* Trigger Condition: Number of Results > 0
* Trigger Actions: Add to Triggered Alerts / Notable Events
* Severity: Medium/High
* Throttle: Off for lab

---

### 4.2 Suspicious Process Execution (4688)

**Search:**

```spl
index=windows_logs EventCode=4688
| eval Suspicious=if(match(New_Process_Name,"(?i)powershell\.exe|cmd\.exe|mimikatz\.exe"),"Yes","No")
| where Suspicious="Yes"
| table _time, Account_Name, host, New_Process_Name, Command_Line
| sort -_time
```

**Alert Configuration:**

* Schedule: Cron (`* * * * *`) – every minute
* Trigger Condition: Number of Results > 0
* Trigger Actions: Add to Triggered Alerts / Notable Events
* Severity: Medium/High
* Throttle: Off for lab

---

## 5. Testing Workflow

1. Generate **failed login events** on Windows → alert triggers.
2. Launch **suspicious processes** (`powershell.exe`, `cmd.exe`, `mimikatz.exe`) → alert triggers.
3. Access **network shares** → events appear in dashboard panel.
4. Confirm **Triggered Alerts** / Notable Events appear in Splunk.
5. Investigate events via dashboard tables, top 10 charts, and “Suspicious Only” panels.

---

## 6. Recommendations / Optional Enhancements

* Add trend charts for failed logins or process creation.
* Include Kali attacker VM simulations for lateral movement.
* Add dropdown filters for Account_Name or host.
* Expand alert coverage (admin share access, RDP logins, etc.).

---

## ✅ Outcome

Following this guide gives you:

* A **fully functional SOC lab dashboard** monitoring key Windows events.
* Alerts for **failed logins** and **suspicious processes**.
* A framework to **simulate SOC investigations** and practice alert triage.
