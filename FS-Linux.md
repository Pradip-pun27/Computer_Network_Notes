# 🐧 Linux File System — The Complete Guide

> Based on the video by **DorianDotSlash**  
> A detailed walkthrough of the Linux root directory structure — explained so you can **understand it clearly** and **teach it to others**.

---

## 📌 Table of Contents

1. [Windows vs Linux — A Different Evolution](#1-windows-vs-linux--a-different-evolution)
2. [The Filesystem Hierarchy Standard (FHS)](#2-the-filesystem-hierarchy-standard-fhs)
3. [Root Directory Tour (`/`)](#3-root-directory-tour-)
   - [`/bin` — Basic Binaries](#31-bin--basic-binaries)
   - [`/sbin` — System Binaries](#32-sbin--system-binaries)
   - [`/boot` — Boot Loader Files](#33-boot--boot-loader-files)
   - [`/dev` — Device Files](#34-dev--device-files)
   - [`/etc` — Configuration Files](#35-etc--configuration-files)
   - [`/lib`, `/lib32`, `/lib64` — Libraries](#36-lib-lib32-lib64--libraries)
   - [`/media` & `/mnt` — Mount Points](#37-media--mnt--mount-points)
   - [`/opt` — Optional / Third-Party Software](#38-opt--optional--third-party-software)
   - [`/proc` — Process Information (Pseudo-FS)](#39-proc--process-information-pseudo-fs)
   - [`/root` — Root User's Home](#310-root--root-users-home)
   - [`/run` — Runtime Data (Tmpfs / RAM)](#311-run--runtime-data-tmpfs--ram)
   - [`/snap` — Snap Packages](#312-snap--snap-packages)
   - [`/srv` — Service Data](#313-srv--service-data)
   - [`/sys` — System / Kernel Interface](#314-sys--system--kernel-interface)
   - [`/tmp` — Temporary Files](#315-tmp--temporary-files)
   - [`/usr` — User System Resources](#316-usr--user-system-resources)
   - [`/var` — Variable Data](#317-var--variable-data)
4. [`/home` — The User's Space](#4-home--the-users-space)
5. [Hidden Files & Dotfiles](#5-hidden-files--dotfiles)
6. [Key Takeaways](#6-key-takeaways)

---

## 1. Windows vs Linux — A Different Evolution

### 🪟 Windows History

- Born from **MS-DOS** (Disk Operating System) — command-line only
- Windows ran **on top of** DOS — you typed `win` to start it
- Used **drive letters**: `A:` and `B:` for floppy disks, `C:` for the hard drive, `D:` onward for more drives
- Applications installed into `C:\Program Files` (since Windows 95)
- Config and binaries scattered across `C:\Windows`, `C:\Program Files`, `C:\Users\...`

### 🐧 Linux History

- Follows **UNIX traditions** (from the 1970s)
- Uses **forward slash** `/` (not backslash `\` like Windows)
- **Case-sensitive** — `File`, `file`, `FILE`, `fiLe` are all **different files**
- Hierarchical single-root tree: **everything starts at `/`**
- macOS is similar because macOS is also UNIX-based (BSD lineage)

### 🧠 Key Concept

> _"In Linux, **everything is a file**."_  
> — This includes hardware devices, processes, and system information.

---

## 2. The Filesystem Hierarchy Standard (FHS)

The **FHS** is maintained by the **Linux Foundation**. It defines:

- What each directory is for
- What type of files go where
- The structure and layout of the filesystem

> ⚠️ **Not all distros follow FHS exactly** — some do their own thing. But most mainstream distros (Ubuntu, Debian, Fedora, Arch) generally follow it.

---

## 3. Root Directory Tour `/`

Let's walk through **every directory** at the root (`/`), one by one.

---

### 3.1 `/bin` — Basic Binaries

- **`bin`** = short for **binaries**
- Contains the most **essential, basic programs** (commands)
- These must be accessible even in **single-user mode** (recovery mode)

| Command | Purpose |
|---------|---------|
| `ls` | List directory contents |
| `cat` | Display file contents |
| `cp` | Copy files |
| `mv` | Move/rename files |
| `echo` | Print text |

> 🔑 These are the commands the system needs to **function at the most basic level**.

---

### 3.2 `/sbin` — System Binaries

- **`sbin`** = **system binaries**
- Contains programs a **system administrator** would use
- A **standard (normal) user** cannot run these without permission (`sudo`)

| Command | Purpose |
|---------|---------|
| `fdisk` | Partition management |
| `mkfs` | Create filesystems |
| `iptables` | Firewall rules |
| `shutdown` | Shut down the system |

> 🔑 Think of it as: **`/bin` = user tools, `/sbin` = admin tools**

Both `/bin` and `/sbin` contain files needed when running in **single-user mode** (network disabled, root-only recovery).

---

### 3.3 `/boot` — Boot Loader Files

- ⚠️ **DO NOT play around in this folder** unless you know what you're doing
- Contains everything the OS needs to **boot**
- Holds:
  - **Kernel images** (`vmlinuz-*`)
  - **Initrd / initramfs** (initial RAM filesystem)
  - **Boot loader config** (GRUB: `grub.cfg`)

💡 **If you break `/boot`, your system won't start.**

---

### 3.4 `/dev` — Device Files

This is where the **"everything is a file"** philosophy shines.

| Device | Path |
|--------|------|
| First SATA disk | `/dev/sda` |
| First partition | `/dev/sda1` |
| Second partition | `/dev/sda2` |
| Webcam | `/dev/video0` |
| Keyboard | `/dev/input/*` |

- These are **not actual files** — they're **interfaces** to hardware
- Applications and drivers interact with hardware through these files
- **Regular users rarely touch this folder directly**

🧠 **Example:** If you plug in a USB drive, it might appear as `/dev/sdb1`.

---

### 3.5 `/etc` — Configuration Files

- The name is debated: **"Editable Text Config"** or just **"et cetera"**
- Contains **system-wide configuration files** (not per-user)
- Think of it as the **control center** of your OS

| File | Purpose |
|------|---------|
| `/etc/apt/sources.list` | APT package repositories |
| `/etc/fstab` | Filesystem table (mount points) |
| `/etc/hosts` | Static hostname mapping |
| `/etc/ssh/sshd_config` | SSH server config |
| `/etc/passwd` | User account info |

> 🎯 **Important distinction:**  
> - System-wide settings → `/etc` (e.g., LibreOffice repo settings)  
> - Per-user settings → `/home/user/.config/` (e.g., LibreOffice preferences)

---

### 3.6 `/lib`, `/lib32`, `/lib64` — Libraries

- Contains **shared libraries** (like `.so` files — Shared Objects)
- Libraries are **code that applications can reuse** (like `.dll` in Windows)
- Required by binaries located in `/bin` and `/sbin`

**Example:** If you install a program, it may need `libc.so.6` (the C library) to function.

| Directory | Purpose |
|-----------|---------|
| `/lib` | Main libraries |
| `/lib32` | 32-bit libraries (on 64-bit systems) |
| `/lib64` | 64-bit libraries |

---

### 3.7 `/media` & `/mnt` — Mount Points

Both are used for **mounting storage devices**, but with different purposes:

| Directory | Who Uses It | Typical Use |
|-----------|-------------|-------------|
| `/media` | **OS / File Manager** | Auto-mounted devices (USB, external drives) |
| `/mnt` | **System Admin (you)** | Manually mounted devices |

**Example paths:**
- USB stick → `/media/username/MyUSB/`
- Manual mount → `/mnt/mydrive/`

> 📁 In file managers like **Nautilus**, mounted drives appear under "Other Locations".

---

### 3.8 `/opt` — Optional / Third-Party Software

- **`opt`** = **optional**
- Where **manually installed software from vendors** lives
- Some repo packages also install here

| Example | Path |
|---------|------|
| VPN software | `/opt/protonvpn/` |
| Printer drivers | `/opt/brother/` |
| Your own apps | `/opt/myapp/` |

> 💡 This is also where you'd put **software you wrote yourself**.

---

### 3.9 `/proc` — Process Information (Pseudo-FS)

- A **pseudo-filesystem** (not real files on disk — generated by the kernel)
- Contains runtime information about **running processes and system resources**
- Like `/dev`, these behave like files but live in RAM

#### How It Works:

Every running process gets its own numbered directory:

```
/proc/2344/    ← process with PID 2344
  ├── status   ← process details (name, memory, state)
  ├── cmdline  ← command that started it
  ├── cwd/     ← symlink to current working directory
  └── fd/      ← open file descriptors
```

**Useful system `/proc` files:**

| File | What It Shows |
|------|---------------|
| `/proc/cpuinfo` | CPU details (model, cores, flags) |
| `/proc/meminfo` | Memory stats |
| `/proc/uptime` | How long the system has been running |
| `/proc/version` | Linux kernel version |

> ⚠️ **Developers love this folder.** Normal users can ignore it.

---

### 3.10 `/root` — Root User's Home

- This is the **home directory for the `root` user**
- **Not** inside `/home` — it's at the root level
- Why? So root always has access to its home, even if `/home` is on a separate drive
- Requires **root permissions** to access

```
/home/elon/    ← normal user home
/root/         ← root user home (separate!)
```

---

### 3.11 `/run` — Runtime Data (Tmpfs / RAM)

- Relatively **new** directory (added in recent years)
- Uses **tmpfs** — runs entirely **in RAM**
- Everything in `/run` is **gone after reboot**
- Stores runtime data for processes that start **early in the boot process**

> ♻️ Think of it as: **temporary runtime storage for system services**

---

### 3.12 `/snap` — Snap Packages

- Used mainly by **Ubuntu** and its derivatives
- **Snap** is a self-contained package format (like a mini-app with all its dependencies)
- Each snap runs in a **sandboxed environment**
- Covered in more detail in dedicated videos/tutorials

---

### 3.13 `/srv` — Service Data

- **`srv`** = **service**
- Contains data for **services/servers** the system runs
- **Probably empty** on your desktop system
- But if you run:
  - A **web server** → `/srv/www/`
  - An **FTP server** → `/srv/ftp/`
- Better security since it's isolated at root level

> 🛡️ External users access files in `/srv` — so it can be mounted on a separate drive for safety.

---

### 3.14 `/sys` — System / Kernel Interface

- Another **pseudo-filesystem** (created at each boot, lives in RAM)
- A way to **interact with the kernel**
- You can **read and sometimes write** to these files to change kernel parameters

**Example:** VGA switcheroo (switching between integrated and dedicated GPU):

```bash
# Write to a sysfs file to switch graphics card
echo "DIS" > /sys/kernel/debug/vgaswitcheroo/switch
```

> ⚠️ Like `/proc`, this is **developer/admin territory**.

---

### 3.15 `/tmp` — Temporary Files

- **`tmp`** = **temporary**
- Applications store **short-lived files** here
- **Cleared on reboot** (usually)

**Real-world example:** When you're writing a document in a word processor, it saves a **temporary recovery copy** here every few minutes. If the app crashes, it checks `/tmp` for the latest saved copy to recover.

```bash
# What happens
$ ls /tmp/
tempfile_doc_abc123   # ← auto-saved document recovery
```

> 🧹 **Cleanup tip:** If `/tmp` gets full of leftover files (hundreds of them eating disk space), boot into single-user mode as root and manually clean it.

---

### 3.16 `/usr` — User System Resources

- **`usr`** = **UNIX System Resources** (historically "user")
- **The big one** — where most user-installed applications live
- Non-essential for basic system operation (unlike `/bin` which is essential)

#### Structure inside `/usr`:

| Directory | Purpose |
|-----------|---------|
| `/usr/bin/` | User programs (most installed apps) |
| `/usr/sbin/` | User system administration tools |
| `/usr/lib/` | Libraries for user programs |
| `/usr/local/` | **Locally compiled / source-built software** |
| `/usr/local/bin/` | Programs installed from source |
| `/usr/local/lib/` | Their libraries |
| `/usr/share/` | Shared data (icons, themes, docs, man pages) |
| `/usr/src/` | Source code (e.g., kernel headers) |

**The flow:**
```
Package Manager installed → /usr/bin/ + /usr/lib/
Source compiled (./configure && make && make install) → /usr/local/bin/
```

> 🧠 **Important:** While FHS defines the structure, not all developers follow it — some apps may be in unexpected places.

---

### 3.17 `/var` — Variable Data

- **`var`** = **variable**
- Contains files that **grow in size** as the system runs

| Directory | What Grows Here |
|-----------|-----------------|
| `/var/log/` | **System and application logs** (constantly growing) |
| `/var/crash/` | Crash reports |
| `/var/spool/` | Print queues, mail spools |
| `/var/tmp/` | Temporary files that persist between reboots |
| `/var/cache/` | Cached package data (apt, pacman) |

**Example:** Check system logs:
```bash
tail -f /var/log/syslog    # Watch system log in real-time
journalctl -xe             # More modern way (systemd)
```

---

## 4. `/home` — The User's Space

### Structure

```
/home/
  ├── elon/         ← User 1's personal files
  │   ├── Documents/
  │   ├── Downloads/
  │   ├── Pictures/
  │   ├── Videos/
  │   └── .config/  ← Hidden settings folder
  ├── alice/        ← User 2's personal files
  └── bob/          ← User 3's personal files
```

### Key Rules

| Rule | Explanation |
|------|-------------|
| **Users are isolated** | Each user can only access their own home folder (unless using `sudo`) |
| **Reinstall-safe** | Many users put `/home` on a **separate partition/drive** — you can reinstall the OS and keep all your files |
| **Hidden files** | Application settings are stored in **hidden folders** (starting with `.`) |

---

## 5. Hidden Files & Dotfiles

Any file or folder starting with a **`.`** (dot) is **hidden by default**.

### How to See Them

| Interface | Method |
|-----------|--------|
| **File Manager** (Nautilus, Dolphin) | `Ctrl + H` |
| **Terminal** | `ls -a` (list all) or `ls -la` (detailed) |

### What's Inside Hidden Folders?

```
~/
  .bashrc          ← Bash shell configuration
  .config/         ← Application settings (e.g., GNOME desktop)
    gnome/         ← Desktop environment settings
    gedit/         ← Text editor settings
  .local/          ← Per-user application data
  .cache/          ← Thumbnails, temporary browser cache
  .themes/         ← Custom themes
  .icons/          ← Custom icons
  .ssh/            ← SSH keys and config
```

### Why They Matter for Backups

```
📁 Visible files       → Your documents, photos, projects
📁 Hidden files (dot)  → Your settings, themes, preferences
```

**Backup strategy:**

| What you want | What to backup |
|---------------|----------------|
| Just files | All visible folders in `/home/you/` |
| Everything (files + settings) | ALL folders including hidden ones |

> 💡 **Pro tip:** If you back up your hidden folders too, when you reinstall:
> 1. Log in with your username
> 2. Restore the hidden folders
> 3. All your themes, wallpapers, and app settings are back
> 4. Just reinstall the applications — they'll find their settings waiting!

---

## 6. Key Takeaways

### 🎯 The Big Picture

```
/                          ← The root (everything starts here)
├── bin/    → Essential commands (ls, cat, cp)
├── sbin/   → Admin commands (fdisk, mkfs)
├── boot/   → Don't touch! (kernel, GRUB)
├── dev/    → Hardware as files (/dev/sda)
├── etc/    → System config (/etc/apt/)
├── home/   → YOUR files (/home/elon/)
├── media/  → Auto-mounted USB drives
├── mnt/    → Manually mounted drives
├── opt/    → Third-party apps
├── proc/   → Running processes (pseudo-FS)
├── root/   → Root user's home
├── run/    → Runtime data (RAM)
├── snap/   → Ubuntu Snap packages
├── srv/    → Server data (web, FTP)
├── sys/    → Kernel interface (pseudo-FS)
├── tmp/    → Auto-cleaned temp files
├── usr/    → User applications & resources
└── var/    → Growing data (logs, caches)
```

### 🧠 Why This Structure Works

1. **Separation of concerns** — System files vs user files vs variable files
2. **Security** — Normal users can't break system files
3. **Resource sharing** — Libraries in `/lib` and `/usr/lib` are shared across apps
4. **Clean uninstalls** — Package managers track where everything goes and clean up
5. **Flexibility** — `/home` on a separate drive, `/srv` on another

### 🆚 Linux vs Windows at a Glance

| Concept | Windows | Linux |
|---------|---------|-------|
| Root | `C:\` | `/` |
| Separator | `\` (backslash) | `/` (forward slash) |
| Case-sensitive | ❌ No | ✅ Yes |
| Drive letters | A:, B:, C:... | Everything is under `/` |
| Programs | `C:\Program Files` | `/usr/bin/`, `/opt/`, `/usr/local/bin/` |
| Config | Registry + `.ini` files | `/etc/` + `/home/user/.config/` |
| Devices | `D:\` drives | `/dev/sda`, `/media/usb` |
| Temp | `C:\Users\...\AppData\Local\Temp` | `/tmp` |

---

> 💬 **"Although it seems like a mess, it's actually a more efficient way of doing things and allows much more sharing of common resources between packages."** — DorianDotSlash

---

*Created from the video: **"Linux File System/Structure Explained!"** by DorianDotSlash*  
*📹 https://youtu.be/HbgzrKJvDRw*
