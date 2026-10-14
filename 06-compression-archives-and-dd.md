# 06. Compression, Archives & Bit-Level Copy (Ch-12)

## 1. Compression Utilities

```bash
# Gzip (.gz)
gzip longfile.txt            # Compress to longfile.txt.gz (deletes original)
ls longfile*                 # Check created archive
gzip -k longfile.txt         # Keep the original
gzip -c longfile.txt > longfile.txt.gz   # Write to stdout, redirected to a file
gzip -9 longfile.txt         # Best compression (-1 fastest)
gunzip longfile.txt.gz       # Decompress
gzip -l longfile.txt.gz      # List compression statistics (gunzip -l also works)
zcat longfile.txt.gz         # View contents of compressed file

# Gzip Directory Recursion
mkdir mydirectory && touch mydirectory/file1 mydirectory/file2
gzip -r mydirectory          # Compress each file within the directory
gunzip -r mydirectory/       # Decompress each file within the directory

# Bzip2 (.bz2) - does NOT support directory recursion
bzip2 longfile.txt           # Compress (deletes original)
bzip2 -k longfile.txt        # Keep original
bzip2 -c longfile.txt > longfile.txt.bz2   # Stream to a file
bzcat longfile.txt.bz2       # View contents
bunzip2 longfile.txt.bz2     # Decompress

# XZ (.xz)
xz longfile.txt              # Compress (deletes original)
xz -k longfile.txt           # Keep original
xz -c longfile.txt > longfile.txt.xz
xzcat longfile.txt.xz        # View contents
unxz longfile.txt.xz         # Decompress
```

| Tool | Extension | Speed | Ratio |
| :--- | :--- | :--- | :--- |
| `gzip` | `.gz` | Fast | Good |
| `bzip2` | `.bz2` | Medium | Better |
| `xz` | `.xz` | Slow | Best |

---

## 2. Packaging & Bit-Level Imaging

```bash
# Tarballs (tar)
# Options: -c create, -x extract, -t list, -f archive file, -v verbose,
#          -z gzip, -j bzip2, -J xz, -C change directory
tar -cvf mydocuments.tar ~/Documents          # Create uncompressed archive
tar -czvf mydocuments.tar.gz ~/Documents      # Create gzip-compressed archive
tar -cjvf mydocuments.tar.bz2 ~/Documents     # Create bzip2-compressed archive
tar -cJvf mydocuments.tar.xz ~/Documents      # Create xz-compressed archive
tar -tvf mydocuments.tar                      # List contents
tar -tzvf mydocuments.tar.gz                  # List gzipped tarball contents
tar -xvf mydocuments.tar                      # Extract everything
tar -xzvf mydocuments.tar.gz -C /tmp          # Extract into /tmp
tar -xvf mydocuments.tar home/user/Documents/file.txt   # Extract one member
tar -rvf mydocuments.tar newfile              # Append a file to an archive

# Zip & Unzip
zip -r myperl.zip /etc/perl  # Compress a directory recursively
unzip -l myperl.zip          # List contents of a zip file
unzip myperl.zip -d /tmp     # Extract into a target directory

# CPIO Archives
# Modes: copy-out (-o), copy-in (-i), copy-pass (-p)
# Flags: -d create directories, -u overwrite, -v verbose, -t list
find . -depth | cpio -ov > archive.cpio       # Create a cpio archive
cpio -it < archive.cpio                       # List contents
cpio -iud < archive.cpio                      # Unpack (create dirs, overwrite)

# Low-Level Bitstream Copy (dd)
# Parameters: if=input, of=output, bs=block size, count=number of blocks
# Units: K (kilobytes), M (megabytes), G (gigabytes)
dd if=/dev/zero of=/var/extraswap bs=1M count=100   # Create a fixed 100 MB file
```

### ➕ Additional Essentials: `dd` Recipes (⚠️ double-check `of=` targets!)

```bash
sudo dd if=/dev/sda of=/backup/sda.img bs=4M status=progress   # Image a whole disk
sudo dd if=/backup/sda.img of=/dev/sdb bs=4M status=progress   # Restore image to disk
sudo dd if=/dev/sda bs=512 count=1 of=mbr.bak                  # Back up the MBR (446 bytes boot code + table)
dd if=ubuntu.iso of=/dev/sdX bs=4M status=progress && sync     # Write ISO to USB stick
dd if=/dev/urandom of=random.bin bs=1M count=10                # 10 MB random data
sha256sum file.iso           # Verify integrity after transfer
md5sum file.iso              # Older checksum
```

### ➕ Additional Essentials: `rsync` for Backups

```bash
rsync -av /src/ /dest/                       # Archive-mode copy, verbose
rsync -avz /src/ user@host:/backup/          # Over SSH with compression
rsync -av --delete /src/ /dest/              # Mirror (removes extras in dest)
rsync -avn /src/ /dest/                      # Dry run
```
