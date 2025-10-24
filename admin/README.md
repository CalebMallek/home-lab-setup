# Admin VM Home Lab Setup

## Overview

This document details the setup of my **Admin VM** for a blue-team home lab, including all steps, errors encountered, and their resolutions. The Admin VM serves as the **central management and monitoring host**, running Ubuntu 22.04, responsible for configuring and managing the lab’s networks and VMs.

**Purpose:**

* Centralized management for all lab VMs
* Network monitoring and logging
* Support for defensive cybersecurity exercises, including SIEM, endpoint monitoring, and simulated attacker scenarios

---

## Virtual Machine Details

| Property           | Value                                           |
| ------------------ | ----------------------------------------------- |
| VM Name            | Admin-VM                                        |
| OS                 | Ubuntu 22.04 LTS                                |
| Role               | Lab management / SIEM support                   |
| Network Interfaces | enp0s3: NAT, enp0s8: Host-only/internal network |
| RAM                | 4–8 GB recommended                              |
| CPU                | 2–4 cores recommended                           |

---

## Step-by-Step Installation

### Step 1: Installing Ubuntu

* Download Ubuntu 22.04 ISO (64-bit) from the official website.
* Create a new VirtualBox VM:

  * Type: Linux → Ubuntu (64-bit)
  * Memory: 4–8 GB
  * Storage: 40 GB dynamically allocated
* Mount the ISO and install Ubuntu using default settings.

**Errors & Fixes:**

* **Ubuntu freezing during installation:** Ensure VirtualBox Guest Additions are not enabled yet and enable VT-x/AMD-V in BIOS. Increase RAM to at least 4GB.

---

### Step 2: Network Configuration

**Goal:** Have two network interfaces:

1. NAT (for internet access)
2. Host-only/internal (for lab VM communication)

**Netplan config applied:**

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: yes
    enp0s8:
      dhcp4: no
      addresses: [192.168.56.10/24]
```

**Errors & Fixes:**

* **Netplan not applying IP correctly:** Ensure YAML spacing is correct (2 spaces per indentation level). Run `sudo netplan apply` after saving.
* **Network unreachable after applying config:** Restart VM or run `sudo systemctl restart systemd-networkd`.

---

### Step 3: Admin Tools Installation

* Update Ubuntu:

```bash
sudo apt update && sudo apt upgrade -y
```

* Install monitoring/admin tools:

```bash
sudo apt install wireshark tcpdump net-tools -y
```

* Add your user to the Wireshark group:

```bash
sudo usermod -aG wireshark $USER
```

**Errors & Fixes:**

* **Wireshark group does not exist error:** Install wireshark first, then create the group if needed using `sudo groupadd wireshark`.
* **apt freeze during updates:** Switch mirrors in Software & Updates to main Ubuntu servers.

---

### Step 4: Verification & Snapshots

* Ping all other VMs on host-only network.
* Test NAT internet connectivity.
* Take a snapshot in VirtualBox: `base/admin-clean`.

**Errors & Fixes:**

* **Ping fails to other VMs:** Check Host-only network adapter is enabled and IP addresses match subnet.
* **Cannot reach internet via NAT:** Verify NAT adapter is attached and VM firewall allows traffic.

---

### Notes / Lessons Learned

* Always check YAML indentation for netplan.
* Take snapshots before major changes.
* Document any errors encountered and resolutions.
* Keep screenshots of network settings and terminal outputs.
