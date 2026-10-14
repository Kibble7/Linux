# 10. Filesystem Administration Labs & Superblock Recovery (Ch-21, Ch-22, Ch-23)

> ⚠️ Labs below overwrite data. Run them only on a scratch disk or VM.

## 1. Capacity & Device Inspection (Ch-21, Ch-22)

```bash
df -h                         # Human-readable free disk space
df -hT                        # Plus filesystem types
df -i                         # Inode usage per filesystem

du -h /bin                    # Space used by /bin (per subdirectory)
du -s /usr                    # Summary size of /usr
du -sh /*                     # Summary for each top-level directory
du /etc | sort -nr | head     # Largest items under /etc
du -h --max-depth=1 /var | sort -h   # Largest subdirectories, human-sorted

lsblk -a                      # All block devices
lsblk -f                      # With filesystems and UUIDs
blkid                         # Show UUIDs, labels, and types
lsof /mnt                     # Open files on a mount point
lsof                          # All open files
fuser -vm /mnt                # Which processes are using a mount
fuser -km /mnt                # Kill processes using the mount
mount                         # Show mounted filesystems
findmnt                       # Tree view of mounts
```

---

## 2. Lab 21: Partitioning, Mounting & Swap Administration

### Create & Mount an Ext4 Partition

```bash
sudo fdisk /dev/sdb
# Commands in fdisk:
#   n -> (type, e.g. l for logical) -> (first sector: Enter) -> (last sector: +200M) -> p -> w
sudo partprobe /dev/sdb       # Re-read table without reboot (or reboot)
ls /dev/sd*                   # Confirm /dev/sdb5 exists

sudo mkfs -t ext4 /dev/sdb5   # Format as ext4
sudo mkdir /data              # Create mount point
sudo mount /dev/sdb5 /data    # Mount
mount | grep /data            # Verify mount

# Configure a persistent mount in /etc/fstab
sudo nano /etc/fstab
# Add the line:
#   /dev/sdb5   /data   ext4   defaults   1   2
tail -n 1 /etc/fstab
sudo umount /data
sudo mount /data              # Mounts using the fstab entry
sudo mount -a                 # Mount everything in fstab (also tests the file for errors)
```

#### `/etc/fstab` Field Reference

```text
<device>     <mount point>  <type>  <options>  <dump>  <pass>
/dev/sdb5    /data          ext4    defaults   0       2
UUID=xxxx    /data          ext4    defaults   0       2     # Preferred: stable across reboots
LABEL=myswap none           swap    sw         0       0
# dump: 0 = no backup; pass: 1 = root fs, 2 = other fs, 0 = skip fsck
```

### Dedicated Swap Partition Lab

```bash
sudo fdisk /dev/sdb
# Commands in fdisk:
#   n -> Enter -> Enter -> t -> 6 -> 82 -> p -> w     (6 = partition number)
sudo partprobe /dev/sdb
ls /dev/sd*

sudo mkswap -L myswap /dev/sdb6   # Format swap with the label 'myswap'
free                              # Check swap metrics
sudo swapon /dev/sdb6             # Activate the swap partition

# Persistent /etc/fstab entry:
#   LABEL=myswap   none   swap   sw   0   0
tail -n 1 /etc/fstab

swapon -s                         # Verify active swap
sudo swapoff /dev/sdb6            # Deactivate
swapon -s
sudo swapon -a                    # Activate all swap entries from /etc/fstab
swapon -s
```

### Dedicated Swap File Lab

```bash
df -h
sudo dd if=/dev/zero of=/var/swapfile bs=1M count=100   # Allocate 100 MB
ls -lh /var/swapfile
sudo chmod 600 /var/swapfile
sudo mkswap /var/swapfile         # Format as swap
sudo swapon /var/swapfile         # Activate

# Persistent /etc/fstab entry:
#   /var/swapfile   none   swap   sw   0   0
tail -n 1 /etc/fstab

swapon -s
sudo swapoff /var/swapfile        # Deactivate
swapon -s
sudo swapon -a                    # Re-activate from /etc/fstab
swapon -s
```

---

## 3. Lab 22: Inode & Reserved Block Tuning

```bash
sudo dumpe2fs /dev/sda1 | head                                   # Filesystem details
sudo dumpe2fs /dev/sda1 | grep "Free inodes" | head -1
sudo dumpe2fs /dev/sda1 | grep "Default mount options"

sudo dumpe2fs /dev/sda1 | grep "Reserved block count"
sudo tune2fs -m 10 /dev/sda1                                     # Reserved blocks: default 5% -> 10%
sudo dumpe2fs /dev/sda1 | grep "Reserved block count"            # Verify the change
```

### ➕ Additional Essentials: `tune2fs` & Resizing

```bash
sudo tune2fs -l /dev/sda1            # List superblock info (like dumpe2fs -h)
sudo tune2fs -L mydata /dev/sdb5     # Set label
sudo tune2fs -c 30 /dev/sdb5         # Check every 30 mounts
sudo tune2fs -i 1m /dev/sdb5         # Check every 1 month
sudo tune2fs -O ^has_journal /dev/sdb5   # Remove the journal (ext3/4 -> ext2 behaviour)
sudo mkfs -t ext4 -i 4096 /dev/sdb5  # More inodes at creation (bytes-per-inode ratio)
sudo resize2fs /dev/sdb5             # Grow ext fs to fill its partition
```

---

## 4. Lab 23: Superblock Corruption & Recovery Lab

```bash
# Check a Filesystem (NEVER run fsck on a mounted filesystem!)
sudo umount /dev/sdb5
sudo fsck /dev/sdb5
sudo fsck -f /dev/sdb5               # Force a check even if marked clean
sudo fsck -y /dev/sdb5               # Auto-answer yes to repairs

# Superblock Corruption Simulation
sudo dumpe2fs /dev/sdb5 | grep -i superblock   # Backup superblock locations
sudo dd if=/dev/zero of=/dev/sdb5 bs=1024 count=1 seek=1   # Overwrite primary superblock
sudo mount /dev/sdb5 /data           # Mount fails because of corruption
dmesg | tail                         # Kernel error log

# Superblock Recovery Procedure
sudo mke2fs -n /dev/sdb5             # DRY RUN (-n): lists backup superblocks (e.g., 8193)
sudo fsck -b 8193 /dev/sdb5          # Repair using backup superblock 8193
# or: sudo e2fsck -b 8193 /dev/sdb5

sudo mount /dev/sdb5 /data           # Remount repaired filesystem
ls /data/lost+found                  # Inspect recovered orphaned files
```

> ⚠️ `mke2fs -n` is safe only with `-n`. Without it, `mke2fs` creates a new filesystem and destroys data.

### ➕ Additional Essentials: Other Filesystems

```bash
sudo xfs_repair /dev/sda3            # Repair XFS (unmounted)
sudo xfs_repair -n /dev/sda3         # Check only, no changes
sudo xfs_info /mnt                   # XFS geometry
sudo xfs_growfs /mnt                 # Grow a mounted XFS filesystem
sudo btrfs check /dev/sdb1           # Btrfs check (unmounted)
sudo fsck.vfat -a /dev/sdb1          # Repair a FAT filesystem
sudo badblocks -sv /dev/sdb          # Scan a disk for bad blocks (read-only test)
sudo smartctl -a /dev/sda            # SMART health report (package: smartmontools)
```
