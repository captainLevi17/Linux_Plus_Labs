# Lab 17 - Process Signals and Management

## Objective

Master process control and signal handling in Linux. By completing this lab, you will learn how Linux signals function, memorize key signal names and numbers, use utilities like `kill`, `pkill`, `killall`, and `pidof` to manage processes, and prevent process termination using `nohup`.

By completing this lab, I should be able to:

* Identify key Linux system signals by name and numerical code (SIGTERM, SIGKILL, SIGHUP, SIGINT, SIGUSR1)

* Send specific signals to processes using `kill`, `killall`, and `pkill`

* Find process identification numbers (PIDs) using `pidof`

* Protect running jobs from terminal disconnections using `nohup`

---

## Part 1 - Essential Linux Signals

Signals are asynchronous notifications sent by the kernel or users to notify a process of an event.

### Core Signals Reference Table

| Signal Name | Number | Keyboard Shortcut | Function / Description | Can Be Caught or Ignored? |
| :--- | :--- | :--- | :--- | :--- |
| **`SIGHUP`** | **1** | — | **Hangup:** Sent when a terminal disconnects. Often forces daemons to reload configuration files. | Yes |
| **`SIGINT`** | **2** | `Ctrl + C` | **Interrupt:** Requests graceful process interruption/termination. | Yes |
| **`SIGKILL`** | **9** | — | **Kill:** Immediately and forcibly terminates a process. Bypasses application cleanup routines. | **NO** (Kernel handles directly) |
| **`SIGUSR1`** | **10** | — | **User-Defined Signal 1:** Custom signal available for application developers. | Yes |
| **`SIGTERM`** | **15** | — | **Terminate:** Default signal sent by `kill`. Polite request for graceful shutdown. | Yes |

---

## Part 2 - Process Termination Utilities

| Command | Usage Example | Target Selection Method |
| :--- | :--- | :--- |
| **`kill`** | `kill -15 1234` or `kill -SIGKILL 1234` | Targets process by exact **PID** |
| **`pidof`** | `pidof nginx` | Returns the **PID(s)** of an exact matching process name |
| **`killall`** | `killall -9 nginx` | Targets all processes matching the **exact name** |
| **`pkill`** | `pkill -f "python.*script"` | Targets processes matching a **pattern or regex** |
| **`nohup`** | `nohup ./backup.sh &` | Ignores **SIGHUP** so the command continues running after log out |

---

## Part 3 - Hands-On Practice Lab

### Step 1: Find PIDs and Send `SIGTERM` / `SIGKILL`

1. Start a dummy sleep process in the background:
   ```bash
   sleep 300 &
   ```
2. Find the PID of the `sleep` command using `pidof`:
   ```bash
   pidof sleep
   ```
3. Send the default termination signal (`SIGTERM` / 15) using `kill`:
   ```bash
   kill <PID>
   ```
4. Verify the process was terminated:
   ```bash
   pidof sleep
   ```

5. Start another dummy process and forcefully kill it with `SIGKILL` (signal 9):
   ```bash
   sleep 300 &
   kill -9 $(pidof sleep)
   ```

### Step 2: Practice Pattern-Based Killing (`pkill` and `killall`)

1. Start multiple dummy processes:
   ```bash
   sleep 400 &
   sleep 400 &
   ```
2. Terminate all matching processes by exact name using `killall`:
   ```bash
   killall sleep
   ```

3. Launch a long-running process and terminate it using pattern matching via `pkill`:
   ```bash
   sleep 500 &
   pkill -f "sleep 50"
   ```

### Step 3: Prevent Disconnection Termination with `nohup`

1. Run a command wrapped in `nohup` to shield it from `SIGHUP` (signal 1):
   ```bash
   nohup sleep 600 > /tmp/nohup_test.log 2>&1 &
   ```
2. Verify that `nohup` detached the process and redirected standard output:
   ```bash
   ps aux | grep sleep
   ```

---

## Completion Checklist

- [x] I know that `kill` sends `SIGTERM` (signal 15) by default
- [x] I understand that `SIGKILL` (signal 9) forcibly stops processes and cannot be caught or ignored
- [x] I can match signals to their numbers: `SIGHUP` (1), `SIGINT` (2 / `Ctrl+C`), `SIGKILL` (9), `SIGUSR1` (10), `SIGTERM` (15)
- [x] I can look up process IDs using `pidof`
- [x] I can terminate processes using `killall` (exact name) and `pkill` (pattern matching)
- [x] I know how `nohup` prevents processes from receiving `SIGHUP` when terminals close
