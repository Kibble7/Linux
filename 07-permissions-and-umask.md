# 07. Permissions, Ownership, Umask & Special Bits (Ch-13)

## 1. Standard Permissions & Ownership

* Octal values: `4` = Read, `2` = Write, `1` = Execute, `0` = None.
* Target categories: `u` = User/Owner, `g` = Group, `o` = Other, `a` = All.
* Modifiers: `+` (add), `-` (remove), `=` (exact assignment).

```text
-rwxr-xr--  1 bob team 120 Jan 1 12:00 file
 │└┬┘└┬┘└┬┘
 │ │  │  └─ other: r--  (4)
 │ │  └──── group: r-x  (5)
 │ └─────── user:  rwx  (7)
 └───────── type: - file, d directory, l symlink
```

```bash
chmod 761 /srv/lab.txt       # User: rwx (7), Group: rw- (6), Other: --x (1)
chmod o+rw /srv/lab.txt      # Grant read and write to other
chmod u=rwx,g=rx,o= file     # Exact assignment using symbolic mode
chmod -R 755 /srv/web        # Recursive change
ls -l /srv/lab.txt           # Verify

# Ownership & Group Management
chown bob file               # Change owner
chown bob:team file          # Change owner and group
chown -R bob:team /srv/data  # Recursive
grep team /etc/group         # Look up the group entry for 'team'
chgrp team /srv/test         # Change group ownership of a file/directory
```

### ➕ Additional Essentials: Users, Groups & `sudo`

```bash
id                           # Your UID, GID, and groups
id bob                       # Same for another user
whoami                       # Current user name
groups bob                   # Groups of a user
who                          # Logged-in users
last                         # Login history

sudo useradd -m -s /bin/bash bob         # Create user with home and shell
sudo passwd bob                          # Set password
sudo usermod -aG team bob                # Add user to a supplementary group (-a is crucial)
sudo usermod -L bob                      # Lock account (-U unlocks)
sudo userdel -r bob                      # Delete user and home directory
sudo groupadd team                       # Create a group
sudo groupdel team                       # Delete a group
sudo chage -l bob                        # Password aging info

su - bob                     # Switch user with login environment
sudo -l                      # List your sudo privileges
sudo visudo                  # Safely edit /etc/sudoers
getent passwd bob            # Look up account entry
# Key files: /etc/passwd, /etc/shadow, /etc/group
```

---

## 2. Umask Calculations

* Directory base permission: `777` (`rwxrwxrwx`).
* File base permission: `666` (`rw-rw-rw-`); new files never get execute automatically.
* Applied permission: Base with the umask bits **removed** (bitwise, not strictly subtraction).

| Umask | Directories | Files |
| :--- | :--- | :--- |
| `022` | `755` (`rwxr-xr-x`) | `644` (`rw-r--r--`) |
| `027` | `750` (`rwxr-x---`) | `640` (`rw-r-----`) |
| `002` | `775` (`rwxrwxr-x`) | `664` (`rw-rw-r--`) |
| `077` | `700` (`rwx------`) | `600` (`rw-------`) |

```bash
umask                        # Show current umask
umask -S                     # Show in symbolic form
umask 002                    # Set umask to 002 (current shell only)
mkdir mydir2                 # Created with 775
touch myfile2                # Created with 664
ls -ld mydir2                # Verify (drwxrwxr-x)
# Make permanent: add 'umask 027' to ~/.bashrc or /etc/profile
```

---

## 3. Special Permission Bits

* **SetUID (SUID)**: Executable runs with the **owner's** privileges (Octal `4000`, `u+s`). Example: `/usr/bin/passwd`.
* **SetGID (SGID)**: Executable runs with the **group's** privileges; on directories, new files inherit the directory's group (Octal `2000`, `g+s`).
* **Sticky Bit**: In a directory, users can only delete or rename files they own (Octal `1000`, `o+t`). Example: `/tmp`.

```bash
# SUID
chmod u+s lab.txt            # Enable SUID
chmod 4755 lab.txt           # SUID via octal (4000 + 755)
chmod u-s lab.txt            # Remove SUID

# SGID
chmod g+s /srv/shared        # Enable SGID (octal 2000)
chmod 2775 /srv/shared       # SGID + rwxrwxr-x

# Sticky Bit
chmod o+t /srv/public        # Enable Sticky bit (octal 1000)
chmod 1777 /srv/public       # Sticky + rwxrwxrwx (like /tmp)

ls -l /usr/bin/passwd        # SUID shows as 's' in the owner execute slot
ls -ld /tmp                  # Sticky shows as 't' in the other execute slot
# Capital S/T means the bit is set but execute is NOT set.
```

---

## 4. ➕ Additional Essentials: ACLs & Attributes

```bash
getfacl file                         # View ACL
setfacl -m u:bob:rw file             # Grant bob rw on a file
setfacl -m g:team:rx /srv/data       # Grant group team r-x
setfacl -x u:bob file                # Remove bob's ACL entry
setfacl -b file                      # Remove all ACLs
setfacl -d -m g:team:rwx /srv/data   # Default ACL inherited by new files

lsattr file                          # List extended attributes
sudo chattr +i file                  # Make immutable (cannot be modified or deleted)
sudo chattr -i file                  # Remove immutability
```
