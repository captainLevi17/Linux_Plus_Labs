# Lab 25 - Universal and Sandboxed Application Formats

## Objective

Master universal Linux application packaging and containerized desktop formats. By completing this lab, you will learn how to manage desktop applications across distributions using Flatpak, Snap, and AppImage, understand dependency isolation and sandboxing, and evaluate trade-offs regarding performance and permissions.

By completing this lab, I should be able to:

* Manage Flatpak remotes, search for applications, and install packages system-wide or per-user (`--user`)
* Manage Snap packages on Ubuntu and other distributions using `snap` CLI commands
* Download, make executable (`chmod +x`), and execute standalone AppImage files without root privileges
* Compare the architecture, repository dependencies, and sandboxing models of Flatpak, Snap, and AppImage
* Recognize performance considerations (e.g., startup overhead) and hardware access constraints of sandboxed apps

---

## Part 1 - Universal Format Architectural Comparison

Traditional package managers (`apt`, `dnf`, `zypper`) share system-wide libraries and dependencies. Universal application formats bundle or sandbox dependencies so software can run reliably across different distributions regardless of host system library versions.

### Format Features Matrix

Format Comparison Summary

####Flatpak

Primary Ecosystem: Cross-distro desktop applications (GNOME / Red Hat backed).

Architecture: Sandboxed applications running on shared runtimes.

Repository System: Uses Remotes (e.g., Flathub).

Installation Scope: System-wide by default, or --user for current user only.

####Snap

Primary Ecosystem: Ubuntu-associated universal packages.

Architecture: Self-contained containerized packages bundled with dependencies.

Repository System: Canonical Snap Store.

Installation Scope: Installed system-wide by default.

####AppImage

Primary Ecosystem: Standalone portable executable format.

Architecture: Compressed filesystem image containing the application and all dependencies.

Repository System: None (downloaded directly from developers as single files).

Installation Scope: No traditional installation or root privileges required.
---

## Part 2 - Detailed Command Breakdown

### 1. Flatpak

Flatpak isolates desktop applications from the host OS using container technologies (`bubblewrap`).

* **Remotes:** Repository sources for Flatpak apps (e.g., Flathub). Installing a `.flatpakref` file automatically configures the required remote and application metadata.
* **Search for applications:**
  ```bash
  flatpak search <term>
  ```
* **Install System-wide:**
  ```bash
  flatpak install <remote> <application-id>
  ```
* **Install for Current User Only (No root required):**
  ```bash
  flatpak install --user <remote> <application-id>
  ```
* **Run Application from CLI:**
  ```bash
  flatpak run <application-id>
  ```

### 2. Snap

Maintained primarily by Canonical (Ubuntu), Snap packages bundle application binaries, libraries, and runtime dependencies.

* **List installed snaps:**
  ```bash
  snap list
  ```
* **Search for a package:**
  ```bash
  snap search <term>
  ```
* **Install a package system-wide:**
  ```bash
  sudo snap install <package>
  ```

### 3. AppImage

AppImages are standalone, self-contained applications contained entirely within a single executable file.

* **Key Advantage:** Extremely simple. Requires no daemon, background service, root access, or package manager installation.
* **Making Executable:**
  ```bash
  chmod +x application.AppImage
  ```
* **Running:**
  ```bash
  ./application.AppImage
  ```
* **Self-Updating:** AppImage apps can check for updates directly when launched.

---

## Part 3 - Sandboxing Trade-offs & Performance Considerations

* **Dependency Isolation Benefit:** Eliminates "dependency hell" and prevents library version conflicts between applications.
* **Startup Overhead:** Sandboxed apps (Snap/Flatpak) can launch more slowly on initial run because runtime environments and compressed mount points must be initialized separately.
* **Hardware & Integration Limits:** Strict sandboxing can occasionally prevent applications from seamlessly accessing host hardware (e.g., webcams, GPU acceleration, external drives) or custom system themes.

---

## Part 4 - Hands-On Practice Lab

### Step 1: Practice Flatpak Management

1. Search for a lightweight Flatpak application:
   ```bash
   flatpak search text-editor
   ```
2. View configured remotes on the system:
   ```bash
   flatpak remotes
   ```

### Step 2: Practice Snap Management (Ubuntu/Kali)

1. List all currently installed Snap packages:
   ```bash
   snap list
   ```
2. Search the Snap Store for a utility:
   ```bash
   snap search micro
   ```

### Step 3: Practice AppImage Deployment

1. Create a dummy file mimicking an AppImage binary:
   ```bash
   echo '#!/bin/bash' > /tmp/sample_app.AppImage
   echo 'echo "AppImage executed successfully!"' >> /tmp/sample_app.AppImage
   ```
2. Make the file executable without needing root:
   ```bash
   chmod +x /tmp/sample_app.AppImage
   ```
3. Run the standalone application:
   ```bash
   /tmp/sample_app.AppImage
   ```

---

## Completion Checklist

- [x] I understand what a Flatpak remote is and how `.flatpakref` files configure repositories
- [x]] I can install Flatpak applications system-wide or per-user using `--user`
- [x] I can list (`snap list`), search (`snap search`), and install (`snap install`) Snap packages
- [x] I know how to make an AppImage executable (`chmod +x`) and run it without root privileges
- [x] I can explain why AppImage differs from Snap/Flatpak (single binary vs repository/daemon model)
- [x] I understand the trade-offs of sandboxed applications (dependency isolation vs startup overhead)
