# Lab 37 - SELinux Administration (Modes, Contexts, Booleans, and Relabeling)

## Objective

Master Security-Enhanced Linux (SELinux) mandatory access control (MAC) operations, including runtime and persistent mode management, security context architecture, policy-driven context restoration, boolean toggles, and filesystem relabeling.

By completing this lab, I should be able to:

* Identify the three SELinux operational modes (`Enforcing`, `Permissive`, `Disabled`) and check current status with `getenforce` and `sestatus`

* Switch SELinux runtime modes temporarily with `setenforce` and persist configuration changes in `/etc/selinux/config`

* Break down the four components of an SELinux security context (`user:role:type:level`) and explain why `type` is central in targeted policies

* Temporarily modify contexts with `chcon`, restore default policy labels with `restorecon`, and trigger full-filesystem relabeling via `/.autorelabel`

* Inspect and permanently toggle SELinux booleans across reboots using `getsebool`, `semanage boolean`, and `setsebool -P`

* Safely undo runtime mode shifts, test file context changes, boolean toggles, and relabel markers for full lab replayability

## Part 1 - SELinux Operational Modes and Configuration

SELinux enforces mandatory access control policies inside the Linux kernel based on security labels attached to processes (subjects) and files/ports/resources (objects).

### The Three Operational Modes

* **Enforcing:** Actively blocks actions that violate the loaded SELinux policy and logs denials (`AVC` messages) to the audit/system logs.

* **Permissive:** Does **not** block policy violations, allowing operations to proceed while still logging denial (`AVC`) events—ideal for troubleshooting applications and building custom policies.

* **Disabled:** Completely turns off SELinux enforcement and policy loading. Changing to or from `Disabled` requires a system reboot because kernel hooks and filesystem labeling behavior depend on boot-time state.

### Checking Current Mode and Status

* **Quick Mode Check:**

  ```
  getenforce
  
  ```

  *(Returns `Enforcing`, `Permissive`, or `Disabled`).*

* **Detailed SELinux Status:**

  ```
  sestatus
  
  ```

  *(Displays SELinux status, mount point, loaded policy name such as `targeted`, and current/configured modes).*

### Runtime vs. Persistent Mode Management

* **Temporarily Switch to Permissive Mode at Runtime:**

  ```
  sudo setenforce 0
  
  ```

  *(To switch back to Enforcing at runtime, use `sudo setenforce 1`. Runtime changes with `setenforce` do not survive a reboot).*

* **Permanently Configure Default Mode (`/etc/selinux/config`):**

  Edit `/etc/selinux/config` (or `/etc/sysconfig/selinux` symlink on older/compatible layouts) and set the `SELINUX=` directive:

  ```
  SELINUX=enforcing
  
  ```

  *(Valid values are `enforcing`, `permissive`, or `disabled`; persistent changes take effect on next boot).*

## Part 2 - Security Context Architecture and Context Management

When listing files or processes with `-Z` (such as `ls -Z` or `ps -eZ`), SELinux displays a four-part security context string:

```
user:role:type:level

```

* **Four Context Components:**

  * **User (`selinux_user`):** An SELinux identity mapped to Linux users (e.g., `unconfined_u`, `system_u`, `staff_u`), distinct from `/etc/passwd` usernames.

  * **Role (`role`):** Defines which SELinux domains/types a user identity can enter (e.g., `object_r` for files/objects, `system_r` for system daemons, `unconfined_r` for unconfined sessions).

  * **Type (`type`):** The primary attribute used by **targeted policy** configurations (the enterprise default on RHEL/Rocky/Alma/Fedora). In targeted policy, unconfined users run in broad domains while specific daemons and files are isolated by distinct **types** (such as `httpd_t` for the web server process and `httpd_sys_content_t` for web content).

  * **Level (`MCS` / `MLS` range):** Sensitivity and category labels used by Multi-Category Security (MCS) or Multi-Level Security (MLS), typically `s0` in standard targeted setups.

### Modifying and Restoring File Contexts

* **Temporarily Change a File's Context (`chcon`):**

  ```
  sudo chcon -t admin_home_t /var/www/html/lab37_test.html
  
  ```

  *(Directly alters the extended attribute label on the target file; however, `chcon` changes are non-persistent against full policy relabels or `restorecon`).*

