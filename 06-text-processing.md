# Lab 06 - Linux Text Processing Utilities

## Objective

Master essential Linux command-line text processing tools including `printf`, `sed`, `awk`, and pipeline redirectional operators for formatting, searching, substituting, and extracting structured data.

By completing this lab, I should be able to:

- Format output using escape characters with `printf`
- Perform basic and global text substitutions and line filtering using `sed`
- Extract specific fields from formatted text using `awk`
- Chain multiple text processing commands together using the pipe operator (`|`)

---

## Part 1 - Text Processing Tools Overview

- **`printf`**
  - **Purpose:** Formats and prints text output.

- **`sed`**
  - **Purpose:** Stream editor for filtering and transforming text.

- **`awk`**
  - **Purpose:** Pattern scanning and field processing language.

---

## Part 2 - Formatted Output (`printf`)

- **New line escape sequence:** `\n`
- **Tab escape sequence:** `\t`

### Common Usage
printf "Name\tScore\nAlice\t95\n"

--

## Part 3 - Stream Editing (`sed`)

- **Substitute first match on each line:** `s/old/new/`
- **Substitute all matches globally per line:** `s/old/new/g`
- **Delete line number N:** `Nd` (e.g., `sed '1d' file.txt` deletes line 1)
- **Print line number N:** `Np` (e.g., `sed -n '3p' file.txt` prints line 3)
- **Suppress default output printing:** `-n`
- **Edit file directly in-place:** `-i` (e.g., `sed -i 's/foo/bar/g' file.txt`)

---

## Part 4 - Field Processing (`awk`)

- **Default field separator:** Space (or tab)
- **Set custom field separator:** `-F` (e.g., `-F":"` for `/etc/passwd`)
- **Print specific field:** `{print $n}`
- **Print third field:** `{print $3}`

---

## Part 5 - Pipeline Redirection

- **Pipe operator:** `|`
- **Purpose:** Passes the standard output (`stdout`) of the command on the left as standard input (`stdin`) to the command on the right.

---

## Part 6 - Hands-On Practice Exercises

Execute the following commands sequentially in your terminal to practice these concepts:

1. **Create sample data file:**
printf "ID\tName\tRole\n1\tAlice\tAdmin\n2\tBob\tUser\n3\tCharlie\tUser\n" > users.txt
2. **Inspect file content:**
cat users.txt
3. **Extract the 2nd field (Names) using `awk`:**
awk '{print $2}' users.txt
4. **Substitute a role using `sed` without changing the file:**
sed 's/User/Client/g' users.txt
5. **Combine tools using a pipe (`|`):**
cat users.txt | sed '1d' | awk '{print $2 " is a " $3}'

---

## Completion Checklist

- [x] I can format output using `printf` escape characters (`\n`, `\t`)
- [x] I can substitute text inline or globally using `sed` (`s/old/new/`, `s/old/new/g`)
- [x] I know how to use `sed -i` for in-place file modifications
- [x] I can extract specific data columns using `awk` and specify custom field separators using `-F`
- [x] I can chain multiple utilities together using the pipe operator (`|`)
