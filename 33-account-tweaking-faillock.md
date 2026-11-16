# Lab 33 - Account Database Security, Password Aging, and Faillock Administration

## Objective

Master the security architecture of Linux account databases (`/etc/passwd`, `/etc/shadow`), safe syntax-checked database editing (`vipw`, `vigr`), user home directory templates (`/etc/skel`), default password aging policies (`/etc/login.defs`, `chage`), and brute-force lockout protection (`pam_faillock` / `faillock`).

By completing this lab, I should be able to:

* Explain what sensitive data is isolated in `/etc/shadow` and verify its restricted file permissions

* Safely edit account and group databases with file locking and syntax validation using `vipw`, `vipw -s`, `vigr`, and `vigr -s`

* Understand how `/etc/skel` populates default files when provisioning new user accounts

* Configure default password aging rules in `/etc/login.defs` and interpret `chage -l` grace period (`inactive`) statuses

* Audit and reset failed login lockouts managed by the `pam_faillock` module using the `faillock` utility

## Part 1 - Account Database Security & Safe Editing (`vipw` / `vigr`)

Linux separates public user identity metadata from sensitive authentication hashes to prevent unauthorized credential harvesting.

### The `/etc/shadow` File

* **Stored Information:** Contains hashed account passwords (alongside salt and hashing algorithm identifiers) and granular password aging/expiration parameters.

* **Access Permissions:** Strictly restricted. Readable only by the **`root`** user and members of the **`shadow`** group (typically permissions `0640` or `0000`).

### Safe Database Editing Utilities

Never edit user or group files directly with a standard text editor (like `nano` or `vi`), as concurrent writes or syntax typos can corrupt system authentication and lock out all users.

* **Edit `/etc/passwd` safely:**

  ```
  sudo vipw
  
  ```

* **Edit `/etc/shadow` safely:**

  ```
  sudo vipw -s
  
  ```

* **Edit `/etc/group` safely:**

  ```
  sudo vigr
  
  ```

* **Edit `/etc/gshadow` safely:**

  ```
  sudo vigr -s
  
  ```

* **Safety Mechanisms:** These utilities place an exclusive lock on the target database file to prevent race conditions and perform automatic syntax verification before saving changes.

## Part 2 - User Provisioning Templates (`/etc/skel`)

* **Purpose:** The `/etc/skel` (skeleton) directory serves as a template for new user home directories.

* **Provisioning Behavior:** Whenever a new user is created with home directory creation enabled (`useradd -m`), all default files and hidden dotfiles (such as `.bashrc`, `.profile`, and `.bash_logout`) inside `/etc/skel` are automatically copied into `/home/<username>` and assigned ownership to the new user.

## Part 3 - Password Aging & Expiration Policies (`chage` & `/etc/login.defs`)

Enforcing password rotation limits reduces the window of exposure if a credential is compromised.

### Global Defaults (`/etc/login.defs`)

The `/etc/login.defs` configuration file defines system-wide default password aging limits applied automatically to newly created user accounts:

* **`PASS_MAX_DAYS`:** Maximum number of days a password may be used before it expires.

* **`PASS_MIN_DAYS`:** Minimum number of days required between password changes.

* **`PASS_WARN_AGE`:** Number of days prior to expiration that the user receives a warning message at login.

### Inspecting Aging Status (`chage -l`)

* **Command:**

  ```
  sudo chage -l <username>
  
  ```

* **Understanding `"Password inactive"` (Grace Period):**

  * Defines the specific number of days *after* a password has expired during which the user is still permitted to log in and immediately set a new password.

  * Once this grace period elapses without a password change, the account becomes completely locked and requires administrator intervention.

## Part 4 - Brute-Force Protection (`pam_faillock` & `faillock`)

Modern Enterprise Linux distributions replace legacy `pam_tally2` with `pam_faillock` to defend against password-guessing attacks.

### How `pam_faillock` Works

* **Mechanism:** Tracks consecutive failed authentication attempts per user account across login services (SSH, console, `su`).

* **Lockout Action:** Once a user exceeds the configured threshold of consecutive failures, `pam_faillock` temporarily locks the account for a configured duration (or until manually unlocked).

### Administrative Management (`faillock`)

* **View failed login attempts for all users:**

  ```
  sudo faillock
  
  ```

* **View failed attempts for a specific user:**

  ```
  sudo faillock --user <username>
  
  ```

* **Clear lockout counter for a specific user:**

  ```
  sudo faillock --user <username> --reset
  
  ```

* **Clear lockout counters for all users:**

  ```
  sudo faillock --reset
  
  ```

## Part 5 - Hands-On Practice Lab

### Step 1: Audit `/etc/shadow` Permissions and `/etc/skel` Contents

1. Inspect permissions on `/etc/passwd` versus `/etc/shadow`:

   ```
   ls -l /etc/passwd /etc/shadow
   
   ```

2. List all hidden template files stored in `/etc/skel`:

   ```
   ls -la /etc/skel
   
   ```

### Step 2: Inspect Global Password Aging Rules and Account Status

1. Query default password aging directives in `/etc/login.defs`:

   ```
   grep -E "^PASS_(MAX|MIN|WARN)_" /etc/login.defs
   
   ```

2. Inspect the password expiration and grace period (`inactive`) status for your current user:

   ```
   sudo chage -l $USER
   
   ```

### Step 3: Practice `faillock` Auditing

1. Check current failed login attempt records on the system:

   ```
   sudo faillock
   
   ```

2. Run a non-destructive reset command syntax check on a specific user:

   ```
   sudo faillock --user $USER --reset
   
   ```

## Completion Checklist

* [x] I know that `/etc/shadow` stores hashed passwords and aging data and is readable only by `root` and the `shadow` group

* [x] I can safely edit account and group files with syntax validation using `vipw`, `vipw -s`, `vigr`, and `vigr -s`

* [x] I understand that `/etc/skel` provides default template files copied into new user home directories

* [x] I can explain the `"Password inactive"` grace period displayed in `chage -l` output

* [x] I know that `/etc/login.defs` configures default password aging parameters (`PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE`)

* [x] I understand how `pam_faillock` protects against brute-force logins and how to unlock accounts using `faillock --user <username> --reset`
