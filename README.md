# Home Cybersecurity Lab Documentation

This repository documents the setup and configuration of my home IT support lab, built to simulate Tier 1 Help Desk workflows (Active Directory account support, password resets, lockouts, domain joins, endpoint troubleshooting) while also supporting blue team/SOC visibility through centralized logging and monitoring. Each VM and network is documented step by step, including installation, configuration, testing, and troubleshooting notes.

---

## Table of Contents
1. [Lab Overview](#lab-overview)
2. [Network Setup](#network-setup)
3. [Virtual Machines](#virtual-machines)
   - [Admin VM](#admin-vm)
   - [Attacker VM](#attacker-vm)
   - [Endpoint VMs](#endpoint-vms)
   - [SIEM / Management VM](#siem--management-vm)
4. [Testing & Verification](#testing--verification)
5. [Notes / Lessons Learned](#notes--lessons-learned)

---

## Lab Overview
- Purpose: Practice defensive SOC operations (network monitoring, endpoint detection, incident response) in a safe, isolated environment.
- Tools used: VirtualBox, Kali Linux, Windows 10/11, Ubuntu Server, Splunk, Wireshark, etc.
- Host OS: Windows 10
- Virtualization: VirtualBox 7.x

---

## Network Setup
| Network Name         | Type         | Purpose                                     | IP Range          |
|---------------------|-------------|-----------------------------------------------|-----------------|
| NAT                 | NAT         | Internet access for VMs                       | Automatic       |
| Lab Management Net  | Host-only   | SIEM / Management access / Log Forwarding     | 192.168.56.0/24 |
| Lab-Internal        | Internal    | Isolated lab network for attacker & endpoints | 192.168.58.0/24 |

**Notes / Steps Taken:**
- Created host-only networks in VirtualBox.
- Created internal-lab network in VirtualBox.
- Verified connectivity between Admin VM and each network.
- IP addressing plan: `Admin VM: 192.168.56.10`, `Kali: 192.168.58.0`, etc.

---

## Virtual Machines

### Admin VM
- **OS:** Ubuntu Server/Desktop
- **RAM / CPU / Disk:** 4GB RAM / 2 CPU / 50GB Disk
- **Network:** NAT + Host-only Lab Net
- **Purpose:** Administration, monitoring, and SIEM tools
- **Key Setup Steps:**
  1. Installed Ubuntu and updates: `sudo apt update && sudo apt upgrade -y`
  2. Installed admin tools: Wireshark, tcpdump, etc.
  3. Configured monitoring groups/users
- **Verification / Tests:**
  - Can ping all other VMs on host-only networks
  - Can access internet via NAT
- **Snapshot:** `base/admin-clean`
For full step-by-step setup of the Admin VM, see [Admin VM Setup](admin/README.md)
---

### Attacker VM
- **OS:** Kali Linux
- **RAM / CPU / Disk:** 4GB RAM / 2 CPU / 30GB Disk
- **Network:** NAT + Internal-only + Host-only Lab Net
- **Purpose:** Offensive testing and traffic generation
- **Key Setup Steps:**
  1. Installed Kali Linux (full or light ISO)
  2. Installed common tools: nmap, metasploit, etc.
  3. Updated system: `sudo apt update && sudo apt full-upgrade -y`
- **Verification / Tests:**
  - Ping Admin VM
  - Perform network scan on host-only subnet
- **Snapshot:** `base/kali-clean`
Setup, configuration, troubleshooting, and quick reference for the Kali attacker VM.  
  [View Attacker VM docs](attacker/README.md)

---

### Endpoint VMs
- **OS Option:** Windows 10.
- **Purpose:** Target system for attacks / monitoring
- **Network:** Host-only + Internal-only Lab Net
- **Setup Notes:**
  - Install OS
  - Configure IPs (static manual)
  - Harden basic security (firewall, updates)
- **Verification / Tests:**
  - Ensure Admin VM can reach endpoints
  - Endpoints can reach each other
 
[Windows Endpoint README](windows-endpoint/README.md)


---

### SIEM / Management VM
- **OS:** Ubuntu Server
- **Tools:** Splunk, SplunkUniversalForwarder, etc.
- **Network:** Host-only + NAT Lab Net
- **Setup Notes:**
  - Install SIEM
  - Connect Admin VM to SIEM dashboard
  - Test logs from endpoints and attacker VM
- **Verification / Tests:**
  - Logs collected and visualized
  - Alerting works on sample events
- For full setup instructions, troubleshooting steps, and Windows endpoint integration, see the [Splunk SIEM Guide](splunk-siem/README.md).
- For detailed dashboard and alert setup instructions, see [Splunk Dashboard & Alerts Setup](splunk-siem/splunk-dashboard-setup/README.md)
---

## Testing & Verification
- Ping between all VMs to ensure connectivity
- Test NAT internet access
- Run sample scans/traffic to confirm SIEM captures events

---

## Notes / Lessons Learned
- Document any errors, fixes, or unique configurations
- Include screenshots of network diagrams, VM settings, or terminal outputs
- Keep snapshots before major changes for rollback

---

