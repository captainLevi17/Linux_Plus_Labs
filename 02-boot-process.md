# Lab 02 - Linux Boot Process, UEFI & GRUB 2

**OS:** Ubuntu 26.04.1 LTS  
**Commands Practiced:** `lsblk`, `fdisk`, `findmnt`, `uname`, `ls`, `file`, `cat`, `grep`, `update-grub`, `grub-mkconfig`

## Overview & Objectives

Understand the Linux boot process by identifying the system firmware, EFI System Partition, GRUB 2 bootloader, Linux kernel, and initramfs.

**Key Learning Outcomes:**
- Identify whether the system boots using BIOS or UEFI
- Identify the EFI System Partition
- Locate the Linux kernel and initramfs
- Understand the role of GRUB 2
- Inspect Ubuntu's GRUB configuration
- Generate a GRUB configuration
- Understand GRUB Advanced Options and kernel rollback

---

## Part 1 - Identify BIOS or UEFI

### Commands

```bash
ls /sys/firmware/efi
```

Then:

```bash
test -d /sys/firmware/efi && echo "UEFI" || echo "BIOS"
```

### Questions

1. **What does the `/sys/firmware/efi` directory indicate?**
   - **Answer:** If it exists, the Linux system was booted using UEFI firmware.

2. **What does it indicate if `/sys/firmware/efi` does not exist?**
   - **Answer:** The system is most likely booted using legacy BIOS/CSM mode.

3. **What does UEFI stand for?**
   - **Answer:** Unified Extensible Firmware Interface.

4. **What does BIOS stand for?**
   - **Answer:** Basic Input/Output System.

---

## Part 2 - Identify the EFI System Partition

If your system is using UEFI, examine the disks and partitions:

### Commands

```bash
lsblk -f
```

```bash
sudo fdisk -l
```

```bash
findmnt /boot/efi
```

### Exercise

Look for a small partition with a filesystem such as:

```text
vfat
```

You may see something similar to:

```text
/dev/sda1   vfat   FAT32
```

### Questions

1. **What partition is used by UEFI to store boot files?**
   - **Answer:** The EFI System Partition (ESP).

2. **What filesystem is commonly used by the EFI System Partition?**
   - **Answer:** FAT32.

3. **Where is the EFI System Partition commonly mounted in Ubuntu?**
   - **Answer:** `/boot/efi`

4. **What does EFI stand for?**
   - **Answer:** Extensible Firmware Interface. In modern systems, the partition is normally referred to as the EFI System Partition (ESP).

5. **Why is UEFI commonly used with GPT on systems with large drives?**
   - **Answer:** UEFI supports modern partitioning schemes such as GPT, which supports disks and partitions beyond the traditional BIOS/MBR limitations.

---

## Part 3 - Locate the Linux Kernel

The Linux kernel is normally stored in `/boot`.

### Commands

```bash
ls -lh /boot/vmlinuz*
```

```bash
uname -r
```

```bash
file /boot/vmlinuz-*
```

### Questions

1. **What does `vmlinuz` represent?**
   - **Answer:** A compressed Linux kernel image.

2. **Where are Linux kernel images commonly stored?**
   - **Answer:** `/boot`

3. **What command displays the currently running kernel version?**
   - **Answer:** `uname -r`

4. **What is the difference between `vmlinuz` and `vmlinux`?**
   - **Answer:** `vmlinuz` is a compressed kernel image, while `vmlinux` generally refers to an uncompressed Linux kernel image.

---

## Part 4 - Locate the initramfs

The kernel requires an initial filesystem during the early stages of boot.

### Commands

```bash
ls -lh /boot/init*
```

Then:

```bash
file /boot/initrd.img-*
```

Display the current kernel version:

```bash
uname -r
```

Inspect the initramfs associated with the running kernel:

```bash
lsinitramfs /boot/initrd.img-$(uname -r) | head
```

### Questions

1. **What is an initramfs?**
   - **Answer:** An initial RAM filesystem containing files, programs, and drivers needed during the early Linux boot process.

2. **What does initrd stand for?**
   - **Answer:** Initial RAM disk.

3. **Why does Linux need an initramfs?**
   - **Answer:** It provides the kernel with the necessary tools, drivers, and filesystems needed to access and mount the real root filesystem during early boot.

4. **Where are initramfs images commonly stored?**
   - **Answer:** `/boot`

