# Linux Notes

## Why Linux matters for cloud engineering
Every AWS EC2 instance runs Linux. You'll SSH into servers, manage files,
debug processes, read logs, and write automation scripts — all from the
Linux command line. This is the foundation everything else is built on.

---

## TIL Log (Today I Learned)

| Date | Lesson | What clicked |
|------|--------|--------------|
| 2026-05-24 | Lesson 2 | `man -k keyword` searches man pages by topic — useful when you forget a command name |
| 2026-05-24 | Lesson 2 | `which bash` shows the full path to a binary — use to verify what's installed |
| 2026-05-24 | Lesson 4 | `mkdir -p` creates nested directories in one shot — use this in every shell script |
| 2026-05-24 | Lesson 4 | Never use a leading `/` with `rm -rf` unless you explicitly mean the filesystem root |

---

## File System

```bash
# Navigation
pwd                        # print current directory
cd /var/log                # go to absolute path
cd ..                      # go up one level
cd ~                       # go to home directory
cd -                       # go to previous directory

# Listing
ls                         # list files
ls -a                      # include hidden files (dotfiles)
ls -l                      # long format (permissions, size, date)
ls -lh                     # long format with human-readable sizes (KB, MB)
ls -lt                     # sort by modification time (newest first)

# File system hierarchy (cloud-relevant paths)
/etc/                      # config files (nginx, ssh, cron)
/var/log/                  # log files — you'll live here debugging
/home/                     # user home directories
/tmp/                      # temporary files, cleared on reboot
/usr/bin/                  # user binaries (installed programs)
```

---

## File Management

```bash
# Creating
touch file.txt             # create empty file / update timestamp
mkdir my-folder            # create directory
mkdir -p a/b/c             # create nested directories in one command

# Copying
cp file.txt /backup/       # copy file to directory
cp -r /etc/nginx /backup/  # copy directory recursively (-r flag)

# Moving & renaming
mv file.txt /tmp/          # move file
mv old-name.txt new-name.txt  # rename file
mv app.log ~               # move to home directory

# Deleting — BE CAREFUL
rm file.txt                # delete file (no undo)
rm -rf ./old-folder        # delete directory and contents (DANGEROUS)
# Always verify path before rm -rf. No recycle bin on Linux servers.

# Safe pattern before deleting
ls ./old-folder            # confirm what's inside first
pwd                        # confirm where you are
rm -rf ./old-folder        # then delete
```

---

## Getting Help

```bash
man ls                     # full manual for a command
ls --help                  # shorter, friendlier summary (use this more)
man -k copy                # search man pages for commands related to "copy"
apropos copy               # same as man -k copy
```

**How to navigate a man page:**
- `/keyword` — search within the page
- `n` — jump to next match
- `q` — quit

---

## SSH (critical for AWS)

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096

# Connect to remote server
ssh username@ip-address
ssh -i ~/.ssh/my-key.pem ec2-user@54.123.45.67   # AWS EC2 pattern

# Copy files to/from remote server
scp file.txt user@server:/remote/path/
scp -r ./folder user@server:/remote/path/
```

---

## Text Editors

```bash
# vim — you'll need this on servers (no GUI)
vim file.txt

# Essential vim commands
i          # enter insert mode (start typing)
Esc        # exit insert mode
:w         # save
:q         # quit
:wq        # save and quit
:q!        # quit without saving
/keyword   # search in file
```

---

## Commands to memorize vs look up

**Memorize (use daily):**
`ls`, `cd`, `pwd`, `cp`, `mv`, `rm`, `mkdir`, `ssh`, `cat`, `grep`, `tail`, `man`

**Know they exist, look up the flags:**
`find`, `tar`, `chmod`, `chown`, `ps`, `kill`, `cron`, `systemctl`, `awk`, `sed`

---

## Connecting to AWS context

| Linux concept | How it shows up in AWS |
|---------------|------------------------|
| SSH keys | EC2 key pairs — how you access instances |
| `/etc/` config files | User data scripts, nginx config on EC2 |
| `/var/log/` | Application logs shipped to CloudWatch |
| File permissions | IAM is the cloud equivalent — same principle |
| cron jobs | EventBridge scheduled rules, Lambda triggers |
| Processes | ECS tasks, pod containers in Kubernetes |
