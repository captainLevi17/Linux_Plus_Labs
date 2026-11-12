# Lab 31 - Linux System Hardening, Defense-in-Depth, and Boot Integrity

## Objective

Master foundational Linux system hardening techniques, defense-in-depth host security practices, default file permission masking (`umask`), password complexity enforcement via PAM, centralized log tamper-resistance, and UEFI Secure Boot verification.

By completing this lab, I should be able to:

* Explain why forwarding logs to a centralized log server protects forensic integrity during host compromises

* Calculate default file and directory permissions using `umask` values (e.g., `0027` -> `640` / `rw-r-----`)

* Identify PAM configuration files responsible for enforcing password complexity (`common-password`, `system-auth`)

* Understand the defense-in-depth necessity of host-based firewalls (`ufw`, `firewalld`, `nftables`) behind perimeter firewalls

* Explain how UEFI Secure Boot prevents boot-level malware and rootkits by verifying cryptographic signatures

## Part 1 - Boot Integrity & Hardware Trust (`Secure Boot`)

Securing an operating system begins before the kernel even loads into memory.

### UEFI Secure Boot Mechanics

* **Primary Function:** Verifies the cryptographic digital signatures of the bootloader (e.g., GRUB2) and the Linux kernel against trusted keys stored in the system's UEFI firmware before execution.

* **Security Benefit:** Prevents unauthorized, unsigned, or malicious code (such as bootkits and kernel-level rootkits) from executing during the startup sequence.

* **Checking Status on Linux:**

  ```
  mokutil --sb-state
  
  ```

## Part 2 - Default Permission Masking (`umask`)

The `umask` (user file-creation mode mask) defines which permission bits are automatically **stripped/removed** from newly created files and directories.

### Permission Calculation Rules

* **Base File Permissions:** `666` (`rw-rw-rw-`) — Linux never grants execute (`x`) permissions to new files by default.

* **Base Directory Permissions:** `777` (`rwxrwxrwx`).

* **Calculation Formula:** Subtract (or bitwise-mask) the `umask` value from the base permission.

### Example: `umask 0027`

* **Mask Breakdown (`0027`):**

  * `0` (Special bits): No change

  * `0` (Owner): Remove `0` bits (keep all)

  * `2` (Group): Remove `2` (write permission)

  * `7` (Others): Remove `7` (read, write, and execute permissions)

* **Resulting New File Permissions (`666 - 027`):**

  * **Octal:** `640`

  * **Symbolic:** `rw-r-----` (Owner: read/write, Group: read-only, Others: no access)

* **Resulting New Directory Permissions (`777 - 027`):**

  * **Octal:** `750`

  * **Symbolic:** `rwxr-x---` (Owner: full access, Group: read/enter, Others: no access)

## Part 3 - PAM Password Policies & Centralized Logging

### 1. Password Complexity Enforcement via PAM

Linux enforces password quality rules (minimum length, character classes, dictionary checks via `pam_pwquality` or `pam_cracklib`) through PAM modules located in `/etc/pam.d/`:

* **Debian / Ubuntu:** `/etc/pam.d/common-password`

* **RHEL / CentOS / Fedora:** `/etc/pam.d/system-auth` and `/etc/pam.d/password-auth`

### 2. Centralized Log Forwarding

* **Security Rationale:** Forwarding system logs (`rsyslog` or `systemd-journal-remote`) in real time to a dedicated, hardened central log server preserves forensic audit trails.

* **Threat Mitigation:** If an attacker compromises a local server and gains `root` access, they cannot alter, truncate, or wipe historical evidence of their intrusion if the logs have already been transmitted off-host.

## Part 4 - Defense-in-Depth: Host-Based Firewalls

Relying solely on an external perimeter or enterprise network firewall leaves internal infrastructure vulnerable.

### Why Host Firewalls (`ufw`, `firewalld`, `nftables`) Are Mandatory

* **Defense-in-Depth:** Creates multiple layers of security controls rather than a single point of failure.

* **Lateral Movement Prevention:** If an attacker compromises one internal workstation or server inside the network perimeter, host-based firewalls block unauthorized east-west traffic between internal hosts.

* **Zero-Trust Alignment:** Ensures each server explicitly restricts incoming ports to only the exact services required for its role.

## Part 5 - Hands-On Practice Lab

### Step 1: Test `umask` Behavior

1. Check your current shell `umask` value:

   ```
   umask
   
   ```

2. Temporarily set a restrictive `umask` of `0027` in your current shell:

   ```
   umask 0027
   
   ```

3. Create a test file and directory, then verify their default permissions:

   ```
   touch /tmp/umask_test_file
   mkdir /tmp/umask_test_dir
   ls -ld /tmp/umask_test_file /tmp/umask_test_dir
   
   ```

   *(Confirm the file shows `-rw-r-----` (`640`) and the directory shows `drwxr-x---` (`750`)).*

4. Clean up test artifacts:

   ```
   rm -rf /tmp/umask_test_file /tmp/umask_test_dir
   
   ```

### Step 2: Inspect PAM Password Policy Configurations

1. Inspect password policy rules on your distribution:

   ```
   # For Debian / Ubuntu:
   sudo cat /etc/pam.d/common-password
   
   # For RHEL / Fedora:
   sudo cat /etc/pam.d/system-auth
   
   ```

### Step 3: Check Secure Boot and Host Firewall Status

1. Check UEFI Secure Boot status (if supported by system firmware/VM):

   ```
   mokutil --sb-state
   
   ```

2. Audit active host-based firewall status:

   ```
   # Check UFW (Ubuntu/Debian):
   sudo ufw status verbose
   
   # Check firewalld (RHEL/Fedora):
   sudo firewall-cmd --state
   
   ```

## Completion Checklist

* [x] I understand how UEFI Secure Boot verifies bootloader and kernel signatures against trusted firmware keys

* [x] I can calculate resulting file (`640` / `rw-r-----`) and directory (`750` / `rwxr-x---`) permissions from `umask 0027`

* [x] I know which PAM files enforce password complexity (`/etc/pam.d/common-password` or `/etc/pam.d/system-auth`)

* [x] I can explain why centralized log forwarding protects forensic evidence from local tampering

* [x] I understand why host-based firewalls (`ufw`, `firewalld`, `nftables`) are essential for defense-in-depth and preventing lateral movement
