# Lab 36 - Privilege Escalation (`su`, `sudo`, `/etc/sudoers`, and PolicyKit)

## Objective

Master Linux privilege escalation mechanisms, including user switching (`su`), granular delegation via `sudo` and `/etc/sudoers.d/`, passwordless command execution (`NOPASSWD:`), and desktop/D-Bus authorization policies managed by PolicyKit (`pkexec`).

By completing this lab, I should be able to:

* Differentiate between `su` (Substitute User) and `sudo` (Superuser Do) authentication models

* Safely configure granular privilege delegation using `visudo` and `/etc/sudoers.d/` drop-in files

* Implement passwordless command execution rules using the `NOPASSWD:` directive

* Locate and inspect PolicyKit (`polkit`) system action definitions under `/usr/share/polkit-1/actions/`

* Clean up custom `sudoers` drop-in files and revert system state for full lab replayability

## Part 1 - User Switching vs. Delegated Privilege (`su` vs. `sudo`)

### `su` (Substitute User / Switch User)

* **Definition:** `su` stands for **substitute user** (or **switch user**). It allows a user to spawn a new shell session under the identity of another account (defaulting to `root` if no username is specified).

* **Authentication Model:** Prompts for the **target user's password** (e.g., the `root` password).

* **Environment Handling:**

  * `su <username>`: Switches user ID while retaining most of the current user's environment variables.

  * `su - <username>` (or `su -l <username>`): Starts a full **login shell**, resetting environment variables (`$HOME`, `$PATH`, `$USER`) and changing to the target user's home directory.

### `sudo` (Superuser Do)

* **Definition:** Executes a single command (or spawns an elevated shell via `sudo -i` or `sudo -s`) with the security privileges of another user (typically `root`) based on granular policy rules.

* **Authentication Model:** Prompts for the **invoking user's own password**, eliminating the need to share the `root` password across multiple administrators and providing full audit logging in `/var/log/auth.log` or `journalctl`.

## Part 2 - `sudo` Administration & Syntax (`/etc/sudoers`)

### Core Policy Files & `visudo`

* **`/etc/sudoers`:** The primary system configuration file defining authorization rules for elevated privileges with `sudo`.

* **`/etc/sudoers.d/`:** The recommended directory for modular drop-in policy files. Placing custom rules here prevents package upgrade conflicts on the main `/etc/sudoers` file.

* **`visudo`:** The mandatory utility for editing `sudoers` files (`sudo visudo` or `sudo visudo -f /etc/sudoers.d/custom`). It acquires an exclusive file lock and performs strict syntax validation before saving, preventing syntax typos from locking out administrative access.

### Rule Structure & `NOPASSWD:` Syntax

A standard `sudoers` rule follows this structure:

<user_or_%group>  <hosts>=(<runas_user>:<runas_group>)  [TAGS]  <command_list>

* **Permitting Passwordless Execution (`NOPASSWD:`):**

  To allow a specific user or `%group` to run an exact command without prompting for a password, place the `NOPASSWD:` directive immediately before the full path of the executable:

  ```text
  username ALL=(ALL:ALL) NOPASSWD: /usr/bin/apt update
  ```

  * **Security Best Practice:** Always specify the full, absolute path to binaries (e.g., `/usr/bin/apt update`) and avoid wildcards (`*`) whenever possible to prevent unintended privilege escalation.

## Part 3 - PolicyKit (`polkit` / `pkexec`) Architecture

While `sudo` governs command-line privilege escalation, **PolicyKit (`polkit`)** provides an authorization framework allowing unprivileged processes and desktop applications to communicate with privileged system services over D-Bus.

### `pkexec`

* Acts as a PolicyKit-aware alternative to `sudo`, executing a command as another user (defaulting to `root`) after authenticating via a graphical or terminal polkit agent.

### Policy Storage Locations

* **Default System Action Definitions (`/usr/share/polkit-1/actions/`):**

  * Contains vendor-supplied XML `.policy` files (e.g., `org.freedesktop.systemd1.policy`) defining specific system actions and their default authorization requirements (`yes`, `no`, `auth_self`, `auth_admin`, `auth_admin_keep`).

  * *Rule:* Never edit files directly inside `/usr/share/polkit-1/actions/`, as package updates will overwrite them.

* **Local Administrator Overrides (`/etc/polkit-1/rules.d/`):**

  * Stores custom JavaScript `.rules` files written by administrators to override default vendor policies persistently.

## Part 4 - Hands-On Practice Lab

### Step 1: Inspect Current `sudo` Privileges

1. List the exact commands and privileges currently granted to your user account:

   ```bash
   sudo -l
   ```

### Step 2: Create a Granular `NOPASSWD:` Rule via `visudo`

1. Create a modular drop-in file inside `/etc/sudoers.d/` using `visudo` so syntax is automatically validated:

   ```bash
   sudo visudo -f /etc/sudoers.d/lab36-nopasswd
   ```

2. Add the following line (replacing `your_username` with your actual Linux username):

   ```text
   your_username ALL=(ALL:ALL) NOPASSWD: /usr/bin/apt update
   ```

   *(Save and exit: in `nano`, press `Ctrl+O`, `Enter`, `Ctrl+X`; in `vi/vim`, press `Esc`, type `:wq`, and press `Enter`).*

3. Verify the drop-in file permissions are strictly `0440`:

   ```bash
   sudo chmod 0440 /etc/sudoers.d/lab36-nopasswd
   sudo ls -l /etc/sudoers.d/lab36-nopasswd
   ```

4. Invalidate your cached `sudo` timestamp so you can verify passwordless execution genuinely works:

   ```bash
   sudo -k
   ```

5. Run the permitted command and confirm it executes immediately without prompting for a password:

   ```bash
   sudo /usr/bin/apt update
   ```

### Step 3: Explore PolicyKit Action Definitions

1. List the default XML policy files stored in `/usr/share/polkit-1/actions/`:

   ```bash
   ls -l /usr/share/polkit-1/actions/ | head -n 15
   ```

2. Inspect the structure of a policy file (such as `systemd` or `policykit` actions) to view `<action id=...>` and `<defaults>` tags:

   ```bash
   head -n 35 /usr/share/polkit-1/actions/org.freedesktop.policykit.policy
   ```

## Part 5 - Undo and Cleanup (Full Lab Replayability)

Execute these steps to remove your test `sudoers.d` rule, reset your `sudo` authentication cache, and verify the system is restored to its baseline state.

### 1. Remove the Custom `sudoers.d` Drop-In File

1. Delete the test policy file:

   ```bash
   sudo rm -f /etc/sudoers.d/lab36-nopasswd
   ```

2. Verify the file no longer exists:

   ```bash
   sudo ls -l /etc/sudoers.d/
   ```

### 2. Reset `sudo` Timestamp and Verify Normal Password Prompting

1. Invalidate any cached credentials:

   ```bash
   sudo -k
   ```

2. Verify that running `sudo -l` now requires your password again and no longer lists the `NOPASSWD: /usr/bin/apt update` entry:

   ```bash
   sudo -l
   ```

## Completion Checklist

* [x] I know that `su` stands for "substitute user" (or "switch user") and uses the target user's password

* [x] I understand that `/etc/sudoers` (and `/etc/sudoers.d/`) defines `sudo` authorization policies and must be edited via `visudo`

* [x] I can write a valid `NOPASSWD:` rule granting passwordless execution for a specific command path

* [x] I know that default PolicyKit system action definitions are stored in `/usr/share/polkit-1/actions/`

* [x] I have deleted `/etc/sudoers.d/lab36-nopasswd` and verified baseline `sudo` behavior
