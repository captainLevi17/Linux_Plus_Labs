# Lab 34 - Host-Based Firewall Administration (UFW, Firewalld, Stateful vs. Stateless, and iptables vs. nftables)

## Objective

Master Linux host-based packet filtering architectures, simple firewall management with UFW, zone-based firewall configuration with `firewalld`, the operational differences between stateful and stateless firewalls, and the evolution from legacy `iptables` to modern `nftables`.

By completing this lab, I should be able to:

* Enable, inspect, and delete basic firewall rules using `ufw` (by rule syntax and by rule index number)

* Manage zone-based security policies and service rules dynamically and permanently using `firewall-cmd`

* Differentiate between stateful connection tracking and stateless packet evaluation

* Contrast legacy multi-tool `iptables` syntax with unified `nftables` architecture

* Safely remove firewall rules and revert test configurations for full lab replayability

## Part 1 - Uncomplicated Firewall (`ufw`) Administration

UFW provides a user-friendly frontend for managing underlying Netfilter/iptables packet filtering rules, commonly utilized on Debian and Ubuntu distributions.

### Basic Operations

* **Enable UFW:**

  ```bash
  sudo ufw enable
  ```

  *(Activates the firewall ruleset and configures rules to persist across system reboots).*

* **Check Status and Active Rules:**

  ```bash
  sudo ufw status verbose
  ```

  *(Displays whether UFW is active, default incoming/outgoing policies, and detailed rule numbering).*

### Permitting Web Traffic

* **Allow HTTP and HTTPS Services:**

  ```bash
  sudo ufw allow http
  sudo ufw allow https
  ```

  *(Alternatively, port-based rules can be specified directly: `sudo ufw allow 80/tcp` and `sudo ufw allow 443/tcp`).*

### Deleting UFW Rules

Rules in UFW can be removed either by repeating the exact rule specification preceded by `delete` or by targeting the rule's assigned index number:

* **Method 1 — Delete by Rule Specification (Syntax):**

  ```bash
  sudo ufw delete allow http
  sudo ufw delete allow 80/tcp
  ```

* **Method 2 — Delete by Rule Number:**

  ```bash
  sudo ufw status numbered
  sudo ufw delete 2
  ```

  *(Note: Rule index numbers shift dynamically after each deletion. Always re-run `sudo ufw status numbered` before deleting additional rules by number).*

## Part 2 - Dynamic Zone-Based Firewalls (`firewalld`)

Commonly deployed on RHEL, Fedora, and CentOS systems, `firewalld` introduces the concept of **zones** to manage network trust levels.

### Key Concepts & Zones

* **Zone:** A predefined security trust level assigned to network interfaces or source addresses (e.g., `public`, `home`, `work`, `trusted`, `drop`).

### Core Inspection & Management Commands (`firewall-cmd`)

* **Display Default Active Zone:**

  ```bash
  firewall-cmd --get-default-zone
  ```

* **List Permitted Services in Active Zone:**

  ```bash
  firewall-cmd --list-services
  ```

* **Runtime (Temporary) Rule Addition:**

  ```bash
  sudo firewall-cmd --add-service=http
  ```

  *(Applies immediately in active memory but is lost upon system reboot or firewall reload).*

* **Permanent Rule Addition (Persistent Across Reboots):**

  ```bash
  sudo firewall-cmd --permanent --add-service=http
  sudo firewall-cmd --reload
  ```

  *(Updates permanent configuration files and reloads the ruleset into runtime memory).*

## Part 3 - Packet Filtering Paradigms (Stateful vs. Stateless)

Understanding how a firewall evaluates network packets determines its performance, security posture, and configuration complexity.

### Stateful Firewall

* **Behavior:** Actively monitors and maintains a state table tracking the lifecycle of active network connections (e.g., TCP handshakes, established streams).

* **Advantage:** Automatically permits return traffic for legitimate established outbound or inbound sessions without requiring explicit return rules.

### Stateless Firewall

* **Behavior:** Evaluates each individual network packet in isolation against static rule criteria without maintaining session tracking tables.

