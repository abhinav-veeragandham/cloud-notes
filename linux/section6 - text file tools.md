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
- tail -1 → get last 1 line of those 3
- Result → line 3 exactly

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

### sudo grep abhinav /etc/\*

```bash
sudo grep abhinav /etc/*
```

- Searches for "abhinav" in every file directly inside /etc/
- NOT recursive — only top-level files in /etc/
- Shows matching lines with filename prefix
- sudo needed because some /etc/ files are root-only
- Use case: find if a username appears in any config file

### sudo grep -Rl abhinav /etc/\*

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
