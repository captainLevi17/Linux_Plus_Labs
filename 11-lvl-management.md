# Lab 11 - Linux Logical Volume Manager (LVM)

## Objective

Master the core concepts and commands of the Linux Logical Volume Manager (LVM), enabling flexible disk space management through Physical Volumes (PVs), Volume Groups (VGs), and Logical Volumes (LVs).

By completing this lab, I should be able to:

- Understand the LVM architecture hierarchy (PV -> VG -> LV)
- Initialize and inspect Physical Volumes using `pvcreate` and `pvdisplay`
- Create, extend, and manage Volume Groups using `vgcreate`, `vgextend`, and `vgdisplay`
- Create, resize, extend, activate, and deactivate Logical Volumes using `lvcreate`, `lvextend`, and `lvchange`
- Understand LVM operational features (online expansion, device paths, and RAID backing storage)

---

## Part 1 - LVM Architecture Hierarchy

LVM abstracts physical storage, allowing flexible partition resizing and management.

- **Physical Volume (PV):** A physical disk or disk partition (e.g., `/dev/sdb1`) initialized for LVM.
- **Volume Group (VG):** A pool or collection of one or more Physical Volumes combined together.
- **Logical Volume (LV):** A virtual block device created out of a Volume Group's storage pool, formatted with a filesystem.
- **LVM Device Path:** `/dev/<vg_name>/<lv_name>`

---

## Part 2 - Physical Volume (PV) Management

- **Initialize a disk/partition as an LVM Physical Volume:**

pvcreate

- **Display detailed information about Physical Volumes:**

pvdisplay

---

## Part 3 - Volume Group (VG) Management

- **Create a Volume Group from one or more PVs:**

vgcreate <vg_name> <pv_device...>

- **Display Volume Group information:**

vgdisplay

- **Extend a Volume Group by adding a new Physical Volume:**

vgextend <vg_name> <new_pv_device>

---

## Part 4 - Logical Volume (LV) Management & Resizing

- **Create a Logical Volume with a specified size and name:**

lvcreate -L <size>  -n <lv_name> <vg_name>

*(Example: `lvcreate -L 10G -n mylv myvg`)*
- **Display Logical Volume information:**

lvdisplay

- **Extend a Logical Volume:**

lvextend -L +<size> /dev/<vg_name>/<lv_name>

- **Extend a Logical Volume AND automatically resize its filesystem:**

lvextend -r -L +<size> /dev/<vg_name>/<lv_name>

*(The `-r` flag resizes the underlying filesystem online).*

---

## Part 5 - Activating and Deactivating Logical Volumes

- **Deactivate a Logical Volume:**

lvchange -an /dev/<vg_name>/<lv_name>

- **Activate a Logical Volume:**

lvchange -ay /dev/<vg_name>/<lv_name>

---

## Part 6 - Key Characteristics and Best Practices

- **Data Redundancy:** LVM itself **does not provide data redundancy** (use RAID for redundancy).
- **Online Expansion:** Supports resizing and extending volumes while they are mounted and in use.
- **Recommended Backing Storage:** It is recommended to use **RAID** underneath LVM for hardware-level reliability.

---

## Part 7 - Hands-On Practice Exercises

*(Note: These commands require root privileges and unassigned disk partitions or loop devices).*

1. **Initialize a partition as a Physical Volume:**

sudo pvcreate /dev/sdb1
pvdisplay

2. **Create a Volume Group using the new PV:**

sudo vgcreate vg_data /dev/sdb1
vgdisplay

3. **Create a 5GB Logical Volume:**

sudo lvcreate -L 5G -n lv_app vg_data
lvdisplay

4. **Extend the Logical Volume by an additional 2GB and resize the filesystem:**

sudo lvextend -r -L +2G /dev/vg_data/lv_app

---

## Completion Checklist

- [x] I understand the difference between PVs, VGs, and LVs
- [x] I can initialize physical volumes using `pvcreate`
- [x] I can create and extend volume groups using `vgcreate` and `vgextend`
- [x] I can create and expand logical volumes using `lvcreate` and `lvextend -r`
- [x] I know how to activate and deactivate LVM devices using `lvchange`
