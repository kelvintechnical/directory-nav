# Lab: Directory Navigation — `cd`, `pwd`, `ls`

**Series:** linux-ops-mastery — RHCSA Essential Tools & File Operations
**Subjects covered:** Filesystem-as-a-tree mental model, the FHS root layout, absolute vs. relative paths, the shortcut tokens `.`, `..`, `~`, `-`, `pwd` (logical vs. physical), `cd` (with `OLDPWD`/`PWD` env vars), `ls` long-listing, hidden files, time-sorted listings, recursive listings, and the universal habit "orient → list → move"
**Career arcs covered:** RHCSA (every exam task gives you a path; you must reach it fast), RHCE (Ansible inventory/role/playbook paths), CKA (`/etc/kubernetes/*`, `/var/lib/kubelet`, `/var/log/pods`), SRE (incident triage starts with `cd /var/log && ls -ltr`), DevOps (project tree navigation, container working directories), AI/MLOps (model registry, dataset, checkpoint paths)
**Prerequisite:** A working shell where you can type a command. No prior Linux knowledge required beyond "I can run `whoami`."
**Time Estimate:** 30 to 45 minutes
**Difficulty arc:** Task 1 foundation (`pwd`) · 2–3 absolute and relative paths · 4 `ls` long/hidden/time sorts · 5 shortcut tokens and the OLDPWD trick · 6 RHCSA exam-realistic capstone

---

## Objective

Move through the Linux filesystem with the same reflex as a senior engineer — without ever pausing to think *"where am I?"* or *"what's in here?"*. Every other lab, every exam task, and every production triage session begins with `pwd`, `ls -la`, and a deliberate `cd`. By the end of this lab those three commands are fingertip-speed reflexes and you understand the path language they speak.

The capstone is an exam-realistic prompt: *"From your home directory, change to `/etc/ssh`, list every file (including hidden) with a long timestamp listing sorted by modification time, save that listing to `/root/sshd-tree.txt`, and return to your prior directory."*

> **Lab safety note:** This lab is read-only on `/etc` and `/var`. Nothing modifies a system file. The only writes are to your home directory and `/tmp/nav-lab`.

---

## Concept: The Linux Filesystem Is One Tree

