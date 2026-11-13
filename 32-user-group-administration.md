# Lab 32 - Local User, Group, and Session Administration

## Objective

Master local Linux user and group account lifecycle management, default provisioning configurations, group membership modifications, and active session monitoring.

By completing this lab, I should be able to:

* View and permanently modify default user creation parameters using `useradd -D`
* Provision user accounts with home directories (`-m`), set passwords (`passwd`), and cleanly purge accounts (`userdel -rf`)
* Create, modify, and remove local groups using `groupadd`, `groupmod`, and `groupdel`
* Safely manage primary (`-g`) and supplementary (`-aG`) group memberships using `usermod`
* Audit user/group identity attributes (`id`) and monitor active user sessions (`who` and `w`)

## Part 1 - User Account Provisioning & Removal

Managing local user accounts requires understanding system defaults and clean lifecycle operations.

### Default Configuration (`useradd -D`)

* **View Default Settings:**
  ```
  useradd -D
  ```
  *(Displays default base home directory, inactivity period, expiration date, default primary group, default shell, and skeleton directory `/etc/skel`).*

* **Permanently Change Default Login Shell:**
  ```
  sudo useradd -D -s /bin/bash
  ```
  *(Updates `/etc/default/useradd` so future accounts automatically inherit `/bin/bash`).*

### Creating, Securing, and Deleting Users

* **Create User with Home Directory (`-m` / `--create-home`):**
  ```
  sudo useradd -m <username>
  ```
  *(Without `-m`, `useradd` may omit creating `/home/<username>` on many distributions).*

* **Assign or Update Password:**
  ```
  sudo passwd <username>
  ```

* **Forcefully Remove User, Home Directory, and Mail Spool (`-r -f`):**
  ```
  sudo userdel -rf <username>
  ```
  * `-r` (`--remove`): Removes the user's home directory and mail spool.
  * `-f` (`--force`): Forces removal even if the user is currently logged in or files are owned by another UID.

## Part 2 - Group Management & Membership Control

Every Linux user has exactly one **primary group** (recorded in `/etc/passwd`) and zero or more **secondary (supplementary) groups** (recorded in `/etc/group`).

### Core Group Commands

* **Create a New Group:**
  ```
  sudo groupadd <groupname>
  ```

* **Modify Group Attributes (Rename or Change GID):**
  ```
  sudo groupmod -n <new_name> <old_name>
  sudo groupmod -g <new_gid> <groupname>
  ```

* **Delete an Existing Group:**
  ```
  sudo groupdel <groupname>
  ```
  *(Note: You cannot delete a group if it is currently assigned as any active user's primary group).*

### Modifying User Group Memberships (`usermod`)

* **Change Primary Group (`-g`):**
  ```
  sudo usermod -g <primary_group> <username>
  ```

* **Append Secondary/Supplementary Group (`-aG`):**
  ```
  sudo usermod -aG <secondary_group> <username>
  ```
  * **CRITICAL WARNING:** Always combine `-a` (append) with `-G` (supplementary groups). Running `usermod -G <group> <user>` without `-a` will overwrite and **remove** the user from all existing supplementary groups!

## Part 3 - Identity & Active Session Auditing

Administrators rely on built-in utilities to verify account UIDs/GIDs and track who is currently connected to the system.

### Identity Inspection (`id`)

* **Command:**
  ```
  id <username>
  ```
* **Output Details:** Displays the user's numeric UID, primary GID, and all supplementary group names and GIDs.

### Active Session Monitoring (`who` vs `w`)

* **`who`**
  * **Function:** Displays a list of currently logged-in users along with their terminal lines (`TTY`/`PTS`), login timestamps, and originating remote hostnames/IPs.

* **`w`**
  * **Function:** Displays currently logged-in users along with system uptime, load averages, idle times, CPU resource usage, and the exact **active process/command ("what" they are doing)** in their session.

## Part 4 - Hands-On Practice Lab

### Step 1: Inspect and Update User Defaults

1. View current default user creation parameters:
   ```
   useradd -D
   ```

2. Verify the contents of `/etc/default/useradd`:
   ```
   cat /etc/default/useradd
   ```

### Step 2: Create Groups and Provision a Test User

1. Create two test groups (`developers` and `auditors`):
   ```
   sudo groupadd developers
   sudo groupadd auditors
   ```

2. Create a test user `jdoe` with a home directory (`-m`) and set a password:
   ```
   sudo useradd -m jdoe
   sudo passwd jdoe
   ```

3. Append `jdoe` to the `developers` supplementary group:
   ```
   sudo usermod -aG developers jdoe
   ```

4. Append `jdoe` to the `auditors` supplementary group without losing `developers`:
   ```
   sudo usermod -aG auditors jdoe
   ```

5. Verify `jdoe`'s UID, primary GID, and supplementary groups using `id`:
   ```
   id jdoe
   ```

### Step 3: Monitor Active Sessions and Clean Up

1. Compare session output between `who` and `w`:
   ```
   who
   w
   ```

2. Rename the `auditors` group to `sec_auditors` using `groupmod`:
   ```
   sudo groupmod -n sec_auditors auditors
   id jdoe
   ```

3. Forcefully remove the test user, their home directory, and clean up the test groups:
   ```
   sudo userdel -rf jdoe
   sudo groupdel developers
   sudo groupdel sec_auditors
   ```

## Completion Checklist

* [x] I can inspect default user settings with `useradd -D` and update the default shell with `useradd -D -s`
* [x] I remember to pass `-m` to `useradd` to create home directories and use `userdel -rf` for clean removal
* [x] I know how to create, modify, and delete groups using `groupadd`, `groupmod`, and `groupdel`
* [x] I understand the difference between `usermod -g` (primary group) and `usermod -aG` (append secondary group)
* [x] I can inspect user UID/GID mappings with `id` and distinguish between `who` and `w` session outputs
