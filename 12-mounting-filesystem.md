# Lab 12 - Filesystem Mounting, Persistence, and Disk Encryption

## Objective

Master essential Linux storage management skills for mounting filesystems, configuring persistent boots via `/etc/fstab`, managing systemd mount units, and securing storage partitions using LUKS disk encryption.

By completing this lab, I should be able to:

- Mount and unmount filesystems dynamically using `mount` and `umount`
- Inspect block devices, UUIDs, and filesystems using `blkid`, `lsblk -f`, and `df -h`
- Configure persistent mounts in `/etc/fstab` with proper options and `fsck` pass numbers
- Manage systemd mount units for automated mounting
- Encrypt partitions, manage mappings, and create filesystems using `cryptsetup` and `mkfs.ext4`

---

## Part 1 - Dynamic Mounting & Unmounting

- **Mount a filesystem:**
mount
- **Unmount a filesystem:**
umount
- **Mount all filesystems listed in `/etc/fstab`:**
mount -a
- **Specify a filesystem type explicitly:**
mount -t
---

## Part 2 - Persistent Mount Configuration (`/etc/fstab`)

To ensure filesystems mount automatically on system boot, entries are configured in system configuration files.

- **System mount configuration file:** `/etc/fstab`
- **Encrypted device configuration file:** `/etc/crypttab`

### Fstab Identifiers and Options
- **Reference device by UUID (recommended):** `UUID=<uuid>`
- **Reference device by label:** `LABEL=<label>`
- **Default mount options (rw, suid, dev, exec, auto, nouser, async):** `defaults`

### Filesystem Check (Pass Number / `fsck` order)
The final field in `/etc/fstab` specifies the order for filesystem checks during boot:
- **Root filesystem (`/`):** `1`
- **Non-root filesystems:** `2`
- **Disable filesystem check:** `0`

---

## Part 3 - Storage & Filesystem Inspection

- **Show block device UUIDs and filesystem types:** `blkid`
- **Show basic block devices:** `lsblk`
- **Show block devices with associated filesystems and UUIDs:** `lsblk -f`
- **Show mounted filesystems and disk space usage:** `df -h`

---

## Part 4 - Systemd Mount Units

Modern Linux distributions convert `/etc/fstab` entries into systemd mount units, or allow manual creation of unit files.

- **Mount unit location:** `/etc/systemd/system`
- **Generated mount units location:** `/run/systemd/generator`
- **Mount unit filename convention:** `<mountpoint>.mount` (e.g., `mnt-data.mount`)

### Systemctl Commands for Mount Units
- **Reload systemd manager configuration:** 
systemctl daemon-reload

- **Start a mount unit immediately:** 
systemctl start <mountname>.mount

- **Enable a mount unit to start on boot:** 
systemctl enable <mountname>.mount


---

## Part 5 - LUKS Disk Encryption & Formatting

To protect sensitive data at rest, partitions can be encrypted using Linux Unified Key Setup (LUKS).

- **Encrypt a partition with LUKS:** 
cryptsetup luksFormat <device path>

- **Open an encrypted device (maps it to a decrypted block device):** 
cryptsetup open <device path> <name>

- **Decrypted device path:** `/dev/mapper/<name>`
- **Create an ext4 filesystem on the decrypted device:** 
mkfs.ext4 /dev/mapper/<name>

- **Close/lock an encrypted device:** 
cryptsetup close </dev/mapper/<name>


---

## Part 6 - Hands-On Practice Exercises

Execute the following commands sequentially in your terminal to inspect mounts and practice configurations:

1. **Inspect block devices, UUIDs, and current mounts:**
lsblk -f
df -h


2. **Check a partition UUID for fstab configuration:**
blkid /dev/sda1


3. **Practice creating an encrypted loop device and formatting it:**
sudo dd if=/dev/zero of=/tmp/encrypted_disk.img bs=1M count=100
sudo losetup -fP /tmp/encrypted_disk.img

(Assume loop device is /dev/loop0, partition /dev/loop0p1 or disk itself)
sudo cryptsetup luksFormat /dev/loop0
sudo cryptsetup open /dev/loop0 secure_vol
sudo mkfs.ext4 /dev/mapper/secure_vol
sudo cryptsetup close secure_vol


---

## Completion Checklist

- [x] I can mount and unmount filesystems using `mount` and `umount`
- [x] I know how to use `lsblk -f` and `blkid` to retrieve UUIDs for persistent mounting
- [x] I understand how to configure `/etc/fstab` using `UUID=`, `defaults`, and proper `fsck` pass numbers
- [x] I know how systemd manages mounts through `.mount` units and `systemctl`
- [x] I can encrypt, open, format (`mkfs.ext4`), and close LUKS partitions using `cryptsetup`
