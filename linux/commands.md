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

# xargs -n 1 — one argument at a time
echo "lisa linda lori" | xargs -n 1 useradd    # safe
echo "lisa linda lori" | xargs useradd          # error — useradd can't handle multiple args

# Commands that NEED -n 1
useradd, userdel, usermod    # user management
passwd                        # password setting

# Commands that DON'T need -n 1
mkdir, rm, chmod, chown, grep, wc    # handle multiple args natively
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

# Section 6 — Text File Tools

## more and less — file pagers

```bash
more filename    # read large files one page at a time
                 # forward only — can't go back on some systems
                 # older tool — avoid in favour of less

less filename    # better version of more
                 # navigate forwards AND backwards
                 # search within file
                 # doesn't load entire file into memory
```

### less navigation
```
Space / f    next page
b            previous page
g            top of file
G            bottom of file
/pattern     search forward
?pattern     search backward
n            next match
N            previous match
q            quit
F            follow mode — live updates (like tail -f)
```

### When to use what
```
cat      → small files, print everything at once
less     → large files, need to navigate and search
tail -f  → live monitoring during deployments
```

---

## head and tail — reading parts of files

```bash
head filename          # first 10 lines (default)
head -3 filename       # first 3 lines
head -n 20 filename    # first 20 lines

tail filename          # last 10 lines (default)
tail -3 filename       # last 3 lines
tail -n 50 filename    # last 50 lines
tail -f filename       # follow — live updates as file grows
```

### head -3 /etc/passwd
```bash
head -3 /etc/passwd
# output:
# root:x:0:0:root:/root:/bin/bash
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
# bin:x:2:2:bin:/bin:/usr/sbin/nologin
```
Shows first 3 lines of /etc/passwd — the user database.
Useful to check the file format without reading all 50+ lines.

### head -3 /etc/passwd | tail -1
```bash
head -3 /etc/passwd | tail -1
# output:
# bin:x:2:2:bin:/bin:/usr/sbin/nologin
```
This is a pipe trick — extracts ONE specific line:
- head -3 → get first 3 lines
- tail -1  → get last 1 line of those 3
- Result   → line 3 exactly

Pattern: `head -N file | tail -1` = extract line number N
```bash
head -7 /etc/passwd | tail -1    # extract line 7
head -42 /etc/passwd | tail -1   # extract line 42
```

### sudo tail -f /var/log/syslog
```bash
sudo tail -f /var/log/syslog
```
The most important tail command for cloud engineering.
- `-f` = follow — stays open and prints new lines as they're written
- Shows system log in real time
- Press Ctrl+C to stop

Real AWS use: watch logs live during a deployment
```bash
# Terminal 1 — deploy your app
./deploy.sh

# Terminal 2 — watch what happens in real time
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/myapp/app.log
```
You will use this every single deployment day.

---

## cat flags — reading files with options

```bash
cat filename      # print file contents
```

### cat -A — show ALL special characters
```bash
cat -A filename
# Shows:
# $ at end of every line (line ending marker)
# ^I for tab characters
# ^M for Windows carriage returns (\r)
```
Real use on AWS: config files copied from Windows have ^M characters
that break Linux scripts. `cat -A` reveals them instantly:
```bash
cat -A deploy.sh
# #!/bin/bash^M$     ← ^M means Windows line endings — will break on Linux
# ./start.sh^M$
```
Fix with: `sed -i 's/\r//' deploy.sh`

### cat -b — number non-empty lines
```bash
cat -b filename
# output:
#      1  first line
#      2  second line
#         (empty line — not numbered)
#      3  third line
```
Numbers only lines that have content. Empty lines skipped.

### cat -n — number ALL lines
```bash
cat -n filename
# output:
#      1  first line
#      2  second line
#      3
#      4  fourth line
```
Numbers every line including empty ones.
Real use: find exact line number before editing with vim:
```bash
cat -n /etc/nginx/nginx.conf | grep "server_name"
# 42  server_name example.com;
# Now you know: vim +42 /etc/nginx/nginx.conf
```

### cat -s — squeeze blank lines
```bash
cat -s filename    # multiple consecutive blank lines → single blank line
```
Makes files with lots of whitespace easier to read.

---

## tac — reverse cat

```bash
tac filename    # prints file in reverse order — last line first
```

