# Admin VM Home Lab Setup

## Overview
This document details the setup of my **Admin VM** for a blue-team home lab, including all steps, errors encountered, and their resolutions. The Admin VM serves as the **central management and monitoring host**, running Ubuntu 22.04, responsible for configuring and managing the lab’s networks and VMs.

**Purpose:**
- Centralized management for all lab VMs
- Network monitoring and logging
- Support for defensive cybersecurity exercises, including SIEM, endpoint monitoring, and simulated attacker scenarios

---

## Virtual Machine Details

| Property            | Value                            |
|--------------------|----------------------------------|
| VM Name            | Admin-VM                         |
| OS                 | Ubuntu 22.04 LTS                 |
| Role               | Lab management / SIEM support    |
| Network Interfaces | enp0s3: NAT, enp0s8: Host-only/internal network |
| RAM                | 4–8 GB recommended               |
| CPU                | 2–4 cores recommended            |

---

## Step-by-Step Installation

### Step 1: Installing Ubuntu
- Download Ubuntu 22.04 ISO (64-bit) from the official website
- Create a new VirtualBox VM:
  - Type: Linux → Ubuntu (64-bit)
  - Memory: 4–8 GB
  - Storage: 40 GB dynamically allocated
- Mount the ISO and install Ubuntu using default settings

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
      addresses: [192.168.56.11/24]

## Detailed VM Setup
For a full step-by-step setup of the Admin VM, including errors, fixes, and configuration, see [Admin VM Setup](Admin-VM-Setup.md).
