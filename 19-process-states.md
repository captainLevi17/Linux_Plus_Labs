# Lab 19 - Process States, Job Control, and Kernel Signals

## Objective

Master the lifecycle of Linux processes, understand process execution states, manage job control directly from the shell, and interpret kernel execution details.

By completing this lab, I should be able to:

- Identify and interpret process state codes in process inspection tools like `ps`
- Manage foreground and background jobs using `jobs`, `fg`, `bg`, and the `&` operator
- Understand the mechanics of Zombie processes and why `SIGKILL` cannot terminate them
- Use keyboard signals (`Ctrl+C`, `Ctrl+Z`, `Ctrl+D`) effectively during CLI sessions
- Customize `ps` output formatting with `-o` and inspect kernel wait channels (`WCHAN`)

---

## Part 1 - Process Execution States

When inspecting processes with tools like `ps aux` or `top`, the kernel reports a single-letter state code for every active PID.

### Process State Reference Table

| Code | State Name | Description |
| :--- | :--- | :--- |
| **`R`** | **Running / Runnable** | The process is currently executing on CPU or waiting in the run queue. |
| **`S`** | **Interruptible Sleep** | Waiting for an event (such as user input, timer, or network packet) to complete. |
| **`D`** | **Uninterruptible Sleep** | Sleeping due to synchronous I/O operations (e.g., waiting for disk access). Cannot be interrupted by signals. |
| **`T`** | **Stopped** | Paused by a job control signal (e.g., `SIGTSTP` via `Ctrl+Z`) or a debugger. |
| **`Z`** | **Zombie** | Terminated child process whose parent process has not yet read its exit status via `wait()`. |

### Zombie Process Mechanics
- **Definition:** A process that has finished execution but still occupies an entry in the system process table.
- **Resource Usage:** Zombies consume **no CPU or memory**; they only hold a Process ID (PID) slot.
- **Termination:** Zombies **cannot be killed with `SIGKILL` (signal 9)** because they are already dead. To eliminate a zombie, you must either send `SIGCHLD` to the parent process, terminate the parent process itself, or restart the system.

---

## Part 2 - Keyboard Controls and Signals

Keyboard combinations allow direct interaction with foreground terminal jobs.

| Keyboard Shortcut | Signal / Action | Result |
| :--- | :--- | :--- |
| **`Ctrl + C`** | `SIGINT` (Signal 2) | Interrupts and requests termination of the foreground process. |
| **`Ctrl + Z`** | `SIGTSTP` (Signal 20) | Sends "Terminal Stop"; pauses the foreground process and places it in the background as a stopped job. |
| **`Ctrl + D`** | **EOF (End-Of-File)** | **Not a signal.** Sends an end-of-file marker to `stdin`, signaling applications or shells to close cleanly. |

---

## Part 3 - Shell Job Control (`jobs`, `fg`, `bg`)

Job control allows managing multiple distinct tasks running within a single shell session.

### Core Job Control Commands

- **Start a command directly in the background:**
  ```bash
  <command> &
  ```
- **List jobs in the current shell:**
  ```bash
  jobs
  ```
  *(Note: The `+` symbol next to a job number indicates the default target for `fg` or `bg` commands).*

- **Resume a stopped job in the background:**
  ```bash
  bg          # Resumes the default job (+) in the background
  bg %2       # Resumes job #2 in the background
  ```

- **Bring a background or stopped job to the foreground:**
  ```bash
  fg          # Brings the default job (+) to the foreground
  fg %1       # Brings job #1 to the foreground
  ```

---

## Part 4 - Advanced `ps` Inspection Options

You can customize `ps` output to inspect specific attributes:

- **Display custom columns:**
  ```bash
  ps -o pid,user,stat,wchan,cmd
  ```

### What is `WCHAN`?
The **`WCHAN`** (Wait Channel) field displays the name of the specific kernel function or event where a sleeping or blocked process (`S` or `D` state) is currently waiting.

---

## Part 5 - Hands-On Practice Lab

Execute these commands in your terminal to practice process states and job control:

### Step 1: Practice Foreground, Background, and Stopping Jobs
1. Launch a long-running process in the foreground:
   ```bash
   sleep 300
   ```
2. Press **`Ctrl + Z`** to pause it. Observe the job creation message.
3. Check active shell jobs:
   ```bash
   jobs
   ```
4. Resume the process in the background:
   ```bash
   bg %1
   ```
5. Verify it is running in the background using `jobs`.
6. Bring it back to the foreground:
   ```bash
   fg %1
   ```
7. Press **`Ctrl + C`** to interrupt and kill the process.

### Step 2: Inspect Process States and Custom Columns
1. Launch a background task using `&`:
   ```bash
   sleep 600 &
   ```
2. View its PID, State, and Kernel Wait Channel using custom `ps` formatting:
   ```bash
   ps -o pid,stat,wchan,cmd -p $(pidof sleep)
   ```

---

## Completion Checklist

- [x] I can identify all 5 core process state letters (`R`, `S`, `D`, `T`, `Z`)
- [x] I understand why Zombie processes (`Z`) cannot be killed with `SIGKILL`
- [x] I know the distinction between `Ctrl+C` (`SIGINT`), `Ctrl+Z` (`SIGTSTP`), and `Ctrl+D` (EOF)
- [x] I can list shell jobs using `jobs` and identify the current default job (`+`)
- [x] I can manipulate jobs using `bg %<number>` and `fg %<number>`
- [x] I know how to use `ps -o` to inspect custom columns like `WCHAN`
