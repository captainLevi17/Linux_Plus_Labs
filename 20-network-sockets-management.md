# Lab 20 - Network Configuration, Socket Auditing, and Management Utilities

## Objective

Master essential Linux networking utilities, socket state auditing tools, persistent network configuration backends, and hostname controls.

By completing this lab, I should be able to:

* Manage and inspect runtime network interfaces, routes, and ARP neighbors using modern `iproute2` commands (`ip addr`, `ip route`, `ip neighbor`)
* Contrast legacy `net-tools` (`ifconfig`, `route`, `arp`, `netstat`) with modern replacements (`ip`, `ss`)
* Inspect active TCP/UDP network sockets using `ss` and combine flags (`-t`, `-u`, `-n`, `-a`, `-l`)
* Manage network connections persistently via `nmcli` (NetworkManager) and understand distro-specific persistent configs (`/etc/netplan/` on Ubuntu, `ifcfg-*` on RHEL/CentOS)
* Display system hostnames and domain attributes using `hostname` flags and configuration files

---

## Part 1 - Modern (`iproute2`) vs Legacy (`net-tools`) Equivalents

Runtime commands modified via `ip` commands take effect **immediately** but are **non-persistent** across system reboots.

| Task | Modern Command (`iproute2` / `ss`) | Legacy Command (`net-tools`) |
| :--- | :--- | :--- |
| **Display IP / Interfaces** | `ip address` / `ip a` | `ifconfig` |
| **Display Routing Table** | `ip route` | `route` |
| **Display ARP Cache** | `ip neighbor` / `ip neigh` | `arp` |
| **Inspect Network Sockets** | `ss` | `netstat` |

### Address Resolution Protocol (ARP)
* **Function:** Resolves local network IP addresses (Layer 3) to physical MAC addresses (Layer 2).

---

## Part 2 - Network Socket Auditing (`ss`)

The `ss` command is the modern, faster replacement for `netstat` used to dump socket statistics.

### Essential Flags Combination Reference

* **`-t`** : Filter **TCP** sockets
* **`-u`** : Filter **UDP** sockets
* **`-n`** : Show **numeric** port and IP addresses (prevents slow DNS reverse-resolution)
* **`-a`** : Show **all** sockets (both listening and non-listening/established connections)
* **`-l`** : Show **listening** sockets only
* **`-p`** : Display process names using the sockets (requires `sudo`)

### Common Command Combinations

```bash
# View all listening TCP and UDP sockets with numeric ports:
ss -tunl

# View all active (including connected) TCP sockets numerically:
ss -tna
```

---

## Part 3 - Persistent Network Configuration & NetworkManager

To make networking changes survive reboots, configurations must be written to disk rather than set via temporary `ip` commands.

### NetworkManager & `nmcli`
NetworkManager is the default network management daemon across desktop and enterprise distributions.

* **Check global networking state:** `nmcli networking`
* **Enable global networking:** `nmcli networking on`
* **Disable global networking:** `nmcli networking off`
* **List network connections:** `nmcli connection show`

### Distro-Specific Persistent Files

* **Ubuntu Server:** Configured via **Netplan** YAML files located in `/etc/netplan/*.yaml`. Applied using `sudo netplan apply`.
* **RHEL / CentOS 8 (Legacy syntax):** Configured via interface scripts located at `/etc/sysconfig/network-scripts/ifcfg-<interface_name>`.
* **Persistent Hostname:** Stored in `/etc/hostname`.

### Hostname Inspection

* **`hostname -f`** : Displays the Fully Qualified Domain Name (FQDN).
* **`hostname -d`** : Displays the DNS domain name component.

---

## Part 4 - Hands-On Practice Lab

### Step 1: Inspect Network Interfaces and Routing Tables

1. View all active network interfaces and IP addresses:
   ```bash
   ip a
   ```
2. Display the system routing table:
   ```bash
   ip route
   ```
3. Inspect known local ARP neighbors:
   ```bash
   ip neighbor
   ```

### Step 2: Audit Active Network Sockets with `ss`

1. List all open listening TCP/UDP ports with numeric formatting:
   ```bash
   ss -tunl
   ```
2. Identify process names associated with open network ports:
   ```bash
   sudo ss -tunlp
   ```

### Step 3: Manage NetworkManager via `nmcli`

1. Check current network state:
   ```bash
   nmcli networking
   ```
2. Inspect network interfaces (devices) status:
   ```bash
   nmcli device status
   ```

---

## Completion Checklist

- [x] I can map modern `iproute2` commands to legacy `net-tools` equivalents (`ip a` vs `ifconfig`, `ip route` vs `route`, `ip neighbor` vs `arp`, `ss` vs `netstat`)
- [x] I know that ARP resolves Layer 3 IP addresses to Layer 2 MAC addresses
- [x] I can build custom `ss` socket queries using `-t`, `-u`, `-n`, `-a`, and `-l` flags
- [x] I understand that commands issued directly via `ip` are temporary and non-persistent
- [x] I know where persistent network configurations live on Ubuntu (`/etc/netplan/`) and RHEL (`ifcfg-*`)
- [x] I can control global networking state using `nmcli networking on/off`
