# Lab 24 - Linux Package Management Ecosystems

## Objective

Master Linux software management across major distribution families. By completing this lab, you will learn the differences between low-level package installers (`dpkg`, `rpm`) and high-level dependency-resolving package managers (`apt`, `dnf`/`yum`, `zypper`), manage transaction histories, perform system upgrades, and understand kernel/security update lifecycles.

By completing this lab, I should be able to:

* Identify native package formats (`.deb`, `.rpm`) and low-level manual tools (`dpkg -i`, `rpm -i`)
* Recognize the dangers of manual dependency resolution ("dependency hell")
* Use high-level package managers (`APT`, `DNF`, `YUM`, `Zypper`) to query, install, and update software
* Inspect and undo DNF transactions using `dnf history` and `dnf history undo <ID>`
* Differentiate standard upgrades (`upgrade`) from distribution upgrades (`dist-upgrade`) across Ubuntu, RHEL, and openSUSE Tumbleweed
* Understand why kernel updates require system reboots

---

## Part 1 - Package Management Family Overview

Linux software installation divides into two layers: **low-level package tools** (which operate on raw single files without resolving dependencies) and **high-level package managers** (which query repositories and resolve dependency chains automatically).

### Distribution Matrix

| Distribution Family | Package Format | Low-Level Installer | High-Level Package Manager |
| :--- | :--- | :--- | :--- |
| **Debian / Ubuntu / Kali** | `.deb` | `dpkg` | `apt` / `apt-get` |
| **RHEL / Fedora / CentOS** | `.rpm` | `rpm` | `dnf` (Dandified YUM) / `yum` |
| **openSUSE / SUSE Linux** | `.rpm` | `rpm` | `zypper` |

---

## Part 2 - Low-Level Tools vs High-Level Managers

### 1. Manual Package Tools & Dependency Hell

Installing raw package files manually bypasses repository metadata:
* **Debian/Ubuntu:** `sudo dpkg -i package.deb`
* **Red Hat/RHEL:** `sudo rpm -i package.rpm`

**Dependency Hell:** Occurs when manually installing a package that requires missing shared libraries or prerequisites. Installing those missing prerequisites manually can trigger further unfulfilled dependencies, leading to a complex, manual chain of failures.

### 2. Debian / Ubuntu (`APT`)

APT (Advanced Package Tool) automates DEB package management and repository metadata tracking.

* **Refresh local package index:**
  ```bash
  sudo apt update
  ```
  *(Fetches latest package lists and versions from configured repositories; does not alter installed packages).*

* **Upgrade installed packages:**
  ```bash
  sudo apt upgrade
  ```
  *(Upgrades installed packages safely without removing existing packages).*

* **Distribution Upgrade:**
  ```bash
  sudo apt dist-upgrade
  ```
  *(Performs major dependency resolution changes, which may install new dependencies or remove obsolete packages).*

* **Routine Maintenance Sequence:**
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

---

### 3. Red Hat / Enterprise Linux (`YUM` & `DNF`)

* **YUM:** Yellow Dog Updater, Modified.
* **DNF:** Dandified YUM (the modern successor to YUM in RHEL 8+ and Fedora).

#### Major DNF Advantage: Transaction Rollback

DNF tracks every installation, removal, or update in a transaction log, allowing administrators to review or revert system states.

```bash
# View transaction history:
dnf history

# Revert a specific transaction ID:
sudo dnf history undo <ID>
```

#### `dnf update` vs `dnf upgrade`
While often aliased in modern DNF, running `dnf upgrade` can carry higher risk than simple updates because it allows the removal of obsolete packages that custom applications or local workflows might still rely on.

---

### 4. SUSE / openSUSE (`Zypper`)

Zypper is the native command-line package manager for SUSE Linux Enterprise (SLES) and openSUSE.

| Command | Function / Notes |
| :--- | :--- |
| **`zypper search <pkg>`** | Searches configured repositories for packages |
| **`sudo zypper update`** | Updates installed software packages safely |
| **`sudo zypper patch`** | Installs **only** official security and bugfix patches |
| **`sudo zypper dist-upgrade`** | Performs distribution-level upgrade (**`dup`**) |

**Rolling-Release Note:** openSUSE **Tumbleweed** is a rolling-release distribution. Unlike fixed-release OSs, Tumbleweed users routinely use `zypper dist-upgrade` (`zypper dup`) to keep the entire distribution synchronized with the latest kernel and package snapshots.

---

## Part 3 - System Maintenance & Reboot Requirements

### Security Updates
Applying security-focused package updates regularly patches known vulnerabilities (CVEs) and prevents remote exploitation.

### Why Kernel Updates Require a Reboot
Updating standard userland packages (like text editors or web servers) takes effect when the application restarts. However, a **kernel update** replaces the fundamental operating system engine at `/boot`. Because the active kernel and its compiled modules are loaded directly into system memory at boot:
1. The newly installed kernel file cannot replace the currently running kernel in RAM dynamically.
2. Kernel module dependencies must match the active running kernel release.
3. A system reboot is required to initialize the new kernel binary and load its associated modules.

---

## Part 4 - Hands-On Practice Lab

### Step 1: Practice APT Maintenance (Debian/Ubuntu/Kali)

1. Refresh local repository indexes:
   ```bash
   sudo apt update
   ```
2. Search for a lightweight package (e.g., `htop` or `tree`):
   ```bash
   apt search htop
   ```
3. Simulate an upgrade without modifying system state:
   ```bash
   apt upgrade --dry-run
   ```

### Step 2: Practice DNF History & Package Operations (RHEL/Fedora)

1. Check past package transaction history:
   ```bash
   dnf history
   ```
2. View detailed information about a specific transaction:
   ```bash
   dnf history info 1
   ```
3. (Optional) To undo a transaction (such as a accidental package install):
   ```bash
   sudo dnf history undo <TRANSACTION_ID>
   ```

### Step 3: Practice Zypper Operations (SUSE/openSUSE)

1. Search for available patches:
   ```bash
   zypper patches
   ```
2. Check for security patches specifically:
   ```bash
   sudo zypper patch --dry-run
   ```

---

## Completion Checklist

- [x] I can match package formats to tools: `.deb` (`dpkg`/`apt`) and `.rpm` (`rpm`/`dnf`/`zypper`)
- [x] I understand "dependency hell" caused by manual single-package installations (`dpkg -i`, `rpm -i`)
- [x] I know that `apt update` refreshes repository metadata while `apt upgrade` updates packages
- [x] I can inspect and revert DNF transactions using `dnf history` and `dnf history undo <ID>`
- [x] I understand why `zypper dist-upgrade` (`dup`) is standard for openSUSE Tumbleweed rolling releases
- [x] I can explain why kernel updates require a system reboot
