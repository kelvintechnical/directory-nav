# Lab 05: Directory Navigation — `cd`, `pwd`, `ls`

**Series:** File Operations & Shell Fundamentals · **Lab 5 of the Novice → RHCA path**  
**Certifications covered:** RHCSA EX200 (foundational), RHCE EX294 (every Ansible path reference), CKA (every kubelet/etcd/containerd path), RHCA building blocks (RH342 troubleshooting, RH358 services, RH236 storage)  
**Prerequisite:** Lab 04 (Shell basics)  
**Time Estimate:** 30–40 minutes  
**Difficulty arc:** Tasks 1–6 foundation · 7–12 practical · 13–17 advanced · 18–20 exam-realistic

---

## 🎯 Objective

By the end of this lab you will move through the Linux filesystem with the same confidence a senior engineer does — without ever pausing to think "where am I?" or "what's in here?". Every other lab, every exam task, and every production troubleshooting session begins with these three commands.

---

## 🧠 Concept: The Linux Filesystem Is a Tree

Linux has a **single root directory**: `/`. Everything — every disk, every user file, every device, every kubelet manifest — lives under `/`. There are no drive letters like `C:\` or `D:\`.

```
/                          ← root of everything
├── bin/                   ← essential user binaries (ls, cp, mv)
├── boot/                  ← kernel + bootloader
├── dev/                   ← device files (disks, USB, terminals)
├── etc/                   ← system configuration (sshd_config, fstab, kubernetes/)
├── home/                  ← regular user home directories
│   └── ec2-user/          ← YOUR home directory
├── root/                  ← the root user's home (NOT the same as /)
├── tmp/                   ← temporary files (often cleared on reboot)
├── usr/                   ← installed software
├── var/                   ← logs, spool, caches, kubelet data
└── opt/                   ← optional/third-party software
```

### Path types in one table

| Type | Starts with | Example | Meaning |
|---|---|---|---|
| **Absolute** | `/` | `/etc/ssh/sshd_config` | Full path from root — works from anywhere |
| **Relative** | not `/` | `ssh/sshd_config` | Path **from where you currently are** |
| **Home** | `~` | `~/notes.txt` | Shortcut to your home directory |
| **Parent** | `..` | `../logs` | Up one level, then into `logs` |
| **Current** | `.` | `./script.sh` | Right here |

> **Rule of thumb on every exam:** Use absolute paths when the task gives you one (e.g., `/var/tmp/file`). Use relative paths only when you are already deep in a directory and want to save typing.

---

## 📚 Command Reference

| Command | Purpose | Critical flags |
|---|---|---|
| `pwd` | **P**rint **W**orking **D**irectory — shows where you are | `-P` (physical/resolve symlinks), `-L` (logical, default) |
| `cd` | **C**hange **D**irectory — moves you | `cd -` (previous), `cd ~user` (another user's home) |
| `ls` | **L**i**S**t directory contents | `-l`, `-a`, `-A`, `-h`, `-R`, `-t`, `-S`, `-d`, `-1`, `-F`, `-i` |

---

## 🛣️ RHCA Pathway Sidebar

| Cert level | Why this lab matters |
|---|---|
| **Foundation** | The Linux Foundation, LFCS, and RHCSA all assume Day-1 fluency |
| **RHCSA EX200** | Every graded task gives you a path — you must navigate to it |
| **RHCE EX294** | Ansible inventories, playbooks, roles all reference paths |
| **CKA** | `/etc/kubernetes/*`, `/var/lib/kubelet`, `/var/log/pods`, `/etc/cni/net.d` |
| **RHCA — RH342 (Troubleshooting)** | "What changed recently?" → `ls -ltr /var/log` is move #1 |
| **RHCA — RH358 (Services)** | Each service has a config directory you must learn to navigate fast |
| **RHCA — RH236 (Storage)** | GlusterFS bricks, Ceph mounts — all under specific paths |

---

## 🔧 The 20 Tasks

> Each task ends with three short callouts: **Switches** (every flag explained), **Output decoded** (every line/column explained), and **Troubleshoot** (what to do if it goes wrong).

---

### Task 1 — Discover where you are with `pwd`

**Purpose:** Before anything else, find out where you stand. Sysadmins call this "orienting yourself" — running `pwd` after `ssh`-ing into a server is a reflex.

```bash
pwd
```

**Expected output:**

```
/home/ec2-user
```

**Switches**

| Token | Meaning |
|---|---|
| `pwd` | **P**rint **W**orking **D**irectory — built-in shell command (also a binary at `/usr/bin/pwd`) |

**Output decoded**

| Line | What it tells you |
|---|---|
| `/home/ec2-user` | The **logical path** of your current shell — the chain of names you took to get here |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `pwd: command not found` | You broke `$PATH` — type `/usr/bin/pwd` directly |
| Output unexpected (e.g., `/`) | You probably `cd /` somewhere and forgot — proceed deliberately |

---

### Task 2 — Resolve symlinks with `pwd -P`

**Purpose:** When the path you see is a symbolic link, the **physical** location may be elsewhere. Container runtimes, GlusterFS bricks, and `/var/run` (often a symlink to `/run`) make this critical.

```bash
cd /var/run
pwd
pwd -P
```

**Expected output:**

```
/var/run
/run
```

**Switches**

| Flag | Meaning |
|---|---|
| (none) | Default = `-L` = logical path (through symlinks) |
| `-P` | **P**hysical path — resolves every symlink to the real directory |
| `-L` | **L**ogical (default) — keep the chain of names you walked |

**Output decoded**

| Line | What it tells you |
|---|---|
| `/var/run` | What your shell thinks — looks like you're under `/var` |
| `/run` | Where you actually are on disk — `/var/run` is a symlink to `/run` |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `pwd` and `pwd -P` always match | No symlinks in your path — that's fine, it's the common case |
| You expected a symlink but `pwd -P` didn't change | The symlink may be lower in the tree — try `readlink -f $(pwd)` |

---

### Task 3 — Move with an absolute path

**Purpose:** Absolute paths always work from anywhere. The exam usually hands you one — never guess.

```bash
cd /etc
pwd
```

**Expected output:**

```
/etc
```

**Switches**

| Token | Meaning |
|---|---|
| `cd` | **C**hange **D**irectory — shell built-in (no separate binary exists) |
| `/etc` | Absolute path — the leading `/` means "start from root" |

**Output decoded**

| Line | What it tells you |
|---|---|
| (silence after `cd`) | **Silence = success.** Almost every navigation command in Linux follows this rule |
| `/etc` | `pwd` confirms the move succeeded |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `bash: cd: /etc: Permission denied` | The directory exists but you lack execute (`x`) on it — try `sudo` for the inspection, or pick a different parent |
| `bash: cd: /eyc: No such file or directory` | Typo — tab-complete: type `cd /e` then Tab |

---

### Task 4 — Move with a relative path

**Purpose:** You can save typing by referencing what's near you instead of repeating the full path. This is how scripts that "just work in their own directory" do it.

```bash
cd ssh
pwd
```

(You should still be in `/etc` from Task 3.)

**Expected output:**

```
/etc/ssh
```

**Switches**

| Token | Meaning |
|---|---|
| `cd` | Change directory |
| `ssh` | Relative path — no leading `/`, so the shell appends it to your current location |

**Output decoded**

| Line | What it tells you |
|---|---|
| `/etc/ssh` | `pwd` shows the shell appended `ssh` to your prior `/etc` location |

**Why a sysadmin needs this:** Configuration directories (`/etc/ssh`, `/etc/httpd`, `/etc/kubernetes`) all sit inside `/etc`. Walking into `/etc` once and then using relative `cd` is faster than typing the full path each time.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `bash: cd: ssh: No such file or directory` | You're not where you thought — run `pwd` then try again |

---

### Task 5 — Move up one level with `..`

**Purpose:** `..` always means "the parent directory." It's how you back out without retyping the entire path.

```bash
cd /etc/ssh
cd ..
pwd
cd ../var/log
pwd
```

**Expected output:**

```
/etc
/var/log
```

**Switches**

| Token | Meaning |
|---|---|
| `..` | Parent of the current directory |
| `../..` | Two levels up |
| `../var/log` | Up one level from `/etc/ssh` to `/etc`, then down into `/etc`'s sibling `/var/log` |

**Output decoded**

| Line | What it tells you |
|---|---|
| `/etc` | After `cd ..` from `/etc/ssh`, you landed in the parent |
| `/var/log` | From `/etc`, `cd ../var/log` went up to `/` then down through `var/log` |

**Why a sysadmin needs this:** When debugging, you often want to jump from `/etc/httpd/conf.d` to `/var/log/httpd` — `cd ../../../var/log/httpd` is faster than retyping.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| You end up at `/` | You used `..` one too many times — `cd` somewhere known: `cd ~` |

---

### Task 6 — Go home three different ways

**Purpose:** Every shell session has a "home" directory. Knowing all three ways to get there saves keystrokes and recovers you from any mess.

```bash
cd /etc
cd ~
pwd
cd /tmp
cd
pwd
cd /var
cd $HOME
pwd
```

**Expected output:**

```
/home/ec2-user
/home/ec2-user
/home/ec2-user
```

**Switches / tokens**

| Token | Meaning |
|---|---|
| `~` | Tilde — shell shorthand for `$HOME` |
| (no arg) | `cd` with no argument also goes home |
| `$HOME` | Environment variable holding your home path |

**Output decoded**

| Line | What it tells you |
|---|---|
| Each `/home/ec2-user` | All three idioms produce identical results |

**Why a sysadmin needs this:** During an emergency, `cd` (no args) is the fastest "panic button" — it always lands you somewhere safe.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `cd ~user` says `No such user` | Username typo or the user doesn't exist on the system |
| `$HOME` is empty | Environment is broken — log out and back in |

---

### Task 7 — Toggle between two directories with `cd -`

**Purpose:** `cd -` swaps you back to your previous directory. Indispensable when bouncing between two locations.

```bash
cd /var/log
cd /etc/ssh
cd -
pwd
cd -
pwd
```

**Expected output:**

```
/var/log
/var/log
/etc/ssh
/etc/ssh
```

**Switches**

| Token | Meaning |
|---|---|
| `cd -` | Move to `$OLDPWD` (the directory you were in **before** the last `cd`) |

**Output decoded**

| Line | What it tells you |
|---|---|
| `/var/log` (first time) | The shell **echoes** the destination it just moved to |
| `/var/log` (second time) | `pwd` confirms |
| `/etc/ssh` | Toggled back — `cd -` is a flip-flop, not a stack |

**Why a sysadmin needs this on CKA:** Reproducible — you bounce between `/etc/kubernetes/manifests` (manifests) and `/var/log/pods` (logs) every other command.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `bash: cd: OLDPWD not set` | This is your first `cd` in this session — do another `cd` first |

---

### Task 8 — Visit another user's home with `~user`

**Purpose:** The `~user` form expands to that user's home directory. Useful for inspecting service accounts (`apache`, `nginx`, `postgres`).

```bash
cd ~root 2>/dev/null && pwd
cd /home && ls -d ~$(whoami)
```

**Expected output (on most systems):**

```
bash: cd: /root: Permission denied
/home/ec2-user
```

**Switches**

| Token | Meaning |
|---|---|
| `~root` | Expands to root's home directory (`/root`) |
| `~$(whoami)` | Expands to **your own** home — `$(whoami)` substitutes your username first |
| `2>/dev/null` | Suppress the permission-denied stderr |

**Output decoded**

| Line | What it tells you |
|---|---|
| `Permission denied` | Expected — non-root users can't enter `/root` |
| `/home/ec2-user` | Your home, shown by `ls -d` |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `bash: cd: ~admin: No such file or directory` | The user has no home directory yet — `sudo useradd -m admin` to give them one |

---

### Task 9 — List with plain `ls`

**Purpose:** See what's in a directory. The default is alphabetical, multi-column, and **hides files starting with `.`**.

```bash
cd /etc
ls
```

**Expected output (truncated):**

```
adjtime         centos-release  chrony.conf   crypttab    ec2_version  ...
alternatives    chkconfig.d     cron.d        dbus-1      environment
audit           chrony.keys     cron.daily    default     ethertypes
```

**Switches**

| Token | Meaning |
|---|---|
| `ls` | List — no flags = short, colored, multi-column, no dotfiles |

**Output decoded**

| Element | Meaning |
|---|---|
| Each name | A file or directory inside `/etc` |
| Colors (if enabled) | Blue = directory, green = executable, cyan = symlink, white = regular file |
| Sort order | Alphabetical, case-insensitive on GNU `ls` |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| No colors | Your shell may not be passing `--color=auto` — set `alias ls='ls --color=auto'` in `~/.bashrc` |

---

### Task 10 — Reveal hidden files with `ls -a`

**Purpose:** Configuration files in your home directory start with `.` — `.bashrc`, `.ssh/`, `.kube/`. Without `-a`, you don't see them.

```bash
cd ~
ls -a
```

**Expected output:**

```
.   ..   .bash_history   .bash_logout   .bash_profile   .bashrc   .ssh
```

**Switches**

| Flag | Meaning |
|---|---|
| `-a` | **A**ll — include entries beginning with `.` |

**Output decoded**

| Entry | Meaning |
|---|---|
| `.` | The current directory itself (every dir contains a reference to itself) |
| `..` | The parent directory |
| `.bash_history` | Log of your previous bash commands |
| `.bashrc` | Per-shell startup config |
| `.ssh` | Directory containing SSH keys and `known_hosts` |

**Why a sysadmin needs this on CKA:** `~/.kube/config` is hidden. Without `-a`, beginners panic that their kubeconfig is "missing."

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `.` and `..` clutter scripts | Use `-A` (Task 11) instead |

---

### Task 11 — "Almost all" with `ls -A`

**Purpose:** Same as `-a` but excludes `.` and `..`. Use this in scripts so you don't accidentally process the current and parent directories.

```bash
cd ~
ls -A
```

**Expected output:**

```
.bash_history   .bash_logout   .bash_profile   .bashrc   .ssh
```

**Switches**

| Flag | Meaning |
|---|---|
| `-A` | **A**lmost all — include dotfiles, exclude `.` and `..` |

**Output decoded**

| Entry | Meaning |
|---|---|
| Same as Task 10 minus `.` and `..` | Cleaner list for piping into other commands |

**Why a sysadmin needs this:** Running `for f in $(ls -a)` would loop over `.` and `..` — disaster. Use `-A`.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Confused which to use | Rule: humans → `-a`, scripts → `-A` |

---

### Task 12 — Read the long listing with `ls -l`

**Purpose:** The most important `ls` flag. Shows type, permissions, owner, group, size, mtime, name. This is where every "is this file set up correctly?" question gets answered.

```bash
ls -l /etc/passwd /etc/ssh
```

**Expected output:**

```
-rw-r--r--. 1 root root 2876 Sep  3 10:42 /etc/passwd
/etc/ssh:
total 612
-rw-r-----. 1 root ssh_keys  492 Mar 14  2024 moduli
-rw-r--r--. 1 root root      577 Mar 14  2024 ssh_config
drwxr-xr-x. 2 root root        6 Mar 14  2024 ssh_config.d
-rw-------. 1 root root     4096 Mar 14  2024 sshd_config
drwxr-xr-x. 2 root root       40 Mar 14  2024 sshd_config.d
```

**Switches**

| Flag | Meaning |
|---|---|
| `-l` | **L**ong listing — full metadata, one entry per line |

**Output decoded — every column of `-rw-r--r--. 1 root root 2876 Sep  3 10:42 /etc/passwd`**

| Column | Value | Meaning |
|---|---|---|
| 1 (chars 1) | `-` | File type (`-` regular, `d` dir, `l` symlink, `c`/`b` device, `s` socket, `p` pipe) |
| 1 (chars 2–4) | `rw-` | Owner permissions (read, write, no execute) |
| 1 (chars 5–7) | `r--` | Group permissions (read only) |
| 1 (chars 8–10) | `r--` | Other permissions (read only) |
| 1 (char 11) | `.` | Has SELinux context (`+` would mean ACL, none = neither) |
| 2 | `1` | Hard link count |
| 3 | `root` | Owning user |
| 4 | `root` | Owning group |
| 5 | `2876` | Size in **bytes** |
| 6 | `Sep 3 10:42` | Last modification timestamp |
| 7 | `/etc/passwd` | The name |
| `total 612` (in dir listing) | Sum of disk blocks (in 1 KiB units) used by listed files |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Size in bytes is hard to read | Add `-h` (Task 13) |
| `total` confuses you | Ignore unless doing disk-usage forensics — it's not file count |

---

### Task 13 — Human-readable sizes with `-h`

**Purpose:** When sizes get big (MB, GB), raw bytes become unreadable. `-h` adds units.

```bash
ls -lh /var/log/messages /var/log/wtmp /var/log/lastlog
```

**Expected output:**

```
-rw-------. 1 root root  3.2M Sep 10 08:15 /var/log/messages
-rw-rw-r--. 1 root utmp  4.2K Sep 10 08:15 /var/log/wtmp
-rw-r--r--. 1 root root  1.2M Sep 10 08:15 /var/log/lastlog
```

**Switches**

| Flag | Meaning |
|---|---|
| `-l` | Long listing |
| `-h` | **H**uman-readable sizes (`K` = 1024, `M` = 1024², `G` = 1024³) |

**Output decoded**

| Token | Meaning |
|---|---|
| `3.2M` | ~3.2 megabytes (3,355,443 bytes) |
| `4.2K` | ~4.2 kilobytes |
| `1.2M` | ~1.2 megabytes |

**Variants to know**

| Flag | Meaning |
|---|---|
| `-h` | Binary units (1K = 1024) |
| `--si` | SI units (1K = 1000) — useful for matching marketing specs |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `-h` does nothing | It only works **with** `-l` or `-s` — combine them |

---

### Task 14 — Show the directory itself with `-d`

**Purpose:** Without `-d`, `ls -l /etc` dumps the contents. With `-d`, you see only `/etc`'s own metadata. Critical for verifying directory permissions.

```bash
ls -ld /etc /home /tmp
```

**Expected output:**

```
drwxr-xr-x. 145 root     root      8192 Sep 10 09:00 /etc
drwxr-xr-x.   4 root     root        37 Aug 12 10:00 /home
drwxrwxrwt.  18 root     root       420 Sep 10 09:30 /tmp
```

**Switches**

| Flag | Meaning |
|---|---|
| `-l` | Long listing |
| `-d` | List **d**irectory entries themselves, not contents |

**Output decoded**

| Token | Meaning |
|---|---|
| `drwxr-xr-x.` | Directory, owner full, group read+execute, other read+execute |
| `drwxrwxrwt.` | The `t` at the end = **sticky bit** — anyone can create in `/tmp` but only the file owner can delete |
| `145` | `/etc` contains 145 directory entries (including `.` and `..`) reflected in its link count |

**Why a sysadmin needs this:** Exam task: "Create `/foo` with mode 0750, owner alice, group developers." Verify with `ls -ld /foo`.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| You see contents instead of metadata | You forgot `-d` — `ls -l` of a directory descends into it |

---

### Task 15 — Sort by time with `-t`, reverse with `-r`

**Purpose:** "What changed recently?" is the #1 troubleshooting question. `ls -ltr` puts newest at the **bottom** — where your eyes naturally land.

```bash
ls -lt /var/log | head -5
ls -ltr /var/log | tail -5
```

**Expected output (first command — newest at top):**

```
total 5240
-rw-------. 1 root   root      8132 Sep 10 09:30 secure
-rw-------. 1 root   root    314722 Sep 10 09:25 messages
-rw-rw-r--. 1 root   utmp      4224 Sep 10 09:00 wtmp
-rw-------. 1 root   root      1024 Sep 10 08:00 audit/audit.log
```

**Switches**

| Flag | Meaning |
|---|---|
| `-l` | Long listing |
| `-t` | Sort by modification **t**ime, newest first |
| `-r` | **R**everse — flip the sort |
| `\| head -5` | Show only top 5 lines |
| `\| tail -5` | Show only last 5 lines |

**Output decoded**

| Token | Meaning |
|---|---|
| `total 5240` | Sum of blocks (1 KiB units) |
| Each row | A log file, with mtime visible in column 6 |
| Order | Newest file on top (`-t`) — flipped if you add `-r` |

**Why on RHCA RH342:** Troubleshooting after a crash: `ls -ltr /var/log | tail -10` shows the last few log files that were touched — often the smoking gun.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| All times look the same | Files updated within the same second — add `--full-time` for sub-second precision |

---

### Task 16 — Sort by size with `-S` and find the biggest

**Purpose:** Disk full? `ls -lS` shows the giants first.

```bash
ls -lhS /var/log | head -5
```

**Expected output:**

```
total 5.2M
-rw-------. 1 root  root   3.2M Sep 10 09:25 messages
-rw-------. 1 root  root   1.2M Sep 10 08:15 lastlog
-rw-------. 1 root  root   600K Sep 10 09:30 secure
-rw-r--r--. 1 root  root   105K Sep  9 10:00 dnf.log
```

**Switches**

| Flag | Meaning |
|---|---|
| `-l` | Long listing |
| `-h` | Human-readable sizes |
| `-S` | Sort by **S**ize, largest first |

**Output decoded**

| Token | Meaning |
|---|---|
| First row | The largest file in the directory |
| Subsequent rows | Descending size order |

**Why a sysadmin needs this:** Pair with `du -sh */ \| sort -h` for directories. RHCSA Task 11 (tar): you need to know which logs are big enough to bother archiving.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Wanted smallest first | Add `-r`: `ls -lShr` |

---

### Task 17 — Recursive listing with `-R` (carefully)

**Purpose:** Walk into every subdirectory. Useful for auditing a small tree; **dangerous** at high levels.

```bash
ls -R /etc/ssh
```

**Expected output:**

```
/etc/ssh:
moduli  ssh_config  ssh_config.d  sshd_config  sshd_config.d

