# Tier 1 IT Help Desk Lab

## Documentation Note

This lab environment was built and iteratively refined over time as part of ongoing hands-on learning and practice. The documentation contained in this repository was consolidated and formalized after the environment reached a stable, repeatable state.

This approach reflects real-world IT workflows, where systems are often implemented first and documented once configurations and processes are validated. The goal of this documentation is to clearly capture the final architecture, troubleshooting steps, and operational procedures for future reference and reproducibility.

---

## Overview

This repository documents the deployment of a Windows Active Directory environment built to simulate real-world **Tier 1 IT Help Desk** workflows. The lab focuses on user account support, password resets, account lockouts, domain-joined endpoint troubleshooting, and least-privilege access using Active Directory.

This environment is used to practice and record help desk simulations for LinkedIn, interviews, and hands-on skill validation.

---

## Lab Architecture

### Domain
- **Domain Name:** lab.local
- **NetBIOS Name:** LAB
- **Directory Services:** Active Directory Domain Services (AD DS)
- **DNS:** AD-integrated DNS

---

## Virtual Machines

### Windows Domain Controller
- **OS:** Windows Server 2025 Standard (Desktop Experience)
- **Hostname:** WIN-DC01
- **Roles:**
  - Active Directory Domain Services
  - DNS Server
- **IP Address:** 192.168.58.5
- **Network:** Internal Lab Network (192.168.58.0/24)
- **Configuration Notes:**
  - Static IP configuration
  - DNS configured to point to itself
  - Single network interface to avoid multi-homed AD issues

---

### Windows Endpoint
- **OS:** Windows 10 Pro
- **Purpose:** Domain-joined client for Tier 1 troubleshooting simulations
- **IP Addresses:**
  - 192.168.58.10 (Internal Lab Network)
  - 192.168.56.12 (SIEM / Management Network)
- **Domain Status:** Joined to lab.local
- **Authentication:** Domain credentials (LAB\\username)

---

### SIEM Server
- **OS:** Ubuntu Server
- **Tool:** Splunk Enterprise
- **IP Address:** 192.168.56.50
- **Purpose:** Centralized log aggregation and security visibility

---

### Kali Linux (Attacker VM)
- **Networks:**
  - NAT (Internet access)
  - 192.168.56.57 (SIEM / Management)
  - 192.168.58.20 (Internal Lab)
- **Purpose:** Attack simulation, credential misuse testing, and account lockout generation

---

### Admin VM
- **OS:** Ubuntu
- **IP Address:** 192.168.56.11
- **Purpose:** Administrative access and lab management

---

## Network Design

| Network | Subnet | Purpose |
|------|-------|--------|
| Internal Lab | 192.168.58.0/24 | Domain authentication and endpoint communication |
| SIEM / Management | 192.168.56.0/24 | Log forwarding, monitoring, and administration |
| NAT | DHCP | Internet access |

This segmented design reflects real enterprise environments where identity services, monitoring, and attack surfaces are separated.

---

## Active Directory Configuration

### Organizational Units (OUs)
- LAB-Users
- LAB-Computers
- LAB-Groups
- LAB-Admins

---

### Tier 1 Help Desk Role Setup

#### Help Desk Security Group
- **Group Name:** Helpdesk-Tier1
- **Group Type:** Global Security Group
- **Purpose:** Role-based Tier 1 permissions

#### Tier 1 User Account
- **Username:** caleb.hd
- **Display Name:** Caleb Helpdesk
- **Group Membership:** Helpdesk-Tier1

---

### Delegated Permissions
The Helpdesk-Tier1 group was delegated permissions on the LAB-Users OU to:
- Reset user passwords
- Force password change at next logon
- Unlock locked user accounts
- Read user account information

No elevated or Domain Admin permissions were assigned, following least-privilege best practices.

---

## Domain Join Process

### Endpoint Preparation
- Windows endpoint upgraded from Home to Pro (required for domain join)
- DNS configured to point to the Domain Controller (192.168.58.5)

### Domain Join
- **Domain:** lab.local
- **Credentials Used:** LAB\\Administrator
- **Verification Command:**
  ```cmd
  systeminfo | findstr /B /C:"Domain"
