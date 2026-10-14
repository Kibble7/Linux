# 🐧 Linux Administration & Engineering Cheatsheet Repository

A structured command and lab reference built from hands-on Linux administration coursework (Chapters 2–23) and standard engineering references.

---

## 📖 Reference Literature

* **Title**: *Linux Pocket Guide: Essential Commands* (4th Edition, 20th Anniversary Release)
* **Author**: Daniel J. Barrett
* **Publisher**: O'Reilly Media, Inc., 2024
* **ISBN**: 978-1-098-15796-8

---

## 📑 Module Index

| # | Module | Covers |
| :-- | :--- | :--- |
| 01 | [System Information & Hardware](01-system-info-and-hardware.md) | `uname`, CPU/RAM diagnostics, `lsusb`, `lspci`, `dmidecode`, kernel modules, sysfs, udev |
| 02 | [Shell Environment & Navigation](02-shell-environment-and-navigation.md) | Aliases, manuals, variables, `PATH`, history, globbing, `find`, `which`/`type` |
| 03 | [File Management & Links](03-file-management-and-links.md) | `ls`, `cp`, `mv`, `rm`, `mkdir`, `stat`, inodes, soft vs. hard links |
| 04 | [Text Processing & Vi](04-text-processing-and-vi.md) | `cut`, `sort`, `uniq`, `tr`, `sed`, `awk`, `grep`, vi |
| 05 | [Redirection & Process Management](05-redirection-and-process-management.md) | Redirection, pipes, `tee`, `xargs`, `ps`, `top`, `kill`, `nice`, `screen`, `tmux` |
| 06 | [Compression, Archives & DD](06-compression-archives-and-dd.md) | `gzip`, `bzip2`, `xz`, `tar`, `zip`, `cpio`, `dd` |
| 07 | [Permissions & Umask](07-permissions-and-umask.md) | `chmod`, `chown`, `chgrp`, umask, SUID/SGID/Sticky, users & groups, ACLs, `sudo` |
| 08 | [Boot Process & Runlevels](08-boot-process-and-runlevels.md) | Boot stages, GRUB, runlevels vs. systemd targets, `systemctl`, `journalctl`, power |
| 09 | [Disk Partitioning & LVM](09-disk-partitioning-and-lvm.md) | MBR vs. GPT, `fdisk`, `parted`, LVM (`pv`/`vg`/`lv`), resizing, swap |
| 10 | [Filesystem Maintenance & Repair](10-filesystem-maintenance-and-repair.md) | `df`, `du`, `/etc/fstab`, `tune2fs`, `fsck`, superblock recovery |
| 11 | [OpenSSH & Networking](11-ssh-and-networking.md) | `ssh`, keys, `scp`, `sftp`, `ip`, `ss`, `ping`, DNS tools, `curl`, `wget`, firewall basics |

---

## 🧭 How to Use

* Commands marked `sudo` need root privileges.
* Lab sections (Lab 15, 17, 21, 22, 23) modify disks and boot configuration. **Run them only in a VM or on a scratch disk.**
* Sections titled **➕ Additional Essentials** contain important commands added beyond the original coursework notes.