Linux has **one root directory**: `/`. Every disk, every device, every user file, every config — everything — lives under `/`. There are no drive letters; there is no `C:\`. A mounted USB stick shows up at `/mnt/usb` or `/media/USERNAME/STICK`. A second hard drive shows up as a directory like `/data`. Everything is one tree.

```
/                              ← root of EVERYTHING
├── bin/        → /usr/bin     ← essential user binaries (ls, cp, mv, cat)
├── boot/                      ← kernel + bootloader
├── dev/                       ← device files (disks, USB, terminals)
├── etc/                       ← system configuration (sshd_config, fstab, kubernetes/)
├── home/                      ← regular user home directories
│   └── ec2-user/              ← YOUR home directory ($HOME, ~)
├── lib/        → /usr/lib     ← essential shared libraries
├── proc/                      ← in-memory kernel state (process info)
├── root/                      ← the root user's home (NOT the same as /)
├── run/                       ← runtime state (PID files, sockets)
├── sbin/       → /usr/sbin    ← system administration binaries
├── srv/                       ← service data
├── sys/                       ← in-memory kernel objects (sysfs)
├── tmp/                       ← temporary files (often cleared on reboot)
├── usr/                       ← installed software
└── var/                       ← logs, spool, caches, kubelet data
```

The fact that the tree starts from a single root means **every file has exactly one absolute path** — a sequence of directory names joined by `/`, starting from `/`. `/etc/ssh/sshd_config` is the same file regardless of where you currently stand.

> **Why this matters:** Every RHCSA task starts *"in `/etc/foo` do X."* If you can `cd /etc/foo` and `ls` it in three seconds, you have already converted one minute of exam time into 57 seconds of solving. Multiply across 20 tasks.

---

## 📜 Why `cd` / `pwd` / `ls` Exist — The Story

These three commands are older than the C programming language. They shipped in **Version 1 Unix (1971)**. The reason for that age is the reason they look the way they do.

When Ken Thompson designed the first Unix shell, he had one driving constraint: the user is sitting at a teletype that prints **80 columns wide** and has no screen — once a line scrolls off, it is gone. He needed commands whose names were as short as possible, because every byte of `cd` was a byte you did not have to type on a slow mechanical keyboard.

That is why:

- `cd` — *change directory*. Two characters.
- `pwd` — *print working directory*. Three characters.
- `ls` — *list*. Two characters.

These names are not abbreviations to make the syntax pretty. They are the **original minimum-typing** that survives because every operating system to come — including macOS, every Linux distro, every BSD, even WSL on Windows — copied them. The path language they share (the `/` separator, `..`, `~`, absolute vs. relative) is also from 1971 and is now universal.

> **The point of the story:** When you learn these three commands you are not learning "shell tricks." You are learning the **lowest-level interface to the filesystem** that every Unix-derived OS in the world has agreed on for 55 years. There is no upgrade. There is no replacement. There is only fluency.

---

## 👪 The Navigation Family — Who Lives There

`cd`, `pwd`, and `ls` are the trunk. A handful of close relatives matter just as much.

### By job

| Job | Command | Notes |
|---|---|---|
| Where am I? | `pwd` | Print working directory |
| Go somewhere | `cd PATH` | Change directory |
| Go home | `cd` or `cd ~` | Empty `cd` = `cd $HOME` |
| Go to previous directory | `cd -` | Swap with `$OLDPWD` |
| What is here? | `ls` | Bare listing |
| What is here, in detail? | `ls -l` | Long listing |
| Include hidden | `ls -a` | Show dotfiles |
| Human-readable sizes | `ls -h` | KB, MB, GB |
| Sort by modification time | `ls -lt` | Newest first |
| Reverse the sort | `ls -ltr` | Oldest first (newest at bottom) |
| Recursive listing | `ls -R` | Walk subtrees |
| What is this thing? | `file PATH` | Identify file type |
| Find a file | `find PATH -name PAT` | Recursive search (Lab 14) |

### Path tokens

| Token | Meaning | Example |
|---|---|---|
| `/` | The root of the tree | `/etc/passwd` |
| `.` | Right here | `./script.sh` |
| `..` | One level up | `cd ..` |
| `~` | Your home directory | `cd ~/projects` |
| `~user` | That user's home | `cd ~root` |
| `-` (with cd) | The previous directory | `cd -` |

### Path types

| Type | Starts with | Example | Resolves to |
|---|---|---|---|
| **Absolute** | `/` | `/etc/ssh/sshd_config` | Same place from anywhere |
| **Relative** | not `/` | `ssh/sshd_config` | Resolved against `$PWD` |
| **Home-anchored** | `~` | `~/notes/today.md` | Resolved against `$HOME` |
| **Parent-anchored** | `..` | `../sibling/file` | Up then over |
| **Self-anchored** | `.` | `./script.sh` | This directory, then a name |

> **The point of the family tree:** Three commands and five tokens cover 95% of navigation. The remaining 5% is `find`, `locate`, and `cd $(somecmd)`.

---

## 🔬 The Anatomy of `ls -l` Output — In One Diagram

```
$ ls -l /etc/passwd
-rw-r--r--. 1 root root 2547 May 21 14:33 /etc/passwd
│ │  │  │  │ │   │    │    └────┬────┘    └────┬────┘
│ │  │  │  │ │   │    │         │              └─ Path / name
│ │  │  │  │ │   │    │         └─ Last modification time (date and time)
│ │  │  │  │ │   │    └─ Size in bytes (use `-h` for K/M/G)
│ │  │  │  │ │   └─ Group owner
│ │  │  │  │ └─ User owner
│ │  │  │  └─ Hard link count (1 = no aliases yet)
│ │  │  └─ SELinux flag ('.' = has context; nothing = no context, '+' = has ACL)
│ │  └─ "Other" permissions (anyone else)
│ └─ "Group" permissions (members of the group owner)
└─ File type + "User" permissions
   - = regular file       d = directory       l = symlink
   c = char device        b = block device    p = named pipe (FIFO)
   s = socket
