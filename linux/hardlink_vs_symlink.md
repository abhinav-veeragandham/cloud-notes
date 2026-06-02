## Hard Links vs Symbolic Links

### How Linux stores files
Every file has two parts:
- filename  → just a label (what you see)
- inode     → the actual data on disk (permissions, size, content)

Think of it as:
  inode    = a house
  filename = the address written on a piece of paper
  
Multiple addresses can point to the same house.
That's the foundation of links.

---

### Hard Links
A hard link is a second filename pointing directly to the same inode.

```bash
ln original.txt hardlink.txt
```

```
original.txt ──────► inode #42 (actual data)
hardlink.txt ──────► inode #42 (same data)
```

- Two names, one file — neither is more "original" than the other
- Data only disappears when ALL names pointing to the inode are deleted
- Deleting original.txt leaves hardlink.txt fully working
- Uses zero extra disk space — same inode, no duplication

**Limitations:**
- Cannot link to directories (explained below)
- Cannot work across different filesystems or drives
- Cannot link across partitions

**Real use:** redundancy — data survives accidental deletion of one name

---

### Symbolic Links (Symlinks / Soft Links)
A symlink is a pointer to a filename, not to the inode directly.

```bash
ln -s /path/to/target linkname    # target first, link name second
```

```
symlink.txt ──────► "original.txt" ──────► inode #42
```

- If original.txt is deleted, symlink becomes dangling — points to nothing
- Can point to directories
- Works across different filesystems and drives
- Can be updated instantly to point to a new target

**Real use:** flexibility — nginx sites-enabled pattern, version switching

---

### Key differences

| | Hard Link | Symbolic Link |
|--|-----------|---------------|
| Points to | inode (data) | filename |
| Survives original deletion? | ✅ Yes | ❌ No — dangling |
| Can link directories? | ❌ No | ✅ Yes |
| Works across filesystems? | ❌ No | ✅ Yes |
| Extra disk space? | No | Tiny (stores path) |
| Create with | `ln file link` | `ln -s target link` |

---

### Why hard links cannot point to directories

If hard links to directories were allowed, this could happen:
/home/user/documents/ ──────► inode #50
└── loop/ ───────────────► inode #50  (points back to itself)

This creates an infinite loop. Running find or rm -rf would
follow the loop forever — never finishing, potentially crashing
the system or corrupting the filesystem.

Linux forbids hard links to directories entirely to prevent this.
Symlinks can point to directories safely because Linux tracks
which symlinks it has already followed and stops when a loop
is detected.

---

### Simple definitions (one line each)

Hard link:  a second name pointing directly to the same data (inode).
            Data survives as long as at least one name exists.

Symlink:    a pointer to a filename.
            Breaks if the original filename is deleted.

---

### The nginx symlink pattern — real production use

```bash
# Store ALL configs safely
/etc/nginx/sites-available/
    company.conf
    api.conf
    maintenance.conf

# Activate only what you need via symlinks
ln -s /etc/nginx/sites-available/company.conf /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/api.conf /etc/nginx/sites-enabled/

# Disable a site instantly without deleting config
rm /etc/nginx/sites-enabled/api.conf
sudo systemctl reload nginx    # never reboot — reload only

# Re-enable instantly
ln -s /etc/nginx/sites-available/api.conf /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

---

### Commands summary

```bash
# Hard link
ln original.txt hardlink.txt

# Symlink — target first, link name second
ln -s /path/to/target linkname

# Check what a symlink points to
ls -la /path/

# Find broken symlinks
find -L /path -xtype l

# Delete broken symlinks
find -L /path -xtype l -delete
```
