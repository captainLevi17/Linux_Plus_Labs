# Lab 23 - Remote File Transfer Utilities

## Objective

Master essential Linux command-line data transfer tools. By completing this lab, you will learn how to download web resources using `curl` and `wget`, mirror web content, perform non-interactive secure file copies with `scp`, and navigate interactive secure sessions using `sftp`.

By completing this lab, I should be able to:

* Download files from web servers using `curl` and `wget` with custom output filenames
* Follow HTTP redirects (`curl -L`) and mirror full directory structures (`wget --mirror`)
* Securely copy files across remote hosts over SSH using `scp`
* Conduct interactive file transfer sessions using `sftp` and manage local versus remote directories
* Understand the security and functional differences between `scp` and `sftp`

---

## Part 1 - Web Data Transfer (`curl` vs `wget`)

Both `curl` and `wget` retrieve data over protocols like HTTP, HTTPS, and FTP, but they serve different primary use cases.

### Core Comparison & Features

| Feature | `curl` | `wget` |
| :--- | :--- | :--- |
| **Primary Design** | Data transfer to/from servers (supports REST APIs, POST requests) | Non-interactive network downloader (built for recursive downloading) |
| **Default Output** | Writes response directly to `stdout` (terminal screen) | Saves output directly to a file on disk |
| **Save to Custom File** | `curl -o <filename> <URL>` | `wget -O <filename> <URL>` |
| **Follow HTTP Redirects**| Requires explicit **`-L`** flag (`curl -L <URL>`) | Follows redirects automatically |
| **Website Mirroring** | Not natively supported | **`wget --mirror`** or **`wget -r`** (recursive) |

---

## Part 2 - SSH-Based Secure File Transfer (`scp` vs `sftp`)

Both `scp` (Secure Copy Protocol) and `sftp` (SSH File Transfer Protocol) run over port 22 using SSH encryption.

### `scp` vs `sftp` Functional Differences

| Feature | `scp` (Secure Copy) | `sftp` (SSH File Transfer Protocol) |
| :--- | :--- | :--- |
| **Interaction Mode** | Command-line scriptable, direct one-shot copy | Interactive shell interface |
| **Session Operations** | Copies files and exits immediately | Allows browsing, creating, deleting, and listing files |
| **Remote Syntax** | `scp local.txt user@host:/remote/path/` | `sftp user@host` |
| **Shell Access Needed?** | Requires interactive shell access | Can be restricted to **SFTP-only** access without shell |

### Interactive `sftp` Command Reference

Once an `sftp` session is established via `sftp user@host`:

* **`get <remote_file>`** : Downloads a file from the remote system to the local directory.
* **`put <local_file>`** : Uploads a file from the local directory to the remote system.
* **`ls`** : Lists files in the **remote** working directory.
* **`lls`** : Lists files in the **local** working directory.
* **`cd` / `lcd`** : Change remote directory / Change local directory.

---

## Part 3 - Hands-On Practice Lab

### Step 1: Practice Web Transfers (`curl` and `wget`)

1. Retrieve a web page's raw HTML and display it on `stdout` using `curl`:
   ```bash
   curl https://example.com
   ```

2. Download a web resource, follow redirects (`-L`), and save it to a specific file (`-o`):
   ```bash
   curl -L -o /tmp/example_curl.html https://example.com
   ```

3. Download the same page using `wget` with a custom output filename (`-O`):
   ```bash
   wget -O /tmp/example_wget.html https://example.com
   ```

### Step 2: Practice SSH Copying (`scp`)

1. Copy a local test file to a remote VM (e.g., your Kali VM or localhost):
   ```bash
   echo "Test payload" > /tmp/scp_test.txt
   scp /tmp/scp_test.txt user@<REMOTE_IP>:/tmp/
   ```

2. Recursively copy an entire local directory to a remote host using `-r`:
   ```bash
   mkdir -p /tmp/testdir && touch /tmp/testdir/file{1..3}
   scp -r /tmp/testdir user@<REMOTE_IP>:/tmp/
   ```

### Step 3: Practice Interactive SFTP Sessions (`sftp`)

1. Connect to a remote host via `sftp`:
   ```bash
   sftp user@<REMOTE_IP>
   ```

2. List remote files, then list local files inside the SFTP prompt:
   ```text
   sftp> ls
   sftp> lls
   ```

3. Download a remote file and upload a local file:
   ```text
   sftp> get /tmp/scp_test.txt /tmp/downloaded_test.txt
   sftp> put /tmp/example_curl.html /tmp/uploaded_example.html
   sftp> exit
   ```

---

## Completion Checklist

- [x] I know that `curl` outputs to `stdout` by default, requiring `-o <file>` to save, and `-L` to follow redirects
- [x] I know that `wget` saves files directly and uses `-O <file>` for custom output filenames
- [x] I understand that `wget` is better suited for site mirroring/recursive downloads than `curl`
- [x] I can copy files non-interactively using `scp user@host:/path`
- [x] I can perform interactive file transfers using `sftp` and use commands like `get`, `put`, and `lls`
- [x] I know that accounts can be locked down for SFTP-only access without shell privileges
