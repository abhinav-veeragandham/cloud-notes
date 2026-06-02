# Section 5 Review Questions

Use these for spaced repetition. Cover the answers and test yourself.
Come back to this file after Week 1, after Week 2, and before interviews.

---

## Links (ln)

1. What is a hard link in one sentence?
2. What is a symbolic link in one sentence?
3. If you delete the original file, what happens to the hard link? What happens to the symlink?
4. Can a hard link point to a directory? Can a symlink?
5. If you edit a file through a hard link, what happens to the original file?
6. Create a hard link called `backup.txt` pointing to `original.txt`
7. Create a symlink called `active.conf` pointing to `/etc/nginx/sites-available/myapp.conf`
8. How do you check what a symlink is pointing to?
9. How do you disable an nginx site without deleting its config?
10. Why does nginx use the sites-available/sites-enabled pattern?
11. What command re-enables a disabled nginx site?
12. True or False: Hard links use extra disk space because they copy file data
13. True or False: Deleting a symlink deletes the original file
14. True or False: Symlinks can work across different drives and partitions
15. What is a dangling symlink?

---

## find command — basics

16. What is the structure of every find command?
17. Find a file called `nginx.conf` anywhere on the system
18. Find all files larger than 500MB on the entire system
19. Find all files modified in the last 24 hours in `/etc`
20. Find all files modified more than 30 days ago in `/var/log`
21. Find all files modified in the last 60 minutes in `/var/log`
22. What does `-type f` mean?
23. What does `-type d` mean?
24. What does `-type l` mean?
25. Find all files owned by user `linda` on the entire system
26. Find all files NOT owned by `ec2-user` in `/home`
27. Find all SUID files on the system, files only
28. Find all world-writable files in `/var/www/html`
29. What does `+` mean with `-mtime` and `-size`?
30. What does `-` mean with `-mtime` and `-size`?

---

## find command — advanced

31. What is `-exec` and when do you use it?
32. What is `xargs` and when do you use it?
33. What is the difference between `-exec` and `xargs`?
34. When would you use `-exec` instead of `xargs`?
35. What does `{}` mean in a `-exec` command?
36. What does `\;` mean in a `-exec` command?
37. Find all files in `/etc` containing the word "root" — use `-exec grep`
38. Find all files in `/var/log` containing "failed" — use `xargs grep`
39. What does `grep -l` do differently from `grep`?
40. Find all files owned by `linda`, copy them to `/root/linda` preserving attributes
41. Find all `.log` files in `/var/log` older than 30 days and delete them
42. Find all files in `/etc` larger than 10KB, print size and path sorted largest first
43. What does `-printf '%s, %p\n'` do?
44. What does `sort -rn` do?
45. What does `2>/dev/null` do?
46. What is the correct order for output redirection?
47. Find broken symlinks in `/etc/nginx/sites-enabled` — what flag goes before the path?
48. What is `-xtype l` — is `l` a letter or number?
49. How do you count the results of a find command?
50. Find files containing "student" in `/etc`, copy matches to `./find/contents/`

---

## find — real AWS scenarios

51. Your EC2 disk is 95% full at 3am — what find command do you run first?
52. A deployment ran 2 hours ago and something broke — how do you find what changed?
53. Security team wants all SUID files listed — write the command and save to a file
54. A user `john` left the company — find and copy all their files preserving attributes
55. You need to clean up log files older than 30 days — write the command
56. Find all config files in `/etc` referencing IP `10.0.1.45`
57. Find all files in `/var/log` larger than 100MB modified in last 7 days, show sizes

---

## which, locate, file

58. What does `which` find?
59. What does `locate` find?
60. What is the difference between `which` and `locate`?
61. What is the difference between `locate` and `find`?
62. If `locate` returns nothing or stale results, what do you run?
63. What does `file /bin/bash` tell you?
64. What does `file backup.tar.gz` tell you?
65. How do you check if two binaries point to the same executable?
66. `which nginx.conf` — will this work? Why or why not?

