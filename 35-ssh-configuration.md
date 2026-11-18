# Lab 35 - SSH Key Management, Client Configuration, and Tunneling

## Objective

Master Secure Shell (SSH) asymmetric key authentication, user-specific client configuration, authentication agent caching, host identity verification, and encrypted tunneling techniques (X11, local port forwarding, and dynamic SOCKS proxying).

By completing this lab, I should be able to:

* Generate asymmetric key pairs with `ssh-keygen` and deploy public keys using `ssh-copy-id`

* Manage passphrase-protected keys in memory using `ssh-agent` and `ssh-add`

* Distinguish between `~/.ssh/config`, `~/.ssh/known_hosts`, and `~/.ssh/authorized_keys`

* Establish encrypted SSH tunnels for graphical X11 forwarding, local port forwarding, and dynamic SOCKS proxies

* Clean up test keys, agent identities, host entries, and client configurations for full lab replayability

## Part 1 - Core `~/.ssh/` Directory Architecture

The `~/.ssh/` directory contains critical files governing client behavior, server trust, and incoming key-based access. Strict permissions (`700` for `~/.ssh` and `600` for private keys/configs) are required by OpenSSH.

* **`~/.ssh/config` (User Client Configuration):**
  * Overrides system-wide `/etc/ssh/ssh_config` settings for the individual user.
  * Defines host aliases, default usernames, custom ports, and specific identity files (`IdentityFile`) to streamline connections.

* **`~/.ssh/known_hosts` (Server Identity & MITM Protection):**
  * Stores the public host key fingerprints of remote servers previously connected to.
  * Allows the SSH client to verify server authenticity on subsequent connections and alert the user to potential Man-in-the-Middle (MITM) attacks or server rebuilds.

* **`~/.ssh/authorized_keys` (Incoming Public Key Access):**
  * Located on the target/remote server inside the target user's home directory.
  * Stores the public keys of external clients permitted to authenticate and log in to that specific user account without a password.

## Part 2 - Key Generation, Deployment, and Agent Caching

### Generating Key Pairs (`ssh-keygen`)

* **Generate an Ed25519 Key Pair (Recommended Modern Standard):**
  ```bash
  ssh-keygen -t ed25519 -C "lab35-key"
  ```
  *(Creates `~/.ssh/id_ed25519` [private key] and `~/.ssh/id_ed25519.pub` [public key]).*

* **Generate a Legacy RSA Key Pair:**
  ```bash
  ssh-keygen -t rsa -b 4096
  ```

### Deploying Public Keys (`ssh-copy-id`)

* **Install Public Key onto Remote Host:**
  ```bash
  ssh-copy-id -i ~/.ssh/id_ed25519.pub <user>@<hostname>
  ```
  *(Authenticates once via password, creates `~/.ssh` on the remote host if absent, sets proper permissions, and appends the public key to the remote `~/.ssh/authorized_keys` file).*

### Caching Passphrases (`ssh-agent` and `ssh-add`)

* **`ssh-agent`:** A background daemon that holds decrypted private keys securely in memory.
* **`ssh-add`:** The utility used to register passphrase-protected private keys into the running `ssh-agent` so you only enter your passphrase once per session.
  * **Start the agent in your current shell (if not already running):**
    ```bash
    eval "$(ssh-agent -s)"
    ```
  * **Register a private key with the agent:**
    ```bash
    ssh-add ~/.ssh/id_ed25519
    ```
  * **List currently cached keys in memory:**
    ```bash
    ssh-add -l
    ```

## Part 3 - SSH Tunneling and Port Forwarding

SSH can encapsulate unencrypted protocols and route network traffic inside an encrypted SSH session.

### 1. X11 Forwarding (`ssh -X`)

* **Purpose:** Securely forwards graphical X Window System applications from a remote Linux server to your local desktop display.
* **Syntax:**
  ```bash
  ssh -X <user>@<hostname>
  ```
  *(Once connected, launching a GUI tool like `xeyes` or `gedit` renders its window locally).*

### 2. Local Port Forwarding (`ssh -L`)

* **Purpose:** Binds a port on your local machine and forwards all traffic sent to that local port through the encrypted SSH tunnel to a specific target host and port reachable from the SSH server.
* **Syntax:**
  ```bash
  ssh -L <local_port>:<target_host>:<target_port> <user>@<ssh_server>
  ```
* **Practical Example (Accessing a remote internal web dashboard on port 80 via local port 8080):**
  ```bash
  ssh -L 8080:127.0.0.1:80 <user>@<ssh_server>
  ```
  *(Navigating to `http://localhost:8080` on your local machine securely accesses port `80` on the remote server).*

### 3. Dynamic Port Forwarding (`ssh -D`)

