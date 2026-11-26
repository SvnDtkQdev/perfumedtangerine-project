# NTFS-REPAIR

**Version `13.0.9`** · **Author: Svn DtkQdev**

> **Terminal NTFS diagnostics & repair for Linux**  
> Menu-driven TUI · multi-volume · progress bars · best-effort repair kits  
> Global commands: `ntfs-repair` · `ntfsre-pair` · `ksvn`
> npm required

---

## Table of contents

1. [About](#about)
2. [Requirements](#requirements)
3. [Install](#install)
4. [Run](#run)
5. [Main menu](#main-menu)
6. [Options reference](#options-reference)
7. [Progress bar](#progress-bar)
8. [Safety](#safety)
9. [Dependencies](#dependencies)
10. [Project layout](#project-layout)
11. [Limits](#limits)
12. [License](#license)

---

## About

**NTFS-REPAIR** is a Linux terminal application that helps diagnose and repair **NTFS** volumes (USB sticks, external disks, dual-boot partitions) **without requiring Windows** when possible.

| | |
|---|---|
| **UI** | Full-screen menus, ANSI clear, key image (`redkeys.png`) |
| **Core tools** | `ntfsfix`, `ntfs-3g`, `ntfsinfo`, `badblocks`, `lsblk`, `blkid` |
| **Goal** | Dirty flag / hibernation / light corruption / force mount |
| **Not a clone of** | Microsoft `chkdsk` (deep MFT rebuild still needs Windows) |

```
┌─────────────────────────────────────────────────────────┐
│  Backup important data before any write operation.      │
│  Some stages change on-disk NTFS metadata.              │
└─────────────────────────────────────────────────────────┘
```

---

## Requirements

- **OS:** Linux (Arch / CachyOS, Debian / Ubuntu, Fedora, openSUSE, …)
- **Node.js:** `>= 18`
- **Privileges:** many repairs need **`sudo` / root**
- **TTY:** interactive terminal (not a pure pipe / non-TTY launcher)

---

## Install

```bash
unzip ntfsrepairclone-13.0.9-full.zip
cd ntfsrepairclone
sudo bash ./install.sh
```

> **What `install.sh` does**
>
> 1. `npm install` (Node packages)  
> 2. System packages via **pacman / apt / dnf / zypper**  
>    (`ntfs-3g`, `e2fsprogs` / `badblocks`, `util-linux`, …)  
> 3. Global commands: `ntfs-repair`, `ksvn`, `ntfsre-pair`  
> 4. If `ntfsfix` is still missing → optional `./install-system-deps.sh` (source build into `./deps`)

**Local run without global install:**

```bash
sudo ./ntfs-repair
# aliases in the same folder:
sudo ./ksvn
sudo ./ntfsre-pair
```

---

## Run

```bash
sudo ntfs-repair
sudo ntfsre-pair
sudo ksvn

```

```
┌─────────────────────────────────────────────────────────┐
│  Workflow (recommended)                                 │
│  1. Option 1 — select volume(s) + tick Back + ENTER     │
│  2. Option 4 — check mounting errors (diagnose first)   │
│  3. Option 2 or 15 — auto-fix / chkdsk kit             │
│  4. Option 17 — force mount only if you accept data risk│
└─────────────────────────────────────────────────────────┘
```

---

## Main menu

After start you see:

- **Title:** `NTFS-REPAIR` (figlet + gradient)
- **Key image:** `redkeys.png` (terminal art, fixed aspect ratio, high quality)
- **Selected volumes:** e.g. `/dev/sdb1 (SanDisk Ultra "DATA" 28.6GB)`
- **Option list:** markers `[x] 01 …` — brackets always **white**; completed options get a **random colored** marker character
- **Clean screen** on every menu enter and return (no leftover text from option 4)

**Markers**

| State | Look |
|--------|------|
| Initial | White random char inside white `[ ]` |
| After complete | New random char + **random color** (brackets stay white) |
| App restart | All markers reset |

---

## Options reference

| # | Name | Role |
|---|------|------|
| **1** | select NTFS volume | Multi-select; shows **company/disk name**, size, label, mount, transport. Confirm with **Back (keep current selection)** + ENTER |
| **2** | auto-fix | Safe multi-pass kit: unmount attempt, `ntfsfix` passes, dirty/bad-sector clear, hiber cleanup, RW verify |
| **3** | check filesystem integrity | Read-only (`ntfsfix -n`, `ntfsinfo`) |
| **4** | check mounting errors | Full diagnostics; result on screen; main menu keeps a **one-line** notice only (clean UI) |
| **5** | fix read-only mount | Dirty / hibernation-related clear + mount hints |
| **6** | bad blocks fix | Read-only surface scan + `ntfsfix` helpers + guidance |
| **7** | repair MFT | Best-effort (deep damage → Windows `chkdsk`) |
| **8** | rebuild $Bitmap | `ntfsfix -b` / related |
| **9** | fix $LogFile | Journal / dirty-state attempts |
| **10** | clear dirty flag | `ntfsfix -d` |
| **11** | informations | Version **13.0.9**, author **Svn DtkQdev**, tools, Node packages, selected volumes |
| **12** | system dependencies | Install / verify local `./deps` builds |
| **13** | help & usage | Shortcuts and safety notes |
| **14** | advanced options | Extra probes / guided utilities |
| **15** | chkdsk NTFS fix - beta | Full multi-stage Linux kit (offline → multi-pass fix → hiber → scan → RW verify → last resort) |
| **16** | dd fix bad blocks | **Dangerous** targeted block overwrite |
| **17** | force mount | **Data loss assumed** — aggressive force RW mount only (no safe path) |

### Option 1 — volume list

Each line can include:

- Device path (`/dev/sdXN`)
- Partition size
- **Company / disk name** (vendor + model)
- Disk total size
- Volume label
- Mount point / ro / transport

Confirm selection by ticking volumes **and** **← Back (keep current selection)**, then **ENTER**.

### Option 15 — chkdsk kit (summary)

```
Stage 0   Force offline (unmount / fuser / blockdev)
Stage 1   Diagnose (ntfsfix -n)
Stage 2–5 Multi-pass fix + dirty + bad-sector list
Stage 6   Hibernation remove_hiberfile cycle
Stage 7   Structure / MFT probe
Stage 8   Surface scan (badblocks read-only)
Stage 9   Final passes
Stage 10  RO / RW + write test
Stage 11+ Last-resort recovery if needed
```

### Option 17 — force mount

```
┌─────────────────────────────────────────────────────────┐
│  WARNING (shown in-app before confirm)                  │
│                                                         │
│  Linux already has NTFS drivers that can read and write │
│  NTFS, and the kernel’s ntfs3 driver supports journal   │
│  replay. But a driver’s job is normal filesystem        │
│  access—not necessarily discovering and safely          │
│  repairing every possible contradiction in a damaged    │
│  filesystem. badblocks is different: it checks a drive  │
│  for unreadable blocks; it doesn’t repair NTFS          │
│  metadata. System avoid its write mode on a disk        │
│  containing data, because it overwrites data.           │
└─────────────────────────────────────────────────────────┘
```

**Double confirm** required. Path: force unmount → multi-pass `ntfsfix` → **read-only** `badblocks` (never `-w`) → force mount (`force,rw,remove_hiberfile`, …) → write test.

---

## Progress bar

Shared wide bar on all work screens:

```
[----------------------------->$]
```

| Char | Color |
|------|--------|
| `[` `]` | Yellow |
| `-` | Orange |
| `>` | Green |
| `$` | Red |
| `#` | Grey |

When the head reaches the end, the last `#` is replaced, then the bar becomes cyan `*` for **3 seconds**.

---

## Safety

```
┌─────────────────────────────────────────────────────────┐
│  • Prefer diagnose (3, 4) before write repairs          │
│  • Unmount volumes when possible before write stages    │
│  • Option 16 / 17 can destroy data — confirm carefully  │
│  • Windows Fast Startup / hibernation often causes      │
│    “dirty” USB mounts — full Windows Shut down helps    │
│  • Deep MFT/index damage may still need:                │
│      chkdsk X: /f /r   (from Windows)                   │
└─────────────────────────────────────────────────────────┘
```

---

## Dependencies

### System (installed by `install.sh`)

| Package / tool | Role |
|----------------|------|
| **ntfs-3g** / ntfsprogs | `ntfsfix`, `ntfs-3g`, `ntfsinfo`, `mkntfs` |
| **e2fsprogs** | `badblocks` |
| **util-linux** | `lsblk`, `blkid`, `findmnt`, `mount`, `umount`, `blockdev` |
| **psmisc / coreutils** | `fuser`, `dd`, `sync` |

### Node (npm)

`inquirer` · `chalk` · `figlet` · `gradient-string` · `boxen` · `static-chafa` · `ora` / `cli-spinners`

See **option 11 — informations** for live paths and ✓/✗ status.

---

## Project layout

```
ntfsrepairclone/
├── index.js                 # main application
├── package.json             # name ntfs-repair, version 13.0.9
├── install.sh               # npm + system packages + global bins
├── install-system-deps.sh   # optional local source build → ./deps
├── ntfs-repair / ksvn / ntfsre-pair   # local launchers
├── redkeys.png              # main-menu key image
├── README.md                # this file
├── CHANGES.md
└── node_modules/
```

---

## Limits

> **Honest limits**
>
> - Linux **cannot** fully replace Microsoft **chkdsk** for every NTFS inconsistency.  
> - Cluster remapping and complex MFT rebuilds may require **Windows**.  
> - This project maximizes **practical Linux recovery** (dirty flag, hibernation, force mount, multi-pass `ntfsfix`).

---

## License

**MIT** (see `package.json`)

---

**NTFS-REPAIR `13.0.9`** · **Svn DtkQdev** · **svn**

```
sudo ntfs-repair
```