5. **What command can you use to inspect an initramfs on Ubuntu?**
   - **Answer:** `lsinitramfs`

---

## Part 5 - Initramfs Tools

Different Linux distributions use different tools for creating and managing initramfs images.

### Commands

```bash
which mkinitrd
```

```bash
which dracut
```

```bash
which update-initramfs
```

### Questions

1. **What command is traditionally associated with creating an initrd image?**
   - **Answer:** `mkinitrd`

2. **What tool is commonly used to create initramfs images on RHEL/Fedora-based systems?**
   - **Answer:** `dracut`

3. **What tool is commonly used to manage initramfs images on Ubuntu?**
   - **Answer:** `update-initramfs`

---

## Part 6 - Identify GRUB 2

GRUB 2 is commonly used as the Linux bootloader.

### Commands

```bash
grub-install --version
```

```bash
grub-mkconfig --version
```

Inspect the GRUB directory:

```bash
ls -lh /boot/grub/
```

### Questions

1. **What bootloader is commonly used by modern Linux distributions?**
   - **Answer:** GRUB 2.

2. **What does GRUB do?**
   - **Answer:** GRUB loads the Linux kernel and initramfs and provides a boot menu that can allow the user to select different operating systems or kernels.

3. **Where are Ubuntu's GRUB files commonly located?**
   - **Answer:** `/boot/grub/`

---

## Part 7 - Ubuntu GRUB Configuration

Ubuntu uses `/etc/default/grub` for administrator-configurable GRUB settings.

### Commands

```bash
cat /etc/default/grub
```

Look for settings such as:

```text
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
```

### Questions

1. **Where is the main Ubuntu GRUB configuration file?**
   - **Answer:** `/etc/default/grub`

2. **What does `GRUB_DEFAULT` control?**
   - **Answer:** It controls which GRUB menu entry is selected by default.

3. **What does `GRUB_TIMEOUT` control?**
   - **Answer:** It controls how long GRUB waits before automatically booting the default entry.

4. **What does `GRUB_CMDLINE_LINUX_DEFAULT` contain?**
   - **Answer:** Default kernel command-line parameters passed when booting Linux. In my case, "quiet splash"

---

## Part 8 - Generate the Ubuntu GRUB Configuration

After changing `/etc/default/grub`, the generated GRUB configuration needs to be updated.

### Commands

```bash
sudo update-grub
```

Then inspect the generated configuration:

```bash
ls -lh /boot/grub/grub.cfg
```

### Questions

1. **What command updates the GRUB configuration on Ubuntu?**
   - **Answer:** `update-grub`

2. **Where is the generated GRUB configuration stored on Ubuntu?**
   - **Answer:** `/boot/grub/grub.cfg`

3. **Should you normally edit `/boot/grub/grub.cfg` manually?**
   - **Answer:** No. Administrator settings should normally be made through `/etc/default/grub` or the appropriate GRUB configuration mechanisms, then the configuration should be regenerated.

---

## Part 9 - Generate GRUB Configuration with `grub-mkconfig`

Ubuntu also provides the lower-level GRUB configuration generator.

### Commands

```bash
sudo grub-mkconfig -o /tmp/grub.cfg
```

Inspect the generated file:

```bash
less /tmp/grub.cfg
```

Search for boot entries:

```bash
grep -n "menuentry" /tmp/grub.cfg
```

Search for the kernel:

```bash
grep -n "vmlinuz" /tmp/grub.cfg
```

Search for initramfs entries:

```bash
grep -n "initrd" /tmp/grub.cfg
```

### Questions

1. **What does `grub-mkconfig` do?**
   - **Answer:** It generates a GRUB configuration file based on the system's installed kernels, configuration, and GRUB scripts.

2. **What does `-o` specify?**
   - **Answer:** The output file.

---

## Part 10 - GRUB Advanced Options

Reboot the system:

```bash
sudo reboot
```

Display the GRUB menu during boot.

Select:

```text
Advanced options for Ubuntu
```

You may see multiple kernel versions.

### Questions

1. **Why might multiple Linux kernels be installed?**
   - **Answer:** A newer kernel can be installed while retaining older kernels so the system has an alternative boot option.

2. **What is the purpose of GRUB Advanced Options?**
   - **Answer:** It provides additional boot choices, including older installed kernels and recovery options when available.

