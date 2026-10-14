# 02. Shell Environment & Navigation (Ch-2, Ch-3, Ch-4, Ch-6)

## 1. Aliases & Manuals (Ch-2)

```bash
alias l='ls -ar'             # Create alias so 'l' runs 'ls -ar'
alias                        # List all aliases
unalias l                    # Remove an alias

man passwd                   # Read manual page for a command
man 5 passwd                 # Read section 5 (file formats) of the page
man -k passwd                # Search man pages by keyword (apropos)
man -f passwd                # Exact-name search (whatis)
whatis passwd                # Same output as man -f passwd
apropos passwd               # Same as man -k
passwd --help                # Quick built-in usage help
# Inside man pages: '/' searches forward, '?' backward, 'n' next match, 'q' quits
```

---

## 2. Variables & Execution History (Ch-3)

* **Local Variables**: Confined to the current shell session.
* **Environment Variables**: Exported to child processes using `export`.

```bash
# Scoping & Inspection
name="bob"                   # Define local variable (no spaces around =)
echo $name                   # Outputs: bob

export AGE=25                # Define environment variable
echo $AGE                    # Outputs: 25

set                          # All variables and functions (local and environment)
env                          # Environment variables only
printenv HOME                # Print a single environment variable
unset name                   # Delete local variable
unset AGE                    # Delete environment variable

# Execution Path
which ls                     # Resolve which binary runs (e.g., /bin/ls)
echo $PATH                   # Directories searched for commands
export PATH=$PATH:/opt/bin   # Append a directory to PATH (current session)

# Shell History Shortcuts
history                      # View previously executed commands
!!                           # Re-execute the previous command
!-4                          # Re-execute the command run 4 lines back
!123                         # Re-execute history entry number 123
!ssh                         # Re-execute most recent command starting with 'ssh'
sudo !!                      # Re-run previous command with sudo
history -c                   # Clear history list from memory
history -w                   # Write current history list to the history file
history -r                   # Read history file and merge into active history
# Ctrl+R                     # Reverse-search history interactively
```

### ➕ Additional Essentials: Persistence & Startup Files

```bash
# Make variables/aliases permanent by adding them to a startup file:
#   ~/.bashrc          Interactive non-login shells (aliases, functions)
#   ~/.bash_profile    Login shells (PATH, exports)
#   /etc/profile       System-wide login settings
#   /etc/environment   System-wide environment variables
source ~/.bashrc             # Reload a startup file in the current shell
echo $SHELL                  # Current login shell
cat /etc/shells              # Valid login shells
```

---

## 3. Globbing & Wildcards (Ch-4)

* `*`: Matches zero or more characters.
* `?`: Matches exactly one character.
* `[abc]`: Matches any one listed character; `[a-z]` is a range.
* `[^abc]` or `[!abc]`: Matches any character **not** listed.
* `{a,b,c}`: Brace expansion (not globbing, but often combined with it).

```bash
echo D?*                     # 'D' followed by at least one more character
echo [PMT]*                  # Names starting with P, M, or T
echo [^a-m]*                 # Names NOT starting with a through m
touch file{1..5}.txt         # Brace expansion: file1.txt ... file5.txt
```

### ➕ Additional Essentials: Quoting & Navigation

```bash
echo "$HOME"                 # Double quotes: variables are expanded
echo '$HOME'                 # Single quotes: literal text
echo $(date)                 # Command substitution
pwd                          # Print working directory
cd ~                         # Home directory
cd -                         # Previous directory
cd ..                        # Parent directory
```

---

## 4. Locating Files & Identifying Commands (Ch-6)

```bash
locate hostname              # Fast database index search
sudo updatedb                # Refresh the locate database

# Real-time Traversal with find
find /etc/ -name hostname    # Search /etc/ for exact name
find ~/ -type d 2> /dev/null # Find directories, discarding errors
find ~/ -empty 2> /dev/null  # Find empty files/dirs, discarding errors

# Command Resolution
whereis find                 # Binaries, man pages, and sources
whereis -b touch             # Binaries only
whereis -m find              # Manual pages only
whereis -s find              # Source files only

type less                    # Nature of a command (alias, builtin, file)
type -t less                 # Output the type word only
type -a less                 # Show all instances of the command
```

### ➕ Additional Essentials: More `find` Power

```bash
find / -name "*.log" -type f 2>/dev/null        # Files matching a pattern
find /var -size +100M                           # Files larger than 100 MB
find /home -user bob                            # Files owned by a user
find . -mtime -7                                # Modified in the last 7 days
find . -mmin -30                                # Modified in the last 30 minutes
find / -perm -4000 -type f 2>/dev/null          # All SUID files (security audit)
find . -name "*.tmp" -delete                    # Delete matches
find . -name "*.txt" -exec grep -l "error" {} \; # Run a command on each match
```
