# 08. Boot Process, Bootloaders & Runlevels (Ch-16, Ch-17, Ch-18)

## 1. Boot Stages (Ch-16)

1. **Firmware Stage**: BIOS/UEFI performs POST and locates the boot device (MBR on BIOS systems; EFI System Partition on UEFI).
2. **Bootloader Stage**: GRUB loads the kernel and `initramfs` into memory.
3. **Kernel Stage**: Hardware detection, driver initialization, root filesystem mount.
4. **Init Stage**: Kernel launches the first process (PID 1), the system initializer.

* `initramfs`: Temporary initial RAM filesystem containing drivers needed to mount the real root filesystem.
* Kernel boot logs: `journalctl -b` or `dmesg` show boot messages.
* Init system locations:
  * Upstart: config in `/etc/init`
  * SysVinit: `/sbin/init`, scripts in `/etc/init.d` and `/etc/rc*.d`
  * systemd: `/sbin/init` is usually a link to systemd; config in `/etc/systemd`

```bash
ps -p 1                       # Which init system is running as PID 1
ls -l /sbin/init              # Often a symlink to systemd
ls /boot                      # Kernels, initramfs images, GRUB files
uname -r                      # Running kernel version
```

### ➕ Additional Essentials: Boot Diagnostics

```bash
systemd-analyze               # Total boot time
systemd-analyze blame         # Time taken by each unit
journalctl -b                 # Logs from the current boot
journalctl -b -1              # Logs from the previous boot
journalctl -p err -b          # Only errors from this boot
cat /proc/cmdline             # Kernel command line used to boot
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS    # Detect firmware type
sudo dracut -f                # Rebuild initramfs (RHEL/Fedora)
sudo update-initramfs -u      # Rebuild initramfs (Debian/Ubuntu)
```

---

## 2. Bootloaders (Ch-17)

* Common bootloaders: GRUB (new version: GRUB 2) and LILO (Linux Loader, obsolete).
* GRUB Legacy stages:
  * Stage 1: Installed in the MBR.
  * Stage 1.5: Filesystem drivers (first ~30 KiB after the MBR).
  * Stage 2: Full bootloader providing the menu interface.
* Config file locations:
  * Fedora / RHEL: `/boot/grub2/grub.cfg`
  * Ubuntu / Debian: `/boot/grub/grub.cfg`
  * User-editable settings: `/etc/default/grub` and scripts in `/etc/grub.d/`
  * **Do not edit `grub.cfg` directly**; it is regenerated.

```bash
# Reinstall & Update
grub-install /dev/sda                  # Install/reinstall GRUB to a disk (Debian/Ubuntu)
grub2-install /dev/sda                 # Same on Fedora/RHEL (BIOS systems)
update-grub                            # Regenerate config on Ubuntu/Debian
grub2-mkconfig -o /boot/grub2/grub.cfg # Regenerate config on Fedora/RHEL

# Lab 17: Editing GRUB Default Settings (Ubuntu)
head /boot/grub/grub.cfg
sudo vi /etc/default/grub
# Configure:
#   GRUB_TIMEOUT_STYLE=menu
#   GRUB_TIMEOUT=30
sudo update-grub
reboot

# Lab 17: GRUB Legacy Configuration (/boot/grub/grub.conf or menu.lst)
cat /boot/grub/grub.conf
sudo vi /boot/grub/grub.conf
# Configure:
#   timeout=60
#   #hiddenmenu            (comment out to show the menu)
# At the GRUB menu: 'e' edit entry, 'b' boot, Ctrl+X boots the edited entry
```

### ➕ Additional Essentials: Recovery Basics

```text
At the GRUB menu: press 'e', append to the 'linux' line:
  single                      # Boot to single-user mode (SysV)
  systemd.unit=rescue.target  # systemd rescue mode
  init=/bin/bash              # Emergency shell (remount rw: mount -o remount,rw /)
Press Ctrl+X (or F10) to boot with the edit (not saved permanently).
```

