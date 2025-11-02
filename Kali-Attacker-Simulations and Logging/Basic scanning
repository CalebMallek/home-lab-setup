# Nmap scans against Windows endpoint (192.168.58.10) — lab notes, commands, and Splunk checks

**Date:** 2025-11-02  
**Author:** Caleb (lab)

---

## Overview & purpose
This document records the reconnaissance and scanning activity performed from a Kali attacker VM to a Windows endpoint VM in a home SOC lab. It contains exact commands used, notes about what was (and wasn't) visible in Splunk, troubleshooting steps taken, and recommended next actions.

**Safety:** only run these commands in your lab on machines you own or have explicit permission to test.

---

## Lab topology (short)
- **Kali attacker VM** — example IP(s): `192.168.56.57`, `192.168.58.20` (attacker)
- **Windows endpoint VM** — IP: `192.168.58.10` (target)
- **Splunk** — centralized logging (Windows events sent to `index=windows_logs` in this lab)
- **Windows host value in Splunk:** `Windows-Endpoin` (literal)

---

## Files to generate / recommended filenames
- `scan_quick_192.168.58.10.nmap`, `.gnmap`, `.xml` — quick top-ports scan output
- `scan_full_192.168.58.10.nmap`, `.gnmap`, `.xml` — full `-p- -A` scan output (if you run it)
- `scan_connect_192.168.58.10.nmap`, `.gnmap`, `.xml` — TCP connect (`-sT`) scan output
- `scan_capture_192.168.58.10.pcap` — optional tcpdump/Wireshark capture

---

## Exact commands run 

### Recon / discovery
```bash
# Check Kali interfaces and routing
ip a
route -n

# Quick host discovery (ICMP/ARP ping sweep)
sudo nmap -sn 192.168.58.0/24
```

### Stealth SYN scan (half‑open) — initial approach
```bash
# top 1000 common ports, SYN scan (root required)
sudo nmap -sS -T4 --top-ports 1000 --stats-every 15s -oA scan_quick_192.168.58.10 192.168.58.10
```

### Full noisy SYN scan (all TCP ports, aggressive scripts) — planned / discussed
```bash
# WARNING: noisy. scans all 65k TCP ports and runs NSE scripts.
sudo nmap -sS -sV -O -A -p- -T3 --stats-every 30s -oA scan_full_192.168.58.10 192.168.58.10
```

### TCP connect scan (completes handshake) — used to generate Windows-side connection logs
```bash
# Connect scan ports 1-1000 (completes TCP handshake; useful for Sysmon EventCode 3)
sudo nmap -sT -p1-1000 -T4 --stats-every 10s -oA scan_connect_192.168.58.10 192.168.58.10
```

### Targeted service/version + safe NSE scripts (non-destructive)
```bash
# Safe enumeration on specific ports
sudo nmap -sS -sV --script=banner,safe -p 22,80,135,139,445,3389 -T4 -oA scan_nse_192.168.58.10 192.168.58.10
```

### Capture network traffic (run in separate terminal during scans)
```bash
sudo tcpdump -i eth1 host 192.168.58.10 -w scan_capture_192.168.58.10.pcap
```

---

## Splunk checks & queries (what was tried and why)
> **Index used in this lab:** `windows_logs` (your environment may differ). Host value for Windows: `Windows-Endpoin`.
Below are the simplified, copy‑pasteable Splunk queries used and the rationale behind each.

### 1) Basic check — show recent events from the Windows host
```spl
index=windows_logs host="Windows-Endpoin" earliest=-60m
| head 20
| table _time, sourcetype, _raw
```
**Purpose:** prove that logs from the host are arriving and inspect raw message format.

### 2) Super-simple IP literal search (works only if IP appears in raw text)
```spl
index=windows_logs host="Windows-Endpoin" "192.168.56.12"
| table _time, host, sourcetype, _raw
```
**Note:** this returns results only if the Kali IP appears literally in `_raw` text of the events.

### 3) Extract any IPv4 strings from `_raw` and search for attacker IP (robust when IPs are in plain text)
```spl
index=windows_logs host="Windows-Endpoin"
| rex "(?<any_ip>\d{1,3}(?:\.\d{1,3}){3})"
| search any_ip="192.168.56.57"
| table _time, host, sourcetype, any_ip, _raw
```
**Purpose:** works even when IPs aren’t structured fields; it pulls IPv4-like tokens from `_raw` and matches them.

### 4) Sysmon network connections (EventCode 3) — works only if Sysmon was installed with `-n`
```spl
index=windows_logs host="Windows-Endpoin" EventCode=3 earliest=-60m
| table _time, SourceIp, SourcePort, DestinationIp, DestinationPort, ProcessName, Image
```
**Important:** Sysmon logs EventCode 3 **only for established connections** and only if Sysmon was installed with the `-n` flag before the connections occurred.

### 5) Windows Firewall connection auditing (EventCode 5156)
```spl
index=windows_logs host="Windows-Endpoin" EventCode=5156 earliest=-60m
| table _time, SourceAddress, SourcePort, DestinationAddress, DestinationPort, Application, _raw
```
**Purpose:** firewall auditing can show inbound connection attempts (including half‑open/blocked attempts depending on settings).

### 6) Broad search across accessible indexes (if you’re unsure which index logs landed in)
```spl
index=* "192.168.56.57"
| table _time, host, index, sourcetype, _raw
```
**Purpose:** find which index and host contain the attacker IP if the `windows_logs` index is not correct for your deployment.

---

## Findings so far (what worked and what didn’t)
- **SYN scans (`-sS`)** were used initially (stealth, half‑open). Those **do not produce Sysmon EventCode 3** entries because the connection handshake is not completed — Sysmon only logs fully established connections on the host side. Therefore, `-sS` scan activity did not appear in Sysmon-derived Splunk searches.
- **Sysmon was running**, but early installs were not configured with network connection logging (`-n`), so historical network events were absent. After reinstalling Sysmon with `-n` (uninstall then `Sysmon64.exe -i -n -accepteula`), new network logs are generated only for connections occurring *after* the reinstall.
- **TCP connect scans (`-sT`)** were run to complete handshakes and thereby generate Windows-side session logs — these are detectable in Sysmon EventCode 3 (if Sysmon has `-n`) or Windows Firewall auditing.
- Initial Splunk searches returned nothing because IPs did not appear in `_raw` and structured fields were not present. This is a data-collection issue (what the forwarder is sending) rather than a Splunk query syntax problem.
- The Windows host in Splunk indexed as `Windows-Endpoin`, and Windows host actual management IP in the network was `192.168.56.12` (note: lab IPs vary per VM, take care to map hostnames ↔ IPs correctly).

---

## Troubleshooting steps taken
1. Verified Splunk had events for `Windows-Endpoin` by using `index=windows_logs | head 20` and inspecting `_raw`.
2. Tried literal searches for the Kali IP; returned nothing because `_raw` did not contain IPs.
3. Tried `rex` extraction of IPv4 strings from `_raw` — still nothing, confirming logs lacked IP addresses.
4. Confirmed Sysmon was installed; found it was originally installed without `-n`, so EventCode 3 entries were missing historically.
5. Uninstalled and reinstalled Sysmon with network logging enabled:  
   ```powershell
   Sysmon64.exe -u
   Sysmon64.exe -i -n -accepteula
   wevtutil cl Microsoft-Windows-Sysmon/Operational   # optional clear
   ```
6. Ran `nmap -sT -p1-1000` to generate completed TCP connections so Sysmon (post-`-n`) and/or firewall logs would have evidence of the scan.

---

## Next recommended steps (short)
1. Re-run the TCP connect scan with `-oA` to save outputs if you haven't already. Attach human-readable `.nmap` summaries into `outputs/`.
2. Verify in Splunk using the `EventCode=3` (Sysmon) or `EventCode=5156` (Windows Firewall) queries after the scan timeframe. If nothing shows, check the forwarder and time picker.
3. Create a Splunk alert that fires when a single source hits > X destination ports in Y minutes (tuning needed for lab). Example detection logic: `stats count by src_ip dest_ip dest_port` and alert when `count` exceeds a threshold for `src_ip`.
4. Document and snapshot VMs before attempting intrusive exploitation or destructive tests.

---

## Notes & lessons learned
- Detection depends on **what** the endpoint forwards to the SIEM. Many analyst frustrations are not query-related but data-availability related. If an IP never appears in indexed fields or `_raw`, no Splunk query can find it.
- Understand the difference between **scanner types**: `-sS` (SYN, stealth) vs `-sT` (connect). Only connect scans complete handshakes and therefore are seen as established connections by endpoint tools like Sysmon.
- Keep detailed artifacts: nmap `.nmap` outputs, greppable `.gnmap`, and small redacted excerpts of `.xml` for automated parsing — but **do not** upload sensitive raw captures to public repos.

---
