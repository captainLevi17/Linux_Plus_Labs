# Lab 03 - Linux Devices and Hardware Inspection


**Commands Practiced:** `ls`, `lspci`, `lsusb`, `dmidecode`, `free`, `grep`, `echo`

## Overview & Objectives

Explore how Linux manages physical and virtual devices within the `/dev/` directory, and practice using system tools to inspect hardware components.

**Key Learning Outcomes:**

- Distinguish between block devices and character devices
- Understand special pseudo-devices (`/dev/zero`, `/dev/null`, `/dev/random`, `/dev/urandom`)
- Inspect hardware specifications using PCI, USB, DMI, and memory management utilities

## Part 1 - Device Files & Types

Device files are located in `/dev/`. In `ls -l` output, the first character indicates the device type:

- **Block Devices (`b`):** Use buffers and caches to transfer data in blocks (e.g., hard drives like `/dev/sda`). Trailing numbers represent partitions (e.g., `sda1`, `sda2`).
- **Character Devices (`c`):** Transfer unbuffered data sequentially as a stream of bytes (e.g., serial ports, printer ports).

### Questions

1. **Where are device files stored in the Linux filesystem?**
   - **Answer:** `/dev/`

2. **What character at the beginning of `ls -l` signifies a block device?**
   - **Answer:** `b` (e.g., `brw-rw----`)

3. **What character at the beginning of `ls -l` signifies a character device?**
   - **Answer:** `c` (e.g., `crw-rw-rw-`)

4. **What is the operational difference between block and character devices?**
   - **Answer:** Block devices transfer data in cached blocks, whereas character devices stream data unbuffered in real-time byte by byte.

5. **What do trailing numbers represent in drive names like `sda1` and `sda2`?**
   - **Answer:** Individual partitions on drive `sda`.

## Part 2 - Special Virtual Devices & Utilities

Linux uses virtual character devices for specific administrative operations:

- **/dev/zero:** Provides an infinite stream of null characters (binary zeros). Common for zeroing out or wiping storage drives.
- **/dev/null:** Known as the "bit bucket." Data written to it disappears forever; reading from it returns nothing immediately (size 0).
- **/dev/random vs. /dev/urandom:**
  - `/dev/random`: Blocks (waits) if system entropy is low.
  - `/dev/urandom`: Non-blocking pseudo-random data, preferred for programmatic use.
- **`$RANDOM`:** Built-in Bash variable that returns a random integer without accessing `/dev/`.

### Questions

1. **What is `/dev/zero` used for?**
   - **Answer:** Generating null character streams, often used to zero-out storage drives.

2. **What happens to data sent to `/dev/null`?**
   - **Answer:** It is immediately discarded ("bit bucket").

3. **Why is `/dev/urandom` preferred over `/dev/random` in scripts?**
   - **Answer:** `/dev/urandom` is non-blocking and provides random data immediately without waiting for system entropy.

4. **How can you print a random number in Bash without querying `/dev/`?**
   - **Answer:** `echo $RANDOM`

## Part 3 - Hardware Inspection Tools

Commands used to interrogate physical and system hardware:

- **`lspci`** — Lists all PCI devices (including motherboard-integrated devices).
- **`lsusb`** — Lists USB controllers and connected USB devices.
- **`dmidecode`** — Reads DMI/SMBIOS tables for detailed hardware information (requires root/`sudo`).
- **`free -h`** — Displays memory usage statistics in human-readable units (MB, GB).

### Exercises

1. Filter PCI devices specifically for network controllers (case-insensitive):
```
lspci | grep -i network
```

2. Inspect processor information using `dmidecode`:
```
dmidecode -t processor
```

3. Display RAM and swap statistics:
```
free -h
```

## Completion Checklist

- [x] Identify block vs. character devices in `/dev/`
- [x] Understand purpose of `/dev/null`, `/dev/zero`, `/dev/random`, and `/dev/urandom`
- [x] List PCI and USB hardware using `lspci` and `lsusb`
- [x] Retrieve system hardware details using `dmidecode`
- [x] Monitor system memory using `free -h`
