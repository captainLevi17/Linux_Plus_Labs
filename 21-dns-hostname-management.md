# Lab 21 - Name Resolution, DNS Tools, and Hostname Management

## Objective

Master Linux name resolution mechanics, system configuration files, DNS record types, diagnostic utilities, and local DNS caching services.

By completing this lab, I should be able to:

* Distinguish between system resolution order (`/etc/nsswitch.conf`) and resolver settings (`/etc/resolv.conf`)
* Perform local hostname mappings via `/etc/hosts` and update hostnames permanently with `hostnamectl`
* Use standalone DNS diagnostic tools (`dig`, `nslookup`, `host`, `whois`) to query specific DNS record types (`A`, `AAAA`, `CNAME`, `MX`, `PTR`)
* Inspect local DNS resolver statistics and clear DNS caches using `resolvectl`

---

## Part 1 - System Name Resolution Order

Linux applications using standard `glibc` routines perform name resolution based on system configuration files.

### Configuration Files Breakdown

| Configuration File | Function / Role | Key Entry Examples |
| :--- | :--- | :--- |
| **`/etc/nsswitch.conf`** | Controls the **order and sources** used for name resolution | `hosts: files dns myhostname` |
| **`/etc/hosts`** | Stores local **static IP-to-hostname** mappings | `127.0.0.1 localhost`<br>`192.168.1.50 server01.local` |
| **`/etc/resolv.conf`** | Defines upstream **nameservers** and search domains | `nameserver 127.0.0.53` (Ubuntu stub)<br>`nameserver 8.8.8.8` |

### Key Distinction: `nsswitch.conf` vs `resolv.conf`

* **`/etc/nsswitch.conf`** dictates *where* the system looks first (e.g., check local `/etc/hosts` file first before querying external DNS).
* **`/etc/resolv.conf`** defines *which* DNS servers to contact when the system falls back to DNS resolution.

---

## Part 2 - Common DNS Record Types

| Record Type | Description & Purpose |
| :--- | :--- |
| **`A`** | Maps a hostname to an **IPv4** address (e.g., `example.com` $\rightarrow$ `93.184.216.34`) |
| **`AAAA`** | Maps a hostname to an **IPv6** address |
| **`CNAME`** | **Canonical Name:** Aliases one hostname to another canonical hostname |
| **`MX`** | **Mail Exchanger:** Points to the mail servers responsible for accepting email on behalf of a domain |
| **`PTR`** | **Pointer Record:** Used for **reverse DNS lookups** (maps an IP address back to a hostname) |

---

## Part 3 - DNS Lookup Utilities

Direct DNS query tools (`dig`, `nslookup`, `host`) **bypass `/etc/nsswitch.conf` and `/etc/hosts`**, communicating directly with DNS servers over port 53.

### Diagnostic Tools Comparison

| Tool | Common Usage Examples | Notes / Key Options |
| :--- | :--- | :--- |
| **`dig`** | `dig example.com`<br>`dig -x 8.8.8.8` | Versatile DNS tool. Use **`-x`** for reverse DNS (PTR) lookups. |
| **`nslookup`** | `nslookup example.com`<br>`nslookup -type=MX example.com` | Interactive or non-interactive query tool; automatically performs reverse lookups when given an IP. |
| **`host`** | `host example.com` | Concise tool for simple, quick DNS queries. |
| **`whois`** | `whois example.com` | Queries public domain registrar databases (ownership, expiration, contact info). |

---

## Part 4 - Systemd Resolved & Hostname Control

Modern Linux distributions (like Ubuntu) use `systemd-resolved` to handle local DNS caching.

### `resolvectl` Management

* **Display resolver status and DNS servers:**
  ```bash
  resolvectl status
  ```
* **Flush local DNS cache:**
  ```bash
  sudo resolvectl flush-caches
  ```

### Hostname Control (`hostnamectl`)

* **Display system hostname and OS metadata:**
  ```bash
  hostnamectl
  ```
* **Permanently change system hostname:**
  ```bash
  sudo hostnamectl hostname new-server-name
  ```

---

## Part 5 - Hands-On Practice Lab

### Step 1: Query DNS Records

1. Perform a standard `A` record query with `dig`:
   ```bash
   dig example.com
   ```
2. Perform a reverse DNS (`PTR`) lookup on an IP address using `dig`:
   ```bash
   dig -x 8.8.8.8
   ```
3. Query Mail Exchanger (`MX`) records for a domain using `nslookup`:
   ```bash
   nslookup -type=MX google.com
   ```
4. Query domain registration details using `whois`:
   ```bash
   whois example.com
   ```

### Step 2: Test Local Hostname Resolution

1. Inspect the local lookup order in `/etc/nsswitch.conf`:
   ```bash
   grep hosts /etc/nsswitch.conf
   ```
2. Add a custom static mapping to `/etc/hosts`:
   ```bash
   echo "127.0.0.2 testserver.local" | sudo tee -a /etc/hosts
   ```
3. Test resolution via system resolver (`ping` or `getent` obeys `nsswitch.conf`):
   ```bash
   getent hosts testserver.local
   ```
4. Test resolution via direct DNS query (`dig` bypasses `/etc/hosts` and will fail to resolve `testserver.local`):
   ```bash
   dig testserver.local
   ```

### Step 3: Manage Local DNS Caching and Hostname

1. View active `systemd-resolved` DNS interfaces:
   ```bash
   resolvectl status
   ```
2. Flush the local DNS cache:
   ```bash
   sudo resolvectl flush-caches
   ```
3. Check system hostname details:
   ```bash
   hostnamectl
   ```

---

## Completion Checklist

- [x] I understand that `/etc/nsswitch.conf` sets resolution order while `/etc/resolv.conf` defines DNS servers
- [x] I know that `dig`, `nslookup`, and `host` query DNS directly, bypassing `/etc/hosts`
- [x] I can match DNS record types to their function: `A` (IPv4), `AAAA` (IPv6), `CNAME` (Alias), `MX` (Mail), and `PTR` (Reverse IP lookup)
- [x] I can run reverse DNS lookups using `dig -x <IP>`
- [x] I can query specific record types using `nslookup -type=<TYPE>`
- [x] I can flush local DNS caches using `resolvectl flush-caches`
- [x] I can set persistent hostnames using `hostnamectl hostname <name>`
