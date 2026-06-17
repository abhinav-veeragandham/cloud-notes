# Section 7 — Text Processing Commands

---

## cut — Extract columns from text

```bash
cut -d: -f1 /etc/passwd          # extract first field, colon delimiter
cut -d, -f2 file.csv             # extract second field, comma delimiter
cut -d: -f1,3 /etc/passwd        # extract fields 1 and 3
cut -c1-5 file                   # extract first 5 characters
cut -d: -f3 /etc/passwd | sort -n  # extract UIDs, sort numerically
```

---

## sort — Sort lines of text

```bash
sort file                        # alphabetical sort
sort -n file                     # numeric sort
sort -r file                     # reverse order
sort -u file                     # remove duplicates
sort -k2 file                    # sort by second field
sort -k2 -n file                 # sort by second field numerically
sort -rn file                    # numeric reverse sort
```

---

## tr — Translate or delete characters

```bash
tr 'a-z' 'A-Z' < file           # lowercase to uppercase
tr -d ' ' < file                 # delete spaces
tr -s ' ' < file                 # squeeze multiple spaces into one
tr ':' ',' < file                # replace colon with comma
tr -d '\n' < file                # remove newlines
```

---

## awk — Field-based text processing

```bash
awk '{print $1}' file                    # print first field
awk '{print $1, $3}' file               # print fields 1 and 3
awk '{print $NF}' file                  # print last field
awk -F: '{print $1}' /etc/passwd        # colon delimiter
awk -F: '{print $1, $3}' /etc/passwd    # username and UID
awk '/error/ {print $0}' app.log        # lines matching pattern
awk '$3 > 100 {print $1}' file          # conditional on field value
awk 'END {print NR}' file               # total line count
```

---

## sed — Stream editor

```bash
sed 's/old/new/' file                   # replace first occurrence per line
sed 's/old/new/g' file                  # replace all occurrences
sed -n '2p' file                        # print line 2 only
sed '3d' file                           # delete line 3
sed -i 's/old/new/g' file              # edit file in place
sed -n '2,4p' file                      # print lines 2 to 4
sed '/pattern/d' file                   # delete lines matching pattern
```

---

## grep — Regex pattern matching

### Basic anchors and wildcards
```bash
grep '^l' file                          # lines starting with l
grep 'anna$' file                       # lines ending with anna
grep '^.$' file                         # single-character lines only
grep '\banna\b' file                    # anna as standalone word
grep '^anna\b' file                     # line starts with anna as whole word
grep 'n.*x' file                        # n followed by anything then x
grep '..*[[:space:]]..*' file           # lines with at least two words
```

### Basic vs Extended regex
```bash
grep -E 'bi+t' file                     # one or more i's
grep -E 'bi?t' file                     # zero or one i (bt or bit)
grep -E 'bi*t' file                     # zero or more i's
grep 'bo\{3\}n' file                    # exactly 3 o's (BRE)
grep -E 'bo{3}n' file                   # exactly 3 o's (ERE)
grep -E 'bo{1,3}n' file                 # between 1 and 3 o's (ERE)
grep -E 'svm|vmx' file                  # OR — match svm or vmx
```

### Real-world patterns
```bash
grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' file   # IP addresses
grep -E '([A-Fa-f0-9]{2}:){5}[A-Fa-f0-9]{2}' /var/log/messages  # MAC addresses
grep -E '\b[0-9]{4}-[0-9]{2}-[0-9]{2}\b' /var/log/messages       # YYYY-MM-DD dates
grep -o -E 'https?://[a-zA-Z0-9./=_-]+' file.txt                 # extract URLs
grep -E '^[0-9]' file                                              # lines starting with a digit
```

---

## Quantifier cheat sheet

| Symbol | Meaning        | Example  | Matches              |
|--------|----------------|----------|----------------------|
| `?`    | Zero or one    | `bi?t`   | `bt`, `bit`          |
| `+`    | One or more    | `bi+t`   | `bit`, `biit`        |
| `*`    | Zero or more   | `bi*t`   | `bt`, `bit`, `biit`  |
| `{3}`  | Exactly 3      | `bo{3}n` | `booon`              |
| `{2,4}`| Between 2 and 4| `bo{2,4}n`| `boon`, `booon`    |

---

## BRE vs ERE syntax

| Feature    | BRE (default) | ERE (`-E`) |
|------------|---------------|------------|
| One or more| `\+`          | `+`        |
| Zero or one| `\?`          | `?`        |
| OR         | `\|`          | `\|`       |
| Exactly n  | `\{n\}`       | `{n}`      |
