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

| Linux concept        | AWS equivalent                   |
| -------------------- | -------------------------------- |
| SSH keys             | EC2 key pairs                    |
| `/etc/` configs      | User data bootstrap scripts      |
| `/var/log/`          | CloudWatch Logs                  |
| File permissions     | IAM policies                     |
| cron jobs            | EventBridge scheduled rules      |
| Processes            | ECS tasks, K8s pods              |
| `mount` + EBS volume | Attach EBS in console then mount |
| `df -h` full disk    | CloudWatch disk alarm            |
| `find` + `grep`      | CloudWatch Logs Insights         |
| `tar` backups        | S3 + AWS Backup                  |
| Symlinks             | ECS task definition versions     |
