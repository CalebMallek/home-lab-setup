# Home Cybersecurity Lab Documentation

This repository documents the setup and configuration of my home cybersecurity lab, designed for **blue team SOC skills** (defensive monitoring, analysis, and testing). Each VM and network is described step by step, including installation, configuration, and testing notes.

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
- Host OS: `YOUR_HOST_OS_HERE`
- Virtualization: VirtualBox 7.x

---

## Network Setup
| Network Name         | Type         | Purpose                                     | IP Range          |
|---------------------|-------------|--------------------------------------------|-----------------|
| NAT                 | NAT         | Internet access for VMs                     | Automatic       |
| Host-only Lab Net    | Host-only   | Isolated lab network for attacker & endpoints | 192.168.56.0/24 |
| Lab Management Net  | Host-only   | SIEM / Management access                    | 192.168.57.0/24 |

**Notes / Steps Taken:**
- Created host-only networks in VirtualBox.
- Verified connectivity between Admin VM and each network.
- IP addressing plan: `Admin VM: 192.168.56.10`, `Kali: 192.168.56.11`, etc.

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
- **Network:** NAT + Host-only Lab Net
- **Purpose:** Offensive testing and traffic generation
- **Key Setup Steps:**
  1. Installed Kali Linux (full or light ISO)
  2. Installed common tools: nmap, metasploit, etc.
  3. Updated system: `sudo apt update && sudo apt full-upgrade -y`
- **Verification / Tests:**
  - Ping Admin VM
  - Perform network scan on host-only subnet
- **Snapshot:** `base/kali-clean`

---

### Endpoint VMs
- **OS Options:** Windows 10/11, Ubuntu Desktop, etc.
- **Purpose:** Target systems for attacks / monitoring
- **Network:** Host-only Lab Net
- **Setup Notes:**
  - Install OS
  - Configure IPs (manual or DHCP from Admin)
  - Harden basic security (firewall, updates)
- **Verification / Tests:**
  - Ensure Admin VM can reach endpoints
  - Endpoints can reach each other (if intended)

---

### SIEM / Management VM
- **OS:** Ubuntu Server / Windows Server
- **Tools:** Splunk, ELK, Security Onion, etc.
- **Network:** Host-only Management Net + NAT
- **Setup Notes:**
  - Install SIEM
  - Connect Admin VM to SIEM dashboard
  - Test logs from endpoints and attacker VM
- **Verification / Tests:**
  - Logs collected and visualized
  - Alerting works on sample events

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

