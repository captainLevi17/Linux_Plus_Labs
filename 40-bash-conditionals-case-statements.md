# Lab 40 - Bash Variables, Environment Inheritance, Conditionals, and Case Statements

## Objective

Master Bash shell variables, unidirectional environment variable inheritance between parent and child processes, variable removal with `unset`, interactive input via `read`, compound string/conditional checks inside `[[ ]]`, and pattern matching with `case` statements.

By completing this lab, I should be able to:

* Export shell variables to child processes and subshells using `export`

* Explain why child processes and subshells cannot alter variables in a parent shell process

* Remove variables or functions from the current shell environment using `unset`

* Prompt for and capture interactive user input in scripts using `read`

* Combine conditions inside `[[ ]]` using `||` and write `case ... in ... ;; ... *) ... ;; esac` blocks

* Clean up test variables, scripts, and temporary files for full lab replayability

## Part 1 - Environment Variables, Inheritance, and `unset`

### Exporting Variables to Child Processes

* **Local vs. Exported Variables:** Assigning `LAB_VAR="hello"` creates a shell variable visible only in the current shell. To make it available to child processes and subshells, export it:

  ```
  export LAB_VAR="hello"
  
  ```

  *(Or assign first, then run `export LAB_VAR`).*

### Unidirectional Process Inheritance

* **Parent-to-Child Only:** Process environment inheritance in Linux is **unidirectional**. A parent process passes exported environment variables down to child processes and subshells, **but a child script or subshell cannot modify or set an environment variable in its parent shell**—when the child exits, its modified environment disappears.

* **Exception (`source` / `.`):** Sourcing a script (`source script.sh` or `. script.sh`) runs commands inside the **current** shell rather than a child process, so variable assignments stay in the current shell.

### Removing Variables with `unset`

* **Clear a Variable or Function:**

  ```
  unset LAB_VAR
  
  ```

  *(Removes `LAB_VAR` from the current shell environment so subsequent commands or child processes no longer see it).*

## Part 2 - Interactive Input (`read`) and Conditionals (`[[ ]]` and `case`)

### Interactive Input with `read`

* **Capture User Input:**

  ```
  read -r user_choice
  
  ```

  *(Pauses execution in a script or shell, reads a line from standard input, and assigns it to `user_choice`; `-r` prevents backslashes from being interpreted as escape characters).*

### Logical OR inside Double Brackets (`[[ ]]`)

* **Combining Conditions with `||`:**

  ```
  if [[ "$user_choice" == "yes" || "$user_choice" == "y" ]]; then
      echo "Confirmed"
  fi
  
  ```

  *(Inside `[[ ]]`, `||` acts as a logical OR between conditional expressions).*

### `case` Statement Syntax and Structure

* **Structure:**

  ```
  case "$variable" in
      pattern1)
          command_one
          ;;
      pattern2)
          command_two
          ;;
      *)
          default_command
          ;;
  esac
  
  ```

* **Key Syntax Rules:**

  * Begins with `case <expression> in` and closes with **`esac`** (`case` spelled backward).

  * Each pattern branch ends with a **double semicolon (`;;`)** to terminate that pattern's command block.

  * An **asterisk (`*`)** acts as the default catch-all wildcard pattern matching any value not matched by earlier patterns.

## Part 3 - Hands-On Practice Lab

### Step 1: Test `export`, Child Process Isolation, and `unset`

1. Set a local variable and an exported variable in your current shell:

   ```
   LOCAL_ONLY="local_val"
   export SHARED_VAL="parent_val"
   
   ```

2. Run a child `bash -c` subcommand that reads both variables and attempts to modify `SHARED_VAL`:

   ```
   bash -c 'echo "Child sees LOCAL_ONLY=[$LOCAL_ONLY] SHARED_VAL=[$SHARED_VAL]"; SHARED_VAL="changed_in_child"'
   echo "Parent still has SHARED_VAL=[$SHARED_VAL]"
   
   ```

   *(Notice `LOCAL_ONLY` is empty in the child, and `SHARED_VAL` in the parent remains `parent_val`).*

3. Remove `SHARED_VAL` and `LOCAL_ONLY` using `unset`:

   ```
   unset SHARED_VAL LOCAL_ONLY
   echo "After unset: LOCAL_ONLY=[$LOCAL_ONLY] SHARED_VAL=[$SHARED_VAL]"
   
   ```

### Step 2: Write and Run a Script Using `read`, `[[ ]]` (`||`), and `case`

1. Create a test script `/tmp/lab40_menu.sh`:

   ```
   cat << 'EOF' > /tmp/lab40_menu.sh
   #!/bin/bash
   
   printf "Enter action (start/stop/status/y): "
   read -r action
   
   if [[ "$action" == "y" || "$action" == "yes" ]]; then
       echo "Quick affirmative matched via [[ ... || ... ]]"
   fi
   
   case "$action" in
       start)
           echo "Starting service..."
           ;;
       stop)
           echo "Stopping service..."
           ;;
       status|y|yes)
           echo "Checking status..."
           ;;
       *)
           echo "Unknown input [$action] matched by default (*) pattern"
           ;;
   esac
   EOF
   chmod +x /tmp/lab40_menu.sh
   
   ```

2. Execute `/tmp/lab40_menu.sh` and test both a matching branch (such as `start` or `y`) and an unmatched value (such as `foobar`) to observe the `*` catch-all branch:

   ```
   /tmp/lab40_menu.sh
   
   ```

## Part 4 - Undo and Cleanup (Full Lab Replayability)

Execute these steps to clear any leftover test variables and remove temporary lab scripts so the environment is clean for replay.

### 1. Unset Test Variables

1. Ensure any test variables from the lab are unset in your current shell:

   ```
   unset LOCAL_ONLY SHARED_VAL action user_choice 2>/dev/null || true
   
   ```

### 2. Remove Temporary Lab Script and Verify Cleanup

1. Delete `/tmp/lab40_menu.sh` and confirm it is gone:

   ```
   rm -f /tmp/lab40_menu.sh
   ls -l /tmp/lab40_menu.sh 2>/dev/null || echo "Clean: /tmp/lab40_menu.sh removed"
   
   ```

## Completion Checklist

* [x] I can use `export` to make variables visible to child processes and subshells

* [x] I understand that environment inheritance is unidirectional and child processes cannot alter parent shell variables

* [x] I can remove shell variables or functions using `unset`

* [x] I can read interactive user input into a variable using `read`

* [x] I can use `||` inside `[[ ]]` for logical OR conditions and write `case` statements closed with `esac`, `;;` terminators, and `*` catch-all patterns

* [x] I have cleaned up test variables and temporary scripts for full lab replayability
