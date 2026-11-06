# Lab 27 - Kernel Module Management and Runtime Kernel Parameters

## Objective

Master the tools and techniques required to manage Linux kernel modules dynamically and configure persistent kernel runtime parameters.

By completing this lab, I should be able to:

* Inspect currently loaded kernel modules using `lsmod` and query module details using `modinfo`

* Insert and remove kernel modules safely using `modprobe` versus manual low-level tools (`insmod`, `rmmod`)

* Monitor kernel initialization and module loading events via `dmesg`

* Inspect and modify runtime kernel parameters using `sysctl` and direct interaction with `/proc/sys/`

* Persist kernel settings across system reboots using `/etc/sysctl.conf` and `/etc/sysctl.d/`

---

## Part 1 - Kernel Module Management (`lsmod`, `modprobe`, `insmod`, `rmmod`)

Kernel modules are pieces of code loaded into the kernel on demand to support hardware, filesystems, or network protocols without requiring a system reboot.

### Module Utilities & Commands

* **`lsmod`**
  * **Function:** Lists all active, inserted kernel modules. Reads directly from `/proc/modules`.

* **`modinfo <module_name>`**
  * **Function:** Displays detailed metadata about a module (author, description, license, module dependencies, and configurable parameters).

* **`modprobe <module_name>`**
  * **Function:** Inserts a module into the kernel while **automatically resolving and loading any dependencies**.
  * **Note:** Requires only the module name, not the absolute file path.

* **`modprobe -r <module_name>`**
  * **Function:** Unloads a module along with any of its **unused dependencies**.

* **`insmod <path_to_file.ko>`**
  * **Function:** Inserts a raw kernel module file into the kernel.
  * **Limitation:** Does **not** resolve dependencies automatically and requires the exact full file path.

* **`rmmod <module_name>`**
  * **Function:** Removes a loaded module from the kernel.
  * **Limitation:** Only removes the specified module, ignoring orphaned dependencies.

---

## Part 2 - Kernel Ring Buffer (`dmesg`)

* **Function:** Displays kernel log messages stored in the kernel ring buffer.
* **Module Troubleshooting:** Useful for verifying whether hardware was detected or whether a module loaded/unloaded successfully.

### Useful Command Formats

* **View recent kernel messages:**
  ```
  dmesg | tail -n 20
  ```

* **Filter messages for module or hardware events:**
  ```
  dmesg | grep -i <module_name>
  ```

---

## Part 3 - Runtime Kernel Parameters (`sysctl` & `/proc/sys/`)

The Linux kernel exposes runtime parameters through a virtual filesystem, allowing administrators to fine-tune networking, security, and memory behavior without recompiling the kernel.

### The `/proc/sys/` Directory Architecture

The files exposed inside `/proc/sys/` map directly to `sysctl` dot-notation keys:

* **Path on Disk:** `/proc/sys/net/ipv4/ip_forward`
* **Corresponding `sysctl` Key:** `net.ipv4.ip_forward`

Writing `1` to `/proc/sys/net/ipv4/ip_forward` is functionally identical to executing:
```
sudo sysctl -w net.ipv4.ip_forward=1
```

### Persistence and Configuration Files

Changes made directly with `sysctl -w` or by writing to `/proc/sys/` are **temporary** and reset on reboot.

* **Primary Configuration File:** `/etc/sysctl.conf`
* **Modular Configuration Directory:** `/etc/sysctl.d/` (e.g., `/etc/sysctl.d/99-custom.conf`)
* **Reload File Settings (Without Rebooting):**
  ```
  sudo sysctl -p
  ```
  *(Loads and applies settings from `/etc/sysctl.conf` or a specified file).*

---

## Part 4 - Hands-On Practice Lab

### Step 1: Inspect Loaded Modules

1. List all active kernel modules and filter for network or storage drivers:
   ```
   lsmod | head -n 15
   ```

2. View information about a common module (e.g., `e1000` or `ext4`):
   ```
   modinfo ext4
   ```

### Step 2: Load and Unload Dummy Modules

1. Practice inserting a harmless testing module using `modprobe`:
   ```
   sudo modprobe dummy
   ```

2. Confirm the module is active:
   ```
   lsmod | grep dummy
   ```

3. Inspect kernel ring buffer logs for the insertion event:
   ```
   dmesg | grep -i dummy | tail -n 5
   ```

4. Safely unload the module and its dependencies using `modprobe -r`:
   ```
   sudo modprobe -r dummy
   ```

### Step 3: View and Modify Kernel Runtime Parameters

1. View the current value of IP forwarding using `sysctl`:
   ```
   sysctl net.ipv4.ip_forward
   ```

2. Temporarily enable IP forwarding:
   ```
   sudo sysctl -w net.ipv4.ip_forward=1
   ```

3. Make a parameter persistent across reboots:
   * Open or create a custom file in `/etc/sysctl.d/`:
     ```
     echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/99-ipforward.conf
     ```
   * Apply settings from the file immediately:
     ```
     sudo sysctl -p /etc/sysctl.d/99-ipforward.conf
     ```

---

## Completion Checklist

* [x] I can view active modules using `lsmod` and query metadata using `modinfo`
* [x] I understand why `modprobe` is preferred over `insmod` (handles dependencies automatically)
* [x] I know that `modprobe -r` removes a module and its unused dependencies, whereas `rmmod` removes only the specified module
* [x] I can check kernel ring buffer messages using `dmesg`
* [x] I understand how `/proc/sys/` paths map to `sysctl` parameter names
* [x] I can persist kernel settings in `/etc/sysctl.conf` or `/etc/sysctl.d/` and reload them using `sysctl -p`