Sounds weird but genuinely useful:
```bash
# Log files are newest-at-bottom
# tac shows newest entries first without loading entire file
tac /var/log/nginx/access.log | less

# Compare: tail shows last 10 lines
# tac shows ALL lines but reversed — useful for searching recent history
```

Real AWS use: quickly find the most recent entries in a large log file
without knowing how many lines to tail.

---

## grep — search file contents

### Basic grep
```bash
grep pattern filename     # search for pattern in file
grep linda *              # search for "linda" in all files in current dir
grep linda /etc/passwd    # search for linda in passwd file
```

### grep with ps — process searching
```bash
ps aux                    # list ALL running processes
# output columns:
# USER  PID  %CPU  %MEM  VSZ  RSS  TTY  STAT  START  TIME  COMMAND

ps aux | grep http        # find all processes related to http/apache/nginx
ps aux | grep ssh         # find all SSH processes
ps aux | grep nginx       # find nginx process and its PID
```

### ps aux | grep ssh | grep -v grep
```bash
ps aux | grep ssh
# output:
# ec2-user  1234  0.0  0.1  ssh
# ec2-user  5678  0.0  0.0  grep ssh   ← this line is grep itself
#                                          we don't want this

ps aux | grep ssh | grep -v grep
# output:
# ec2-user  1234  0.0  0.1  ssh        ← only the actual SSH process
```
`grep -v grep` removes the grep process from its own results.
This is one of the most common Linux one-liners you'll write.

---

## grep flags — the important ones

### grep -i — case insensitive
```bash
grep -i "error" /var/log/syslog
# matches: error, Error, ERROR, ErRoR
```
Always use -i when searching logs — log levels vary in capitalization.

### grep -v — invert match (exclude)
```bash
grep -v "DEBUG" /var/log/app.log    # show everything EXCEPT DEBUG lines
ps aux | grep ssh | grep -v grep    # exclude the grep process itself
```
Real use: filter out noise from logs to see only what matters.

### grep -l — show filenames only
```bash
grep -l "pattern" /etc/*    # list files containing pattern, not the lines
```
Use when you want to know WHICH files contain something,
not what the matching lines say.

### grep -R — recursive search
```bash
grep -R "pattern" /etc/         # search all files in directory recursively
grep -R "127.0.0.1" /etc/       # find all configs referencing localhost
grep -R "password" /var/www/    # security audit — find hardcoded passwords
```
Same as find + xargs grep but simpler syntax for quick searches.

### grep -Rl — recursive, filenames only
```bash
grep -Rl "pattern" /etc/        # list files containing pattern recursively
```
Combines -R (recursive) and -l (filenames only).

### grep -A, -B, -C — context lines
```bash
grep -A5 "pattern" file    # show 5 lines AFTER match (After)
grep -B5 "pattern" file    # show 5 lines BEFORE match (Before)
grep -C5 "pattern" file    # show 5 lines BEFORE AND AFTER match (Context)
```
Critical for log analysis — errors rarely make sense without context:
```bash
# Just the error line — not very useful
grep "FATAL" /var/log/app.log
# Jan 15 02:14:33 FATAL: Connection refused

# With context — now you understand what happened
grep -C5 "FATAL" /var/log/app.log
# Jan 15 02:14:28 INFO: Attempting database connection
# Jan 15 02:14:29 INFO: Retry attempt 1
# Jan 15 02:14:30 INFO: Retry attempt 2
# Jan 15 02:14:31 INFO: Retry attempt 3
# Jan 15 02:14:32 WARN: Connection timeout
# Jan 15 02:14:33 FATAL: Connection refused     ← now this makes sense
# Jan 15 02:14:33 INFO: Shutting down
# Jan 15 02:14:33 INFO: Cleanup complete
```
You will use -C5 constantly when debugging production issues on AWS.

---

## Real scenarios — command breakdowns

### sudo grep abhinav /etc/*
```bash
sudo grep abhinav /etc/*
```
- Searches for "abhinav" in every file directly inside /etc/
- NOT recursive — only top-level files in /etc/
- Shows matching lines with filename prefix
- sudo needed because some /etc/ files are root-only
- Use case: find if a username appears in any config file

### sudo grep -Rl abhinav /etc/*
```bash
sudo grep -Rl abhinav /etc/*
```
- -R = recursive — searches /etc/ AND all subdirectories
- -l = filenames only — no matching lines shown
- Returns: list of files that contain "abhinav"
- More useful than above when you want to know WHICH files to edit