/etc/ssh/ssh_config.d:
05-redhat.conf

/etc/ssh/sshd_config.d:
50-redhat.conf
```

**Switches**

| Flag | Meaning |
|---|---|
| `-R` | **R**ecursive — descend into every subdirectory |

**Output decoded**

| Element | Meaning |
|---|---|
| `/etc/ssh:` | Header — the directory whose contents follow |
| Items below | Names inside that directory |
| Blank line | Separator between directories |

> ⚠️ **Never run `ls -R /`** — it tries to list the entire filesystem.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Wall of text | Pipe to `less` or restrict with `find /etc/ssh -maxdepth 2` |

---

### Task 18 — One per line with `-1` and classify with `-F`

**Purpose:** Script-friendly output; visual cues for file types.

```bash
ls -1F /etc/ssh
ls /etc | wc -l
```

**Expected output:**

```
moduli
ssh_config
ssh_config.d/
sshd_config
sshd_config.d/
```

(`wc -l` returns a number — e.g., `184`.)

**Switches**

| Flag | Meaning |
|---|---|
| `-1` | One entry per **line** |
| `-F` | Classify — append a character indicating type |
| `\| wc -l` | Count lines |

**Classification suffixes from `-F`**

| Suffix | Meaning |
|---|---|
| `/` | Directory |
| `*` | Executable file |
| `@` | Symbolic link |
| `=` | Socket |
| `\|` | FIFO/named pipe |
| (none) | Regular file |

**Output decoded**

| Token | Meaning |
|---|---|
| `ssh_config.d/` | The trailing `/` says: this is a directory |
| `wc -l` output | Total number of entries (counts dotfiles only if you used `ls -A`) |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Want only the count | `ls -1A /path \| wc -l` is the idiom |

---

### Task 19 — Inode numbers with `-i`

**Purpose:** Inodes are the disk-level identity of a file. `-i` reveals the inode number, which is essential when working with hard links (Lab 09) or recovering files by inode.

```bash
ls -li /etc/passwd /etc/shadow
```

**Expected output:**

```
4456 -rw-r--r--. 1 root root 2876 Sep  3 10:42 /etc/passwd
4457 -rw-------. 1 root root 1432 Sep  3 10:42 /etc/shadow
```

**Switches**

| Flag | Meaning |
|---|---|
| `-l` | Long listing |
| `-i` | Show **i**node number (first column) |

**Output decoded**

| Token | Meaning |
|---|---|
| `4456` | Inode number for `/etc/passwd` — unique per filesystem |
| `4457` | Inode for `/etc/shadow` — different number = different file |

**Why a sysadmin needs this:** If `find / -inum 4456` returns more than one path, those paths are hard links to the same file.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Inode numbers very large (millions) | Normal on big filesystems — they're not file sizes |

---

### Task 20 — Exam-style scenario: combine everything

**Task statement (RHCSA-style):** *"In `/var/log`, identify the three most recently modified files, confirm `/var/log/messages` exists and is owned by root, then return to your prior directory."*

```bash
cd /var/log
pwd
ls -ltrh | tail -3
ls -ld /var/log/messages
stat -c '%U %G %a %n' /var/log/messages
cd -
pwd
```

**Expected output:**

```
/var/log
-rw-r--r--. 1 root root  105K Sep  9 10:00 dnf.log
-rw-------. 1 root root  600K Sep 10 09:30 secure
-rw-------. 1 root root  3.2M Sep 10 09:35 messages
-rw-------. 1 root root 3258880 Sep 10 09:35 /var/log/messages
root root 600 /var/log/messages
/home/ec2-user
/home/ec2-user
```

**Step-by-step rationale**

| Step | Why |
|---|---|
| `cd /var/log` | Move to where the action is |
| `pwd` | Confirm — never trust without verifying |
| `ls -ltrh \| tail -3` | Long, time-sorted, reversed (newest last), human sizes — last 3 lines = 3 newest files |
| `ls -ld /var/log/messages` | Verify the specific file (metadata, not contents) |
| `stat -c '%U %G %a %n' ...` | One-line owner/group/mode/name audit (preview of Lab 06) |
| `cd -` | Toggle back to the previous directory |
| Final `pwd` | Audit trail — proves the toggle worked |

**Output decoded**

| Line | What it tells you |
|---|---|
| First `/var/log` | You moved successfully |
| Three `ls -ltrh` rows | The three most recently changed log files, biggest visible |
| `ls -ld` row | `/var/log/messages` exists, root owns it, mode `600` |
| `stat` row | Quick scan: owner = root, group = root, mode = 600, file = `/var/log/messages` |
| Second `/home/ec2-user` | `cd -` succeeded |
| Third `/home/ec2-user` | `pwd` confirmation |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `cd -` says `OLDPWD not set` | This was your first `cd` of the session — try again after another `cd` |
| `ls: /var/log/messages: No such file` | On some distros it's `/var/log/syslog` — substitute accordingly |

---

## 🔍 Navigation Decision Guide

```
Where am I?               → pwd          (add -P to resolve symlinks)
What's here?              → ls
What's REALLY here?       → ls -a
What does this look like? → ls -l        (add -h for sizes, -d for dir itself)
What changed recently?    → ls -ltrh
What's the biggest?       → ls -lhS
Inode for link debugging? → ls -li
One per line for scripts? → ls -1A

