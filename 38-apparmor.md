# Lab 38 - AppArmor Administration (Profile Modes, Utilities, Local Overrides, and SELinux Comparison)

## Objective

Master Linux Mandatory Access Control (MAC) using AppArmor, including profile operational modes, core inspection and mode-toggling utilities, safe local profile overrides, interactive log-driven policy refinement, service lifecycle management, and architectural comparison against SELinux.

By completing this lab, I should be able to:

* Differentiate between AppArmor's three primary operational modes (`Enforce`, `Complain`, and `Disable`)

* Verify AppArmor kernel enablement and inspect loaded profiles using `aa-enabled` and `aa-status`

* Toggle profile enforcement modes using `aa-enforce`, `aa-complain`, and `aa-disable`

* Implement update-safe custom profile overrides under `/etc/apparmor.d/local/` and refine policies interactively with `aa-logprof`

* Start and stop the AppArmor service via `systemctl` and contrast AppArmor's path-based model with SELinux's label-based model

* Safely revert profile modes, remove local override rules, and clean up test profiles for full lab replayability

## Part 1 - AppArmor vs. SELinux Architectural Comparison

Both AppArmor and SELinux provide kernel-level Mandatory Access Control (MAC), but they enforce policies using fundamentally different paradigms:

* **AppArmor (Path-Based Application Profiles):**

  * Enforces security policies based on **individual application profiles bound to executable filesystem paths** (e.g., `/usr/sbin/tcpdump` or `/usr/bin/man`).

  * Default MAC framework on Ubuntu, Debian, and SUSE distributions; generally simpler to read and author because rules mirror standard filesystem paths and capabilities.

* **SELinux (Label-Based Security Contexts):**

  * Enforces rules using **security contexts** (`user:role:type:level`) attached as extended attributes to files, processes, ports, and kernel objects.

  * Default MAC framework on RHEL, Fedora, Rocky Linux, and AlmaLinux; policies remain attached to objects regardless of path changes or hard links.

## Part 2 - Operational Modes and Service Management

### The Three Primary Profile Modes

* **Enforce:** Actively blocks actions that violate the application's profile rules and logs the blocked attempts to system/audit logs.

* **Complain (Learning Mode):** Allows policy violations to execute without blocking them, while logging every out-of-policy action—ideal for profiling new applications or troubleshooting broken workflows.

* **Disable:** Completely turns off profile enforcement and logging for the specified application by linking the profile into `/etc/apparmor.d/disable/`.

### Service and Kernel Enablement Checks

* **Check if AppArmor is Enabled in the Kernel:**

  ```
  aa-enabled
  ```

  *(Outputs `Yes` and exits `0` if AppArmor is active).*

* **Display Active Status, Loaded Profiles, and Confined Processes:**

  ```
  sudo aa-status
  ```

  *(Lists total loaded profiles, how many are in `enforce` vs. `complain` mode, and active PIDs currently confined).*

* **Start or Stop the AppArmor Service:**

  ```
  sudo systemctl start apparmor
  sudo systemctl stop apparmor
  ```

  *(Note: Stopping `apparmor` via `systemctl` unloads user-space service state, though kernel profiles already loaded may persist until unloaded with `aa-teardown` or a reboot).*

## Part 3 - Profile Administration, Overrides, and Interactive Auditing

AppArmor profile definitions reside in `/etc/apparmor.d/`, named after the executable's path with slashes replaced by dots (e.g., `/usr/sbin/tcpdump` maps to `/etc/apparmor.d/usr.sbin.tcpdump`).

### Toggling Profile Modes (`apparmor-utils`)

*(Note: On Debian/Ubuntu, install `apparmor-utils` if `aa-enforce` or `aa-logprof` are not yet present).*

* **Switch a Profile to Enforce Mode:**

  ```
  sudo aa-enforce /etc/apparmor.d/usr.sbin.tcpdump
  ```

  *(Can also reference the binary path directly, e.g., `sudo aa-enforce /usr/sbin/tcpdump`).*

* **Switch a Profile to Complain Mode:**

  ```
  sudo aa-complain /etc/apparmor.d/usr.sbin.tcpdump
  ```

* **Temporarily Disable a Specific Profile:**

  ```
  sudo aa-disable /etc/apparmor.d/usr.sbin.tcpdump
  ```

  *(Creates a symlink in `/etc/apparmor.d/disable/` and unloads the profile from the kernel).*

### Update-Safe Local Overrides (`/etc/apparmor.d/local/`)

* **Purpose:** Package upgrades overwrite main profile files inside `/etc/apparmor.d/`. To prevent updates from wiping custom permissions, main profiles include an `#include <local/...>` directive pointing to `/etc/apparmor.d/local/`.

* **Workflow:**

  1. Add custom path/capability rules (terminated with a comma `,`) to `/etc/apparmor.d/local/<profile_name>`.

  2. Reload the parent profile into the kernel:

     ```
     sudo apparmor_parser -r /etc/apparmor.d/<profile_name>
     ```

### Interactive Log Auditing (`aa-logprof`)

* **Purpose:** Scans `/var/log/audit/audit.log` or `/var/log/syslog` (`journalctl`) for AppArmor `ALLOWED` (complain mode) and `DENIED` (enforce mode) events.

* **Usage:**

  ```
  sudo aa-logprof
  ```

  *(Interactively prompts the administrator to `(A)llow`, `(D)eny`, `(I)gnore`, or `(N)ew` glob paths for each logged event and automatically writes approved rules back into the profile).*

## Part 4 - Hands-On Practice Lab

