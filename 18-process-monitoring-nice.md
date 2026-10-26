# Lab 18 - Process Monitoring, Open Files, and Nice Priorities

## Objective

Master Linux process inspection, open file tracking, and CPU scheduling priority management. By completing this lab, you will learn how to monitor active processes with `ps`, `top`, and `htop`, audit open files and network sockets using `lsof`, and adjust CPU scheduling priorities using `nice` and `renice`.

By completing this lab, I should be able to:

* Take process snapshots with `ps aux` and track real-time resource utilization with `top` / `htop`

* Audit files and network sockets opened by specific users or processes using `lsof`

* Understand the nice priority scale from -20 to 19 (where lower values equal higher priority)

* Launch processes with custom CPU priority using `nice -n <level>`

* Adjust running process priority using `renice -n <level> -p <PID>` and recognize unprivileged user restriction boundaries

---

## Part 1 - Process Inspection Utilities (`ps`, `top`, `htop`, `lsof`)

### Process Monitoring Tools

| Tool | Type | Key Features / Notes |
| :--- | :--- | :--- |
| **`ps`** | Static Snapshot | Displays process state at command execution. `PID 1` is assigned to systemd/init. |
| **`top`** | Dynamic Real-Time | Interactive viewer; sorts by **CPU usage** by default. Press `h` for help, `>` to shift sort column, `R` to reverse sort order. |
| **`htop`** | Dynamic Real-Time | Enhanced, colorized process viewer with mouse support. Not installed by default on all distributions. |
| **`lsof`** | File/Socket Audit | **List Open Files**. Tracks files, directories, pipes, and network sockets opened by processes. |

### Common `lsof` Commands

* **Filter open files by user:**
  ```bash
  lsof -u <username>
  ```
* **Filter for network connections/ports:**
  ```bash
  lsof -i
  ```
* **Filter by active network port:**
  ```bash
  lsof -i :22
  ```

---

## Part 2 - CPU Scheduling Priorities (`nice` & `renice`)

Linux uses a "niceness" scale to dictate CPU priority. Higher niceness values make a process "nicer" to other processes by lowering its CPU priority.

### The Niceness Scale

* **Default Nice Level:** `0`
* **Highest Priority:** `-20` (or `-19` depending on distro tools)
* **Lowest Priority:** `19`
* **Rule:** **Lower** numerical value = **Higher** priority.

### Privileges & Syntax Rules

* **Regular Users:** Can only **increase** the nice value (lower the priority) of processes they own.
* **Root User:** Required to set negative nice values or decrease the nice value (raise priority) of any process.
* **Recommended Syntax:** Always use `nice -n <value>` explicitly (e.g., `nice -n 10 <command>`) to prevent ambiguity with negative numbers.

### Command Comparison

| Command | Target | Syntax Example |
| :--- | :--- | :--- |
| **`nice`** | **New** process before launch | `nice -n 10 <command>` |
| **`renice`** | **Existing** running process | `renice -n 19 -p <PID>` |

---

## Part 3 - Hands-On Practice Lab

### Step 1: Inspect Running Processes and Open Files

1. List all processes on the system and filter for specific commands:
   ```bash
   ps aux | grep systemd
   ```
2. Launch `top`, change the sorting column using `>`, reverse sort order with `R`, and exit with `q`.
3. Audit open network files using `lsof`:
   ```bash
   sudo lsof -i
   ```
4. View all files opened by your current user:
   ```bash
   lsof -u $USER
   ```

### Step 2: Manage CPU Priorities (`nice` & `renice`)

1. Start a long-running process in the background with a lowered priority (nice level 10):
   ```bash
   nice -n 10 sleep 600 &
   ```
2. Verify its PID and niceness level using `ps`:
   ```bash
   ps -o pid,ni,cmd -p $(pidof sleep)
   ```
3. Lower the process priority further to `19` using `renice`:
   ```bash
   renice -n 19 -p $(pidof sleep)
   ```
4. Attempt to raise the process priority back to `0` without `sudo` (verify privilege error):
   ```bash
   renice -n 0 -p $(pidof sleep)
   ```
5. Successfully raise priority back to `0` using `sudo`:
   ```bash
   sudo renice -n 0 -p $(pidof sleep)
   ```

---

## Completion Checklist

- [x] I understand that `ps` provides static snapshots while `top`/`htop` provide real-time resource tracking
- [x] I know that `lsof` stands for "List Open Files" and can filter by user (`-u`) or network socket (`-i`)
- [x] I understand the nice scale: `-20` is highest priority, `0` is default, `19` is lowest priority
- [x] I can start new tasks with specific nice levels using `nice -n <value>`
- [x] I can adjust existing process priorities using `renice -n <value> -p <PID>`
- [x] I know that root privileges are required to assign negative nice values or raise process priority