```

> **Reading rule:** Always read left-to-right and stop when you find what you need. `-` at column 1 = regular file. `d` at column 1 = directory. Permission triads `rwxrwxrwx` tell you who can read, write, execute. You will read this 10,000 times in your career.

---

## 📚 Navigation Reference Table

| Task | Command | Notes |
|---|---|---|
| Print current path | `pwd` | Use `pwd -P` to resolve symlinks |
| Change to absolute path | `cd /etc/ssh` | Works from anywhere |
| Change to relative path | `cd config/sshd_config.d` | From `$PWD` |
| Change to home | `cd` or `cd ~` | Empty = home |
| Change to previous directory | `cd -` | Toggles with `$OLDPWD`; prints the new path |
| Go up one level | `cd ..` | Repeat `..` for multiple levels |
| Combine up and over | `cd ../sibling-dir` | Move up then sideways |
| Show working directory in environment | `echo $PWD` | Set by `cd`; never edit manually |
| Show previous directory in environment | `echo $OLDPWD` | The target of `cd -` |
| Bare listing | `ls` | Names only |
| Long listing | `ls -l` | Permissions, owner, size, time |
| Long + hidden | `ls -la` | Includes dotfiles |
| Human-readable | `ls -lh` | KB / MB / GB |
| Sort by mtime, newest first | `ls -lt` | Recently changed at top |
| Sort by mtime, oldest first | `ls -ltr` | Recently changed at bottom (better for tailing) |
| Sort by size | `ls -lS` | Biggest first |
| One file per line | `ls -1` | Scriptable output |
| Append type indicators | `ls -F` | `/` = dir, `*` = exec, `@` = symlink |
| Recursive listing | `ls -R` | Walk subtree |
| Listing of a single dir, not its contents | `ls -ld DIR` | Treat dir as a file |
| Single inode number | `ls -i` | Useful for hard-link debugging |

> **Rule one of navigation:** When you sit down at any Linux shell, the first three commands you type are `whoami; pwd; ls`. Skip those and you will waste minutes troubleshooting "why doesn't this script find my file?"

---

## 🎯 Career Pathway Sidebar

| Level | Why this lab matters |
|---|---|
| **RHCSA candidate** | Every graded task is path-specific. `cd /etc/ssh && ls -la` in three keystrokes is real exam time saved. |
| **RHCE candidate** | Ansible roles assume the working dir is the role root; navigation reflex matters when debugging variable precedence. |
| **CKA candidate** | `/etc/kubernetes/manifests`, `/var/lib/kubelet`, `/var/log/pods/*` — knowing the layout is half the troubleshooting. |
| **SRE / Platform** | "What just changed in `/var/log`?" → `cd /var/log && ls -ltr` shows the newest file at the bottom in one second. |
| **DevOps** | Build pipelines change `CWD` between steps; understanding `$PWD` and `cd -` keeps scripts portable. |
| **AI / MLOps** | Datasets, checkpoints, logs, and configs each live under conventional paths (`/data`, `/scratch`, `/models`); fluent `cd` and `ls -ltr` make experiment management painless. |

---

## 🔧 The 6 Tasks

> Six exam-realistic phases that build the **orient → list → move → verify** habit.

---

### Task 1 — Orient yourself with `pwd` and the FHS

**Purpose:** Open a fresh shell, find out where the shell put you, and develop the reflex of "always know where you are before doing anything."

```bash
whoami
pwd
echo "HOME=$HOME"
echo "PWD=$PWD"
ls /
ls /etc | head -n 10
```

**Human-Readable Breakdown:** Print the current user, the current directory, the value of `$HOME`, then inspect the FHS root and a sample of `/etc`. By the end you should know what the prompt's expected starting directory is.

**Reading it left to right:** `whoami` prints your effective user. `pwd` prints the shell's current working directory. `$HOME` and `$PWD` are environment variables maintained by the shell. `ls /` shows the top-level FHS directories. `ls /etc | head -n 10` previews the most common configuration location.

**The story:** Every senior engineer's first reflex after `ssh user@host` is `whoami; pwd`. It takes one second and tells you (a) who the shell thinks you are, and (b) where the shell put you — the two facts every subsequent command depends on.

**Expected output:**

```text
ec2-user
/home/ec2-user
HOME=/home/ec2-user
PWD=/home/ec2-user
bin   boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
adjtime
aliases
alsa
alternatives
audit
bash_completion.d
bashrc
bindresvport.blacklist
binfmt.d
chrony.conf
```

**Switches**

| Token | Meaning |
|---|---|
| `whoami` | Print effective username |
| `pwd` | Print working directory |
| `pwd -P` | Resolve symlinks (physical path) |
| `pwd -L` | Show the logical path including symlinks (default) |
| `echo $VAR` | Print an env variable |
| `head -n N` | First N lines |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `pwd` printed a path with a tilde | Tilde is shell-display sugar — the real value of `$PWD` never has `~` |
| `$HOME` differs from `pwd` | The shell launched in a different dir — `cd ~` to go home |
| `ls /` is missing some entries | Modern RHEL has `/bin → /usr/bin`, `/sbin → /usr/sbin` symlinks |

---

### Task 2 — Use absolute paths to jump anywhere

**Purpose:** Practice `cd /path/to/anywhere` and prove that absolute paths work from any starting location.

```bash
cd /
pwd
ls

cd /etc/ssh
pwd
ls

cd /var/log
pwd
ls -ltr | tail -n 5
```

**Human-Readable Breakdown:** Jump to `/`, list it. Jump to `/etc/ssh`, list it. Jump to `/var/log`, list the five most recently modified entries (the `-ltr` long-time-reverse trick).

**Reading it left to right:** Each `cd /path` is unambiguous: starts at root, traverses by name, lands at the target. `pwd` confirms. `ls` shows the contents. `ls -ltr` sorts by mtime ascending so the newest file is at the bottom of the output — perfect for spotting "what changed last."

**The story:** Absolute paths are the safe default. They produce the same result regardless of where you currently stand. Relative paths are convenient for short hops in the same directory, but a script that uses relative paths will silently break the moment somebody runs it from the wrong working directory.

**Expected output:**

```text
/
bin   boot  dev  etc  home  lib  lib64  ...
/etc/ssh
ssh_config         ssh_config.d       sshd_config       sshd_config.d
ssh_host_ecdsa_key ssh_host_ed25519_key ...
/var/log
-rw------- 1 root root  ... May 26 12:48 messages
-rw-r--r-- 1 root root  ... May 26 12:48 dnf.log
-rw-r----- 1 root root  ... May 26 12:48 secure
-rw-r--r-- 1 root root  ... May 26 12:48 wtmp
-rw-r----- 1 root root  ... May 26 12:48 audit
```

**Switches**

| Token | Meaning |
|---|---|
| `cd /` | Go to root |
| `cd /etc/ssh` | Go to a deep path absolutely |
| `ls -ltr` | Long, sorted by mtime, oldest at top → newest at bottom |
| `tail -n N` | Last N lines |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `cd: /etc/ssh: No such file or directory` | Typo — `ls /etc/` to confirm the spelling |
| `Permission denied` listing `/root` | Only root can read it — `sudo ls /root` |
| `ls` shows nothing in `/var/log` | You are not in `/var/log` — `pwd` |

---

### Task 3 — Use relative paths and the shortcut tokens

**Purpose:** Practice `.`, `..`, `~`, and `cd -` so short hops in the filesystem take one keystroke instead of typing a full path.

```bash
mkdir -p /tmp/nav-lab/a/b/c
cd /tmp/nav-lab/a/b/c
pwd

cd ..
pwd

cd ../..
pwd

cd .
pwd

cd ~
pwd

cd -
pwd

cd /tmp/nav-lab
ls -F
```

**Human-Readable Breakdown:** Build a deep test tree, navigate down to the leaf, walk back up with `..`, prove `.` does nothing, jump home with `cd ~`, then bounce back with `cd -`. Finally, list the tree with `-F` to see directory markers.

**Reading it left to right:** `mkdir -p` makes every missing parent. `cd ..` goes one level up. `cd ../..` two levels. `cd .` stays put. `cd ~` jumps home. `cd -` swaps the shell between `$PWD` and `$OLDPWD` — your fastest "bounce back" toggle.

**The story:** `cd -` is the single most underused shortcut in Linux. Senior engineers do most navigation as a ping-pong between two directories — say, a project source tree and `/var/log`. `cd -` toggles between them with one keypress and prints the new directory each time. Learn it once, save it for life.

**Expected output:**

```text
/tmp/nav-lab/a/b/c
/tmp/nav-lab/a/b
/tmp/nav-lab
/tmp/nav-lab
/home/ec2-user
/tmp/nav-lab
/tmp/nav-lab
a/
```

**Switches**

| Token | Meaning |
|---|---|
| `mkdir -p P/Q/R` | Make all missing parents |
| `cd ..` | Up one level |
| `cd ../..` | Up two levels |
| `cd .` | Here (no-op) |
| `cd ~` | Home |
| `cd -` | Toggle to `$OLDPWD`; prints new path |
| `ls -F` | Append type markers (`/` for dir) |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `cd: too many arguments` | You wrote `cd ../..` (good) but with a stray space turning it into `cd .. ..` |
| `cd -` errors with `OLDPWD not set` | First-ever `cd` in this shell — make any other `cd` first |
| `~bob` does not expand | The user `bob` does not exist |

---

### Task 4 — Inspect with `ls -l`, hidden files, and time sorts

**Purpose:** Read the long-listing output, surface hidden files, and use time-sorted listings to find what changed recently.

```bash
cd /etc/ssh
ls
ls -l
ls -la
ls -lha

cd /var/log
ls -ltr | head -n 10
ls -ltr | tail -n 5

cd ~
touch test1.txt && touch test2.txt
ls -lt | head -n 5
rm test1.txt test2.txt
```

**Human-Readable Breakdown:** List `/etc/ssh` plain, then long, then long + hidden, then long + hidden + human sizes. Then in `/var/log` find the oldest and newest files via `-ltr`. Finally create two test files in your home dir and watch them appear at the top of `-lt`.

**Reading it left to right:** Each `ls` flag adds one dimension: `-l` adds permissions / owner / size / time. `-a` adds hidden files. `-h` makes sizes human-readable. `-t` sorts by mtime, `-r` reverses. `touch` creates an empty file and updates its mtime; `-lt` shows newest first.

**The story:** Long listings are how Linux exposes the metadata that matters: who owns what, when was it last changed, how big is it. Senior engineers read `ls -l` faster than English; you can too, with practice.

**Expected output:**

```text
ssh_config  ssh_config.d  sshd_config  sshd_config.d  ssh_host_ecdsa_key ...
-rw-r--r--. 1 root root 1872 May 21 14:33 ssh_config
drwxr-xr-x. 2 root root  175 May 21 14:33 ssh_config.d
-rw-------. 1 root root 4434 May 21 14:33 sshd_config
...
total 24
drwxr-xr-x.  4 root root   80 May 21 14:33 .
drwxr-xr-x. 89 root root 8192 May 26 12:48 ..
-rw-r--r--.  1 root root 1872 May 21 14:33 ssh_config
-rw-------.  1 root root 4434 May 21 14:33 sshd_config
...
total 24K
... 1.9K May 21 14:33 ssh_config
...
-rw-r--r-- 1 root root      0 May  1 03:21 oldest.log
...
-rw-r--r-- 1 root root  41234 May 26 12:48 dnf.log
-rw------- 1 root root 102433 May 26 12:48 messages
-rw-r----- 1 root root  41922 May 26 12:48 secure
...
total 16
-rw-r--r--. 1 ec2-user ec2-user    0 May 26 13:11 test2.txt
-rw-r--r--. 1 ec2-user ec2-user    0 May 26 13:11 test1.txt
drwxr-xr-x. 4 ec2-user ec2-user  117 May 26 13:11 .
...
```

**Switches**

| Token | Meaning |
|---|---|
| `ls -l` | Long listing |
| `ls -a` | Include `.` and `..` and hidden files |
| `ls -h` | Human-readable sizes |
| `ls -t` | Sort by mtime, newest first |
| `ls -r` | Reverse sort |
| `ls -F` | Add type indicators |
| `touch FILE` | Create / update mtime |
| `rm FILE` | Remove file (Lab 11) |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `ls` does not show your dotfile | Use `ls -a` |
| Sizes shown as huge raw bytes | Add `-h` |
| Order looks alphabetical, not by time | Add `-t` |
| Want the oldest first instead of newest | Add `-r` to make `-ltr` |

---

### Task 5 — Recursive listings, single-directory listings, and the `$OLDPWD` trick

**Purpose:** Use `ls -R` to walk a subtree, `ls -ld` to inspect a directory's metadata (not its contents), and chain `cd -` between two working directories for fast bouncing.

```bash
cd /tmp/nav-lab
mkdir -p src/{app,lib,tests} build/{debug,release}
touch src/app/main.c src/lib/util.c src/tests/test_main.c
touch build/debug/output.log build/release/output.log

ls -R
ls -ld src
ls -ld src/app
ls -ld /etc /etc/ssh

cd src
pwd
cd /etc/ssh
pwd
cd -
pwd
cd -
pwd
```

**Human-Readable Breakdown:** Build a multi-level project tree with brace expansion, walk the whole tree with `-R`, inspect each directory's own metadata with `-ld`, then bounce between two unrelated locations with `cd -`.

**Reading it left to right:** `mkdir -p src/{app,lib,tests}` creates three directories in one command (brace expansion). `ls -R` recurses into every subdir. `ls -ld DIR` shows DIR's permissions/owner/size/time **as a single line** rather than listing its contents. `cd -` toggles between `$PWD` and `$OLDPWD`.

**The story:** `ls -R` is the cheap-and-easy tree visualizer. `ls -ld` is critical when debugging "why can't user X enter this directory" — you need the directory's own permissions, not the contents. `cd -` is the senior engineer's project-vs-logs ping-pong.

**Expected output:**

```text
.:
build  src

./build:
debug  release

./build/debug:
output.log

./build/release:
output.log

./src:
app  lib  tests
...
drwxr-xr-x. 5 ec2-user ec2-user 36 May 26 13:13 src
drwxr-xr-x. 2 ec2-user ec2-user 22 May 26 13:13 src/app
drwxr-xr-x. 89 root root 8192 May 26 12:48 /etc
drwxr-xr-x. 4 root root  175 May 21 14:33 /etc/ssh
/tmp/nav-lab/src
/etc/ssh
/tmp/nav-lab/src
/etc/ssh
```

**Switches**

| Token | Meaning |
|---|---|
| `mkdir -p A/{B,C,D}` | Brace expansion creates A/B, A/C, A/D |
| `ls -R` | Recursive listing |
| `ls -ld DIR` | Long-list the directory itself, not its contents |
| `cd -` | Bounce between `$PWD` and `$OLDPWD` |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `ls -R` printed too much | Pipe through `\| head -n 50` or run in a smaller directory |
| `ls -ld` printed every file | You forgot the directory argument |
| `cd -` complains "OLDPWD not set" | Do one regular `cd` first |
| Brace expansion did not happen | You quoted it: `'{a,b,c}'` is literal; remove quotes |

---

### Task 6 — Capstone: RHCSA-realistic save-listing-and-return

**Task statement:** *"From your home directory, change to `/etc/ssh`, list every file (including hidden) using `-la` sorted by modification time newest-first, save that listing to `/root/sshd-tree.txt`, then return to your prior working directory."*

**Purpose:** Execute a real exam-style path navigation end-to-end, capture the artifact a grader would inspect, and prove you can return cleanly.

```bash
sudo -i

START=$PWD
cd /etc/ssh

ls -lat > /root/sshd-tree.txt

wc -l /root/sshd-tree.txt
head -n 5 /root/sshd-tree.txt
cd "$START"
pwd

# alternative one-liner without env var (uses cd - to bounce back):
cd /etc/ssh
ls -lat > /root/sshd-tree.txt
cd -

test -s /root/sshd-tree.txt && echo "VERIFY: file exists and is non-empty"
grep -c '^-' /root/sshd-tree.txt
grep -c '^d' /root/sshd-tree.txt
```

**Human-Readable Breakdown:** Become root, save the starting directory in a shell variable, `cd` to `/etc/ssh`, capture the `-lat` listing into `/root/sshd-tree.txt`, return to the starting directory. The verification block confirms the file is non-empty, counts regular files and directories, and prints the head for a sanity check.

**Layer stack you built:**

```text
/root/sshd-tree.txt
   │
   ├── ls -lat output from /etc/ssh    ← captured by `>`
   │     ├── . and .. entries          (hidden)
   │     ├── sshd_config(.d), ssh_config(.d)
   │     └── host key files (mode 0600, root-owned)
   └── one line per file               ← grep -c to count file types
```

**The story:** This is the **canonical 60-second exam answer.** Memorize the spine: `START=$PWD → cd /target/path → ls -OPTS > /save/path → cd "$START"`. Either `cd -` or a saved env variable returns you home. Either is acceptable on the exam — the file content is what graders check.

**Expected verification output:**

```text
6 /root/sshd-tree.txt
total 192
drwxr-xr-x.  4 root root   175 May 21 14:33 .
drwxr-xr-x. 89 root root  8192 May 26 12:48 ..
-rw-------.  1 root root  4434 May 21 14:33 sshd_config
-rw-------.  1 root root   513 May 21 14:33 ssh_host_ed25519_key
/home/ec2-user
/etc/ssh
VERIFY: file exists and is non-empty
4
2
```

**Cleanup**

```bash
rm -rf /tmp/nav-lab
rm -f /root/sshd-tree.txt
exit
```

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Listing is missing hidden files | Add `-a` to the `ls` flags |
| Listing is alphabetic, not by time | Add `-t` |
| You cannot return to start dir | Re-run with `START=$PWD` saved at the top |
| Wrote the listing to `/etc/ssh/...` | The `>` destination must be `/root/sshd-tree.txt`, not a relative path |

---

## 🔍 Navigation Decision Guide

```
Need to move around the filesystem?
  │
  ├── "Where am I right now?"
  │       └── ✅ pwd
  │
  ├── "What's in here?"
  │       └── ✅ ls            (bare)
  │       └── ✅ ls -la        (everything, with metadata)
  │       └── ✅ ls -ltr       (sorted by time, newest at bottom — find what changed)
  │
  ├── "Go to a specific known path"
  │       └── ✅ cd /etc/ssh
  │
  ├── "Go home"
  │       └── ✅ cd            (or cd ~)
  │
  ├── "Go up"
  │       └── ✅ cd ..         (one level)
  │       └── ✅ cd ../..      (two levels)
  │
  ├── "Bounce between two directories"
  │       └── ✅ cd -          (toggles with $OLDPWD)
  │
  ├── "Save current location so I can return later"
  │       └── ✅ START=$PWD; cd /elsewhere; ...; cd "$START"
  │
  ├── "Show me the directory ITSELF, not what's in it"
  │       └── ✅ ls -ld DIR
  │
  └── "Where is this file actually?"
          └── ✅ realpath FILE  (resolves all symlinks)
```

---

## ✅ Lab Checklist (6 Tasks)

- [ ] 01 Orient with `whoami`, `pwd`, `$HOME`, `ls /`, and the FHS top-level
- [ ] 02 Practice absolute paths: `cd /`, `cd /etc/ssh`, `cd /var/log`, with `ls -ltr` to spot recent changes
- [ ] 03 Practice relative paths and tokens: `..`, `.`, `~`, `cd -`, brace expansion in `mkdir -p`
- [ ] 04 Read `ls -l`/`-la`/`-lh`/`-lt`/`-ltr` until each field is obvious at a glance
- [ ] 05 Walk subtrees with `ls -R`, inspect dirs with `ls -ld`, ping-pong with `cd -`
- [ ] 06 Execute the RHCSA capstone — save `/etc/ssh`'s `-lat` listing to `/root/sshd-tree.txt` and return to the starting directory

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| `cd` with no argument expected to do nothing | Lands in `$HOME` | Empty `cd` is `cd ~` by design |
| `cd ..` typed twice as `cd .. ..` | "too many arguments" | One `cd` per argument |
| Used relative path in a script | Breaks when run from a different `cwd` | Use absolute paths or anchor on `$BASH_SOURCE` |
| `ls` does not show hidden files | Forgot `-a` | `ls -la` |
| Sort feels random | Default is alphabetical | Add `-t` for time |
| `ls -l` of a directory dumps its contents | That's the default | Add `-d` to see only the directory itself |
| `pwd` shows a symlink path | That is logical (`-L`) pwd | Use `pwd -P` to resolve symlinks |
| `cd -` says OLDPWD not set | First-ever `cd` | Do another `cd` first |
| Brace expansion is treated as a literal | Quoted | Remove quotes around `{a,b,c}` |
| `ls -R /` runs forever | Too much output | Limit with `\| head` or narrow the path |

---

## 🎯 Career & Interview Strategy

**RHCSA candidate**
- Train the muscle memory: `pwd; ls -la` on entering every directory. Use `cd -` to ping-pong between the exam task's working directory and `/etc/foo`.

**RHCE candidate**
- Ansible inventory and role paths matter. `ansible-playbook` resolves paths relative to the playbook directory — `pwd` discipline avoids "file not found" errors during graded tasks.

**SRE / Platform interview**
- "How would you find out what just changed in `/var/log`?" → `cd /var/log && ls -ltr | tail`. The newest mtime is at the bottom.

**DevOps**
- CI scripts that `cd` into a build directory should also `cd -` (or restore `$START`) before exiting, so subsequent steps run in the expected directory.

**AI / MLOps**
- Experiment scripts standardize on `cd $EXPDIR; ls -lt runs/ | head`. Find your most recent training run in one second.

---

## 🔗 Related Labs

| Lab | Connection |
|---|---|
| Lab 06 — Listing Files and SELinux Contexts | The `-Z` extension to `ls` |
| Lab 08 — Copying Files and Directories | First job after navigating: copy |
| Lab 12 — Creating Nested Directories | The `mkdir -p` primitive used here |
| Lab 14 — File Searching with `find` | When you do not know the path yet |
| Lab 15 — Instant File Searching with `locate` | The pre-indexed alternative to `find` |

---

## 👤 Author

**Kelvin R. Tobias**
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
