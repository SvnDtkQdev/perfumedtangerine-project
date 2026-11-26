# NTFS-REPAIR

 Version `13.0.9` · Svn DtkQdev
          
----------------------------------------------                                       
                                     


## Table of contents

Hx 1. [About](#about)    
Hx 3. [Requirements](#requirements)   
Hx 5. [Install](#install)    
Hx 6. [Run](#run)    
Hx 9. [Options reference](#options-reference)    
Hx 0. [Safety](#safety)    
H 11. [Dependencies](#dependencies)    
H 13. [Project layout](#project-layout)    
H 15. [Limits](#limits)   
H 19. [License](#license)   
 


## About

**NTFS-REPAIR** is a Linux terminal application that helps diagnose and repair **NTFS** volumes (USB sticks, external disks, dual-boot partitions) **without requiring Windows** when possible.

| | |
|---|---|
| **UI** | Full-screen menus, ANSI clear, key image (`redkeys.png`) |
| **Core tools** | `ntfsfix`, `ntfs-3g`, `ntfsinfo`, `badblocks`, `lsblk`, `blkid` |
| **Goal** | Dirty flag / hibernation / light corruption / force mount |
| **Not a clone of** | Microsoft `chkdsk` (deep MFT rebuild still needs Windows) |



  Backup important data before any write operation. 
  Some stages change on-disk NTFS metadata.      


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
> > E1. `npm install` (Node packages)  
> > E3.  System packages via **pacman / apt / dnf / zypper** `ntfs-3g`, `e2fsprogs` / `badblocks`, `util-linux`)  
> > E0.  Global commands: `ntfs-repair`, `ksvn`, `ntfsre-pair`  
> > E6.  If `ntfsfix` is still missing → optional `./install-system-deps.sh` (source build into `./deps`)


---

## Run commands to open ntfs-repair interface menu:

```bash
sudo ntfs-repair
sudo ntfsre-pair
sudo ksvn

```



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


---

## Dependencies

### System (installed by `install.sh`)

| Package / tool | Role |
|----------------|------|
| **ntfs-3g** / ntfsprogs | `ntfsfix`, `ntfs-3g`, `ntfsinfo`, `mkntfs` |
| **e2fsprogs** | `badblocks` |
| **util-linux** | `lsblk`, `blkid`, `findmnt`, `mount`, `umount`, `blockdev` |
| **psmisc / coreutils** | `fuser`, `dd`, `sync` |

### Node (npm) required

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

>
> - Linux **cannot** fully replace Microsoft **chkdsk** for every NTFS inconsistency.  
> - Cluster remapping and complex MFT rebuilds may require **Windows**.  
> - This project maximizes **practical Linux recovery** (dirty flag, hibernation, force mount, multi-pass `ntfsfix`).

---

## License

**MIT** (see `package.json`)