```bash
sudo efibootmgr -v             # List/manage UEFI boot entries
```

---

## 3. Runlevels vs. Systemd Targets (Ch-18)

* System operational states (SysV):
  * `0`: Halt
  * `1`: Single-user mode
  * `2`: Multi-user (no networking; Debian: full multi-user)
  * `3`: Multi-user (with networking, text mode)
  * `4`: Unused / custom
  * `5`: GUI mode
  * `6`: Reboot
* `runlevel` or `who -r`: Display the current runlevel.
* `init 5` or `telinit 5`: Switch to GUI mode.

| Runlevel | systemd target |
| :--- | :--- |
| 0 | `poweroff.target` |
| 1 | `rescue.target` |
| 2, 3, 4 | `multi-user.target` |
| 5 | `graphical.target` |
| 6 | `reboot.target` |

```bash
systemctl get-default                    # Show the default target
sudo systemctl set-default multi-user.target   # Boot to text mode by default
sudo systemctl set-default graphical.target    # Boot to GUI by default
sudo systemctl isolate rescue.target     # Switch to single-user mode now
sudo systemctl isolate graphical.target  # Switch to GUI mode now
systemctl list-units --type=target       # Active targets
```

---

## 4. Service Control, Power Management & ACPI (Ch-18)

```bash
# Systemctl Service Management
systemctl status ssh                     # Status of the SSH service (sshd on RHEL)
sudo systemctl start httpd.service       # Start now
sudo systemctl stop httpd.service        # Stop now
sudo systemctl restart httpd.service     # Stop then start
sudo systemctl reload httpd.service      # Reload config without restart
sudo systemctl enable httpd.service      # Start automatically at boot
sudo systemctl disable httpd.service     # Do not start at boot
sudo systemctl enable --now httpd.service  # Enable and start together
systemctl is-active httpd.service        # active / inactive
systemctl is-enabled httpd.service       # enabled / disabled
sudo systemctl mask httpd.service        # Prevent any starting
systemctl list-units --all               # All loaded units
systemctl list-unit-files                # All installed unit files and state
systemctl --failed                       # Failed units
sudo systemctl daemon-reload             # Reload unit files after editing
sudo systemctl poweroff                  # Shut down
sudo systemctl reboot                    # Restart

# SysVinit & Init Scripts
chkconfig --list                         # Services across runlevels
chkconfig --list httpd                   # One service (scripts in /etc/rc.d/init.d)
sudo chkconfig httpd on                  # Enable in default runlevels (2-5)
sudo chkconfig httpd off                 # Disable
sudo /etc/init.d/sshd status             # Status of sshd
sudo /etc/init.d/sshd stop               # Stop sshd
sudo /etc/init.d/sshd start              # Start sshd
service sshd status                      # Wrapper that works on both systems

# Broadcast, ACPI & Shutdown
sudo wall testfile                       # Broadcast contents of a file to all terminals
echo "Maintenance at 10" | sudo wall     # Broadcast a message
ls -l /proc/acpi                         # ACPI devices and wakeup state
acpi -b                                  # Battery info (-a AC, -t thermal, -c cooling, -V all)
sudo shutdown now                        # Immediate shutdown
sudo shutdown -h +10 "Down in 10 min"    # Halt in 10 minutes with message
sudo shutdown -r now                     # Reboot immediately (or: init 6)
sudo shutdown -c                         # Cancel a pending shutdown
```

### ➕ Additional Essentials: Journals & Cron-like Timers

```bash
journalctl -u sshd                       # Logs for one unit
journalctl -u sshd -f                    # Follow logs live
journalctl --since "1 hour ago"          # Time-filtered logs
systemctl list-timers                    # systemd timers (cron replacement)
crontab -e                               # Edit your cron jobs
crontab -l                               # List your cron jobs
# Cron format: minute hour day-of-month month day-of-week command
#   30 2 * * 1  /usr/local/bin/backup.sh     (Mondays at 02:30)
```