3. **How can GRUB help if a newly installed kernel fails to boot?**
   - **Answer:** You can select an older working kernel from GRUB's Advanced Options.

4. **What is the Linux+ concept associated with using an older kernel?**
   - **Answer:** Kernel rollback.

---

## Part 11 - Kernel Rollback Scenario

### Scenario

A new kernel was installed on the server.

After rebooting, the system fails to boot correctly.

An older kernel is still installed.

### Exercise

Use GRUB to boot the previous kernel.

The general process is:

```text
GRUB menu
    ↓
Advanced options for Ubuntu
    ↓
Older kernel
    ↓
Boot
```

After successfully booting, verify the kernel:

```bash
uname -r
```

### Questions

1. **Where would you find older installed kernels during boot?**
   - **Answer:** GRUB → Advanced Options.

2. **What command verifies which kernel is currently running?**
   - **Answer:** `uname -r`

3. **Why is keeping an older kernel useful?**
   - **Answer:** It provides an alternative boot option if the newer kernel has a boot or compatibility problem.

---

## Part 12 - Network Boot

Linux systems can also boot from a network instead of local storage.

### Questions

1. **What does PXE stand for?**
   - **Answer:** Preboot Execution Environment.

2. **What is PXE used for?**
   - **Answer:** Booting a computer from network-provided boot resources.

3. **Why might an organization use network booting?**
   - **Answer:** It can allow systems to boot installation environments, deployment environments, or other boot resources without requiring a locally installed operating system.

---

## Part 13 - Memory Testing & Kernel Panics

### Concepts

Linux systems may provide memory-testing options through the boot menu.


A kernel panic indicates that the kernel encountered a condition from which it could not safely continue.

### Commands

Inspect kernel messages:

```bash
dmesg | less
```

Or:

```bash
journalctl -k
```

Search for relevant messages:

```bash
journalctl -k | grep -iE "panic|error|failed"
```

### Questions

1. **What tool is associated with memory testing?**
   - **Answer:** `memtest` / Memtest86-family tools.

2. **What is a kernel panic?**
   - **Answer:** A critical kernel failure that prevents the kernel from continuing normal operation.

3. **Can hardware problems cause a kernel panic?**
   - **Answer:** Yes. Faulty RAM, storage, or other hardware can contribute to kernel failures.

4. **Can software cause a kernel panic?**
   - **Answer:** Yes. Kernel bugs, faulty drivers, and other software-related problems can also cause kernel panics.

5. **What command can display kernel messages from the system journal?**
   - **Answer:** `journalctl -k`

---

## Part 14 - RHEL GRUB Reference

Ubuntu and RHEL-based systems use related GRUB concepts, but some commands and paths differ.

### Ubuntu

```text
/etc/default/grub
/boot/grub/grub.cfg
update-grub
grub-mkconfig
```

### RHEL

```text
/boot/grub2/grub.cfg
grub2-mkconfig
```

---

## Part 15 - Complete Boot Chain

Put the following components in the correct order:
```text
GRUB
UEFI
initramfs
Linux kernel
Firmware
Root filesystem
```

### Answer

```text
Firmware
    ↓
UEFI
    ↓
GRUB
    ↓
Linux kernel (vmlinuz)
    ↓
initramfs
    ↓
Root filesystem
```

> **Note:** This is a simplified conceptual model. The exact firmware/bootloader/kernel handoff varies depending on firmware mode and distribution configuration.

---

## Completion Checklist

- [x] Identify whether the system uses BIOS or UEFI
- [x] Identify the EFI System Partition
- [x] Explain BIOS and UEFI
- [x] Locate the Linux kernel in `/boot`
- [x] Explain `vmlinuz`
- [x] Locate and inspect the initramfs
- [x] Explain `initrd` and `initramfs`
- [x] Identify `mkinitrd` and `dracut`
- [x] Identify GRUB 2
- [x] Inspect `/etc/default/grub`
- [x] Run `update-grub`
- [x] Use `grub-mkconfig`
- [x] Locate `/boot/grub/grub.cfg`
- [x] Understand GRUB Advanced Options
- [x] Explain kernel rollback
- [x] Explain PXE
- [x] Explain iPXE
- [x] Understand the purpose of `memtest`
- [x] Explain kernel panic
- [x] Identify the RHEL GRUB paths and commands