---

## tar

67. What does tar stand for conceptually?
68. What do these flags mean: `-c` `-x` `-v` `-f` `-z` `-j` `-J` `-C`
69. Create a gzip compressed archive of `/etc/nginx` called `backup.tar.gz`
70. Extract `backup.tar.gz` to `/tmp/restore/`
71. View contents of `backup.tar.gz` without extracting
72. Create a bzip2 compressed archive of `/var/log`
73. What is the difference between tar and gzip?
74. When do you use tar vs gzip directly?

---

## Compression

75. How do you compress a single file called `app.log` with gzip?
76. How do you decompress `app.log.gz`?
77. What happens to the original file when you run `gzip file.txt`?
78. How do you compress without removing the original?
79. What command compresses with bzip2?
80. What command decompresses a `.bz2` file?
81. What command compresses with xz?
82. What command decompresses a `.xz` file?
83. Compare gzip, bzip2, xz — speed and compression ratio
84. Which compression tool would you use for log files on AWS? Why?
85. Which compression tool would you use for long-term cold storage? Why?

---

## Mounting

86. What does mounting mean in one sentence?
87. What command shows all currently mounted filesystems?
88. What command shows a tree view of all mounts?
89. What command shows disk space usage in human readable format?
90. What command lists all block devices and mount points?
91. Write the full 4-step workflow to attach a new EBS volume `/dev/xvdf` to `/mnt/data`
92. What is `mkfs.ext4` and why do you run it first?
93. How do you unmount a filesystem?
94. How do you make a mount persistent across reboots?
95. What file controls persistent mounts on Linux?

---

## xargs deep dive

96. What is the purpose of xargs in one sentence?
97. Why is `find ... | xargs grep` faster than `find ... -exec grep {} \;`?
98. When would you use `-print0 | xargs -0`?
99. Can you use xargs and -exec together in the same command?
100. Write three different examples of xargs with different commands (not grep)

---

## Answers

### Links
1. A second name pointing to the same inode (same data on disk)
2. A pointer to a filename — breaks if original is deleted
3. Hard link survives — data still accessible. Symlink breaks — dangling symlink
4. Hard link: no. Symlink: yes
5. The original changes too — they're the same data
6. `ln original.txt backup.txt`
7. `ln -s /etc/nginx/sites-available/myapp.conf active.conf`
8. `ls -la /path/` — shows `->` arrow
9. `rm /etc/nginx/sites-enabled/myapp.conf`
10. Separates storage (sites-available) from activation (sites-enabled) — safe, reversible
11. `ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/`
12. False — same inode, no extra storage
13. False — only removes the pointer, original untouched
14. True — unlike hard links
15. A symlink whose original file has been deleted — pointer goes nowhere

### find — basics
16. `find [WHERE] [WHAT CONDITIONS] [ACTION]`
17. `find / -name "nginx.conf"`
18. `find / -type f -size +500M`
19. `find /etc -type f -mtime -1`
20. `find /var/log -type f -mtime +30`
21. `find /var/log -type f -mmin -60`
22. Regular files only — excludes directories, symlinks
23. Directories only
24. Symlinks only
25. `find / -user linda -type f`
26. `find /home ! -user ec2-user -type f`
27. `find / -type f -perm /4000`
28. `find /var/www/html -type f -perm /002`
29. + means MORE THAN (older/larger)
30. - means LESS THAN (newer/smaller)

