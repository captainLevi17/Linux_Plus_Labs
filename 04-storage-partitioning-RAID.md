# Lab 04 - Storage, Partitioning, RAID, and Source Compilation

**OS:** Ubuntu / RHEL

**Commands Practiced:** `./configure`, `make`, `sudo make install`, `umount`, `sshfs`, `which`, `dnf`

## Overview & Objectives

Understand Linux storage paradigms, disk partitioning standards, FUSE filesystems, software RAID configurations, and the steps required to compile and install software from source code.

**Key Learning Outcomes:**

- Distinguish between File, Block, and Object storage types
- Identify MBR vs. GPT partitioning standards
- Utilize FUSE for userspace filesystems like `sshfs`
- Differentiate RAID levels (RAID 0, 1, 5) by capacity and tolerance
- Compile and install C/C++ packages from source on Linux distributions

## Part 1 - Storage Types & Concepts

- **File Storage:**
  - **Structure:** File + directory hierarchy
  - **Metadata:** Supported natively by filesystem

- **Block Storage:**
  - **Best For:** Databases and high-performance transactional data
  - **Metadata:** None at the block layer

- **Object Storage:**
  - **Access Method:** API (HTTP / REST)
  - **Identifier:** UUID
  - **Metadata:** Supported
  - **Example:** Amazon S3

## Part 2 - Partition Tables (MBR vs. GPT)

- **BIOS Partition Table:** MBR (Master Boot Record)
- **UEFI Partition Table:** GPT (GUID Partition Table)
- **GPT Backward Compatibility:** Protective MBR

## Part 3 - Filesystem in Userspace (FUSE)

- **FUSE Meaning:** Filesystem in Userspace
- **Key Capability:** Allows mounting filesystems without root privileges
- **SSH Filesystem Utility:** `sshfs`
- **Unmount Command:** `umount`

## Part 4 - Software RAID Overview

- **RAID Meaning:** Redundant Array of Independent Disks

- **RAID 0:**
  - **Layout:** Stripe
  - **Usable Space:** 100%
  - **Redundancy / Failure Tolerance:** None
  - **Primary Advantage:** Speed

- **RAID 1:**
  - **Layout:** Mirror
  - **Usable Space:** 50%
  - **Primary Advantage:** Redundancy

- **RAID 5:**
  - **Layout:** Striping + parity
  - **Usable Space:** N - 1 drives
  - **Drive Failures Tolerated:** 1
  - **Primary Advantage:** Speed + redundancy

## Part 5 - Hands-On Exercise: Compiling Software from Source

Practice compiling and installing **GNU Hello** (a standard open-source C program) from source code.

### Step 1: Install Required Build Tools

Before compiling software, ensure the system build utilities (`gcc`, `make`, `tar`, `wget`) are installed.

On Ubuntu / Debian:

sudo apt update && sudo apt install -y build-essential wget

On RHEL / CentOS / Fedora:

sudo dnf groupinstall -y "Development Tools" && sudo dnf install -y wget

### Step 2: Download and Extract Source Code

1. Navigate to the temporary directory:

cd /tmp

2. Download the source tarball:

wget https://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz

3. Extract the archive:

tar -xzf hello-2.10.tar.gz

4. Navigate into the source code directory:

cd hello-2.10

### Step 3: Configure, Compile, and Install

1. Check dependencies and generate the Makefile:

./configure

2. Compile the source code:

make

3. Install the compiled binary into `/usr/local/bin`:

sudo make install

### Step 4: Verify the Installation

1. Find where the system binary was installed:

which hello

*Expected Output:* `/usr/local/bin/hello`

2. Execute the compiled binary:

*Expected Output:* `Hello, world!`

### Step 5: Clean Up and Uninstall (Optional)

1. Uninstall the binary using the Makefile:

sudo make uninstall

2. Verify it has been removed:


```

## Part 6 - Command Reference

- **Locate executable binary path:** `which` (e.g., `which python3`)

## Completion Checklist

- [x] Distinguish between File, Block, and Object storage
- [x] Identify MBR vs. GPT partition schemes
- [x] Understand FUSE userspace capabilities
- [x] Compare RAID 0, 1, and 5 layouts and fault tolerance
- [x] Understand compilation steps (`./configure`, `make`, `sudo make install`)
