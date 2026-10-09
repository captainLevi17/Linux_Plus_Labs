# Lab 07 - Command-Line Text Editors

## Objective

Master essential Linux command-line text editors including `nano`, `vi`, and `vim`, understanding their operational modes, shortcut conventions, and key differences across system environments.

By completing this lab, I should be able to:

- Identify the best text editor for specific Linux environments and skill levels
- Navigate, edit, save, and exit files using `nano`
- Transition between Command Mode and Insert Mode in `vi`/`vim`
- Use core `vi`/`vim` commands for editing, line deletion, saving, and quitting
- Recognize operational differences between traditional `vi` and `vim`

---

## Part 1 - Text Editors Overview & Defaults

- **Nano:** Beginner-friendly, easy-to-use terminal text editor
- **vi:** Legacy text editor installed almost everywhere on Unix/Linux systems
- **vim:** Improved version of `vi` (*Vi IMproved*) with enhanced features

### System Environment Defaults

- **Ubuntu Desktop default editor:** `vi`
- **Ubuntu Server default editor:** `vim`

---

## Part 2 - Using Nano

- **Open or create a file:** `nano <file>`
- **Save file (WriteOut):** `Ctrl+O`
- **Exit editor:** `Ctrl+X`
- **Cut line:** `Ctrl+K`
- **Paste line:** `Ctrl+U`
- **Show help:** `Ctrl+G`

---

## Part 3 - Using vi / vim

`vi` and `vim` are modal editors that operate primarily in two modes:

- **Command mode:** Default mode upon opening; used for navigation, line manipulation, saving, and quitting.
- **Insert mode:** Editing mode used for typing and modifying text content.

### Switching Modes

- **Enter Insert mode:** `i`
- **Return to Command mode:** `Esc`

### Saving and Exiting (from Command Mode)

- **Save file:** `:w`
- **Quit:** `:q`
- **Save and quit:** `:wq`
- **Quit without saving (force quit):** `:q!`

### Deletion Shortcuts (in Command Mode)

- **Delete character:** `dl` (or `x`)
- **Delete word:** `dw`
- **Delete entire line:** `dd`

---

## Part 4 - Key Differences: vi vs. vim

- **Syntax highlighting:** Supported in `vim` (No native highlighting in legacy `vi`)
- **Arrow key navigation in Insert mode:** Supported in `vim` (Not supported in legacy `vi`)

---

## Part 5 - Hands-On Practice Exercises

Execute the following commands sequentially in your terminal to practice using both editors:

1. **Practice with Nano:**
nano practice_nano.txt
- Type a few lines of text.
- Press `Ctrl+O` then `Enter` to save.
- Press `Ctrl+X` to exit.

2. **Practice with Vim:**
vim practice_vim.txt
- Press `i` to enter Insert Mode and type `Hello Linux!`.
- Press `Esc` to return to Command Mode.
- Type `dd` to delete the line you just wrote.
- Type `:wq` and press `Enter` to save and quit.

---

## Completion Checklist

- [x] I can create and edit files using `nano`
- [x] I can enter and exit Insert Mode in `vi`/`vim` using `i` and `Esc`
- [x] I know how to save and quit in `vi`/`vim` using `:w`, `:q`, `:wq`, and `:q!`
- [x] I can delete characters, words, and full lines in `vi`/`vim` using `dl`, `dw`, and `dd`
- [x] I understand the core feature differences between `vi` and `vim`