### find — advanced
31. Runs a command on each file found. Use when chaining two actions on same file
32. Takes list of filenames and passes them to a command. Use for speed
33. -exec = one process per file (slow). xargs = one process for all files (fast)
34. When chaining two actions on same file, or command only handles one file at a time
35. Placeholder replaced by each filename found
36. Marks end of -exec command
37. `find /etc -type f -exec grep -l "root" {} \;`
38. `find /var/log -type f | xargs grep "failed"`
39. `grep -l` prints only filenames. `grep` prints matching lines
40. `sudo find / -user linda -type f -exec cp -a {} /root/linda \;`
41. `find /var/log -name "*.log" -type f -mtime +30 -delete 2>/dev/null`
42. `find /etc -type f -size +10K -printf '%s, %p\n' | sort -rn`
43. Prints file size and path for each result, separated by comma
44. Sort numerically in reverse order (largest first)
45. Redirects error messages to /dev/null — silences them
46. `> output.txt 2>/dev/null` — stdout first, stderr second
47. `-L` — must come right after `find` before the path
48. Lowercase letter L — for symLink. Never number 1
49. `find ... | wc -l`
50. `mkdir -p find/contents && find /etc -exec grep -l "student" {} \; -exec cp {} find/contents/ \; 2>/dev/null`

### find — AWS scenarios
51. `find / -type f -size +500M`
52. `find /etc -type f -mmin -120`
53. `sudo find / -type f -perm /4000 > /tmp/suid-audit.txt 2>/dev/null`
54. `sudo mkdir -p /backup/john && sudo find / -user john -type f -exec cp -a {} /backup/john \;`
55. `sudo find /var/log -name "*.log" -type f -mtime +30 -delete 2>/dev/null`
56. `sudo find /etc -type f | xargs grep "10.0.1.45"`
57. `sudo find /var/log -type f -size +100M -mtime -7 -printf '%s, %p\n' | sort -rn`

### which, locate, file
58. Executable binaries in your PATH only
59. Any file on the system using a pre-built database
60. which=executables in PATH only. locate=any file via database
61. locate=uses database (fast, may be outdated). find=real-time search (slower, always current)
62. `sudo updatedb`
63. Type of the file — ELF 64-bit LSB executable
64. gzip compressed data
65. `ls -la /usr/bin/python3` and `ls -la /usr/bin/python`
66. No — which only finds executables. nginx.conf is a config file, not a binary

### tar
67. Tape ARchive — bundles multiple files into one
68. -c=create -x=extract -v=verbose -f=filename -z=gzip -j=bzip2 -J=xz -C=destination
69. `tar -czvf backup.tar.gz /etc/nginx`
70. `tar -xzvf backup.tar.gz -C /tmp/restore/`
71. `tar -tzvf backup.tar.gz`
72. `tar -cjvf logs.tar.bz2 /var/log`
73. tar=bundles files into archive. gzip=compresses a single file
74. tar for directories/multiple files. gzip for single files

### Compression
75. `gzip app.log` — creates app.log.gz
76. `gunzip app.log.gz` or `gzip -d app.log.gz`
77. Original is removed automatically — gzip replaces it with .gz version
78. `gzip -k file.txt` — keep original
79. `bzip2 file.txt`
80. `bunzip2 file.bz2`
81. `xz file.txt`
82. `unxz file.xz`
83. gzip=fastest/least compression. bzip2=middle. xz=slowest/best compression
84. gzip — speed matters more than size for frequently accessed logs
85. xz — best compression ratio for rarely accessed archives

### Mounting
86. Making a storage device accessible at a specific directory path
87. `mount`
88. `findmnt`
89. `df -h`
90. `lsblk`
91. `sudo mkfs.ext4 /dev/xvdf` → `sudo mkdir -p /mnt/data` → `sudo mount /dev/xvdf /mnt/data` → `df -h`
92. Formats the volume with ext4 filesystem — must do before first use
93. `sudo umount /mnt/data`
94. Add entry to `/etc/fstab`
95. `/etc/fstab`

### xargs
96. Takes a list of items from stdin and passes them as arguments to a command
97. xargs runs one grep process for all files. -exec spawns a new process per file
98. When filenames contain spaces — -print0 uses null separator instead of space
99. No — they're two different ways to do the same thing, use one or the other
100. `find /tmp -name "tmp_*" | xargs rm`
     `find /var/www -type f -perm /002 | xargs chmod 644`
     `find /var/log -name "*.log" | xargs wc -l`
