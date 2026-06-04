# Vim — Cloud Engineer Survival Guide

## The honest truth
You only need vim on AWS when SSH'd into a server with no other editor.
Learn the survival commands cold. Everything else is optional.

---

## Phase 1 — Survival commands (learn these now)

### Modes — the most important concept in vim
Vim has modes. This is what confuses everyone at first.
You are ALWAYS in one of these modes:

```
Normal mode   → default when you open vim
                navigation, deletion, copy/paste
                press Esc to return here from anywhere

Insert mode   → for typing text
                press i to enter from normal mode

Visual mode   → for selecting text
                press v to enter from normal mode
                press V for whole line selection
```

The #1 vim mistake: trying to type when you're in normal mode.
Always check your mode. When in doubt — press Esc first.

---

### Opening and quitting

```bash
vim filename          # open file
vim +42 filename      # open file at line 42
vim +/pattern file    # open file at first match of pattern
```

```
# Inside vim — saving and quitting (most important)
:wq       save and quit
:w        save without quitting
:q!       quit WITHOUT saving (force)
:wq!      save and quit (force)
:x        save and quit (same as :wq)
```

---

### Navigation (normal mode)

```
# Basic movement
h         move left
j         move down
k         move up
l         move right

# Word movement
w         move to beginning of next word
b         move to beginning of previous word
e         move to end of current word

# Line movement
0         move to beginning of line
^         move to first non-blank character
$         move to end of line

# File movement
gg        go to top of file
G         go to bottom of file
:42       go to line 42
Ctrl+f    page down
Ctrl+b    page up

# Screen positioning
zt        current line to top of screen
zz        current line to middle of screen
zb        current line to bottom of screen
H         jump to highest line on screen
M         jump to middle line on screen
L         jump to lowest line on screen
```

---

### Editing (normal mode)

```
# Insert mode entry points
i         insert before cursor
a         insert after cursor
o         open new line below cursor
O         open new line above cursor
I         insert at beginning of line
A         insert at end of line

# Deleting
x         delete character under cursor
X         delete character before cursor
dd        delete entire line
3dd       delete 3 lines
dw        delete word
D         delete to end of line
d$        delete to end of line (same as D)
d0        delete to beginning of line

# Copying (yanking)
yy        copy entire line
2yy       copy 2 lines
yw        copy word
y$        copy to end of line

# Pasting
p         paste after cursor
P         paste before cursor

# Undo / redo
u         undo last change
Ctrl+r    redo

# Changing
cw        delete word and enter insert mode
cc        delete line and enter insert mode
C         delete to end of line and enter insert mode
s         delete character and enter insert mode
S         delete line and enter insert mode
r         replace single character (stays in normal mode)
```

---

### Searching

```
/pattern      search forward for pattern
?pattern      search backward for pattern
n             jump to next match
N             jump to previous match
*             search for word under cursor (forward)
#             search for word under cursor (backward)
```

---

### Visual mode (selecting text)

```
v             enter visual mode (character selection)
V             enter visual mode (line selection)
Ctrl+v        enter visual block mode (column selection)

# After selecting:
y             copy selection
d             delete selection
>             indent selection
<             unindent selection
```

---

### Line numbers and settings

```
:set number       show line numbers
:set nonumber     hide line numbers
:set hlsearch     highlight search results
:set ignorecase   case-insensitive search
:syntax on        enable syntax highlighting
```

---

### Multiple files and windows

```
:sp filename      split horizontally, open file
:vsp filename     split vertically, open file
Ctrl+w w          switch between splits
Ctrl+w q          close current split
:e filename       open file in current window
```

---

## Phase 2 — Advanced motions (learn after hired)

```
# Jump motions
fc        jump to next occurrence of character c
Fc        jump to previous occurrence of character c
tc        jump to character before next c
Tc        jump to character after previous c
;         repeat last f/F/t/T motion forward
,         repeat last f/F/t/T motion backward

# Text objects (powerful combinations)
diw       delete inner word
daw       delete word including spaces
di"       delete inside quotes
da"       delete quotes and content
ci(       change inside parentheses
```

---

## Real AWS usage patterns

```bash
# Edit nginx config
sudo vim /etc/nginx/nginx.conf

# Quick edit — go to line, make change, save
vim +42 /etc/nginx/nginx.conf    # open at line 42
# press i → make change → Esc → :wq

# Find and fix a config value
vim /etc/ssh/sshd_config
/PermitRootLogin                  # find the line
i                                 # enter insert mode
# make change
Esc → :wq                        # save and quit
sudo systemctl reload sshd        # apply change

# Quick view without editing
vim -R filename                   # read-only mode
# or just use cat/less for viewing
```

---

## The 8 commands you must know cold

```
i          enter insert mode
Esc        return to normal mode
:wq        save and quit
:q!        quit without saving
hjkl       navigate
dd         delete line
u          undo
/pattern   search
```

If you only learn 8 things from this file — learn these 8.
Everything else you can look up.

---

## Learning tip — vimtutor
The best way to learn vim is built into Linux:
```bash
vimtutor    # interactive 30-minute tutorial
            # run this once, it covers everything in Phase 1
```