How do I move?
  ├── Full path known       → cd /absolute/path
  ├── Nearby                → cd relative/path
  ├── Up one level          → cd ..
  ├── Home                  → cd ~  or  cd  or  cd $HOME
  └── Back where I was      → cd -
```

---

## ✅ Lab Checklist (20 Tasks)

- [ ] 01 `pwd` prints the working directory
- [ ] 02 `pwd -P` resolves symlinks
- [ ] 03 `cd /etc` with absolute path
- [ ] 04 `cd ssh` with relative path
- [ ] 05 `cd ..` and chained parents
- [ ] 06 `cd`, `cd ~`, `cd $HOME` all go home
- [ ] 07 `cd -` toggles to previous directory
- [ ] 08 `cd ~user` references another user's home
- [ ] 09 Plain `ls` shows visible files only
- [ ] 10 `ls -a` reveals dotfiles
- [ ] 11 `ls -A` skips `.` and `..`
- [ ] 12 `ls -l` long listing — every column decoded
- [ ] 13 `ls -lh` human-readable sizes
- [ ] 14 `ls -ld` for directory metadata
- [ ] 15 `ls -ltr` for newest at the bottom
- [ ] 16 `ls -lhS` for largest first
- [ ] 17 `ls -R` recursive (scoped)
- [ ] 18 `ls -1F` for scripts and visual classification
- [ ] 19 `ls -li` for inode numbers
- [ ] 20 Exam combo: navigate, list, verify, return

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| Forgetting the leading `/` on absolute paths | `cd: etc: No such file or directory` | Add `/`: `cd /etc` |
| Typing `cd ~/etc` | `No such file or directory` | `/etc` is system, not in your home — drop the `~` |
| Running `ls -R /` | Terminal hangs for minutes | Always scope: `ls -R /specific/path` |
| Confusing `-a` and `-A` | Scripts process `.` and `..` accidentally | Humans use `-a`, scripts use `-A` |
| Stacking too many `..` | End up at `/` and lost | `pwd` after every `cd` until comfortable |
| Forgetting `-d` for a directory's own metadata | Floods screen with contents | `ls -ld /path` |
| `cd file.txt` on a regular file | `cd: Not a directory` | Use `cat`/`less` to read files |

---

## 📌 Exam Strategy

**RHCSA EX200**
- Always start a task with `pwd && ls -la` to confirm context.
- Use **absolute paths** when the task statement provides one.
- `ls -ld /path` is your one-line verification for any "create directory with these permissions" task.

**RHCE EX294 (Ansible)**
- Playbook paths are relative to the **playbook file**, not your shell's `pwd`.
- Inventory files commonly live at `/etc/ansible/hosts` or `./inventory` — confirm with `ls -la`.

**CKA**
- Memorize and `cd` to these without thinking:
  - `/etc/kubernetes/manifests` (static pods)
  - `/etc/kubernetes/pki` (certs)
  - `/var/lib/kubelet` (kubelet state)
  - `/var/log/pods` (container logs)
  - `/etc/cni/net.d` (CNI config)
- `cd -` saves real seconds when bouncing between manifests and logs.

**RHCA**
- RH342 (Troubleshooting): `ls -ltr /var/log` is the universal first move.
- RH358 (Services): each service has a config directory; muscle-memory paths win.
- RH236 (GlusterFS): bricks live under `/var/lib/glusterd` and `/data/...` paths.

---

## 🔗 Related Labs

| Lab | Connection |
|---|---|
| Lab 06 — `ls -l`, `ls -Z` | Decodes the long listing in depth |
| Lab 07 — `touch` | Creates files you'll then list with `ls` |
| Lab 08 — `cp` | Requires absolute/relative path mastery |
| Lab 10 — `mv` | Same path rules apply to renames and moves |
| Lab 11 — `rm` | Wrong directory + `rm -rf` = disaster — `pwd` first |

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
