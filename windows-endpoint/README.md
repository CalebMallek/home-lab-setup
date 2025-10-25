[windows_endpoint_readme.md](https://github.com/user-attachments/files/23136762/windows_endpoint_readme.md)
# Windows 10 Endpoint VM Setup

This document details the full setup of the **Windows 10 Endpoint VM** for the home lab, including network configuration, security tooling, and troubleshooting steps. This VM acts as a monitored endpoint to simulate a corporate workstation for SOC exercises.

---

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Network Configuration](#network-configuration)
4. [Security Tools](#security-tools)
    - Sysmon
    - Winlogbeat
5. [Shared Folder Setup](#shared-folder-setup)
6. [Troubleshooting & Tweaks](#troubleshooting--tweaks)
7. [Next Steps](#next-steps)

---

## Overview

- **Role:** Endpoint for monitoring attacks and log collection.
- **VM Type:** Windows 10 Pro (64-bit).
- **Networks:**
  - `lab-internal` → isolates attacker traffic.
  - `management` → connects to Admin VM and future Splunk SIEM.

---

## Installation

1. Created a new VM in VirtualBox:
   - Name: `Windows-Endpoint`
   - Type: `Windows 10 (64-bit)`
   - Memory: 4 GB
   - Hard disk: 40 GB VDI, dynamically allocated

2. Installed Windows 10 via ISO:
   - Chose **personal use** account setup.
   - Username: `Caleb` (consistent with attacker VM for lab simplicity).
   - Password: unique from host account.
   - Privacy settings adjusted for lab: minimal telemetry, no browser data sharing.

3. Completed Windows updates post-installation.

---

## Network Configuration

- Adapter 1: `lab-internal` → manually set static IP for endpoint.
- Adapter 2: `management` → static IP for SIEM log forwarding.

**Notes from setup:**

- Initial IPs showed `169.x.x.x` → ran `ipconfig /renew` to obtain correct `192.168.56.x` addresses.
- Confirmed connectivity by pinging Admin VM on management network and Kali VM on lab-internal network.
- Switched network adapters as needed when IP addresses did not match intended network (initially misassigned management IP).

---

## Security Tools

### Sysmon

- Installed **Sysmon (Sysinternals)** for advanced event logging.
- Steps:
  1. Copied Sysmon package from host via VirtualBox **shared folder**.
  2. Extracted to `C:\Tools\Sysinternals`.
  3. Installed via PowerShell:
     ```powershell
     cd C:\Tools\Sysinternals
     .\sysmon64.exe -i -accepteula
     ```
  4. Verified installation:
     ```powershell
     Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
     ```

**Troubleshooting Notes:**
- Initially couldn’t copy files via shared folder — resolved by installing **VBox Guest Additions** via ISO.
- Execution policy blocked script running → temporarily bypassed with:
  ```powershell
  Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
  ```
  After Sysmon install, verified local machine policy returned to `Restricted`.

### Winlogbeat

- Installed **Winlogbeat** to forward logs to SIEM (Splunk planned).
- Steps:
  1. Extracted Winlogbeat ZIP to `C:\Tools\Winlogbeat`.
  2. Ran PowerShell script to install service:
     ```powershell
     Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
     cd "C:\Tools\Winlogbeat\winlogbeat-9.2.0-windows-x86_64"
     .\install-service-winlogbeat.ps1
     Start-Service winlogbeat
     Get-Service winlogbeat
     ```
  3. Verified service status: **Running**

**Troubleshooting Notes:**
- Script execution blocked initially → resolved by temporary execution policy bypass.
- Executable paths changed in Winlogbeat 9.x → had to ensure PowerShell navigated to the correct subfolder.

---

## Shared Folder Setup

- Created a shared folder between host and VM for transferring tools:
  - Folder on host: `vm-isos/tools`
  - Mounted in VM under `C:\Tools`
- Enabled **auto-mount** via VirtualBox settings.
- Installed **VBox Guest Additions** to enable full shared folder functionality.
- Disconnected shared folder after tool transfer for security hygiene.

---

## Troubleshooting & Tweaks

- **IP issues:** Initial 169.x.x.x addresses → ran `ipconfig /renew`.
- **Ping issues:** Confirmed network adapter and IP alignment.
- **Sysmon service name mismatch:** PowerShell `Get-Service` returned nothing — used `Get-Service | Where-Object {$_.DisplayName -like "*Sysmon*"}`.
- **Execution policy:** Temporarily bypassed for script execution; verified default restored after installation.
- **Shared folder mount:** Required VBox Guest Additions; removed ISO after install to prevent mount errors.

---

## Next Steps

1. Create **Splunk SIEM VM** (Ubuntu Server).
2. Configure **Winlogbeat output** in `winlogbeat.yml` to forward logs to Splunk.
3. Optionally deploy **Suricata IDS VM** on lab-internal network to monitor Kali attacks.
4. Begin **attack-detect-analyze cycles** for SOC training.

