# 03. File Management & Links (Ch-5, Ch-14)

## 1. File Operations & Directory Management (Ch-5)

```bash
# Directory Listings (ls)
ls -a                        # Show all hidden dot files
ls -l                        # Detailed long listing
ls -d testdir/               # Show the directory itself, not its contents
ls -S                        # Sort by size
ls -r                        # Reverse order
ls -t                        # Sort by last modification time
ls -lh                       # Long listing with human-readable sizes
ls -R                        # Recursive listing

# File Identification & Creation
file /etc/hosts              # Inspect file content type
touch csnt                   # Create empty file or update timestamp
touch ~/Documents/hello1.txt ~/Documents/hello2.txt   # Create multiple files
touch -t 202101021435 testfile   # Set timestamp: [[CC]YY]MMDDhhmm[.ss]
stat testfile                # Detailed metadata: size, inode, permissions, times

# Copy, Move & Remove
cp file1 file2               # Copy a file
cp -r csnt ~/Documents/      # Recursive copy (directories and contents)
cp -a csnt ~/Documents/      # Archive copy (preserves timestamps, permissions, links)
cp -i file1 file2            # Prompt before overwriting
mv testfile ~/Documents      # Move or rename a file
mv old.txt new.txt           # Rename
rm file                      # Remove a file
rm -i file                   # Prompt before removing
rm -r testdir                # Remove a directory and its contents
rm -rf testdir               # Force-remove without prompts (dangerous!)
mkdir testdir                # Create a directory
mkdir -p a/b/c               # Create nested directories in one step
rmdir testdir                # Remove an EMPTY directory only
```

> ⚠️ `rmdir` has no `-r` option. To delete a directory that has contents, use `rm -r`.

### ➕ Additional Essentials: Viewing & Disk Layout

```bash
tree -L 2                    # Directory tree, 2 levels deep
basename /etc/hosts          # Output: hosts
dirname /etc/hosts           # Output: /etc
readlink -f /etc/rc.local    # Resolve symlink to its real path
shred -u secret.txt          # Overwrite then delete a file
```

---

## 2. Links: Soft (Symbolic) vs. Hard Links (Ch-14)

* **Soft Link**: A pointer to another file's pathname (`ln -s`); can cross filesystems and link to directories; breaks if the target is removed.
* **Hard Link**: An additional directory entry pointing to the same inode (`ln`); shares data blocks, cannot cross filesystems, and cannot link directories; survives removal of the original name.
* `ls -i`: Display inode numbers.

```bash
# Symbolic (Soft) Links
ln -s /tmp/hosts /var/hosts  # Create soft link /var/hosts -> /tmp/hosts
ls -l /etc/rc.local          # Inspect symlink arrow and target
find . -xtype l              # Find broken symlinks

# Hard Links
mkdir /test                  # Create test directory
cp /etc/crontab /test        # Copy file
cd /test
ln crontab hcrontab          # Create hard link sharing the same inode

# Verify Inodes
ls -i                        # crontab and hcrontab show the same inode number
ls -l                        # Link count increases to 2
stat crontab                 # Shows Inode and Links fields
```

### Soft vs. Hard at a Glance

| Feature | Soft link | Hard link |
| :--- | :--- | :--- |
| Command | `ln -s target link` | `ln target link` |
| Own inode | Yes | No (same inode) |
| Cross filesystems | Yes | No |
| Link to directory | Yes | No |
| If original deleted | Link breaks (dangling) | Link still works |
