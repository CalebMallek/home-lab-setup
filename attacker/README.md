# Kali Attacker VM — SOC Lab README

## Overview
This document records the full setup, troubleshooting, and verification steps used to create the **Kali attacker VM** for my small SOC home lab.

---

## Lab Network Summary
- **Management (host-only)**: `192.168.56.0/24` — existing adapter at `192.168.56.57` (admin VM / Splunk)
- **Lab-internal (host-only)**: `192.168.58.0/24` — attacker + endpoints — existing adapter at `192.168.58.20`
- **NAT**: VirtualBox NAT for internet access (automatic 10.0.2.x addressing)

---

## Kali VM Details
- **Name:** kali-attacker  
- **Hostname:** kali-attacker  
- **User:** caleb (sudo-enabled)  
- **Disk:** 20 GB (dynamically allocated)  
- **RAM:** 2048–4096 MB recommended  
- **Network adapters:**  
  - Adapter 1: NAT (internet)  
  - Adapter 2: Host-only (management `192.168.56.0/24`)
  - Adapter 3: Host-only (lab-internal `192.168.58.0/24`)

---

## Installation Steps (brief)
1. Create internal adapter for lab-internal (VirtualBox → File → Host Network Manager)  
   - IPv4: `192.168.58.20` `/24`  
2. Create VM: Name `kali-attacker`, Linux → Debian (64-bit), 2048 MB RAM, 20 GB disk.
3. Attach Kali ISO to optical drive and install (Graphical install).
4. During install: Hostname `kali-attacker`, domain blank or `lab.local`, create user `caleb`.
5. After install: take snapshot `attacker-clean-slate`.

---

## Post-install commands
Open a terminal (`Ctrl+Alt+T`) and run:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y openssh-server net-tools iputils-ping wget vim
# attacker tools
sudo apt install -y nmap metasploit-framework hydra john sqlmap
sudo systemctl enable --now ssh
```

---

## Troubleshooting (errors encountered and fixes)
1. **`sub-process /usr/bin/dpkg returned an error code (1)`**  
   Fix:
   ```bash
   sudo apt --fix-broken install
   sudo dpkg --configure -a
   sudo apt clean
   sudo apt autoclean
   sudo apt full-upgrade -y
   ```

2. **`dhclient: command not found`**  
   Fix:
   ```bash
   sudo apt update
   sudo apt install -y isc-dhcp-client
   ```

3. **`dhclient` appears to “hang” (runs in foreground)**  
   - Run in background or verbose:
   ```bash
   sudo dhclient -v eth1 &
   ```
   - Or open a new terminal (`Ctrl+Alt+T`).

4. **Host-only interface UP but no IPv4 (only inet6 or loopback)**  
   - DHCP on host-only adapter likely disabled or not working. Quick fix: static IP.
   ```bash
   sudo ip addr add 192.168.58.101/24 dev eth1
   sudo ip link set eth1 up
   ```
   - Alternative: enable/configure DHCP in VirtualBox Host Network Manager.

5. **Lost NAT IP on eth0 after setting static on eth1**  
   Fix:
   ```bash
   sudo ip link set eth0 up
   sudo dhclient eth0
   sudo ip route add default via 10.0.2.2 dev eth0
   ```

6. **Stuck in multiline shell prompt (dangling quote/backslash)**  
   - Cancel with `Ctrl + C` to return to a normal prompt.

---

## Verification Checklist (commands & expected results)
1. Show interfaces and IPs:
```bash
ip addr show
# Expect:
# - eth0 (NAT): inet 10.0.2.15/24
# - eth1 (host-only): inet 192.168.56.101/24
# - eth2 (internal): inet 192.168.58.101/24
```

2. Verify NAT gateway:
```bash
ping -c 3 10.0.2.2
```

3. Verify host-only gateway:
```bash
ping -c 3 192.168.57.1
```

4. Test internet & DNS:
```bash
ping -c 3 8.8.8.8
ping -c 3 google.com
```

5. Check SSH:
```bash
sudo systemctl status ssh --no-pager
# Expect active (running)
```

6. Check attacker tools:
```bash
dpkg -l | grep -E "metasploit|hydra|john|sqlmap"
# Optional version checks:
msfconsole --version
hydra -h
john --version
sqlmap --version
```

---

## Final State & Notes
- **eth0 (NAT)**: Internet access available.  
- **eth1 (host-only)**: Static IP `192.168.56.101` assigned for log forwarding traffic.
- **eth2 (internal)**: Static IP `192.168.58.101` assigned for internal attacks on windows endpoint.  
- SSH enabled and attacker tools installed (as requested).  
- Snapshot `attacker-clean-slate` saved after setup — use it to revert before noisy attacks.

---

## Next steps 
- Create Windows endpoint VM with two host-only adapters:
  - Adapter for lab-internal: `192.168.58.x`
  - Adapter for management: `192.168.56.x`
- Install Splunk Universal Forwarder or Winlogbeat on Windows endpoint.
- Create Splunk VM on management network (`192.168.56.0/24`) and configure receiving port.
- Configure SplunkUniversalForwarder to forward data to Splunk.
