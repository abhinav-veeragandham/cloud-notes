# 🐧 Linux Commands

## Why Linux matters for cloud engineering
Every AWS EC2 instance runs Linux. You'll SSH into servers, manage files,
debug processes, read logs, and write automation scripts — all from the
Linux command line. This is the foundation everything else is built on.

---

## Section 2 — Essential tools

```bash
# Getting help
man ls                     # full manual for a command
man -k keyword             # search man pages by keyword (same as apropos)
apropos keyword            # search man pages by keyword
ls --help                  # shorter, friendlier summary
info ls                    # alternative to man, more detailed

# Navigation
pwd                        # print current directory
cd /var/log                # go to absolute path
cd ..                      # go up one level
cd ~                       # go to home directory
cd -                       # go to previous directory

# Listing files
ls                         # list files
ls -a                      # include hidden files (dotfiles)
ls -l                      # long format (permissions, size, date)
ls -lh                     # long format with human-readable sizes
ls -lt                     # sort by modification time newest first
ls -la                     # long format including hidden files

# Finding binaries
which bash                 # show full path to a binary/executable
type bash                  # show if command is built-in, alias, or external

# Who am I
whoami                     # print current logged-in user
id                         # show user ID, group ID

# Network info
ip a                       # show all network interfaces and IP addresses

# File contents
cat file.txt               # print file contents
passwd                     # change current user password

# Creating empty files
touch file.txt             # create empty file or update timestamp

# Man page navigation
# /keyword   search within man page
# n          jump to next match
# q          quit
```

---

## Section 4 — File management

```bash
# Creating
mkdir my-folder            # create directory
mkdir -p a/b/c             # create nested directories in one command

# Copying
cp file.txt /backup/       # copy file to directory
cp -r /etc/nginx /backup/  # copy directory recursively
cp -a /etc/nginx /backup/  # copy preserving permissions + timestamps

# Moving and renaming
mv file.txt /tmp/          # move file
mv old.txt new.txt         # rename file
mv app.log ~               # move to home directory

# Deleting
rm file.txt                # delete file (no undo)
rm -f file.txt             # force delete, no prompt
rm -rf ./old-folder        # delete directory and all contents
rmdir empty-folder         # delete empty directory only

# Safe deletion pattern — always do this first
ls ./folder                # confirm what's inside
pwd                        # confirm where you are
rm -rf ./folder            # then delete

# Viewing disk usage
du -sh /var/log            # size of a directory human readable
du -sh *                   # size of everything in current directory
```

---

## Section 5 — Advanced file management

### Links
```bash
# Hard links — two names, one inode (same data)
ln original.txt hardlink.txt
# - survives deletion of original
# - cannot link directories
# - only works within same filesystem

# Symbolic links — pointer to a filename
ln -s /path/to/target linkname    # target first, link name second
ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/myapp
# - breaks if original deleted (dangling symlink)
# - can link directories
# - works across filesystems

# Check what a symlink points to
ls -la /etc/nginx/sites-enabled/

# nginx pattern — enable/disable sites without deleting configs
ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/  # enable
rm /etc/nginx/sites-enabled/myapp                                   # disable
```

### find command
```bash
# Basic structure
find [WHERE] [WHAT CONDITIONS] [ACTION]

# Search by name
find / -name "nginx.conf"          # exact name
find / -name "*.log"               # wildcard
find / -name "hosts*"              # starts with hosts

# Search by type
find / -type f                     # files only
find / -type d                     # directories only
find / -type l                     # symlinks only

# Search by owner
find / -user linda                 # owned by linda
find / ! -user ec2-user            # NOT owned by ec2-user (space after !)

# Search by size
find / -size +500M                 # larger than 500MB
find / -size -10M                  # smaller than 10MB
find / -size +100M -size -1G       # between 100MB and 1GB

# Search by time
find / -mtime -1                   # modified in last 24 hours
find / -mtime +30                  # modified more than 30 days ago
find / -mmin -60                   # modified in last 60 minutes

# + vs - rule
# +  means MORE THAN (older, larger)
# -  means LESS THAN (newer, smaller)

# Search by permissions
find / -perm /4000                 # SUID files (security audit)
find / -perm /002                  # world-writable files

# Broken symlinks
find -L /path -xtype l             # -L before path, l = letter not number 1

# Output formatting
find /etc -printf '%s, %p\n'       # size, path
find /etc -printf '%m, %s, %p\n'   # permissions, size, path
find /etc -printf '%s, %p\n' | sort -rn   # sorted largest first

# Actions
find /etc -type f -delete                  # delete results
find /etc -type f > /tmp/output.txt        # save results to file
find /etc -type f 2>/dev/null              # suppress errors

# -exec — runs command on each file (one process per file)
find / -type f -exec grep -l "pattern" {} \;
find / -user linda -type f -exec cp -a {} /dest/ \;
# Chained exec — file must pass first before second runs
find /etc -exec grep -l "student" {} \; -exec cp {} /dest/ \; 2>/dev/null

# xargs — batches all filenames into one process (faster)
find /etc -type f | xargs grep "pattern"
find /tmp -name "tmp_*" | xargs rm
find /var/www -type f -perm /002 | xargs chmod 644

# Handle filenames with spaces
find /etc -type f -print0 | xargs -0 grep "pattern"

# -exec vs xargs
# -exec  = one process per file = flexible, use for chaining two actions
# xargs  = one process for all  = faster, use for searching/processing

# Count results
find / -type f -perm /4000 | wc -l

# Output order — stdout before stderr
find / -type f > /tmp/output.txt 2>/dev/null

# Real AWS scenarios
find / -size +500M -type f                              # disk full — find culprit
find /etc -mtime -1 -type f                             # what changed recently
find / -name "nginx.conf"                               # locate config file
find / -perm /4000 -type f                              # security audit
find /var/log -name "*.log" -mtime +30 -delete          # clean old logs
find / -type f -size +100M | wc -l                      # count large files
find /etc -type f | xargs grep "192.168.1.100"          # find IP in configs
find /etc -name "*.conf" -printf '%s, %p\n' | sort -rn  # audit config sizes
```

