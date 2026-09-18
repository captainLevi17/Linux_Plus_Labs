# Lab 01 - Filesystem Hierarchy Standard (FHS)

**OS:** Ubuntu 26.04.1 LTS  
**Commands Practiced:** `pwd`, `ls`, `cd`

## Overview & Objectives

Understand the Linux Filesystem Hierarchy Standard (FHS) by exploring common directories and practicing path navigation.

**Key Learning Outcomes:**
- Navigate using absolute and relative paths (`.`, `..`)
- Identify core FHS directory purposes
- Inspect directory structures using `tree`

---

## Part 1 - Root Directory & Exploration

### Commands
```
pwd
ls /
```

### Questions
1. **What directory represents the top level of the Linux filesystem?**
   - **Answer:** `/`

2. **What directories can you identify from the root directory listing?**
   - **Answer:** `boot`, `cdrom`, `dev`, `etc`, `home`, `lost+found`, `media`, `mnt`, `opt`, `proc`, `root`, `run`, `snap`, `srv`, `sys`, `tmp`, `usr`, `var`

---

## Part 2 - Filesystem Hierarchy Standard (FHS) Reference

- **/home** — Non-root user home directories
- **/root** — Root user home directory
- **/boot** — Kernel and bootloader files
- **/etc** — Systemwide configuration files
- **/opt** — Add-on software packages
- **/media** — Mount point for removable media
- **/mnt** — Temporary mount directory
- **/tmp** — Temporary files (cleared on reboot)
- **/var/tmp** — Persistent temporary files across reboots
- **/var** — Variable data (logs, databases, spool files)
- **/bin** — Essential command binaries
- **/sbin** — Essential system administration binaries
- **/usr/bin** — Non-essential user binaries
- **/usr/lib** — Libraries for `/usr/bin` and `/usr/sbin`
- **/usr/local** — Local software and scripts
- **/dev** — Device nodes/files
- **/proc** — Virtual filesystem for process & kernel information
- **/sys** — Virtual filesystem for system/hardware device information

---

## Part 3 - Navigation & Special Directories

### Concepts
- **Absolute path:** Full path starting from `/` (e.g., `/usr/bin`)
- **Relative path:** Path relative to current location (e.g., `../lib`)
- **`.`** — Current directory
- **`..`** — Parent directory

### Exercise
1. Navigate to `/usr/bin`: `cd /usr/bin`
2. Verify location: `pwd`
3. Navigate to `/usr/lib` using relative path: `cd ../lib`

### Questions
1. **What is the difference between an absolute path and a relative path?**
   - **Answer:** An absolute path is the complete path starting from the root directory (`/`), whereas a relative path is evaluated relative to the current working directory.

2. **What do `.` and `..` represent?**
   - **Answer:** `.` represents the current working directory, and `..` represents the parent directory.

---



## Completion Checklist

- [x] Navigate using absolute and relative paths
- [x] Understand `.` and `..` directory shortcuts
- [x] Identify common FHS directory purposes
- [x] Explore directory trees using `tree`