* **Purpose:** Creates a local **SOCKS proxy server** (SOCKS4/SOCKS5) that dynamically routes all application or web browser traffic through the remote SSH host, regardless of destination IP or port.
* **Syntax:**
  ```bash
  ssh -D <local_socks_port> <user>@<ssh_server>
  ```
* **Practical Example:**
  ```bash
  ssh -D 9050 <user>@<ssh_server>
  ```
  *(Configure a browser or `curl --socks5 127.0.0.1:9050` to route traffic out through `<ssh_server>`).*

## Part 4 - Hands-On Practice Lab

*(Note: You can perform this entire lab safely against `localhost` if a second VM is unavailable).*

*(If you're SSHing on to a desktop distro, you might need to install the ssh server there by typing sudo apt install openssh-server)*

### Step 1: Generate a Dedicated Lab Key Pair

1. Create a distinct test key pair with a passphrase so you do not overwrite your primary keys:
   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/id_lab35 -C "lab35-test"
   ```
   *(Enter a test passphrase when prompted).*

### Step 2: Register the Key with `ssh-agent`

1. Ensure `ssh-agent` is active and load the private key:
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_lab35
   ```
2. Verify the key fingerprint is cached in memory:
   ```bash
   ssh-add -l
   ```

### Step 3: Deploy the Public Key and Configure `~/.ssh/config`

1. Copy the public key to your target host (or `localhost`):
   ```bash
   ssh-copy-id -i ~/.ssh/id_lab35.pub $USER@localhost
   ```
2. Create or append a host alias in `~/.ssh/config`:
   ```bash
   nano ~/.ssh/config
   ```
   Add the following block:
   ```text
   Host lab35-local
       HostName 127.0.0.1
       User your_username
       IdentityFile ~/.ssh/id_lab35
   ```
   *(Replace `your_username` with your actual Linux username, then save and exit).*
3. Enforce strict permissions on `~/.ssh/config`:
   ```bash
   chmod 600 ~/.ssh/config
   ```
4. Test passwordless/passphrase-cached login using your alias:
   ```bash
   ssh lab35-local
   ```
   *(Type `exit` once logged in to return to your original shell).*

### Step 4: Test Dynamic SOCKS Proxy Tunneling

1. Open a background dynamic SOCKS tunnel on local port `9050`:
   ```bash
   ssh -f -N -D 9050 lab35-local
   ```
   *(`-f` backgrounds SSH after authentication; `-N` instructs SSH not to execute a remote command).*
2. Verify the local listening socket is active:
   ```bash
   ss -tulpn | grep 9050
   ```

## Part 5 - Undo and Cleanup (Full Lab Replayability)

Execute these steps to remove all test keys, clear cached agent identities, terminate background tunnels, and restore `~/.ssh/` to its baseline state.

### 1. Terminate Background SSH Tunnels

1. Find and terminate any background `ssh -D 9050` tunnel process:
   ```bash
   pkill -f "ssh -f -N -D 9050"
   ```
2. Verify port `9050` is closed:
   ```bash
   ss -tulpn | grep 9050
   ```

### 2. Evict Keys from `ssh-agent`

1. Remove the specific lab key from `ssh-agent` memory (or use `ssh-add -D` to clear all cached identities):
   ```bash
   ssh-add -d ~/.ssh/id_lab35.pub
   ```
2. Verify the agent no longer holds the key:
   ```bash
   ssh-add -l
   ```

### 3. Remove the Public Key from `authorized_keys`

1. Open `~/.ssh/authorized_keys` on the target host (or locally if you tested against `localhost`):
   ```bash
   nano ~/.ssh/authorized_keys
   ```
2. Delete the line ending with `lab35-test`, then save and close (`Ctrl+O`, `Enter`, `Ctrl+X`).

### 4. Delete Test Key Files and Revert `~/.ssh/config`

1. Remove the generated private and public key pair:
   ```bash
   rm -f ~/.ssh/id_lab35 ~/.ssh/id_lab35.pub
   ```
2. Open `~/.ssh/config` and remove the `Host lab35-local` block (or remove `~/.ssh/config` if it did not exist prior to the lab):
   ```bash
   nano ~/.ssh/config
   ```
3. *(Optional)* Remove the `127.0.0.1` entry from `~/.ssh/known_hosts` if you want to re-test the initial host fingerprint prompt:
   ```bash
   ssh-keygen -R 127.0.0.1
   ```

## Completion Checklist

* [x] I understand how `~/.ssh/config` overrides global SSH client defaults
* [x] I can generate key pairs with `ssh-keygen` and deploy them with `ssh-copy-id`
* [x] I know how `ssh-agent` and `ssh-add` cache decrypted private keys in memory
* [x] I can distinguish the security roles of `~/.ssh/known_hosts` and `~/.ssh/authorized_keys`
* [x] I can construct SSH commands for X11 forwarding (`-X`), local port forwarding (`-L`), and dynamic SOCKS proxies (`-D`)
