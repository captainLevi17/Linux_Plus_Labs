# Lab 39 - Linux File Permissions, Special Bits, ACLs, and File Attributes

## Objective

Master foundational POSIX file permissions, symbolic permission modification with `chmod`, special permission bits (`SUID`, `SGID`, and `Sticky bit`), extended Access Control Lists (`getfacl` and `setfacl`), and filesystem attributes (`lsattr` and `chattr`).

By completing this lab, I should be able to:

* Identify the three POSIX permission types (`r`, `w`, `x`) and three entity levels (`u`, `g`, `o`)

* Modify permissions symbolically using `chmod` (such as `chmod g-w,o+r test`)

* Explain and apply `SUID`, `SGID` (on files and directories), and the `Sticky bit`

* Inspect and modify extended ACL entries with `getfacl` and `setfacl -m`

* Inspect filesystem attributes with `lsattr` and safely clean up all test files, ACLs, and attributes for full replayability

## Part 1 - Standard POSIX Permissions and `chmod`

Linux file security builds upon three permission types across three entity levels:

* **Permission Types:**

  * **Read (`r`):** View file contents or list directory entries.

  * **Write (`w`):** Modify file contents or create/remove entries inside a directory (provided execute permission is also set on the directory).

  * **Execute (`x`):** Run an executable file or traverse/enter a directory (`cd` / access files inside).

* **Entity Levels:**

  * **User/Owner (`u`):** The user account that owns the file or directory.

  * **Group (`g`):** The group associated with the file or directory.

  * **Other (`o`):** All other users on the system who are neither the owner nor members of the owning group.

* **Symbolic Modification with `chmod`:**

  Combine entity levels, operators (`+` to add, `-` to remove, `=` to set exact), and permission letters separated by commas without spaces:

  ```
  chmod g-w,o+r test
  ```

  *(Removes write permission from the group and adds read permission for others on `test`).*

## Part 2 - Special Permission Bits (`SUID`, `SGID`, and Sticky Bit)

Beyond standard `rwx` bits, Linux provides three special mode bits:

* **Set User ID (`SUID`):**

  * When set on an executable file, the process runs with the privileges of the **file's owner** rather than the user invoking it.

  * Displayed as `s` (or `S` if execute is not set) in the owner's execute position (`rwsr-xr-x`).

* **Set Group ID (`SGID`):**

  * On an **executable**, runs the program with the **owning group's privileges**.

  * On a **directory**, causes new files/subdirectories created inside to automatically **inherit the directory's owning group** rather than the creator's primary group (displayed as `s` or `S` in the group's execute position, e.g., `rwxrwsr-x`).

* **Sticky Bit:**

  * Primarily applied to shared directories (such as `/tmp`).

  * Prevents users from deleting or renaming files owned by other users inside that directory, even if they have write access to the directory itself (displayed as `t` or `T` in the other's execute position, e.g., `rwxrwxrwt`).

## Part 3 - Access Control Lists (`getfacl` and `setfacl`)

Standard POSIX permissions restrict access to one owner, one group, and everyone else. **Access Control Lists (ACLs)** let you grant or restrict permissions for specific additional users or groups.

* **View ACL Entries (`getfacl`):**

  ```
  getfacl test
  ```

  *(Displays the file owner, owning group, base permissions, mask, and any named user/group ACL entries).*

* **Grant or Modify a User ACL Entry (`setfacl -m`):**

  ```
  setfacl -m u:susie:rwx test
  ```

ch  *(Modifies `test` so user `susie` explicitly receives `rwx` permissions and updates the effective ACL `mask` if needed).*

* **Remove Specific or All ACL Entries:**

  ```
  setfacl -x u:susie test
  setfacl -b test
  ```

  *(`-x` removes a specific entry; `-b` removes all extended ACL entries).*

## Part 4 - File Attributes (`lsattr` and `chattr`)

Filesystem attributes control low-level kernel/filesystem behaviors on supported filesystems (such as `ext4`), independent of standard `rwx` mode bits.

* **List File Attributes (`lsattr`):**

  ```
  lsattr test
  ```

  *(Displays attribute flags such as `i` for immutable, which prevents modification, deletion, renaming, or linking even by `root` until cleared).*

* **Set or Clear Attributes (`chattr`):**

  ```
  sudo chattr +i test
  sudo chattr -i test
  ```

  *(Use `+i` to set immutable and `-i` to remove immutable before deleting or modifying a protected file).*

## Part 5 - Hands-On Practice Lab

### Step 1: Practice Standard Permissions and `chmod`

1. Create a workspace and test file `test`:

   ```
   mkdir -p /tmp/lab39_work
   cd /tmp/lab39_work
   echo "initial content" > test
   chmod 0664 test
   ls -l test
   ```

2. Remove group write and add other read (or adjust to `chmod g-w,o+r test` after testing changes):

   ```
   chmod g-w,o+r test
   ls -l test
   ```

### Step 2: Test Special Directory Bits (`SGID` and Sticky Bit)

1. Create a shared directory and apply `SGID` (`2`) and `Sticky bit` (`1`) with mode `3770` (or `1777` / `2770`):

   ```
   mkdir -p /tmp/lab39_work/shared_dir
   chmod 2770 /tmp/lab39_work/shared_dir
   ls -ld /tmp/lab39_work/shared_dir
   ```

2. Apply the sticky bit to a world-writable test directory and verify the `t` bit:

   ```
   mkdir -p /tmp/lab39_work/sticky_dir
   chmod 1777 /tmp/lab39_work/sticky_dir
   ls -ld /tmp/lab39_work/sticky_dir
   ```

### Step 3: Configure and Inspect Extended ACLs

1. Grant user `susie` `rwx` permissions on `test` (if `susie` does not exist on your system, you can test against your current user `$USER` or create a temporary test account):

   ```
   sudo setfacl -m u:susie:rwx test
   getfacl test
   ```

### Step 4: Inspect File Attributes with `lsattr`

1. Check the attributes on `test` and `/tmp/lab39_work`:

   ```
   lsattr test
   lsattr -d /tmp/lab39_work
   ```

## Part 6 - Undo and Cleanup (Full Lab Replayability)

Execute these steps to clear any special attributes, strip ACLs, and remove all temporary lab files and directories cleanly.

### 1. Clear Attributes and Remove ACLs

1. Remove immutable or other test attributes if set:

   ```
   sudo chattr -i /tmp/lab39_work/test 2>/dev/null || true
   ```

2. Strip extended ACL entries from `test`:

   ```
   sudo setfacl -b /tmp/lab39_work/test 2>/dev/null || true
   ```

### 2. Remove Lab Workspace and Verify Cleanup

1. Return to your home directory and remove `/tmp/lab39_work`:

   ```
   cd ~
   rm -rf /tmp/lab39_work
   ```

2. Verify the workspace no longer exists:

   ```
   ls -ld /tmp/lab39_work 2>/dev/null || echo "Clean: /tmp/lab39_work removed"
   ```

## Completion Checklist

* [x] I can name the three POSIX permission types (`r`, `w`, `x`) and three entity levels (`u`, `g`, `o`)

* [x] I can modify permissions symbolically using `chmod` (such as `chmod g-w,o+r test`)

* [x] I can explain `SUID`, `SGID` (executable vs. directory group inheritance), and the `Sticky bit`

* [x] I can inspect ACLs with `getfacl` and grant user permissions with `setfacl -m u:susie:rwx test`

* [x] I can list file attributes using `lsattr` and clean up test artifacts for full replayability
