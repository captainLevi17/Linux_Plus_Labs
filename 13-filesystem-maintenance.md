# Lab 13 - Filesystem Creation, Maintenance, and Tuning

## Objective

Master essential Linux storage administration tasks for creating ext4 and XFS filesystems, repairing and checking filesystems, inspecting metadata, configuring tuning parameters, and resizing volumes online or offline.

By completing this lab, I should be able to:

- Create ext4 and XFS filesystems using `mkfs` utilities
- Diagnose, check, and repair corrupted filesystems using `fsck` and `xfs_repair`
- Tune ext filesystem parameters, labels, and check intervals using `tune2fs` and `e2label`
- Inspect deep filesystem metadata using `dumpe2fs` and `xfs_info`
- Scan for physical hardware block issues using `badblocks`
- Resize ext and XFS filesystems dynamically using `resize2fs` and `xfs_growfs`

---

## Part 1 - Filesystem Creation (`mkfs`)

- **Create an ext4 filesystem:**
mkfs.ext4

- **Create an XFS filesystem:** 
mkfs.xfs


---

## Part 2 - Filesystem Checking and Repair (`fsck` & `xfs_repair`)

*Note: Never run `fsck` on a mounted, write-active filesystem, as it can cause catastrophic data corruption.*

- **Check and repair a filesystem:** 
fsck

- **Force a filesystem check (even if marked clean):** 
fsck -f

- **Automatically repair errors without prompting:** 
fsck -y

- **Repair an XFS filesystem (operates on unmounted filesystems):** 
xfs_repair


---

## Part 3 - Filesystem Tuning & Metadata Inspection (ext filesystems)

- **Show all ext filesystem settings and superblocks:** 
tune2fs -l

- **Set an ext filesystem label:** 
tune2fs -L

- **Set the mount count threshold before a forced `fsck` runs:** 
tune2fs -c

- **Set a time-based `fsck` interval (e.g., `30d`, `6w`):** 
tune2fs -i

- **Display comprehensive ext filesystem metadata and block groups:** 
dumpe2fs

- **Display or set ext filesystem labels directly:** 
e2label 

---

## Part 4 - Filesystem Tuning & Inspection (XFS filesystems)

- **Display XFS filesystem layout and configuration (requires a mounted path):** 
xfs_info

- **Set an XFS filesystem label:** 
xfs_admin -L


---

## Part 5 - Identification & Bad Block Scanning

- **Display filesystem UUIDs:** 
blkid

- **Display block devices and their associated filesystems:** 
lsblk -f

- **Scan a device for bad physical sectors/blocks:** 
badblocks


---

## Part 6 - Filesystem Resizing Matrix

- **Filesystem UUID Purpose:** A unique system-generated identifier used to prevent mount errors if drive letters shift.
- **Filesystem Label Purpose:** A human-readable identifier assigned to a volume.

### Resizing Capabilities and Commands

- **ext2 / ext3 / ext4 Online Resize Support:** Yes (can be resized while mounted)
- **XFS Shrink Support:** **No** (XFS filesystems can never be shrunk, only grown)
- **Resize an ext filesystem:** 
resize2fs

- **Grow a mounted XFS filesystem:** 
xfs_growfs


---

## Part 7 - Hands-On Practice Exercises

Execute the following commands sequentially in your terminal to practice these concepts:

NOTE: This lab assumes that you already have spare partitions and devices to play with preferable on a VM to reduce the risk of breaking your own system.

1. **Check block devices and existing filesystems:**
lsblk -f


2. **Create and tune an ext4 filesystem on a test volume:**
sudo mkfs.ext4 /dev/sdb1
sudo tune2fs -L "DataVolume" /dev/sdb1
sudo tune2fs -l /dev/sdb1 | grep "Filesystem volume name"

3. **Install Required XFS Tools on Ubuntu/Debian systems**
sudo apt update && sudo apt install -y xfsprogs

4. **Format the Disk with XFS**
sudo mkfs.xfs /dev/sdb2

5. **Inspect the Filesystem UUID**
sudo blkid /dev/sdb2

6. **Mount and Inspect the XFS Filesystem**
	***Create a mount point directory:***
	sudo mkdir -p /mnt/xfs_lab

	*** Mount the XFS filesystem:***
	sudo mount /dev/sdb2

	*** View the XFS layout configuration using xfs_info: ***
	sudo xfs_info /mnt/xfs_lab

	*** Check space usage: ***
	df -h /mnt/xfs_lab

---

## Completion Checklist

- [x] I can format block devices with ext4 (`mkfs.ext4`) and XFS (`mkfs.xfs`)
- [x] I know how to check and safely repair filesystems using `fsck` and `xfs_repair`
- [x] I understand how UUIDs and Labels differ and why they matter in `/etc/fstab`
- [x] I know how to expand ext4 filesystems with `resize2fs` and XFS filesystems with `xfs_growfs`