Difference:
```
grep abhinav /etc/*       → searches top level only, shows matching lines
grep -Rl abhinav /etc/*   → searches recursively, shows filenames only
```

### ps faux | grep -B5 faux
```bash
ps faux
# Shows processes in a tree format (f = forest/tree view)
# faux = f (forest) + aux (all processes)
# Shows parent-child relationships between processes

ps faux | grep -B5 faux
# Searches tree output for "faux"
# -B5 shows 5 lines BEFORE each match
# Shows the parent processes of any process containing "faux"
# Useful for understanding process hierarchy
```

### sudo grep -A5 -i root /etc/ssh/sshd_config
```bash
sudo grep -A5 -i root /etc/ssh/sshd_config
```
Breaking it down:
```
sudo                    → need root to read sshd_config
grep                    → search command
-A5                     → show 5 lines after each match
-i                      → case insensitive (matches Root, ROOT, root)
root                    → search pattern
/etc/ssh/sshd_config    → SSH server configuration file
```

Output example:
```
PermitRootLogin no          ← matched line (contains "root")
# the settings below...    ← line 1 after
AllowUsers ec2-user         ← line 2 after
MaxAuthTries 3              ← line 3 after
PubkeyAuthentication yes    ← line 4 after
PasswordAuthentication no   ← line 5 after
```

Real AWS use: quickly check SSH security settings.
`PermitRootLogin no` should always be set on production servers.
The -A5 context shows the surrounding security settings too.

---

## grep with regex — preview (covered fully in Section 7)

```bash
grep "^root" /etc/passwd      # ^ = start of line
grep "bash$" /etc/passwd      # $ = end of line
grep "ro*t" /etc/passwd       # * = zero or more of previous char
grep "r.t" /etc/passwd        # . = any single character
grep -E "root|admin" file     # -E = extended regex, | = OR
grep -E "^[A-Z]" file         # lines starting with capital letter
```

---

## Commands summary

```bash
# Pagers
less filename                         # navigate large files
more filename                         # forward only (use less instead)

# Head and tail
head -n 10 filename                   # first 10 lines
tail -n 10 filename                   # last 10 lines
head -3 file | tail -1                # extract specific line (line 3)
tail -f /var/log/nginx/error.log      # live log monitoring

# cat flags
cat -A filename    # show special characters (find Windows line endings)
cat -b filename    # number non-empty lines
cat -n filename    # number all lines
cat -s filename    # squeeze multiple blank lines

# tac
tac filename       # reverse cat — last line first

# grep flags
grep -i "pattern" file    # case insensitive
grep -v "pattern" file    # invert — exclude matches
grep -l "pattern" /etc/*  # filenames only
grep -R "pattern" /etc/   # recursive
grep -Rl "pattern" /etc/  # recursive, filenames only
grep -A5 "pattern" file   # 5 lines after match
grep -B5 "pattern" file   # 5 lines before match
grep -C5 "pattern" file   # 5 lines before and after match

# Real patterns
ps aux | grep nginx | grep -v grep          # find process, exclude grep itself
sudo grep -Rl "pattern" /etc/              # find files recursively
sudo grep -A5 -i root /etc/ssh/sshd_config # check SSH config with context
tail -f /var/log/app.log | grep -i error   # live error monitoring
```

---

## AWS real-world usage

```bash
# Live deployment monitoring
tail -f /var/log/nginx/access.log
tail -f /var/log/myapp/app.log | grep -i error

# Find broken config
grep -R "old-server.internal" /etc/    # find stale hostname in configs
grep -Rl "127.0.0.1" /etc/nginx/       # find files referencing localhost

# Security audit
sudo grep -A5 -i "PermitRootLogin" /etc/ssh/sshd_config
grep -R "password" /var/www/html/      # find hardcoded passwords

# Process management
ps aux | grep nginx | grep -v grep     # find nginx PID for restart
ps faux | grep -B5 defunct             # find zombie process parents

# Log analysis
grep -C5 "FATAL" /var/log/app.log      # understand context around errors
grep -i "error\|warn\|fatal" /var/log/syslog   # find all issues at once
tail -n 1000 /var/log/app.log | grep -i error  # errors in last 1000 lines
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

