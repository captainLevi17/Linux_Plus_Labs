# Lab 41 - Bash Loops and Arithmetic (`while`, `until`, `for`, and Double Parentheses)

## Objective

Master Bash iteration constructs (`while`, `until`, and `for` loops), loop delimiters (`do` and `done`), wildcard directory iteration, and double-parentheses arithmetic evaluation (`(( ... ))` and `$(( ... ))`).

By completing this lab, I should be able to:

* Enclose loop bodies using `do` and `done`

* Differentiate `while` loops (run while condition is true) from `until` loops (run while condition is false / until true)

* Iterate sequentially over fixed lists, ranges, or wildcard file matches using `for` loops

* Evaluate integer arithmetic and increment variables using `(( variable++ ))` and `variable=$(( variable + 1 ))`

* Clean up temporary scripts and test directories for full lab replayability

## Part 1 - Loop Constructs and Delimiters (`do` and `done`)

Every `while`, `until`, and `for` loop in Bash encloses the commands executed on each iteration between the **`do`** and **`done`** keywords.

### `while` Loop (Execute While Condition Is True)

* **Behavior:** Evaluates the test condition before each pass and continues executing as long as the condition returns true (exit status `0`).

* **Syntax:**

  ```
  while [ condition ]; do
      commands
  done
  ```

### `until` Loop (Execute While Condition Is False)

* **Behavior:** Evaluates the test condition before each pass and continues executing as long as the condition is **false** (non-zero exit status), stopping once the condition becomes true.

* **Syntax:**

  ```
  until [ condition ]; do
      commands
  done
  ```

### `for` Loop (Iterate Over a Fixed List, Range, or Glob)

* **Behavior:** Steps sequentially through a fixed list of items, words, files, or numeric ranges, executing the `do ... done` block once per item.

* **Wildcard Directory Iteration Syntax:**

  ```
  for file in /path/to/dir/*; do
      commands
  done
  ```

## Part 2 - Arithmetic with Double Parentheses (`(( ... ))`)

Bash supports integer arithmetic directly inside double parentheses without needing external tools like `expr` or `bc`.

* **Increment / Arithmetic Statement (`(( ... ))`):**

  ```
  (( variable++ ))
  ```

  *(Evaluates the expression in arithmetic context; no `$` prefix is needed on variables inside `(( ... ))`).*

* **Arithmetic Expansion (`$(( ... ))`):**

  ```
  variable=$(( variable + 1 ))
  ```

  *(Evaluates the arithmetic expression and substitutes/assigns the resulting integer value).*

## Part 3 - Hands-On Practice Lab

### Step 1: Practice `while`, `until`, and Double-Parentheses Arithmetic

1. Create `/tmp/lab41_loops.sh`:

   ```
   cat << 'EOF' > /tmp/lab41_loops.sh
   #!/bin/bash

   count=1
   while (( count <= 3 )); do
       echo "while pass: count=$count"
       (( count++ ))
   done

   until (( count <= 1 )); do
       echo "until pass: count=$count"
       count=$(( count - 1 ))
   done
   EOF
   chmod +x /tmp/lab41_loops.sh
   ```

   * **Parameter notes:** `cat << 'EOF' > /tmp/lab41_loops.sh` (`<< 'EOF'` single-quotes the delimiter so Bash does not expand variables while writing the file; `>` writes/overwrites the target file). `chmod +x` adds execute permission so the script can be run directly.

2. Execute `/tmp/lab41_loops.sh`:

   ```
   /tmp/lab41_loops.sh
   ```

### Step 2: Iterate Over Files in a Directory Using a Wildcard `for` Loop

1. Create a sample directory with test files:

   ```
   mkdir -p /tmp/lab41_files
   touch /tmp/lab41_files/a.txt /tmp/lab41_files/b.txt /tmp/lab41_files/c.log
   ```

   * **Parameter notes:** `mkdir -p` creates `/tmp/lab41_files` and any missing parent directories without failing if it already exists.

2. Loop through all entries matching `/tmp/lab41_files/*`:

   ```
   for file in /tmp/lab41_files/*; do
       echo "Found: $file"
   done
   ```

## Part 4 - Undo and Cleanup (Full Lab Replayability)

Execute these steps to remove all temporary scripts and test directories created during the lab.

### 1. Remove Lab Artifacts and Verify Cleanup

1. Delete `/tmp/lab41_loops.sh` and `/tmp/lab41_files`:

   ```
   rm -rf /tmp/lab41_loops.sh /tmp/lab41_files
   ```

   * **Parameter notes:** `rm -rf` (`-r` removes directories recursively; `-f` avoids prompts or errors if an item is already missing).

2. Confirm both paths are gone:

   ```
   ls -ld /tmp/lab41_loops.sh /tmp/lab41_files 2>/dev/null || echo "Clean: Lab 41 artifacts removed"
   ```

   * **Parameter notes:** `ls -ld` (`-l` shows long format; `-d` lists directory entries themselves rather than their contents) and `2>/dev/null` discards missing-file error messages cleanly.

## Completion Checklist

* [x] I know `do` and `done` enclose the command block in `while`, `until`, and `for` loops

* [x] I can distinguish `while` (runs while true) from `until` (runs while false)

* [x] I can write a wildcard `for` loop (`for file in /path/to/dir/*; do ...; done`) to step through directory items

* [x] I can use `(( variable++ ))` and `variable=$(( variable + 1 ))` for integer arithmetic

* [x] I have removed all Lab 41 test scripts and directories for full replayability
