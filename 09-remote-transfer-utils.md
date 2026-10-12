# Lab 09 - Remote File Transfer and Network Utilities

## Objective

Master essential Linux command-line utilities for transferring files securely across networks, synchronizing directories efficiently, and performing raw network socket operations using `scp`, `rsync`, and `nc` (netcat).

By completing this lab, I should be able to:

- Copy files and directories securely between local and remote hosts using `scp`
- Synchronize files and directories efficiently using `rsync` (leveraging delta-transfer and SSH)
- Listen for connections, send data, and transfer files raw over network ports using `nc` (netcat)

---

## Part 1 - Secure Copy (`scp`)

The `scp` command uses SSH for secure data transfer between hosts.

- **Secure Copy command:** `scp`
- **Copy file to remote host:**

scp <local file/directory>  user@host:path

- **Copy file from remote host:**

scp user@host:path <local file/directory>

- **Copy directory recursively:** `-r`

---

## Part 2 - Remote Synchronization (`rsync`)

`rsync` is a powerful tool for efficient file copying and synchronization. Unlike `scp`, it only copies **changed files** (using delta-transfer algorithms to minimize data sent over the network).

- **Remote sync command:** `rsync`
- **Default transport protocol:** SSH
- **Sync directory to remote:**

rsync -r <local file/directory> user@host:path

- **Sync remote directory locally:**

rsync -r user@host:path <local file/directory>

- **Recursive mode option:** `-r`
- **Verbose mode option (shows detailed progress):** `-v`

---

## Part 3 - Network Utility & Raw Data Transfer (`nc` / netcat)

Often referred to as the "Swiss Army knife" of networking, `nc` reads and writes data across network connections using TCP or UDP.

- **Network utility command:** `nc`
- **Listen on a port (Receiver):**

nc -l <port number>

- **Connect to a host/port (Sender):**

nc <host> <port>

- **Send a file over netcat:**

nc <host> <port>  < file 

- **Receive and save a file over netcat:**

nc -l  > file

---

## Part 4 - Hands-On Practice Exercises

Execute the following commands sequentially in your terminal to practice these concepts:
NOTE: You will need two machines here, a local one, and a remote computer to transfer/receive files to/from.

1. **Test `scp` syntax (dry run concept):**
touch local_file.txt
scp local_file.txt user@remote-server:/path/to/destination/

2. **Test `rsync` with verbose output:**

rsync -rv local_directory/ user@remote-server:/path/to/backup/

3. **Test Netcat local file transfer (open two terminal windows):**
- *Terminal A (Receiver):*
  ```
  nc -l 8080 > received_file.txt
  ```
- *Terminal B (Sender):*
  ```
  echo "Testing Netcat Transfer" > test.txt
  nc localhost 8080 < test.txt
  ```

---

## Completion Checklist

- [x] I can copy files and directories securely using `scp` in both directions
- [x] I understand how `rsync` optimizes transfers by copying only changed files
- [x] I know how to use `rsync -r` and `-v` for recursive and verbose synchronizations
- [x] I understand how to use `nc` (netcat) to listen on ports and stream files between systems



