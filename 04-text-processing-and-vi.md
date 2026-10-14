# 04. Text Processing, Filters & Vi Editor (Ch-7, Ch-8, Ch-9)

## 1. Slicing, Display & Character Translation (Ch-7)

```bash
# Viewing & Line Numbering
cat file.txt                 # Print a file
cat -n file.txt              # Number all output lines
nl -ba file.txt              # Number all lines, including blank ones
less largefile.txt           # Paginated viewing (q quits, /text searches)
tac file.txt                 # Print lines in reverse order

# Head & Tail Operations
head -n 2 myfile.txt         # First 2 lines (same as head -2)
head -n -2 myfile.txt        # Everything EXCEPT the last 2 lines
tail -n 2 myfile.txt         # Last 2 lines
tail -n +2 myfile.txt        # Everything from line 2 onward (skip the first line)
tail -f /var/log/syslog      # Follow a file as it grows

# Slicing & Joining
split -l 10 data.txt         # Split into chunks of 10 lines (xaa, xab, ...)
split -b 1M data.bin         # Split into 1 MB chunks
split -d words output        # Numeric suffixes with prefix 'output'
paste file1 file2            # Merge files side by side, line by line
join animals.txt numbers.txt # Join two sorted files on a shared field

# Extraction, Sorting & De-duplication
cut -d: -f1,3,4 /etc/passwd  # Delimiter ':', extract fields 1, 3, and 4
cut -c1-5 file.txt           # Extract characters 1-5 of each line
sort -t: -k1 /etc/passwd     # Delimiter ':', sort on key 1
sort -n numbers.txt          # Numeric sort
sort -r file.txt             # Reverse sort
sort -u output.txt           # Sort and remove duplicates
sort output | uniq -c        # Count occurrences (uniq needs sorted input)
sort output | uniq -d        # Show only duplicated lines

# Character Translation (tr)
echo 'aaaappleeeeeee' | tr -s 'ae'    # Squeeze repeated characters
cat /etc/hosts | tr 'a-z' 'A-Z'       # Lowercase to uppercase
tr -d '\r' < dos.txt > unix.txt       # Delete carriage returns (DOS to Unix)

# Stream Editing (sed)
sed 's/host/name/g' /etc/hosts        # Global find and replace
sed '/127.0.0.1/i\Report' /etc/hosts  # Insert 'Report' before matching lines
sed -i 's/pattern/replacement/' file  # Edit the file in place
sed -i.bak 's/old/new/g' file         # In place, keeping a .bak backup
sed '/localhost/d' /etc/hosts         # Delete lines matching localhost
sed -n '5,10p' file.txt               # Print only lines 5-10

# Metrics
wc -l animals.txt            # Line count
wc -w animals.txt            # Word count
wc -c animals.txt            # Byte count
wc -m animals.txt            # Character count
```

### ➕ Additional Essentials: `awk`, `diff`, `column`

```bash
awk '{print $1}' file.txt                 # Print the first whitespace-separated field
awk -F: '{print $1, $3}' /etc/passwd      # Custom delimiter; print user and UID
awk -F: '$3 >= 1000 {print $1}' /etc/passwd   # Conditional: regular users
awk '{sum += $1} END {print sum}' nums.txt    # Sum a column
diff file1 file2             # Show line differences
diff -u file1 file2          # Unified diff (patch-friendly)
cmp file1 file2              # Byte-by-byte comparison
comm file1 file2             # Compare two sorted files in three columns
column -t file.txt           # Align output into columns
rev file.txt                 # Reverse characters on each line
```

---

## 2. Pattern Matching with Grep (Ch-8)

* Metacharacters: `^` (line start), `$` (line end), `.` (any one char), `*` (0 or more of the previous), `[ ]` (character set).
* Extended regex (`grep -E` / `egrep`) adds `?` (0 or 1), `+` (1 or more), `|` (alternation), `( )` (groups), `{n,m}` (repetition).

```bash
grep '^hello' /etc/myfile.txt   # Lines beginning with 'hello'
grep 'World$' /etc/myfile.txt   # Lines ending with 'World'
grep 'abc*' file.txt            # 'ab' followed by zero or more 'c's
grep -v 'pattern' file.txt      # Invert match (non-matching lines)
grep -i 'pattern' file.txt      # Case-insensitive
grep -w 'word' file.txt         # Whole words only
grep -o 'word' file.txt         # Print only the matched text
grep -n 'pattern' file.txt      # Show line numbers
grep -c 'pattern' file.txt      # Count matching lines
grep -r 'pattern' /etc/         # Recursive search through a directory
grep -l 'pattern' *.txt         # List only names of files that match
grep -A2 -B2 'error' log.txt    # Show 2 lines after and before each match

fgrep 'test*' file.txt          # Fixed string, no regex (grep -F)
egrep 'abc+|def' file.txt       # Extended regex (grep -E)
```

---

## 3. Vi Text Editor Quick Reference (Ch-9)

```text
Modes:
  i       : Insert before cursor
  a       : Append after cursor
  A       : Append at end of line
  I       : Insert at beginning of line
  o / O   : Open a new line below / above
  ESC     : Return to Command mode
  :       : Enter Last-line (ex) mode

Directional Navigation (Command mode):
  h       : Left
  j       : Down
  k       : Up
  l       : Right
  w / b   : Next / previous word
  0 / $   : Start / end of line
  1G      : First line of file
  G       : Last line of file
  :25     : Go to line 25

Edit Commands:
  x       : Delete character
  dd      : Delete (cut) current line
  5dd     : Delete 5 lines
  dw      : Delete current word
  D       : Delete from cursor to end of line
  yy      : Yank (copy) current line
  3yy     : Yank 3 lines
  p       : Paste after cursor/line
  P       : Paste before cursor/line
  u       : Undo
  Ctrl+R  : Redo
  .       : Repeat last command

Search & Replace:
  /paste  : Search forward for 'paste'
  ?paste  : Search backward for 'paste'
  n / N   : Next / previous match
  :%s/old/new/g   : Replace all occurrences in the file

File Control:
  :w          : Save
  :w name     : Save as 'name'
  :q          : Quit
  :q!         : Quit without saving
  :wq  or ZZ  : Save and quit
  :wq!        : Write and quit forcefully
  :set nu     : Show line numbers
```
