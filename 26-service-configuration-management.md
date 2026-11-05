# Lab 26 - Service Reloading, Configuration Files, and Repository Management

## Objective

Master Linux service configuration management, service control safety, package repository configuration across APT, YUM, and DNF, and understand modular `.d` directory design alongside RPM configuration file backups (`.rpmnew` and `.rpmsave`).

By completing this lab, I should be able to:

* Safely update and reapply service configurations using `reload` instead of `restart`
* Contrast `systemctl reload` with legacy `service` control options
* Manage APT repository sources (`/etc/apt/sources.list` and `/etc/apt/sources.list.d/`)
* Manage YUM/DNF repository sources (`/etc/yum.repos.d/` and `/etc/dnf/`)
* Understand modular `.d` configuration directory conventions across Linux systems
* Handle `.rpmnew` and `.rpmsave` files created during RPM package upgrades

## Part 1 - Service Management: Restart vs Reload

When modifying service configuration files (such as Apache, Nginx, or SSH), choose the correct service action to avoid unnecessary service downtime.

### Action Comparison

* **Restart (`systemctl restart <service>`)**
  * **Mechanism:** Completely stops the process and launches a fresh instance.
  * **Risk:** If the new configuration file contains syntax errors, the service will fail to start upon stopping, leading to total service unavailability.
  * **Use Case:** Required when changing major environment variables or binary options.

* **Reload (`systemctl reload <service>`)**
  * **Mechanism:** Keeps the main process running in memory while signaling it to read and apply the updated configuration files.
  * **Benefit:** Zero downtime. If the configuration contains errors, the running process rejects the change and stays online.
  * **Command:** `sudo systemctl reload <service>`

### Legacy SysVinit Control

* On legacy or SysV-compatible systems, the `service` wrapper command is used:
  * Syntax: `service <service> <action>` (e.g., `service httpd reload`)

## Part 2 - Modular Configuration Directories (`.d` Pattern)

Linux widely adopts modular `.d` directories (such as `/etc/httpd/conf.d/`, `/etc/apt/sources.list.d/`, `/etc/yum.repos.d/`).

### Purpose & Benefits of `.d` Directories

* **Avoid Monolithic Editing:** Eliminates the need to directly edit large central configuration files (like `/etc/apt/sources.list`).
* **Package Management:** Third-party applications can drop their own `.conf` or `.repo` file into a `.d` directory upon installation and clean it up cleanly upon uninstallation without corrupting host settings.
* **Easy Removal:** Disabling or removing a configuration or repository source simply requires deleting its specific file inside the `.d` folder.

## Part 3 - Package Repository Configurations

Package managers maintain configuration files exclusively under the `/etc` directory. Neither YUM nor DNF rely on running daemons; they read configuration files fresh each time a command is executed.

### APT Repository Structure (Debian/Ubuntu)

* **Main Config Directory:** `/etc/apt/`
* **Primary Repository File:** `/etc/apt/sources.list`
* **Modular Repository Directory:** `/etc/apt/sources.list.d/`
* **Removing Repositories:** Delete the target `.list` file from `/etc/apt/sources.list.d/` and run `sudo apt update`.

### YUM & DNF Repository Structure (RHEL/CentOS/Fedora)

* **DNF Configuration Directory:** `/etc/dnf/` (Contains global settings like `dnf.conf`).
* **Shared Repository Directory:** `/etc/yum.repos.d/`
* **Interoperability:** Both YUM and DNF read `.repo` files from `/etc/yum.repos.d/`.

## Part 4 - RPM Config Updates: `.rpmnew` vs `.rpmsave`

During system upgrades on RPM-based distributions (RHEL, Fedora, openSUSE), RPM handles modified configuration files carefully to avoid overwriting admin settings.

* **`.rpmnew` File**
  * **Trigger:** Created when an administrator has modified an existing configuration file, and a package update includes a newer version of that configuration file.
  * **Outcome:** RPM leaves the user's modified file active and saves the vendor's updated defaults as `<configfile>.rpmnew`.

* **`.rpmsave` File**
  * **Trigger:** Created during an upgrade or removal when RPM needs to replace or delete an existing configuration file that was edited locally.
  * **Outcome:** RPM replaces or removes the file but preserves a copy of the old modified configuration as `<configfile>.rpmsave`.

## Part 5 - Hands-On Practice Lab

### Step 1: Practice Safe Reloading (Apache/Nginx example)

1. Verify the location of Web server configuration files on Red Hat:
   * Location: `/etc/httpd/` and `/etc/httpd/conf.d/`

2. Safely reload a service after making changes:
   ```bash
   sudo systemctl reload httpd
   ```

3. Test configuration file syntax *before* restarting or reloading (best practice):
   ```bash
   # For Apache (RHEL/Fedora):
   httpd -t

   # For Nginx:
   nginx -t
   ```

### Step 2: Inspect Repository Structures

1. List active APT repository configuration files on Debian/Ubuntu:
   ```bash
   ls -l /etc/apt/sources.list.d/
   ```

2. List active RPM repository configuration files on RHEL/Fedora:
   ```bash
   ls -l /etc/yum.repos.d/
   ```

3. Find any remaining `.rpmnew` or `.rpmsave` files on an RPM system:
   ```bash
   find /etc -name "*.rpmnew" -o -name "*.rpmsave"
   ```

---

## Completion Checklist

* [x] I understand why `systemctl reload` is safer than `systemctl restart` for live services
* [x] I can locate Apache configuration files on RHEL (`/etc/httpd/` and `/etc/httpd/conf.d/`)
* [x] I know why `.d` configuration directories are used across Linux systems
* [x] I can differentiate `.rpmnew` (vendor update file) from `.rpmsave` (local backup file)
* [x] I know where APT (`/etc/apt/sources.list.d/`) and YUM/DNF (`/etc/yum.repos.d/`) store repository configurations
* [x] I know that YUM and DNF do not use persistent background daemons and re-read configs on every launch
