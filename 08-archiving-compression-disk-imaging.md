# Lab 08 - Archiving, Compression, and Disk Imaging Utilities

## Objective

Master essential Linux command-line archiving, compression, and disk imaging tools, including `tar`, `gzip`, `bzip2`, `xz`, `zip`, `cpio`, and `dd`.

By completing this lab, I should be able to:

- Distinguish between archiving (bundling) and compression (reducing size)
- Create and extract compressed archives using `tar` with compression flags (`-z`, `-j`, `-J`)
- Compress and decompress files using `gzip`, `bzip2`, `xz`, and `zip`
- Preserve original files during compression using the `-k` option
- Create low-level disk images and raw copies using `dd`

---

## Part 1 - Core Concepts: Archiving vs. Compression

- **Archiving:** Combines multiple files and directories into one single archive file without reducing file size.
- **Compression:** Reduces file size using algorithms to save storage space and transmission bandwidth.

---

## Part 2 - High-Level Archiving & Compression (`tar`)

The `tar` (Tape Archive) utility is the standard tool for creating and extracting archives.

### Primary Flags
- **Create archive:** `-c`
- **Extract archive:** `-x`
- **Archive filename:** `-f` (Must immediately precede the archive name)

### Integrated Compression Flags
- **`tar` + `gzip` option:** `-z` (creates `.tar.gz` or `.tgz`)
- **`tar` + `bzip2` option:** `-j` (creates `.tar.bz2`)
- **`tar` + `xz` option:** `-J` (creates `.tar.xz` or `.txz`)

### Common `tar` Command Syntax

- **Create a gzipped tarball:**
tar -czvf archive.tar.gz /path/to/directory
- **Extract a gzipped tarball:**
tar -xzvf archive.tar.gz
---

## Part 3 - Single-File Compression Utilities

To retain the original file during compression with `gzip`, `bzip2`, or `xz`, use the **`-k`** (keep) option.

### `gzip`
- **Compress command:** `gzip <filename>`
- **Decompress option:** `gzip -d <filename.gz>`
- **Decompress alias command:** `gunzip <filename.gz>`
- **Extension:** `.gz`
- **Shorthand archive extension:** `.tgz` (equivalent to `.tar.gz`)

### `bzip2`
- **Compress command:** `bzip2 <filename>`
- **Decompress option:** `bzip2 -d <filename.bz2>`
- **Decompress alias command:** `bunzip2 <filename.bz2>`
- **Extension:** `.bz2`

### `xz`
- **Compress command:** `xz <filename>`
- **Decompress option:** `xz -d <filename.xz>`
- **Decompress alias command:** `unxz <filename.xz>`
- **Extension:** `.xz`
- **Shorthand archive extension:** `.txz` (equivalent to `.tar.xz`)

---

## Part 4 - Zip and Legacy Utilities

### `zip` & `unzip`
- **Compress recursively (include subdirectories):** `zip -r archive.zip /path/to/directory`
- **Extract archive:** `unzip archive.zip`

### `cpio`
- **Create archive:** `cpio -o`
- **Extract archive:** `cpio -i`

### `dd` (Data Duplicator / Disk Imaging)
- **Input file/device option:** `if=`
- **Output file/device option:** `of=`
- **Common use case:** Disk imaging, cloning drives, or creating bootable USB installation media.
- **Example usage:**

dd if=/dev/sda of=/path/to/disk_image.img status=progress

---

## Part 5 - Hands-On Practice Exercises

Execute the following commands sequentially in your terminal to practice these concepts:

1. **Create a test directory and file structure:**

cd /tmp
mkdir -p lab08_test/subdir
echo "Linux Archiving Test" > lab08_test/file1.txt
echo "Compression Test" > lab08_test/subdir/file2.txt

2. **Create a gzipped tar archive:**
tar -czvf lab08_test.tar.gz lab08_test

3. **Practice single-file compression and keep original file (`-k`):**
cp lab08_test/file1.txt .
gzip -k file1.txt
ls -l file1.txt*

4. **Decompress the `.gz` file:**
gunzip file1.txt.gz

5. **Extract the `.tar.gz` archive:**
tar -xzvf lab08_test.tar.gz -C /tmp

6. **Clean up:**
rm -rf lab08_test lab08_test.tar.gz file1.txt

---

## Completion Checklist

- [ ] I understand the difference between archiving and compression
- [ ] I can create and extract compressed tarballs using `-z`, `-j`, and `-J` flags
- [ ] I know how to compress and decompress files using `gzip`, `bzip2`, `xz`, and `zip`
- [ ] I know how to preserve the original file when compressing using the `-k` flag
- [ ] I understand the basic syntax of `dd` (`if=` and `of=`) for disk imaging

