# Splunk SIEM Server & Windows Endpoint Integration

This guide documents the full setup, configuration, and troubleshooting process for creating and integrating a **Splunk SIEM Server VM** with a **Windows Endpoint** and managing everything through the **Admin VM**. It covers every step taken to achieve a functional SOC environment capable of collecting, forwarding, and analyzing Windows event logs.

---

## 🧩 Lab Environment Overview

| VM Name              | Role            | OS             | Purpose                                                  |
| -------------------- | --------------- | -------------- | -------------------------------------------------------- |
| **splunk-siem**      | SIEM Server     | Ubuntu Server  | Hosts Splunk Enterprise for log aggregation and analysis |
| **windows-endpoint** | Endpoint        | Windows 10     | Simulated workstation sending logs to Splunk             |
| **admin-vm**         | Admin Console   | Ubuntu Desktop | Used to manage Splunk via web interface (port 8000)      |
| **kali-attacker**    | Attacker        | Kali Linux     | Used for future attack simulation                        |

> Network: All machines are connected via the internal lab network (`192.168.56.0/24`).

---

## ⚙️ Splunk SIEM Server Setup (Ubuntu Server)

### 1. Install Splunk Enterprise

```bash
cd /opt
sudo dpkg -i splunk_package.deb
sudo /opt/splunk/bin/splunk start --accept-license
```

Create an admin user when prompted.
Splunk web interface runs on **http://<Splunk-IP>:8000** (e.g., `http://192.168.56.50:8000`).

### 2. Enable Boot Start

```bash
sudo /opt/splunk/bin/splunk enable boot-start
```

### 3. Verify Service and Listening Ports

```bash
sudo systemctl status Splunkd
sudo ss -tulpen | grep 9997
```

Port `9997` must show as **LISTEN**, which confirms Splunk is ready to receive logs.

---

## 💻 Windows Endpoint Integration (Universal Forwarder)

### 1. Install Splunk Universal Forwarder

Download and install the Windows UF on the endpoint.

During setup:

* Forwarder type: **Forward logs to a Splunk indexer**
* Indexer hostname/IP: **192.168.56.50** (Splunk SIEM)
* Port: **9997**
* Admin credentials: configure a local Splunk UF user

### 2. Verify Forwarder Connection

In PowerShell:

```powershell
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe list forward-server
```

Output should show:

```
Active forwards:
    192.168.56.50:9997
Configured but inactive forwards:
    None
```

If “inactive,” verify:

* The Splunk server is running (`sudo systemctl status Splunkd`)
* Windows firewall allows outbound port 9997
* The UF service is running (`services.msc → SplunkForwarder → Running`)

### 3. Configure Event Log Collection

Create the file:

```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

Add:

```
[WinEventLog://Application]
disabled = 0
index = windows_logs
sourcetype = WinEventLog:Application

[WinEventLog://System]
disabled = 0
index = windows_logs
sourcetype = WinEventLog:System

[WinEventLog://Security]
disabled = 0
index = windows_logs
sourcetype = WinEventLog:Security
```

Restart the forwarder service:

```powershell
net stop splunkforwarder
net start splunkforwarder
```

---

## 🌐 Admin VM: Accessing Splunk Web

From the Admin VM (or host browser):

```
http://192.168.56.50:8000
```

Login with your Splunk admin credentials.

### Search & Reporting

Navigate to:
**Search & Reporting → Search → index=windows_logs**

Run:

```spl
index=windows_logs | stats count
```

This shows the total number of logs ingested from Windows.

---

## 🧪 Generating Windows Logs for Testing

In PowerShell:

```powershell
EventCreate /ID 100 /L APPLICATION /T INFORMATION /SO TestApp /D "Test Application log"
EventCreate /ID 200 /L SYSTEM /T INFORMATION /SO TestSystem /D "Test System log"
```

Or create multiple entries:

```powershell
for ($i=1; $i -le 20; $i++) {
    EventCreate /ID $i /L APPLICATION /T INFORMATION /SO TestApp /D "Test Application log $i"
}
```

Then search again:

```spl
index=windows_logs | stats count
```

---

## 🔍 Troubleshooting Summary

| Issue                                                    | Cause                                                | Fix                                                                 |
| -------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------- |
| **Splunk not starting / missing directory**              | Install incomplete or incorrect path                 | Verified `/opt/splunk` and reinstalled correctly                    |
| **“No such file or directory” during enable boot-start** | Wrong installation path                              | Confirmed correct `/opt/splunk/bin/splunk`                          |
| **UF “inactive” forwarder**                              | Port or service issue                                | Restarted Splunk & SplunkForwarder; confirmed port 9997 open        |
| **Server certificate hostname validation disabled**      | Network config mismatch                              | Ignored safely in isolated lab                                      |
| **No new events appearing**                              | UF not monitoring event logs                         | Created `inputs.conf` and restarted service                         |
| **inputs.conf named incorrectly**                        | Saved as `inputs.conf.txt`                           | Fixed by renaming to `inputs.conf`                                  |
| **PowerShell “not recognized” errors**                   | Ran commands outside UF directory                    | Used full path `cd "C:\Program Files\SplunkUniversalForwarder\bin"` |
| **EventCreate ID out of range**                          | IDs >1000 invalid                                    | Used IDs 1–1000                                                     |
| **Service account confusion**                            | SplunkForwarder runs as `NT SERVICE\SplunkForwarder` | Confirmed service account permissions                               |
| **Logs not updating live**                               | UF only forwards periodic batches                    | Confirmed logs update within ~30 seconds of event creation          |

---

## 📊 Validation Steps

1. In Splunk Web:

   ```spl
   index=windows_logs | stats count by host, sourcetype
   ```
2. Check **Data Summary → Hosts / Sourcetypes** for `WinEventLog:Application`, `WinEventLog:System`, and `WinEventLog:Security`.
3. Generate test logs again — verify count increases.
4. Confirm port 9997 open on Splunk Server:

   ```bash
   sudo ss -tulpen | grep 9997
   ```

---

## ✅ SOC Lab Progress Summary

| Component          | Status                           | Notes                            |
| ------------------ | -------------------------------- | -------------------------------- |
| Splunk SIEM Server | ✅ Configured & running on Ubuntu | Listening on 9997                |
| Windows Endpoint   | ✅ Forwarding event logs          | Tested via EventCreate           |
| Admin VM           | ✅ Access via Splunk Web          | Dashboard functional             |
| Kali Attacker      | ⚙️ Ready for attack simulation   | Not yet used                     |
| Windows DC         | ⏸️ Planned for later phase       | Domain log integration postponed |

---

## 🧠 Next Steps

* Create detection dashboards and basic alerts (failed logins, privilege escalations, etc.).
* Simulate attacks from **Kali VM** and analyze detections.
* Later: deploy **Windows DC** for Active Directory log ingestion.

---

## 🧾 Credits

All configuration and troubleshooting performed manually in a VirtualBox SOC lab environment for educational and portfolio purposes.