* **Restore Default Policy Context (`restorecon`):**

  ```
  sudo restorecon -v /var/www/html/lab37_test.html
  
  ```

  *(Consults the installed SELinux file-context policy database and resets the file's context back to its expected policy default, such as `httpd_sys_content_t`. Note: Directories like `/tmp` do not have explicit file-context database mappings for arbitrary files, so `restorecon` will warn `no default label` if run on `/tmp/*` files).*

### Full-Filesystem Relabeling on Next Boot

* **Trigger Automatic Relabel at Boot:**

  ```
  sudo touch /.autorelabel
  
  ```

  *(Creating `/.autorelabel` at the root of the filesystem signals the boot process to relabel files across the system before starting regular services, then removes `/.autorelabel` upon completion).*

## Part 3 - SELinux Booleans

SELinux **booleans** are runtime/persistent policy switches (`on`/`off` or `1`/`0`) that allow administrators to adjust specific policy behaviors without writing custom SELinux policy modules.

### Listing Booleans

* **List All Booleans and Current Values:**

  ```
  getsebool -a
  
  ```

  *(Alternatively, `semanage boolean -l` lists booleans along with current vs. default state and descriptions).*

### Permanently Toggling a Boolean Across Reboots

* **Enable a Boolean Persistently (e.g., `httpd_enable_homedirs`):**

  ```
  sudo setsebool -P httpd_enable_homedirs on
  
  ```

  *(The `-P` flag writes the setting into the persistent SELinux policy store so it survives reboots; omitting `-P` only changes the active runtime value).*

## Part 4 - Hands-On Practice Lab

*(Note: If you are running on a Debian/Ubuntu host without SELinux enabled, you can still inspect the command syntax and verify whether SELinux tools are active via `sestatus` or `getenforce`).*

### Step 1: Inspect Current SELinux Mode and Contexts

1. Check the active mode and detailed status:

   ```
   getenforce
   sestatus
   
   ```

2. Inspect the security context of a file and a running process:

   ```
   ls -Z /etc/passwd
   ps -eZ | head -n 10
   
   ```

### Step 2: Test Runtime Mode Switching (`setenforce`)

1. Switch to `Permissive` mode at runtime and verify:

   ```
   sudo setenforce 0
   getenforce
   
   ```

2. Return to `Enforcing` mode at runtime and verify:

   ```
   sudo setenforce 1
   getenforce
   
   ```

### Step 3: Practice Context Changes and Restoration (`chcon` / `restorecon`)

1. Verify the policy-defined default context for `/var/www/html/lab37_test.html`:

   ```
   matchpathcon /var/www/html/lab37_test.html
   
   ```

2. Create the test file under `/var/www/html/` and check its initial context:

   ```
   sudo mkdir -p /var/www/html
   sudo touch /var/www/html/lab37_test.html
   ls -Z /var/www/html/lab37_test.html
   
   ```

3. Temporarily change its type to `admin_home_t` using `chcon`, verify the change, then restore its policy default (`httpd_sys_content_t`) with `restorecon`:

   ```
   sudo chcon -t admin_home_t /var/www/html/lab37_test.html
   ls -Z /var/www/html/lab37_test.html
   sudo restorecon -v /var/www/html/lab37_test.html
   ls -Z /var/www/html/lab37_test.html
   
   ```

### Step 4: Inspect and Toggle an SELinux Boolean

1. Check the current state of `httpd_enable_homedirs`:

   ```
   getsebool httpd_enable_homedirs
   
   ```

2. Enable it persistently across reboots using `-P`:

   ```
   sudo setsebool -P httpd_enable_homedirs on
   getsebool httpd_enable_homedirs
   
   ```

## Part 5 - Undo and Cleanup (Full Lab Replayability)

Execute these steps to revert test files, restore boolean defaults, ensure `Enforcing` mode is active, and remove any relabel markers before replaying the lab.

### 1. Remove Temporary Test Files and Any Relabel Marker

1. Delete the test file created during the lab (and clean up `/var/www/html` only if it was empty and created solely for this lab):

   ```
   sudo rm -f /var/www/html/lab37_test.html
   sudo rmdir --ignore-fail-on-non-empty /var/www/html /var/www 2>/dev/null || true
   
   ```

2. If you created `/.autorelabel` during testing and do **not** want a full reboot relabel, remove it:

   ```
   sudo rm -f /.autorelabel
   
   ```

### 2. Revert Boolean Changes Back to Default

1. Turn `httpd_enable_homedirs` back off persistently:

   ```
   sudo setsebool -P httpd_enable_homedirs off
   getsebool httpd_enable_homedirs
   
   ```

### 3. Restore Enforcing Mode and Verify `/etc/selinux/config`

1. Ensure the active runtime mode is set to `Enforcing`:

   ```
   sudo setenforce 1
   getenforce
   
   ```

2. Inspect `/etc/selinux/config` to confirm the persistent `SELINUX=` setting matches your intended baseline (such as `SELINUX=enforcing`):

   ```
   grep -E '^SELINUX=' /etc/selinux/config
   
   ```

## Completion Checklist

* [x] I can name the three SELinux modes (`Enforcing`, `Permissive`, `Disabled`) and check them with `getenforce` and `sestatus`

* [x] I can temporarily switch modes at runtime with `setenforce` and know `/etc/selinux/config` controls persistent boot behavior

* [x] I can identify the four context fields (`user:role:type:level`) and explain why `type` drives targeted policies

* [x] I can modify contexts with `chcon`, restore policy defaults with `restorecon`, and trigger a boot relabel via `/.autorelabel`

* [x] I can list booleans with `getsebool -a` (or `semanage boolean -l`) and persist changes across reboots with `setsebool -P`

* [x] I have cleaned up test files, relabel markers, boolean overrides, and runtime mode changes