### xargs
```bash
# xargs -n flag — control arguments per command
echo "lisa linda lori" | xargs -n 1 useradd    # create 3 users
echo "dir1 dir2 dir3" | xargs -n 1 mkdir       # create 3 directories

# -n 1 = run command once per item (most useful)
# without -n = pass everything at once (default)
```

### which, locate, file
```bash
# which — finds executable binaries in PATH only
which nginx                # output: /usr/sbin/nginx
which python3              # output: /usr/bin/python3

# locate — finds any file using pre-built database (fast)
locate nginx.conf          # find any file named nginx.conf
sudo updatedb              # rebuild locate database (run if results are stale)

# file — identifies what type a file is
file /bin/bash             # ELF 64-bit LSB executable
file backup.tar.gz         # gzip compressed data
file notes.txt             # ASCII text

# which vs locate vs find
# which   = find a runnable command in PATH
# locate  = find any file fast via database (may be outdated)
# find    = find any file in real-time (always current, slower)

# Check if two binaries are the same
ls -la /usr/bin/python3
ls -la /usr/bin/python
```

### tar — archiving
```bash
# tar flags
# -c  create archive
# -x  extract archive
# -v  verbose (print each file being processed)
# -f  filename (next argument is archive name)
# -z  gzip compression
# -j  bzip2 compression
# -J  xz compression
# -C  destination directory for extraction

# Create compressed archive
tar -czvf backup.tar.gz /etc/nginx         # gzip
tar -cjvf backup.tar.bz2 /etc/nginx        # bzip2
tar -cJvf backup.tar.xz /etc/nginx         # xz

# Extract archive
tar -xzvf backup.tar.gz                    # extract here
tar -xzvf backup.tar.gz -C /tmp/restore/   # extract to specific directory

# View contents without extracting
tar -tzvf backup.tar.gz
```

### Compression — single files
```bash
# gzip — fastest, least compression
gzip file.txt              # creates file.txt.gz, removes original
gunzip file.txt.gz         # decompress
gzip -d file.txt.gz        # same as gunzip
gzip -k file.txt           # keep original file

# bzip2 — middle ground
bzip2 file.txt             # creates file.txt.bz2
bunzip2 file.txt.bz2       # decompress

# xz — slowest, best compression
xz file.txt                # creates file.txt.xz
unxz file.txt.xz           # decompress

# Speed vs compression ratio
# gzip  → fastest, least compression  (use for logs on AWS)
# bzip2 → middle
# xz    → slowest, best compression   (use for long-term archives)

# tar vs compression tools
# tar    = bundles multiple files into one archive
# gzip   = compresses a single file
# combine: tar -czvf = bundle AND compress together
```

### Mounting
```bash
# Show mounted filesystems
mount                      # all currently mounted filesystems
findmnt                    # tree view of all mounts
df -h                      # disk space usage human readable
lsblk                      # list all block devices and mount points

# Mount a device
sudo mount /dev/xvdf /mnt/data     # device first, mount point second
sudo umount /mnt/data              # unmount

# Full EBS volume workflow on AWS EC2 — memorize this
sudo mkfs.ext4 /dev/xvdf           # Step 1: format new volume
sudo mkdir -p /mnt/data            # Step 2: create mount point
sudo mount /dev/xvdf /mnt/data     # Step 3: mount it
df -h                              # Step 4: verify it mounted

# Make mount persistent across reboots — add to /etc/fstab
echo '/dev/xvdf /mnt/data ext4 defaults 0 0' | sudo tee -a /etc/fstab
```

---

## Quick reference — commands to memorize vs look up

### Memorize (use daily on AWS)
```bash
ls, cd, pwd, cp, mv, rm, mkdir     # file operations
ssh, scp                           # remote access
cat, tail, grep                    # reading files
find, xargs                        # searching
tar, gzip, gunzip                  # archiving
df -h, lsblk, mount                # disk management
which, file                        # identifying files
sudo updatedb, locate              # finding files
whoami, id, ip a                   # system info
```

### Know they exist, look up the flags
```bash
find -printf, find -exec, find -perm    # advanced find
tar -j, tar -J                          # bzip2 and xz with tar
findmnt, du                             # disk details
```

---

## Linux → AWS context

| Linux concept | AWS equivalent |
|---------------|----------------|
| SSH keys | EC2 key pairs |
| `/etc/` configs | User data bootstrap scripts |
| `/var/log/` | CloudWatch Logs |
| File permissions | IAM policies |
| cron jobs | EventBridge scheduled rules |
| Processes | ECS tasks, K8s pods |
| `mount` + EBS volume | Attach EBS in console then mount |
| `df -h` full disk | CloudWatch disk alarm |
| `find` + `grep` | CloudWatch Logs Insights |
| `tar` backups | S3 + AWS Backup |
| Symlinks | ECS task definition versions |

