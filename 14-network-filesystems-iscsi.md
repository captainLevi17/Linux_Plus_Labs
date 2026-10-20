# Lab 14 - Network Filesystems and iSCSI Storage

## Objective

Master essential Linux network storage and enterprise connectivity techniques, including mounting remote filesystems via NFS and SMB/CIFS, and configuring a complete block storage SAN environment using iSCSI across multiple virtual machines.

By completing this lab, I should be able to:

- Install and configure NFS and SMB/CIFS network share services and mount them on clients
- Understand the core architectural differences between remote file storage (NFS/SMB) and remote block storage (iSCSI)
- Build an iSCSI storage target on an Ubuntu server using `targetcli`
- Discover, log into, and manage iSCSI block storage targets from a client system using `iscsiadm`

---

## Part 1 - Network Filesystems (NFS & SMB/CIFS)

Network filesystems allow remote directories to be accessed over a network as local directories.


### Mounting Commands & Syntax

- **Mount an NFS share:**
  ```bash
  sudo mount -t nfs <server>:<share> <mountpoint>
  ```
- **Mount an SMB/CIFS share (interactive password):**
  ```bash
  sudo mount -t cifs //<server>/<share> <mountpoint> -o username=<user>,password=<pass>
  ```
- **Mount an SMB/CIFS share using a secure credentials file:**
  ```bash
  sudo mount -t cifs //<server>/<share> <mountpoint> -o credentials=<file>
  ```
  *(Note: The credentials file contains `username=...` and `password=...` on separate lines).*

---

## Part 2 - Remote Block Storage via iSCSI (VM Lab Guide)

iSCSI allows transmission of SCSI commands over IP networks, presenting remote storage arrays as local **block devices** (like physical hard drives).

- **iSCSI provides:** Remote block storage (allowing you to format partitions with ext4/XFS and mount them locally).
- **Core configuration file:** `/etc/iscsi/iscsid.conf`
- **Auto-login configuration setting:** `node.startup=automatic`

---

## Part 3 - Hands-On VM Practice Lab: Ubuntu Target & Kali Client

Practice building a live SAN storage environment using two VMs (The one tested for this lab are both Ubuntu-based systems)

### Step 1: Set Up the Ubuntu iSCSI Storage Server
1. **Install target utilities on the storage server:**
   ```bash
   sudo apt update && sudo apt install -y targetcli-fb
   ```
2. **Launch the configuration shell:**
   ```bash
   sudo targetcli
   ```
3. **Configure a block-backed storage object (using a raw partition or file) and target:**
   ```text
   /backstores/block create name=disk1 dev=/dev/sdc1
   /iscsi create iqn.2026-10.com.example:storage.target01
   /iscsi/iqn.2026-10.com.example:storage.target01/tpg1/luns create /backstores/block/disk1
   /iscsi/iqn.2026-10.com.example:storage.target01/tpg1 set attribute authentication=0 demo_mode_write_protect=0 generate_node_acls=1
   saveconfig
   exit
   ```
4. **Open the firewall port:**
   ```bash
   sudo ufw allow 3260/tcp
   ```

### Step 2: Connect from the Linux Client
1. **Install iSCSI initiator tools on the client:**
   ```bash
   sudo apt update && sudo apt install -y open-iscsi
   sudo systemctl enable --now open-iscsi
   ```
2. **Discover the target on your Ubuntu server IP:**
   ```bash
   sudo iscsiadm --mode discovery --type sendtargets --portal <UBUNTU_IP_ADDRESS>
   ```
3. **Log in to the target:**
   ```bash
   sudo iscsiadm --mode node --targetname iqn.2026-10.com.example:storage.target01 --portal <UBUNTU_IP_ADDRESS> --login
   ```
4. **Verify and use the block device locally on the client machine:**
   ```bash
   lsblk
   sudo mkfs.ext4 /dev/sdb
   sudo mkdir -p /mnt/remote_storage
   sudo mount /dev/sdb /mnt/remote_storage
   ```

### Step 3: Clean Disconnect & Teardown
- **On the client machine (Log out):**
  ```bash
  sudo umount /mnt/remote_storage
  sudo iscsiadm --mode node --targetname iqn.2026-10.com.example:storage.target01 --portal <UBUNTU_IP_ADDRESS> --logout
  ```
- **On Ubuntu Server (If you need to clear the device lock to mount locally again):**
  ```bash
  sudo targetcli
  /> /iscsi delete iqn.2026-10.com.example:storage.target01
  /> /backstores/block delete disk1
  /> saveconfig
  /> exit
  ```

---

## Completion Checklist

- [x] I can install and mount NFS shares using `mount -t nfs` and `nfs-common`
- [x] I can mount SMB/CIFS shares using `cifs-utils` and credentials files
- [x] I understand the structural difference between remote file storage (NFS/SMB) and remote block storage (iSCSI)
- [x] I can configure an Ubuntu iSCSI storage target using `targetcli`
- [x] I can discover, log into, format, and disconnect remote block storage targets using `iscsiadm`
