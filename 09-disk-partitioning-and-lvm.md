# 09. Disk Partitioning, Formatting & LVM (Ch-19, Ch-20)

> ⚠️ These commands change disks. Practice in a VM or on a scratch disk, and double-check device names with `lsblk` first.

## 1. Storage Concepts (Ch-19)

* Definitions: **Partition** (slice of a disk), **Filesystem** (data organization), **Mounting** (attaching to the directory tree).
* Steps: 1. Divide (`fdisk`) ➔ 2. Format (`mkfs.ext4`) ➔ 3. Mount (`mount`).
* Physical architecture (HDD): platters, heads (read/write), spindle, tracks/cylinders. SSDs have no moving parts.
* Partition limits:
  * **MBR** (Master Boot Record): max 4 primary partitions (or 3 primary + 1 extended holding logical partitions); disks up to 2 TiB.
  * **GPT** (GUID Partition Table): up to 128 partitions by default; disks beyond 2 TiB; required for UEFI boot.
* Device naming: `/dev/sda`, `/dev/sdb` (SCSI/SATA/USB), `/dev/nvme0n1` (NVMe, partitions `nvme0n1p1`), `/dev/vda` (virtio).

```bash
lsblk                         # Tree view of disks and partitions
lsblk -f                      # With filesystem type, label, UUID
sudo fdisk -l                 # List all partition tables
cat /proc/partitions          # Kernel's view of partitions
```

---

## 2. Partitioning Tools (Ch-20)

### Fdisk Interactive Commands

* `m`: Help menu
* `p`: Print the partition table
* `n`: New partition (`p` primary, `e` extended, `l` logical)
* `d`: Delete partition
* `t`: Change partition type code (`83` = Linux, `82` = Linux swap, `8e` = Linux LVM, `fd` = RAID)
* `l`: List known type codes
* `g` / `o`: Create new GPT / DOS (MBR) table
* `q`: Quit without saving
* `w`: Write changes to disk and exit

```bash
sudo fdisk /dev/sdb                      # Interactive partitioning
sudo fdisk -cu /dev/sda                  # Older fdisk: -u sectors, -c disable DOS compatibility
sudo fdisk -l /dev/sdb                   # List partitions non-interactively
sudo partprobe /dev/sdb                  # Ask kernel to re-read the partition table
```

### Scripted Partitioning & GPT Tools

```bash
# sfdisk (Scriptable Automation)
sudo sfdisk -s /dev/sda                  # Size of the disk (in KiB)
sudo sfdisk -d /dev/sda > sda.disk       # Back up the partition table to a file
sudo sfdisk /dev/sda < sda.disk          # Restore the partition table from the file

# gdisk (GPT Management; works on whole disks, not partitions)
sudo gdisk /dev/sdb                      # Interactive: n new, p print, v verify, o new empty GPT, w write

# GNU Parted
lsblk
sudo parted /dev/sdb print               # View existing partitions
sudo parted /dev/sdb mklabel msdos       # Create an MBR partition label (use 'gpt' for GPT)
sudo parted /dev/sdb mkpart primary ext4 0% 50%   # Primary partition using the first 50% of the disk
sudo parted /dev/sdb rm 1                # Remove partition 1
```

---

## 3. Logical Volume Management (LVM)

* Structure: Physical Volume (PV) ➔ Volume Group (VG) ➔ Logical Volume (LV).
* Benefits: resize volumes online, span multiple disks, snapshots.

```bash
# 1. Initialize PVs
sudo pvcreate /dev/sdb
sudo pvcreate /dev/sdc
sudo pvcreate /dev/sdd

# 2. Combine PVs into a Volume Group
sudo vgcreate vol1 /dev/sdb /dev/sdc /dev/sdd
sudo vgextend vol1 /dev/sde              # Add another PV to the VG

# 3. Create Logical Volumes
sudo lvcreate -L 200M -n logical_vol1 vol1     # 200 MB LV named logical_vol1 in VG vol1
sudo lvcreate -l 100%FREE -n data vol1         # Use all remaining free space

# 4. Format and mount the LV
sudo mkfs -t ext4 /dev/vol1/logical_vol1
sudo mkdir /mnt/lv1
sudo mount /dev/vol1/logical_vol1 /mnt/lv1
```

### ➕ Additional Essentials: Inspect, Resize & Remove LVM

```bash
sudo pvs  ;  sudo pvdisplay              # Physical volume summary / detail
sudo vgs  ;  sudo vgdisplay              # Volume group summary / detail
sudo lvs  ;  sudo lvdisplay              # Logical volume summary / detail

# Grow an LV and its filesystem in one step
sudo lvextend -L +100M -r /dev/vol1/logical_vol1   # -r resizes the filesystem too
sudo lvextend -l +100%FREE -r /dev/vol1/data

# Manual resize (ext4)
sudo lvextend -L +100M /dev/vol1/logical_vol1
sudo resize2fs /dev/vol1/logical_vol1
# XFS can only grow:  sudo xfs_growfs /mnt/lv1

# Shrink (ext4 only; unmount first, check, shrink fs, then LV)
sudo umount /mnt/lv1
sudo e2fsck -f /dev/vol1/logical_vol1
sudo resize2fs /dev/vol1/logical_vol1 150M
sudo lvreduce -L 150M /dev/vol1/logical_vol1

# Move data off a disk and remove it
sudo pvmove /dev/sdb
sudo vgreduce vol1 /dev/sdb
sudo pvremove /dev/sdb

# Snapshots
sudo lvcreate -s -L 50M -n snap1 /dev/vol1/logical_vol1

# Tear down
sudo lvremove /dev/vol1/logical_vol1
sudo vgremove vol1
```

---

## 4. Filesystems & Swap Setup

```bash
# Create Filesystems
sudo mkfs -t vfat /dev/sdb1              # FAT32 (also: mkfs.vfat)
sudo mkfs -t ext4 /dev/sdb1              # ext4 (also: mkfs.ext4)
sudo mkfs.xfs /dev/sdb1                  # XFS
sudo e2label /dev/sdb1 mydata            # Set an ext label

# Swap Partition
sudo fdisk /dev/sda                      # Press 't' and set type 82 (Linux swap)
sudo mkswap /dev/sda3                    # Format partition as swap
sudo swapon /dev/sda3                    # Enable swap partition
swapon -s                                # Display active swap devices (also: swapon --show)

# Swap File
sudo dd if=/dev/zero of=/var/extraswap bs=1M count=100   # Create a 100 MB file
sudo chmod 600 /var/extraswap            # Swap files must not be world-readable
sudo mkswap /var/extraswap               # Format file as swap
sudo swapon /var/extraswap               # Enable swap file
```
