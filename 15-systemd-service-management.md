# Lab 15 - Systemd Service Management and Control

## Objective

Master the fundamentals of Linux service management using `systemctl`. By completing this lab, you will learn how to inspect, start, stop, restart, enable, disable, and mask systemd services, while understanding the underlying mechanics of service states and symlinks.

By completing this lab, I should be able to:

* Query and interpret service states using `systemctl status`
* Control runtime service states (start/stop/restart) versus boot configurations (enable/disable)
* Use service masking to permanently prevent accidental or automatic service execution
* Locate and understand the symlink mechanics behind `systemctl mask`

---

## Part 1 - Service Management Quick Reference

### Core Service Control Commands

| 	Action 	| 	Command 	 | 	 Scope / Impact  	|
| **Check Status** | `systemctl status <service>` | View current runtime state and boot setting |
| **Start Immediately** | `systemctl start <service>` | **Current session only** (does not persist across reboots) |
| **Stop Immediately** | `systemctl stop <service>` | **Current session only** (does not persist across reboots) |
| **Restart Service** | `systemctl restart <service>` | Restarts service immediately in current session |
| **Enable at Boot** | `systemctl enable <service>` | **Boot time only** (does not start service immediately) |
| **Disable at Boot** | `systemctl disable <service>` | **Boot time only** (does not stop service immediately) |
| **Mask Service** | `systemctl mask <service>` | Prevents **both** manual and automatic starting |
| **Unmask Service** | `systemctl unmask <service>` | Re-enables service capability to start |

---

## Part 2 - Key Systemd Concepts & States

### Runtime vs. Boot Persistence
* **Runtime Control (`start`/`stop`):** Modifies the service state in memory for the current session only.
* **Boot Control (`enable`/`disable`):** Modifies symlinks under `/etc/systemd/system/` to determine if a service launches during startup.
* **Combine Enable & Start:** To enable and start a service in a single command, run:
  ```bash
  sudo systemctl enable --now <service>
  ```

### Common `systemctl status` States

| State String | Meaning |
| :--- | :--- |
| **`active (running)`** | Service is running normally |
| **`inactive (dead)`** | Service is currently stopped |
| **`masked`** | Service is locked and cannot be started by any user, process, or boot target |

### Mechanics of Service Masking
Masking a service prevents dependencies or administrators from starting it. Systemd achieves this by pointing the service unit file to `/dev/null`:

$$\text{Symlink Path: } /etc/systemd/system/<service>.service \longrightarrow /dev/null$$

---

## Part 3 - Hands-On Practice Lab

Follow these steps using a dummy or test service (such as `apache2`, `nginx`, or `cron`/`cronie`) to practice service lifecycle management.

### Step 1: Inspect Service Status
Check the current state of a service (e.g., `cron` or `ssh`):
```bash
systemctl status cron
```
*Observe whether the output shows `active (running)` or `inactive (dead)`, and whether it is `enabled` or `disabled`.*

### Step 2: Manage Runtime States
1. **Stop the service:**
   ```bash
   sudo systemctl stop cron
   systemctl status cron
   ```
2. **Start the service:**
   ```bash
   sudo systemctl start cron
   systemctl status cron
   ```

### Step 3: Practice Service Masking
1. **Mask the service:**
   ```bash
   sudo systemctl mask cron
   ```
2. **Verify the `/dev/null` symlink:**
   ```bash
   ls -l /etc/systemd/system/cron.service
   ```
3. **Attempt to start the masked service:**
   ```bash
   sudo systemctl start cron
   ```
   *(Notice the error stating that the service is masked).*
4. **Unmask the service:**
   ```bash
   sudo systemctl unmask cron
   sudo systemctl start cron
   ```

---

## Completion Checklist

* [x] I understand the difference between runtime control (`start`/`stop`) and boot control (`enable`/`disable`)
* [x] I can check service status and identify `active (running)` vs `inactive (dead)` states
* [x] I can mask a service and verify that `/etc/systemd/system/<service>.service` points to `/dev/null`
* [x] I can unmask a service and return it to normal operation
