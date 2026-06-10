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
