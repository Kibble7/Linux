# 01. System Information & Hardware Diagnostics (Ch-2, Ch-15)

## 1. System Architecture (`uname`)

Display parameters about the running kernel and operating environment.

| Option | Description |
| :--- | :--- |
| `-a` | All system information |
| `-s` | Kernel name (e.g., `Linux`) |
| `-n` | Network nodename / hostname |
| `-r` | Kernel release |
| `-v` | Kernel build version |
| `-m` | Machine hardware architecture (e.g., `x86_64`) |
| `-p` | Processor type |
| `-i` | Hardware platform |
| `-o` | Operating system name |

```bash
uname -a
```

### ➕ Additional Essentials: OS & Host Identity

```bash
cat /etc/os-release          # Distribution name and version
hostnamectl                  # Hostname, OS, kernel, virtualization info
hostname                     # Print hostname
uptime                       # Uptime and load averages
who -b                       # Last boot time
date                         # Current date/time
timedatectl                  # Time, timezone, NTP sync status
```

---

## 2. Hardware Subsystems & Bus Topologies (Ch-15)

* `usb-devices | tail -n 15`: View connected USB device topology and parameters.
* `tree`: Show a hierarchical tree of files and directories.
* `sysfs`: Virtual filesystem mounted at `/sys`, exposing kernel hardware attributes and configuration.
* `HAL`: Deprecated hardware abstraction layer; replaced by `udev` and `D-Bus` (`lshal` used to list device attributes).
* `udev`: Manages dynamic device nodes in `/dev` via configured rules.
* `D-Bus`: Inter-process communication and hardware event notifications between components.
* `systemd`: Integrates with `udev` for device event handling.

```bash
# CPU Metrics
lscpu                        # CPU architecture overview
cat /proc/cpuinfo            # Detailed core-by-core processor parameters
nproc                        # Number of processing units available

# Memory Metrics
free                         # Free, total, and used memory
free -h                      # Same, human-readable
cat /proc/meminfo            # Detailed memory statistics

# USB & PCI Bus Inspection
lsusb                        # USB devices attached to the system
lsusb -v -d 0624:0248 | less # Verbose details for a vendor:product ID
lspci                        # Devices on the PCI bus
lspci -nn                    # PCI devices with numeric vendor/device codes
lspci -k                     # PCI devices with the kernel driver in use
```

### ➕ Additional Essentials: Hardware Inventory

```bash
sudo dmidecode -t system     # System manufacturer/model/serial (BIOS/DMI)
sudo dmidecode -t memory     # RAM slots and module details
sudo dmidecode -t bios       # BIOS vendor and version
sudo lshw -short             # Concise hardware summary
lsblk                        # Block devices (disks, partitions)
lsscsi                       # SCSI/SATA devices
sensors                      # Temperatures/fans (package: lm-sensors)
```

---

## 3. Kernel Modules (Lab 15)

```bash
lsmod | less                 # Currently loaded kernel modules
modinfo dm_mod | less        # Metadata and parameters for dm_mod
lsmod | grep fat             # Check whether fat/vfat modules are loaded
sudo modprobe vfat           # Load the vfat module (and dependencies)
sudo modprobe -r fat         # Remove the fat module from memory
```

### ➕ Additional Essentials: Kernel & Logs

```bash
dmesg | tail                 # Recent kernel ring-buffer messages
dmesg -T | grep -i usb       # Kernel messages with readable timestamps
sudo insmod ./mod.ko         # Insert a module file directly (no dependency resolution)
sudo rmmod mod               # Remove a module directly
cat /proc/version            # Kernel version string
sysctl -a | less             # All runtime kernel parameters
sudo sysctl -w net.ipv4.ip_forward=1   # Change a parameter at runtime
```
