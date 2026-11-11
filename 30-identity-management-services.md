# Lab 30 - Identity Management, PAM, Directory Services, and Enterprise Authentication

## Objective

Master central concepts of Linux enterprise identity management, user access controls, authentication vs. authorization frameworks, Pluggable Authentication Modules (PAM), Lightweight Directory Access Protocol (LDAP), and the System Security Services Daemon (SSSD).

By completing this lab, I should be able to:

* Differentiate conceptually between Authentication ("who you are") and Authorization ("what you can do")

* Identify Multi-Factor Authentication (MFA) requirements across authentication factor types

* Explain how Pluggable Authentication Modules (PAM) abstract authentication mechanisms away from application source code

* Understand central directory management using LDAP and active client management/offline caching using SSSD

* Inspect and audit PAM rules (`/etc/pam.d/`) and SSSD client configurations (`/etc/sssd/sssd.conf`)

## Part 1 - Core Identity Concepts: Authentication vs Authorization

Enterprise security relies on a clear distinction between identifying users and controlling their privileges.

### Identity Principles

* **Authentication (AuthN)**

  * **Concept:** Verifies identity ("Who you are").

  * **Examples:** Entering a password, presenting an SSH private key, scanning a fingerprint, or providing a Time-based One-Time Password (TOTP) code.

* **Authorization (AuthZ)**

  * **Concept:** Determines permitted privileges ("What you can do").

  * **Examples:** File permissions (`rwx`), `sudoers` privileges, SELinux policies, or LDAP group-based resource access.

* **Multi-Factor Authentication (MFA)**

  * Security framework requiring two or more distinct verification factors before granting access:

    1. **Something You Know:** Passwords, PINs, passphrase.

    2. **Something You Have:** Hardware token, smartphone authenticator app, smart card.

    3. **Something You Are:** Biometric markers (fingerprint, facial recognition, iris scan).

## Part 2 - Pluggable Authentication Modules (PAM)

PAM provides a flexible, modular architecture for Linux authentication. Instead of embedding authentication logic directly into applications (like `sshd`, `sudo`, `login`, or `su`), applications call the PAM API.

### PAM Architecture & Benefits

* **Decoupled Security:** Administrators can swap, update, or add authentication backends (e.g., local `/etc/shadow`, LDAP, Kerberos, MFA tokens) without recompiling or modifying application source code.

* **Configuration Directory:** `/etc/pam.d/`

* **Common Service Files:**

  * `/etc/pam.d/sshd` (Controls SSH authentication policies)

  * `/etc/pam.d/sudo` (Controls privilege elevation policies)

  * `/etc/pam.d/system-auth` or `/etc/pam.d/common-auth` (Global authentication defaults)

## Part 3 - Enterprise Directory Services (`LDAP` & `SSSD`)

In enterprise environments, managing local user accounts on hundreds of individual Linux servers is inefficient. Enterprise networks centralize user management using directory services.

### Directory Services Framework

* **LDAP (Lightweight Directory Access Protocol)**

  * **Role:** Vendor-neutral network protocol used to query, manage, and store directory objects (users, groups, systems, access lists) over IP networks.

  * **Function:** Serves as the central user registry across systems like OpenLDAP or Active Directory.

* **SSSD (System Security Services Daemon)**

  * **Role:** Linux system service (`sssd`) that manages access between client systems and remote identity providers (LDAP, Active Directory, FreeIPA).

  * **Primary Feature:** Provides **offline credential caching**. If a server loses network connectivity to the central LDAP/AD domain controller, users can still log in using cached credentials.

  * **Configuration File:** `/etc/sssd/sssd.conf`

## Part 4 - Hands-On Practice Lab

### Step 1: Inspect Local PAM Configuration Files

1. List the PAM configuration files present on your system:

   ```
   ls -l /etc/pam.d/
   
   
   ```

2. Inspect how PAM handles SSH service logins:

   ```
   cat /etc/pam.d/sshd
   
   
   ```

3. View PAM rules enforcing password complexity or authentication modules in common settings:

   ```
   grep -v '^#' /etc/pam.d/system-auth 2>/dev/null || grep -v '^#' /etc/pam.d/common-auth 2>/dev/null
   
   
   ```

### Step 2: Audit SSSD Client Daemon Setup

1. Check if the SSSD daemon is active on the host system:

   ```
   systemctl status sssd
   
   
   ```

2. View SSSD configuration file permissions (SSSD requires strict permissions, typically `0600`, or it will refuse to start):

   ```
   ls -l /etc/sssd/sssd.conf 2>/dev/null
   
   
   ```

3. Query system user and group identities across local and remote SSSD providers using `getent`:

   ```
   getent passwd
   
   
   ```

## Completion Checklist

* [x] I can explain the difference between Authentication ("who you are") and Authorization ("what you can do")

* [x] I know the three core factor types required for MFA (something you know, have, or are)

* [x] I understand how PAM (`/etc/pam.d/`) decouples application code from underlying authentication modules

* [x] I know that LDAP centralizes directory information over IP networks

* [x] I know that SSSD handles remote domain logins and provides offline credential caching

* [x] I know that SSSD's primary configuration file is `/etc/sssd/sssd.conf`
