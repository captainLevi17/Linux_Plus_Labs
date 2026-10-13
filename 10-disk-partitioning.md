# Lab 10 - Linux Disk Partitioning, Storage Inspection, and Usage Management

## Objective

Master essential Linux storage management utilities for inspecting block devices, creating and modifying disk partitions with `fdisk` and `parted`, reloading partition tables, wiping storage headers, and monitoring disk and directory usage.

By completing this lab, I should be able to:

- Inspect block devices, UUIDs, and SCSI hardware using `lsblk`, `blkid`, and `lsscsi`
- Create and manage disk partitions interactively using `fdisk` and scriptably using `parted`
- Reload kernel partition tables dynamically using `partprobe`
- Safely wipe disk partition headers using `dd`
- Monitor filesystem disk space (`df`) and directory sizes (`du`)

---

## Part 1 - Storage & Block Device Inspection

- **List block devices:** `lsblk`
- **Show block device UUIDs and filesystem types:** `blkid`
- **List SCSI storage devices:** `lsscsi`

---

## Part 2 - Interactive Partitioning with `fdisk`

`fdisk` is a traditional menu-driven program for partitioning MBR and GPT disks.

- **Open disk for partitioning:** `fdisk /dev/sdX`
- **Show interactive help menu:** `m`
- **Print current partition table:** `p`
- **Create a new partition:** `n`
- **Write changes to disk and exit:** `w`
- **Quit without saving changes:** `q`

### Partition Table Standards Recap
- **Legacy BIOS standard:** MBR (Master Boot Record)
- **UEFI standard:** GPT (GUID Partition Table)

---

## Part 3 - Advanced Partitioning with `parted`

`parted` supports both MBR and GPT, handles very large disks (>2TB), and can be used interactively or via command-line arguments.

- **Open partition editor:** `parted /dev/sdX`
- **Show help menu:** `help`
- **Print current partition table:** `print`
- **Create a GPT partition table:** `mklabel gpt`
- **Create a new partition:** `mkpart`
- **Quit editor:** `quit`

---

## Part 4 - Partition Table Management & Disk Wiping

- **Reload partition table into kernel without rebooting:**
partprobe
- **Wipe beginning of disk / partition headers:**
dd if=/dev/zero of=/dev/sdX count=100
*(Overwrites the first 100 blocks with zeros to clear old partition tables and filesystem signatures).*

---

## Part 5 - Disk Space & Directory Usage Monitoring

- **Show filesystem disk usage (human-readable format):** `df -h`
- **Show directory size summary (summarize + human-readable):** `du -sh <directory>`

---

## Part 6 - Fibre Channel & Enterprise Storage

- **Show Fibre Channel adapter information and statistics:** `FCstat`

---

## Part 7 - Hands-On Practice Exercises

Execute the following commands sequentially in your terminal to practice these concepts:

1. **Inspect connected block devices and UUIDs:**
lsblk
blkid
2. **Check overall disk usage:**
df -h
3. **Check the size of a specific directory:**
du -sh /var/log
4. **Open a disk safely in parted (read-only / inspect mode):**
sudo parted /dev/sda print
---

## Completion Checklist

- [x] I can list block devices and identify UUIDs using `lsblk` and `blkid`
- [x] I know how to navigate the interactive `fdisk` menu (`m`, `p`, `n`, `w`, `q`)
- [x] I can create GPT partition tables and partitions using `parted`
- [x] I know how to use `partprobe` to update kernel partition tables without rebooting
- [x] I can monitor disk and directory usage using `df -h` and `du -sh`