*(Note: Run this lab on an AppArmor-enabled host such as Ubuntu or Debian. If you are on a RHEL/Fedora host running SELinux, review the command syntax and inspect `/etc/apparmor.d/` if installed).*

### Step 1: Verify Service and Kernel Enablement

1. Confirm AppArmor is enabled and check overall status:

   ```
   aa-enabled
   sudo aa-status
   ```

2. Verify the `systemd` unit status:

   ```
   systemctl status apparmor --no-pager
   ```

### Step 2: Create a Safe Standalone Test Script and Profile

*(Using a dedicated test script avoids altering critical system daemons during practice).*

1. Create a simple executable script `/usr/local/bin/lab38-test.sh`:

   ```
   sudo tee /usr/local/bin/lab38-test.sh > /dev/null << 'EOF'
   #!/bin/bash
   echo "Lab 38 AppArmor test running" > /tmp/lab38_output.txt
   cat /tmp/lab38_output.txt
   EOF
   sudo chmod 755 /usr/local/bin/lab38-test.sh
   ```

2. Create a minimal profile `/etc/apparmor.d/usr.local.bin.lab38-test.sh` that initially omits `/tmp/lab38_output.txt` write permission and includes a local override file:

   ```
   sudo mkdir -p /etc/apparmor.d/local
   sudo touch /etc/apparmor.d/local/usr.local.bin.lab38-test.sh
   sudo tee /etc/apparmor.d/usr.local.bin.lab38-test.sh > /dev/null << 'EOF'
   #include <tunables/global>

   profile lab38-test /usr/local/bin/lab38-test.sh {
     #include <abstractions/base>
     #include <abstractions/bash>

     /usr/local/bin/lab38-test.sh r,
     /bin/bash ix,
     /usr/bin/bash ix,
     /bin/cat ix,
     /usr/bin/cat ix,

     #include <local/usr.local.bin.lab38-test.sh>
   }
   EOF
   ```

3. Load the profile into the kernel in **Enforce** mode and verify it appears in `aa-status`:

   ```
   sudo apparmor_parser -r /etc/apparmor.d/usr.local.bin.lab38-test.sh
   sudo aa-status | grep lab38-test
   ```

4. Run the script while enforced—observe that writing to `/tmp/lab38_output.txt` is blocked (`Permission denied`):

   ```
   /usr/local/bin/lab38-test.sh
   ```

### Step 3: Toggle Between Complain, Enforce, and Local Override

1. Switch the profile to **Complain** mode and run the script again (it now succeeds while logging an `ALLOWED` audit entry):

   ```
   sudo aa-complain /etc/apparmor.d/usr.local.bin.lab38-test.sh
   /usr/local/bin/lab38-test.sh
   ```

2. Add the required rule safely inside `/etc/apparmor.d/local/usr.local.bin.lab38-test.sh`:

   ```
   echo "/tmp/lab38_output.txt rw," | sudo tee /etc/apparmor.d/local/usr.local.bin.lab38-test.sh
   ```

3. Switch the profile back to **Enforce** mode, reload it, and verify the script now succeeds even under full enforcement:

   ```
   sudo aa-enforce /etc/apparmor.d/usr.local.bin.lab38-test.sh
   /usr/local/bin/lab38-test.sh
   ```

### Step 4: Test Disabling the Profile

1. Disable the test profile using `aa-disable` and confirm it is no longer listed as enforced:

   ```
   sudo aa-disable /etc/apparmor.d/usr.local.bin.lab38-test.sh
   sudo aa-status | grep lab38-test || echo "Profile unloaded/disabled"
   ```

## Part 5 - Undo and Cleanup (Full Lab Replayability)

Execute these steps to remove all test scripts, profiles, local override files, disable symlinks, and temporary output files so the lab can be replayed cleanly.

### 1. Unload and Remove the Test Profile and Symlinks

1. Ensure the profile is unloaded from the kernel (ignore errors if already unloaded by `aa-disable`):

   ```
   sudo apparmor_parser -R /etc/apparmor.d/usr.local.bin.lab38-test.sh 2>/dev/null || true
   ```

2. Remove the disable symlink, local override file, and main profile file:

   ```
   sudo rm -f /etc/apparmor.d/disable/usr.local.bin.lab38-test.sh
   sudo rm -f /etc/apparmor.d/local/usr.local.bin.lab38-test.sh
   sudo rm -f /etc/apparmor.d/usr.local.bin.lab38-test.sh
   ```

### 2. Remove the Test Script and Temporary Output

1. Delete `/usr/local/bin/lab38-test.sh` and `/tmp/lab38_output.txt`:

   ```
   sudo rm -f /usr/local/bin/lab38-test.sh /tmp/lab38_output.txt
   ```

### 3. Verify Baseline AppArmor State

1. Confirm the test profile is completely absent and `apparmor` service is healthy:

   ```
   sudo aa-status | grep lab38-test || echo "Clean: no lab38 profiles loaded"
   sudo systemctl status apparmor --no-pager
   ```

## Completion Checklist

* [x] I can describe AppArmor's three primary operational modes (`Enforce`, `Complain`, and `Disable`)

* [x] I can check kernel enablement with `aa-enabled` and inspect loaded profiles with `aa-status`

* [x] I can toggle profile states using `aa-enforce`, `aa-complain`, and `aa-disable`

* [x] I know that custom profile rules belong in `/etc/apparmor.d/local/` so package updates do not overwrite them

* [x] I understand how `aa-logprof` interactively updates profiles from logged audit events

* [x] I can distinguish AppArmor (path-based profiles) from SELinux (label-based security contexts) and start/stop `apparmor` via `systemctl`

* [x] I have removed all test profiles, overrides, symlinks, and scripts for full lab replayability