* **Trade-off:** Requires explicit, independent rules to permit both inbound requests and outbound responses (doubling rule-base maintenance).

## Part 4 - Netfilter Frameworks (`iptables` vs. `nftables`)

Linux kernel packet filtering has evolved to resolve performance bottlenecks and syntax redundancy in legacy tooling.

### `iptables` (Legacy)

* **Architecture:** Utilizes separate, protocol-specific userspace tools and binary frameworks (`iptables` for IPv4, `ip6tables` for IPv6, `arptables` for ARP).

* **Drawback:** Prone to kernel rule bloat and packet traversal inefficiencies under high loads due to discrete hooks across separate modules.

### `nftables` (Modern Standard)

* **Architecture:** Provides a single, unified kernel backend engine and userspace command utility (`nft`) handling IPv4, IPv6, bridge, and network address translation seamlessly.

* **Advantage:** Offers dramatically improved performance, simplified syntax, atomic rule updates, and native support for sets and maps.

## Part 5 - Hands-On Practice Lab

### Step 1: Configure and Inspect UFW Rules

1. Check current UFW status:

   ```bash
   sudo ufw status
   ```

2. Add rules permitting SSH and HTTP traffic, then inspect their assigned index numbers:

   ```bash
   sudo ufw allow ssh
   sudo ufw allow http
   sudo ufw status numbered
   ```

### Step 2: Query and Test `firewalld` Zones and Services (If Installed)

1. Check the default system zone and list active services:

   ```bash
   firewall-cmd --get-default-zone
   firewall-cmd --list-services
   ```

2. Add HTTP temporarily to the runtime environment, then add it permanently:

   ```bash
   sudo firewall-cmd --add-service=http
   sudo firewall-cmd --permanent --add-service=http
   sudo firewall-cmd --reload
   ```

### Step 3: Compare Raw Netfilter Rulesets

1. View current raw Netfilter rules via legacy tools (if installed):

   ```bash
   sudo iptables -L -v -n
   ```

2. View modern `nftables` ruleset structure:

   ```bash
   sudo nft list ruleset
   ```

## Part 6 - Undo and Cleanup (Full Lab Replayability)

Execute these steps to remove all test rules and restore your firewall to its exact baseline state before replaying the lab.

### 1. Remove UFW Rules and Restore Baseline State

* **Option A — Delete specific rules by syntax:**

  ```bash
  sudo ufw delete allow http
  sudo ufw delete allow ssh
  ```

* **Option B — Delete rules by index number:**

  1. Display numbered rules:

     ```bash
     sudo ufw status numbered
     ```

  2. Delete a rule by its bracketed number (e.g., rule `1`) and confirm with `y`:

     ```bash
     sudo ufw delete 1
     ```

     *(Re-run `sudo ufw status numbered` after each deletion because remaining rule numbers shift upward).*

* **Option C — Full UFW Reset (Wipes all custom rules and disables UFW):**

  ```bash
  sudo ufw reset
  ```

* **Disable UFW (if it was inactive prior to starting the lab):**

  ```bash
  sudo ufw disable
  ```

* **Verify UFW cleanup:**

  ```bash
  sudo ufw status verbose
  ```

### 2. Remove `firewalld` Rules and Reload (If Used)

* **Remove permanent and runtime HTTP rules:**

  ```bash
  sudo firewall-cmd --permanent --remove-service=http
  sudo firewall-cmd --reload
  ```

* **Verify `firewalld` cleanup:**

  ```bash
  firewall-cmd --list-services
  firewall-cmd --permanent --list-services
  ```

## Completion Checklist

* [x] I can enable and verify firewall policies using `ufw enable` and `ufw status`

* [x] I know how to allow and delete HTTP/HTTPS traffic via UFW service names, TCP ports, or rule numbers

* [x] I understand `firewalld` zones and can query active default zones and permitted services

* [x] I can apply and remove temporary runtime changes vs. permanent rules (`--permanent` followed by `firewall-cmd --reload`)

* [x] I can distinguish between stateful connection-tracking firewalls and stateless packet evaluators

* [x] I recognize the architectural advantages of `nftables` over legacy multi-tool `iptables` frameworks
