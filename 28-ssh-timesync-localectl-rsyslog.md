# Lab 28 - SSH Hardening, System Logging, Localization, and Time Synchronization

## Objective

Master essential system administration tasks for server security configuration, log lifecycle management, localization parameters, and system clock synchronization across `systemd` and enterprise Linux platforms.

By completing this lab, I should be able to:

* Distinguish between SSH client (`ssh_config`) and server daemon (`sshd_config`) files and enforce key-only root logins (`PermitRootLogin prohibit-password`)

* Locate system logging rules (`/etc/rsyslog.conf`, `/etc/rsyslog.d/`) and understand log retention via `logrotate`

* View, audit, and configure system language, keyboard layouts, and locales using `localectl`

* Manage system clocks, time zones, and NTP synchronization using `timedatectl` and `systemd-timesyncd`

* Monitor and administer enterprise NTP synchronization using `chrony` and `chronyc`

---

## Part 1 - OpenSSH Configuration Management

OpenSSH separates settings for outgoing client connections from incoming server daemon sessions.

### Key SSH Files & Parameters

* **Server Daemon Config:** `/etc/ssh/sshd_config`
  * Defines listening ports, allowed authentication methods, key policies, and access controls for incoming SSH connections.

* **Client Config:** `/etc/ssh/ssh_config`
  * Sets system-wide defaults for outgoing connections initiated from the local system.

* **Root Access Security (`PermitRootLogin prohibit-password`)**
  * Setting `PermitRootLogin` to `prohibit-password` (or legacy `without-password`) permits `root` logins **only** via public key authentication.
  * Completely disables password-based authentication for the `root` account over SSH.

---

## Part 2 - System Logging & Log Management

Linux logs events, system facility messages, and security notifications using `rsyslog` and manages log file disk consumption using `logrotate`.

### Configuration Files & Utilities

* **Primary `rsyslog` Configuration:** `/etc/rsyslog.conf`
  * Main ruleset defining how syslog messages (facilities like `auth`, `cron`, `daemon`) are routed to log files in `/var/log/`.

* **Modular Logging Directory:** `/etc/rsyslog.d/` (e.g., `50-default.conf` on Debian/Ubuntu systems).

* **Log Lifecycle Management (`logrotate`)**
  * Utility that automatically rotates, compresses, truncates, and mails log files.
  * Prevents `/var/log/` partition exhaustion while preserving historical audit logs.

---

## Part 3 - System Localization (`localectl`)

Systemd provides `localectl` to manage system language, character encoding (`LANG`), and keyboard layouts.

### Essential `localectl` Commands

* **View current locale configuration (`LANG`, `LC_*` settings):**
  ```bash
  localectl status
  ```

* **List all installed/available system locales:**
  ```bash
  localectl list-locales
  ```

* **Set system locale:**
  ```bash
  sudo localectl set-locale LANG=en_US.UTF-8
  ```

---

## Part 4 - Time Synchronization (`timedatectl`, `systemd-timesyncd`, `chrony`)

Accurate system time is critical for security logging, kerberos authentication, network protocols, and TLS certificate validation.

### 1. Control Utility: `timedatectl`

* **View current clock, timezone, and NTP status:**
  ```bash
  timedatectl status
  ```

* **Enable or disable network time synchronization:**
  ```bash
  sudo timedatectl set-ntp true
  ```
  *(Accepts `true`/`false` or `1`/`0`).*

### 2. Lightweight Client: `systemd-timesyncd`

* Included natively with `systemd` as a lightweight, client-only SNTP daemon for basic desktop/server clock syncing.

* **Check `systemd-timesyncd` status, active server, and clock offset:**
  ```bash
  timedatectl timesync-status
  ```

### 3. Enterprise Time Synchronization: `chrony`

* Modern NTP client/server implementation (daemon name: `chronyd`) that replaces legacy `ntpd`.
* Designed for fast clock synchronization, rapid response to frequency drift, and robust behavior on systems with intermittent network connectivity.

* **Administration Utility:** `chronyc`

* **Key Commands:**
  * View active time sources: `chronyc sources`
  * Check tracking statistics and offset: `chronyc tracking`

---

## Part 5 - Hands-On Practice Lab

### Step 1: Audit SSH Server Configuration

1. Inspect key security settings in `/etc/ssh/sshd_config`:
   ```bash
   grep -E "^(PermitRootLogin|PasswordAuthentication|Port)" /etc/ssh/sshd_config
   ```

2. Test configuration file syntax without restarting the daemon:
   ```bash
   sudo sshd -t
   ```

### Step 2: Check System Locale and Time Settings

1. Query current system locale parameters:
   ```bash
   localectl status
   ```

2. Search for available US UTF-8 locales:
   ```bash
   localectl list-locales | grep -i "en_US"
   ```

3. Display complete clock, timezone, and network time synchronization details:
   ```bash
   timedatectl status
   ```

### Step 3: Inspect Network Time Synchronization (`chrony` or `systemd-timesyncd`)

1. Check detailed status of `systemd-timesyncd` (if active):
   ```bash
   timedatectl timesync-status
   ```

2. Query `chrony` sources and tracking state (if installed):
   ```bash
   chronyc tracking
   chronyc sources -v
   ```

---

## Completion Checklist

- [x] I can differentiate between `/etc/ssh/sshd_config` (daemon) and `/etc/ssh/ssh_config` (client)
- [x] I know that `PermitRootLogin prohibit-password` restricts root SSH logins exclusively to SSH keys
- [x] I know where `rsyslog` configurations reside (`/etc/rsyslog.conf`, `/etc/rsyslog.d/`) and what `logrotate` does
- [x] I can check and list locales using `localectl status` and `localectl list-locales`
- [x] I can manage system time sync using `timedatectl set-ntp true` and inspect `systemd-timesyncd` via `timedatectl timesync-status`
- [x] I can monitor `chrony` (`chronyd`) using `chronyc tracking` and `chronyc sources`
