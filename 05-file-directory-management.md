# Lab 05 - File and Directory Management

## Objective

Master essential Linux terminal commands for filesystem navigation, file creation and removal, inspecting metadata, viewing directory trees, and working with hard vs. symbolic links.

By completing this lab, I should be able to:

- Navigate the Linux filesystem using standard path shortcuts
- Create, inspect, and safely remove files and directories
- Distinguish between Hard Links and Symbolic Links
- Differentiate between file timestamps (`atime`, `mtime`, `ctime`) using `stat`
- Inspect detailed file listings and directory tree structures

---

## Part 1 - Navigation & Directory Shortcuts

- **Print working directory:** `pwd`
- **Change directory:** `cd`
- **Home directory shortcut:** `~` (e.g., `cd ~`)
- **Current directory shortcut:** `.` (e.g., `cd .`)
- **Parent directory shortcut:** `..` (e.g., `cd ..`)

---

## Part 2 - Directory & File Manipulation

- **Create empty file (or update timestamp):** `touch <filename>`
- **Create directory:** `mkdir <dirname>`
- **Remove empty directory:** `rmdir <dirname>`
- **Remove file:** `rm <filename>`
- **Remove directory recursively:** `rm -r <dirname>`
- **Force remove (no prompts):** `rm -f <file_or_dir>`

---

## Part 3 - File Inspection & Metadata

- **Display directory tree:** `tree`
- **Show full file contents:** `cat <filename>`
- **Determine file type:** `file <filename>`
- **Display detailed file metadata:** `stat <filename>`

### Understanding File Timestamps (`stat` output)
- **`atime` (Access Time):** The last time the file was read or accessed.
- **`mtime` (Modify Time):** The last time the actual content of the file was modified.
- **`ctime` (Change Time):** The last time the file's metadata or permissions were changed.

---

## Part 4 - Directory Listing Options

- **List all files (including hidden files starting with `.`):** `ls -a`
- **Long listing format (permissions, ownership, size, date):** `ls -l`
- **Combine long listing with hidden files:** `ls -la`

---

## Part 5 - Links: Hard Links vs. Symbolic Links

- **Create hard link:** `ln <original_file> <link_name>`
- **Create symbolic (soft) link:** `ln -s <original_file> <link_name>`

### Key Differences Matrix

- **Hard Link**
  - **Points to:** Inode (direct disk data pointer)
  - **If original file is deleted:** Link still works (data remains accessible)

- **Symbolic Link (Symlink)**
  - **Points to:** Filename / File path
  - **If original file is deleted:** Link breaks (becomes a dangling pointer)

---

## Part 6 - Hands-On Practice Exercises

Execute the following commands sequentially in your terminal to practice these concepts:

1. **Navigate and create a test folder:**
cd ~
mkdir lab05_test
cd lab05_test
2. **Create a file and check its metadata:**
touch original.txt
file original.txt
stat original.txt
3. **Practice Links:**
ln original.txt hardlink.txt
ln -s original.txt symlink.txt
ls -l
4. **Test deleting the original file:**
rm original.txt
ls -l
*(Notice `symlink.txt` is now broken, but `hardlink.txt` still retains access to the data.)*

5. **Clean up:**
cd ..
rm -rf lab05_test

---

## Completion Checklist

- [x] I can navigate using `~`, `.`, and `..` shortcuts
- [x] I can create and safely remove files and directories
- [x] I understand the difference between `ls -a` and `ls -l`
- [x] I know how to check file metadata using `stat` and identify `atime`, `mtime`, and `ctime`
- [x] I understand how Hard Links differ from Symbolic Links when the original file is deleted
