# 🏰 Arch Fortress: Reforged — Modern, Secure & Minimal Arch Linux Installation Guide

![Linux](https://img.shields.io/badge/OS-Linux-black?style=flat-square&logo=linux&logoColor=white)
![Arch Linux](https://img.shields.io/badge/Distro-Arch-blue?style=flat-square&logo=arch-linux)
![EFI](https://img.shields.io/badge/Firmware-EFI-white?style=flat-square&logo=rocket&logoColor=white)
![UKI](https://img.shields.io/badge/Boot-UKI-purple?style=flat-square&logo=linuxfoundation&logoColor=white)
![LUKS2 + TPM2](https://img.shields.io/badge/Encryption-LUKS2%20%2B%20TPM2-orange?style=flat-square&logo=cryptpad&logoColor=white)
![Secure Boot](https://img.shields.io/badge/Secure%20Boot-Enabled-teal?style=flat-square&logo=socket&logoColor=white)
![LVM](https://img.shields.io/badge/Storage-LVM-darkslategray?style=flat-square&logo=discogs&logoColor=white)
![Ext4](https://img.shields.io/badge/Filesystem-Ext4-deepskyblue?style=flat-square&logo=buffer&logoColor=white)
![Systemd](https://img.shields.io/badge/Init-Systemd-slateblue?style=flat-square&logo=circle&logoColor=white)
![zRam](https://img.shields.io/badge/zRam-Enabled-limegreen?style=flat-square&logo=cashapp&logoColor=white)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square&logo=open-source-initiative)](LICENSE)

**Arch Fortress: Reforged** is a modern, security-focused Arch Linux installation guide designed for users who value **stability, simplicity and long-term maintainability**.

Rather than chasing every available feature, this guide focuses on building a **robust and predictable system** using carefully selected technologies. It deliberately favors **LVM + Ext4** for their maturity, proven reliability and consistent performance, while adopting modern security features such as **Unified Kernel Images (UKI)**, **LUKS2 with TPM2**, and **Secure Boot**.

> 🛡️ Built on: **EFI**, **UKI**, **LUKS2 + TPM2**, **Secure Boot**, **LVM**, **Ext4**, **Systemd**, **zRAM**

---

## 📚 Table of Contents

- [🎯 Overview](#-overview)
- [⚙️ Features](#️-features)
- [📦 Project Structure](#-project-structure)
- [🗂️ Disk Layout & LVM Architecture](#️-disk-layout--lvm-architecture)
- [🔧 Mount Options Summary](#-mount-options-summary)
- [📖 Manual Installation (Step-by-step)](#-manual-installation-step-by-step)
- [❓ FAQ](#-faq)
- [🛠 Requirements](#-requirements)
- [📜 License](#-license)
- [👤 Author](#-author)

---

## 🎯 Overview

**Arch Fortress: Reforged** is not a distribution or a preconfigured Arch setup — it's a **step-by-step installation guide** for building a modern, secure and production-ready Arch Linux system from scratch.

Designed around **stability, simplicity and long-term maintainability**, it combines proven storage technologies with modern Linux security features to provide a clean, fully encrypted and predictable system.

- 💽 **LVM + Ext4** for a mature, reliable and high-performance storage stack
- 🔐 **LUKS2** full-disk encryption with **TPM2** auto-unlocking and passphrase fallback
- 🚀 **Direct EFI boot** using a signed **Unified Kernel Image (UKI)** — no traditional bootloader required
- 💥 Full **Secure Boot** support
- 🧠 Modern `mkinitcpio` using **systemd init hooks**
- 🧵 **zRam** enabled for fast compressed in-memory swap
- 💾 Encrypted **swap partition** as zRam fallback
  - Uses a transient encryption key generated at boot from `/dev/urandom`  
  - ⚠️ Hibernation is not possible (non-persistent encryption key)
- 🛟 **UKI Fallback & Previous** for boot recovery and kernel rollback
- 🗄️ **Automatic EFI partition backup** with a rolling history of previous backups in `/boot/efibackup`

---

## ⚙️ Features

### 🔐 Security
- Full `/` system encryption with **LUKS2 + TPM2**
- Fallback passphrase support
- Secure Boot ready with signed kernels

### 🧊 Filesystem
- **Ext4 on LVM** using physical volumes, volume groups and logical volumes:
  - `lv_root`, `lv_home`, `lv_var`, etc.
- **zRam** enabled to provide fast compressed RAM-based swap
- Encrypted **swap partition** as a fallback to zRam

### ⚙️ Boot Process
- **No traditional bootloader** — no GRUB or systemd-boot
- EFI directly loads a **signed Unified Kernel Image (UKI)**
- UKIs are built with `mkinitcpio` and contain:
  - Kernel
  - Initramfs
  - Kernel command line
  - CPU microcode

### 🧠 Init System
- `mkinitcpio` using:
  - `systemd`, `sd-vconsole`, `sd-encrypt`
- No legacy hooks like `udev`, `usr`, `resume`, `keymap`, `consolefont`, `encrypt`
- Faster, cleaner, future-proof boot

### 🛟 UKI Recovery & Rollback 
- **Fallback UKI** built with a more generic initramfs for recovery
- **Previous UKI** automatically preserved before kernel-related updates
- Provides a quick boot option if a newly generated UKI or kernel causes a boot failure

### 🗄️ Automatic EFI Partition Backup
- The `/efi` EFI System Partition (ESP) is automatically backed up before relevant system updates
- Backups are stored in `/boot/efibackup`
- A rolling history of the **5 most recent EFI backups** is maintained
- Provides an additional recovery layer in case of EFI or UKI-related issues

---

## 📦 Project Structure

<details>
<summary>📁 <code>arch-fortress-reforged/</code> (click to expand)</summary>

```
arch-fortress/
├── LICENSE
└── README.md
```

</details>

---

## 🗂️ Disk Layout & LVM Architecture

> This is the storage layout used by **Arch Fortress: Reforged**, built around a secure, modular and maintainable architecture combining **LUKS2**, **LVM**, **Ext4**, and **EFI boot with UKI**.

### 💽 Partition Table (GPT - `/dev/nvme0n1`)

| Partition        | Type              | FS    | Mount Point | Size | Description                         |
|------------------|-------------------|-------|-------------|------|-------------------------------------|
| `/dev/nvme0n1p1` | EFI System (ef00) | FAT32 | `/efi`      | 500M | EFI System Partition (boot via UKI) |
| `/dev/nvme0n1p2` | Linux LUKS (8309) | LUKS2 | [LUKS]      | ~2TB | Encrypted root volume               |

---

### 🔐 Encrypted Storage

- `/dev/nvme0n1p2` is encrypted using **LUKS2** with **TPM2** auto-unlocking.
- Opened as `/dev/mapper/cryptarch`.
- The encrypted container hosts a **single LVM Physical Volume (PV)**.
- The **Volume Group (VG)** contains multiple **Ext4 Logical Volumes (LVs)**.

---

### 🌳 LVM Structure

| Physical Volume         | Virtual Group Name | Virtual Group Mapper    | Virtual Group Device |
|-------------------------|--------------------|-------------------------|----------------------|
| `/dev/mapper/cryptarch` | `vg_system`        | `/dev/mapper/vg_system` | `/dev/vg_system`     |

| Logical Volumes Names | Logical Volumes Mapper           | Logical Volumes Devices   | Mount Point               | Description                      |
|-----------------------|----------------------------------|---------------------------|---------------------------|----------------------------------|
| `lv_root`             | `/dev/mapper/vg_system-lv_root`  | `/dev/vg_system/lv_root`  | `/`                       | Root system                      |
| `lv_boot`             | `/dev/mapper/vg_system-lv_boot`  | `/dev/vg_system/lv_boot`  | `/boot`                   | Boot data                        |
| `lv_home`             | `/dev/mapper/vg_system-lv_home`  | `/dev/vg_system/lv_home`  | `/home`                   | User data                        |
| `lv_var`              | `/dev/mapper/vg_system-lv_var`   | `/dev/vg_system/lv_var`   | `/var`                    | Variable system data             |
| `lv_log`              | `/dev/mapper/vg_system-lv_log`   | `/dev/vg_system/lv_log`   | `/var/log`                | System logs                      |
| `lv_tmp`              | `/dev/mapper/vg_system-lv_tmp`   | `/dev/vg_system/lv_tmp`   | `/var/tmp`                | Temporary files                  |
| `lv_cache`            | `/dev/mapper/vg_system-lv_cache` | `/dev/vg_system/lv_cache` | `/var/cache`              | Application and package caches   |
| `lv_virt`             | `/dev/mapper/vg_system-lv_virt`  | `/dev/vg_system/lv_virt`  | `/var/lib/libvirt/images` | Virtual machine images           |
| `lv_opt`              | `/dev/mapper/vg_system-lv_opt`   | `/dev/vg_system/lv_opt`   | `/opt`                    | Optional third-party software    |
| `lv_games`            | `/dev/mapper/vg_system-lv_games` | `/dev/vg_system/lv_games` | `/opt/games`              | Games and game libraries         |
| `lv_srv`              | `/dev/mapper/vg_system-lv_srv`   | `/dev/vg_system/lv_srv`   | `/srv`                    | Server data                      |
| `lv_swap`             | `/dev/mapper/vg_system-lv_swap`  | `/dev/vg_system/lv_swap`  | `[SWAP]`                  | Encrypted swap volume (e.g. 4GB) |

---

🧠 **Why this layout?**

This architecture is designed to provide:

- 🔒 **Security** through isolated mount points and dedicated mount options.
- 🛟 **Backups** stored in dedicated boot data volume
- 🛠️ **Maintainability** by separating system data into dedicated logical volumes.
- 📈 **Scalability** by allowing logical volumes to be resized independently.
- 🚀 **Performance** by isolating write-intensive workloads (logs, cache, VMs, games).
- 🧩 **Flexibility** to add, remove or resize volumes without repartitioning the disk.

---

### 🖼️ Layout Diagram

```
Disk: /dev/nvme0n1 (GPT)
┌──────────────────────────────────────────────────────┐
│ Partition Table                                      │
│──────────────────────────────────────────────────────│
│ /dev/nvme0n1p1   → EFI System (FAT32, 500M)          │
│                  └── Mounted at /efi                 │
│                                                      │
│ /dev/nvme0n1p2   → LUKS2 Encrypted Volume (~2TB)     │
│                  └── mapper/cryptarch                │
│                      └── LVM Pysical Volume          │
│                          └── LVM Virtual Group       │
│                              └── LVM Logical Volumes │
│                                  └── Ext4 Filesystem │
└──────────────────────────────────────────────────────┘
```

#### LVM Logical Volumes (inside /dev/mapper/cryptarch):

```
┌───────────────────────────────────────────────────────────────────────────┐
│ lv_root   → /                                      ← Root filesystem      │
│ lv_boot   → /boot                                                         │
│ lv_home   → /home                                                         │
│ lv_var    → /var                                                          │
│ lv_log    → /var/log                                                      │
│ lv_tmp    → /var/tmp                                                      │
│ lv_cache  → /var/cache                                                    │
│ lv_virt   → /var/lib/libvirt/images                                       │
│ lv_opt    → /opt                                                          │
│ lv_games  → /opt/games                                                    │
│ lv_srv    → /srv                                                          │
│ lv_swap   → [SWAP]                   ← Encrypted swap partition (e.g. 4G) │
└───────────────────────────────────────────────────────────────────────────┘
```

#### Boot process:

```text
[ EFI Firmware ]
    ↓
[ UKI Image (.efi) in /efi ]
    ↓
[ systemd (init) in initramfs ]
    ↓
[ Unlock LUKS via TPM2 ]
    ↓
[ Activate LVM Volume Group ]
    ↓
[ Mount Ext4 Logical Volumes ]
    ↓
[ Boot into secure, modern Arch Fortress 🔐🛡️ ]
```

---

## 🔧 Mount Options Summary

### 📂 Mount Points and Options

| 📍 Mount Point | 💽 Devices | 🗂️ Logical Volumes | ⚙️ Mount Options |
|----------------|-----------|-------------------|------------------|
| `/` | `/dev/vg_system/lv_root` | `lv_root` | `rw,noatime,nodiratime,errors=remount-ro` |
| `/efi` | `/dev/nvme0n1p1` | *(N/A)* | `rw,noatime,nodiratime,nodev,nosuid,noexec,fmask=0022,dmask=0022` |
| `/boot` | `/dev/vg_system/lv_boot` | `lv_boot` | `rw,noatime,nodiratime,nodev,nosuid,noexec` |
| `/home` | `/dev/vg_system/lv_home` | `lv_home` | `rw,noatime,nodiratime,nodev,nosuid` |
| `/var` | `/dev/vg_system/lv_var` | `lv_var` | `rw,noatime,nodiratime,nodev,nosuid` |
| `/var/log` | `/dev/vg_system/lv_log` | `lv_log` | `rw,noatime,nodiratime,nodev,nosuid,noexec` |
| `/var/tmp` | `/dev/vg_system/lv_tmp` | `lv_tmp` | `rw,noatime,nodiratime,nodev,nosuid,noexec` |
| `/var/cache` | `/dev/vg_system/lv_cache` | `lv_cache` | `rw,noatime,nodiratime,nodev,nosuid,noexec` |
| `/var/lib/libvirt/images` | `/dev/vg_system/lv_virt` | `lv_virt` | `rw,noatime,nodiratime,nodev,nosuid,noexec` |
| `/opt` | `/dev/vg_system/lv_opt` | `lv_opt` | `rw,noatime,nodiratime,nodev,nosuid` |
| `/opt/games` | `/dev/vg_system/lv_games` | `lv_games` | `rw,noatime,nodiratime,nodev,nosuid` |
| `/srv` | `/dev/vg_system/lv_srv` | `lv_srv` | `rw,noatime,nodiratime,nodev,nosuid,noexec` |
| `[SWAP]` | `/dev/vg_system/lv_swap` | `lv_swap` | `pri=0` |

---

### 📖 Mount Options Explanation

| ⚙️ Option | 🔎 Description | 🏷️ Category |
|-----------|---------------|-------------|
| `rw` | Mount filesystem in read-write mode | 🔧 Default |
| `noatime` | Do not update file access timestamps (reduces unnecessary SSD writes) | 🚀 Performance |
| `nodiratime` | Do not update directory access timestamps *(redundant with `noatime`, kept for consistency)* | 🚀 Performance |
| `nodev` | Prevent character/block device files from being interpreted on this filesystem | 🔒 Security |
| `nosuid` | Ignore SUID and SGID permission bits | 🔒 Security |
| `noexec` | Prevent execution of binaries from this filesystem | 🔒 Security |
| `errors=remount-ro` | Remount the filesystem read-only if a filesystem error is detected, helping prevent further corruption | 🛡️ Reliability |
| `fmask=0022` | File permission mask for the FAT32 EFI System Partition | 🔒 Security |
| `dmask=0022` | Directory permission mask for the FAT32 EFI System Partition | 🔒 Security |
| `pri=0` | Swap mount option with low priority | 🔧 zRam Fallback |

---

### 🔎 Why these mount options?

These options are carefully chosen for:

- 🚀 **Performance**: reduce unnecessary SSD writes using `noatime` and `nodiratime`.
- 🔒 **Security**: apply `nodev`, `nosuid` and `noexec` only where they make sense.
- 🛡️ **Reliability**: `errors=remount-ro` protects the root filesystem against further corruption.
- 📦 **Isolation**: logs, cache, temporary files, virtual machine images and game libraries are isolated in dedicated Logical Volumes.
- 💽 **Flexibility**: LVM allows online resizing of individual filesystems without repartitioning the disk.
- 🎮 **Game preservation**: `/opt/games` is isolated in its own Logical Volume, allowing Steam and standalone game libraries to be preserved and reused independently of the operating system during a reinstall.

---

### ✅ Quick Summary

| 🎯 Aspect | ⚙️ Strategy |
|------------|------------|
| SSD optimization | `noatime`, `nodiratime` + `fstrim.timer` |
| Reduce writes | `noatime`, `nodiratime` |
| Security hardening | `nodev`, `nosuid`, `noexec` where appropriate |
| Filesystem reliability | `errors=remount-ro` |
| Storage architecture | LUKS2 + LVM + Ext4 |
| Flexible storage | Independent Logical Volumes |
| OS reinstallation | Preserve `/home`, `/opt/games` and VM images independently |

---

**✅ READY FOR PRODUCTION 🖥️**

---

## 📖 Manual Installation (Step-by-step)

> 🧠 For advanced users or educational purposes.

> ⚠️ **Secure Boot must be set to "Setup Mode" in the BIOS/UEFI before installation.**
> This is required to enroll your own Secure Boot keys with `sbctl`.

This section provides every command used to build the system from a blank disk, explaining each step of the installation process.

The guide covers:

- 🧱 GPT partitioning and filesystem creation
- 🔐 LUKS2 encryption with TPM2 auto-unlocking
- 📦 LVM Physical Volume (PV), Volume Group (VG) and Logical Volume (LV) creation
- 💽 Ext4 filesystem creation and mounting
- 📦 Base system installation
- 🧰 System configuration inside the chroot environment
- 🧬 Unified Kernel Image (UKI) generation and Secure Boot signing
- ⚙️ EFI configuration and boot entry creation
- 🧵 zRAM configuration
- 🌀 Encrypted swap configuration
- 🛡️ Post-install hardening and system optimization
- 🛟 Setup hooks for previous and backups UKI

### 🧱 Step 1 — Pre-Installation Setup

- ⌨️ (Optional) Set keyboard layout to French

```bash
loadkeys fr
```

- 🧼 Clean existing EFI entries if needed (replace X with the entry number)

```bash
efibootmgr
efibootmgr -b X -B
```

- 🔐 Update GPG keys from live environment (recommended before installing)

```bash
pacman -Sy archlinux-keyring
```

### 💽 Step 2 — Disk Partitioning (GPT)

- ⚙️ Partition the disk: EFI (500MB) + LUKS root (rest of disk)

```bash
sgdisk --clear --align-end \
  --new=1:0:+500M --typecode=1:ef00 --change-name=1:"EFI system partition" \
  --new=2:0:0 --typecode=2:8309 --change-name=2:"Linux LUKS" \
  /dev/nvme0n1
```

### 🧼 Step 3 — Main Filesystem Creation

- 🧴 Format EFI partition (optimized for NVMe 4K sector size)

```bash
mkfs.vfat -F 32 -n "SYSTEM" -S 4096 -s 1 /dev/nvme0n1p1
```

- 🔐 Create LUKS2 encrypted container with strong encryption options

```bash
cryptsetup --type luks2 --cipher aes-xts-plain64 --hash sha512 \
  --iter-time 5000 --key-size 512 --pbkdf argon2id \
  --label "Linux LUKS" --sector-size 4096 --use-urandom \
  --verify-passphrase luksFormat /dev/nvme0n1p2
```

- 🔓 Open the LUKS container as /dev/mapper/cryptarch

```bash
cryptsetup --allow-discards --persistent open --type luks2 \
  /dev/nvme0n1p2 cryptarch
```

- 🧱 Initialize the LVM Physical Volume (PV)

```bash
pvcreate /dev/mapper/cryptarch
```

- 🗃️ Create the Volume Group (VG)

```bash
vgcreate vg_system /dev/mapper/cryptarch
```

### 🌳 Step 4 — Logical Volume Creation & Formatting

- 📦 Create the Logical Volumes (LVs)

> 💡 **Adjust the sizes below according to your storage requirements.**

```bash
lvcreate -L 15G -n lv_root vg_system
lvcreate -L 2G -n lv_boot vg_system
lvcreate -L 50G -n lv_home vg_system
lvcreate -L 5G -n lv_var vg_system
lvcreate -L 5G -n lv_log vg_system
lvcreate -L 5G -n lv_tmp vg_system
lvcreate -L 5G -n lv_cache vg_system
lvcreate -L 100G -n lv_virt vg_system
lvcreate -L 50G -n lv_opt vg_system
lvcreate -L 200G -n lv_games vg_system
lvcreate -L 100M -n lv_srv vg_system
lvcreate -L 4G -n lv_swap vg_system
```

> 💡 **Note:** The swap Logical Volume (`lv_swap`) is created in this step to reserve the required storage space, but it will be initialized later in **Step 6** after all filesystems have been created and mounted.

- 🧊 Format the Ext4 Logical Volumes
```bash
mkfs.ext4 -b 4096 -L "Linux Root" -m 1 /dev/vg_system/lv_root
mkfs.ext4 -b 4096 -L "Linux Boot" -m 1 /dev/vg_system/lv_boot
mkfs.ext4 -b 4096 -L "Linux Home" -m 1 /dev/vg_system/lv_home
mkfs.ext4 -b 4096 -L "Linux Var" -m 1 /dev/vg_system/lv_var
mkfs.ext4 -b 4096 -L "Linux Log" -m 1 /dev/vg_system/lv_log
mkfs.ext4 -b 4096 -L "Linux Tmp" -m 1 /dev/vg_system/lv_tmp
mkfs.ext4 -b 4096 -L "Linux Cache" -m 1 /dev/vg_system/lv_cache
mkfs.ext4 -b 4096 -L "Linux Virt" -m 1 /dev/vg_system/lv_virt
mkfs.ext4 -b 4096 -L "Linux Opt" -m 1 /dev/vg_system/lv_opt
mkfs.ext4 -b 4096 -L "Linux Games" -m 1 /dev/vg_system/lv_games
mkfs.ext4 -b 4096 -L "Linux Srv" -m 1 /dev/vg_system/lv_srv
```

---

### 🛠️ Step 5 — Mount Logical Volumes & Prepare the Installation

- 🔧 Mount the root filesystem

```bash
mount -o rw,noatime,nodiratime,errors=remount-ro /dev/vg_system/lv_root /mnt
```

- 🗂️ Create the required mount points

```bash
mkdir -p /mnt/{efi,boot,home,var/{log,tmp,cache,lib/libvirt/images},opt/games,srv}
```

- 🖥️ Mount the EFI System Partition

```bash
mount -o rw,noatime,nodiratime,nodev,nosuid,noexec,fmask=0022,dmask=0022 /dev/nvme0n1p1 /mnt/efi
```

- 🧷 Mount the remaining Logical Volumes

```bash
mount -o rw,noatime,nodiratime,nodev,nosuid,noexec /dev/vg_system/lv_boot /mnt/boot
mount -o rw,noatime,nodiratime,nodev,nosuid /dev/vg_system/lv_home /mnt/home
mount -o rw,noatime,nodiratime,nodev,nosuid /dev/vg_system/lv_var /mnt/var
mount -o rw,noatime,nodiratime,nodev,nosuid,noexec /dev/vg_system/lv_log /mnt/var/log
mount -o rw,noatime,nodiratime,nodev,nosuid,noexec /dev/vg_system/lv_tmp /mnt/var/tmp
mount -o rw,noatime,nodiratime,nodev,nosuid,noexec /dev/vg_system/lv_cache /mnt/var/cache
mount -o rw,noatime,nodiratime,nodev,nosuid,noexec /dev/vg_system/lv_virt /mnt/var/lib/libvirt/images
mount -o rw,noatime,nodiratime,nodev,nosuid /dev/vg_system/lv_opt /mnt/opt
mount -o rw,noatime,nodiratime,nodev,nosuid /dev/vg_system/lv_games /mnt/opt/games
mount -o rw,noatime,nodiratime,nodev,nosuid,noexec /dev/vg_system/lv_srv /mnt/srv
```

---

### 💾 Step 6 — Initialize the Swap Logical Volume

- 🛏️ Initialize the dedicated LVM Swap Logical Volume

```bash
mkswap -L "Linux Swap" /dev/vg_system/lv_swap
```

> 🔐 **Note:** The swap Logical Volume is **not enabled at this stage**.
>
> Arch Fortress uses an **ephemeral encrypted swap** configured through **`/etc/crypttab`**, using a random key generated from **`/dev/urandom`** at every boot.
>
> The encrypted swap mapping and the `swapon` command will therefore be configured later in the installation guide, once the encrypted swap device has been properly configured.

### 📦 Step 7 — Install Base System

- 🧱 Install base packages, kernel + firmwares, EFI tools, text editor, secure boot tools, splash screen and zRam generator service

```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware amd-ucode neovim efibootmgr sbctl plymouth zram-generator
```

> 💡 `base-devel` is required for building packages from the AUR or compiling software from source, and `linux-headers` is needed for DKMS to build and maintain kernel modules.

### 🗂️ Step 8 — Generate fstab

- 📄 Generate fstab with UUIDs

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

- 🔍 (Optional) Review fstab and check `dump` and `fsck` settings for each filesystem

📝 `fstab` uses the last two fields to control `dump` and filesystem checks:

| Value | 🔎 Description |
|-------|----------------|
| `0 0` | 🚫 Disable `dump` and do not check the filesystem automatically with `fsck`. |
| `0 1` | 💾 Disable `dump` and check the filesystem first with `fsck` — used for `/`. |
| `0 2` | 💾 Disable `dump` and check the filesystem after `/` — used for other filesystems. |

> 💡 **Note:** The first value controls the legacy `dump` backup utility and is normally set to `0`. The second value controls the order in which filesystems are checked by `fsck`: `1` for the root filesystem and `2` for other filesystems.

```bash
cat /mnt/etc/fstab
```

- Content:

```bash
UUID=<EXT4-UUID-ROOT>       /                          ext4    rw,noatime,nodiratime,errors=remount-ro                                 0 1
UUID=<FAT32-UUID>           /efi                       vfat    rw,noatime,nodiratime,nodev,nosuid,noexec,fmask=0022,dmask=0022         0 2
UUID=<EXT4-UUID-BOOT>       /boot                      ext4    rw,noatime,nodiratime,nodev,nosuid,noexec                               0 2
UUID=<EXT4-UUID-HOME>       /home                      ext4    rw,noatime,nodiratime,nodev,nosuid                                      0 2
UUID=<EXT4-UUID-VAR>        /var                       ext4    rw,noatime,nodiratime,nodev,nosuid                                      0 2
UUID=<EXT4-UUID-LOG>        /var/log                   ext4    rw,noatime,nodiratime,nodev,nosuid,noexec                               0 2
UUID=<EXT4-UUID-TMP>        /var/tmp                   ext4    rw,noatime,nodiratime,nodev,nosuid,noexec                               0 2
UUID=<EXT4-UUID-CACHE>      /var/cache                 ext4    rw,noatime,nodiratime,nodev,nosuid,noexec                               0 2
UUID=<EXT4-UUID-VIRT>       /var/lib/libvirt/images    ext4    rw,noatime,nodiratime,nodev,nosuid,noexec                               0 2
UUID=<EXT4-UUID-OPT>        /opt                       ext4    rw,noatime,nodiratime,nodev,nosuid                                      0 2
UUID=<EXT4-UUID-GAMES>      /opt/games                 ext4    rw,noatime,nodiratime,nodev,nosuid                                      0 2
UUID=<EXT4-UUID-SRV>        /srv                       ext4    rw,noatime,nodiratime,nodev,nosuid,noexec                               0 2
```

> 💡 **Note:** The swap Logical Volume is intentionally not listed here yet. It will be configured later through `/etc/crypttab` as an **encrypted swap device**, using a randomly generated key from `/dev/urandom`.

> 🔐 **Note:** The `UUID=<...>` values above are placeholders. `genfstab` will automatically generate the correct UUID entries for the mounted filesystems. Review the generated file and verify that the mount options and filesystem check values are correct before continuing.

### 🚪 Step 9 — Enter Chroot

- 🌀 Change root into new system

```bash
arch-chroot /mnt
```

### 🌐 Step 10 — Keyboard & Locale Configuration

- ⌨️ Set virtual console keyboard to French

```bash
nvim /etc/vconsole.conf
```

- Content:

```bash
KEYMAP=fr
FONT=lat9w-16
```

- 🧩 Set X11 keyboard layout

```bash
localectl set-x11-keymap fr pc105 azerty compose:rctrl
```
> 💡 This ensures correct keyboard compatibility with Xorg/XWayland apps and proper layout support in display managers like Plasma Login Manager (used by KDE Plasma).

- 🌍 Set system-wide locale

```bash
nvim /etc/locale.conf
```

- Content:

```bash
LANG=fr_FR.UTF-8
LC_COLLATE=C
LC_MESSAGES=en_US.UTF-8
```

- 🔓 Enable required locales

```bash
nvim /etc/locale.gen
```

- Uncomment:

```bash
en_US.UTF-8 UTF-8
fr_FR.UTF-8 UTF-8
```

- ⚙️ Generate locale definitions

```bash
locale-gen
```

### 🔢 Step 11 — TTY Behavior (Enable NumLock)

- 🧷 Create drop-in to activate NumLock automatically on TTY login

```bash
mkdir /etc/systemd/system/getty@.service.d
nvim /etc/systemd/system/getty@.service.d/activate-numlock.conf
```

- Content:

```bash
[Service]
ExecStartPre=/bin/sh -c 'setleds -D +num < /dev/%I'
```

> 💡 This enables NumLock automatically when a getty session starts on a virtual terminal (TTY).

### 🖥️ Step 12 — Host Identity Configuration

- 🏷️ Set system hostname

```bash
nvim /etc/hostname
```

- Content:

```bash
lianli-arch
```

- 🧭 Set hosts file entries for local networking

```bash
nvim /etc/hosts
```

- Content:

```bash
127.0.0.1      localhost
::1            localhost
127.0.1.1      lianli-arch.zenitram lianli-arch
192.168.1.101  lianli-arch.zenitram lianli-arch
```

### 🕒 Step 13 — Timezone & Clock Setup

- 🌍 Set system timezone

```bash
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
```

- ⏱️ Sync hardware clock with system time

```bash
hwclock --systohc
```

> 💡 This synchronizes the hardware clock with the configured system time.

### 🧩 Step 14 — Initramfs Configuration (AMDGPU Module, Systemd, LUKS, Keyboard)

- ⚙️ Edit initramfs modules and hooks to include AMDGPU driver before anything, systemd & encryption

```bash
nvim /etc/mkinitcpio.conf
```

- Content:

```bash
MODULES=(amdgpu)

HOOKS=(systemd plymouth autodetect microcode modconf kms keyboard sd-vconsole block sd-encrypt lvm2 filesystems fsck)
```

- 🔐 Setup encrypted volume for systemd to unlock via TPM2

```bash
nvim /etc/crypttab.initramfs
```

- Content:

```bash
cryptarch UUID=<NVME-UUID> none tpm2-device=auto,password-echo=no,x-systemd.device-timeout=0,timeout=0,no-read-workqueue,no-write-workqueue,discard
```

- Get `<NVME-UUID>` on neovim:

```bash
:read ! lsblk -dno UUID /dev/nvme0n1p2
```

### 🧵 Step 15 — Kernel Command Line Configuration (UKI + disable zswap)

- ⚙️ Configure the root filesystem and boot parameters

```bash
nvim /etc/cmdline.d/01-root.conf
```

- Content:

```bash
root=UUID=<EXT4-UUID-ROOT> rw loglevel=3 quiet
```

- Get `<EXT4-UUID-ROOT>` on neovim:

```bash
:read ! blkid -s UUID -o value /dev/vg_system/lv_root
```

> 💡 `rw` is required because the `fsck` hook is used in `mkinitcpio`. This allows the root filesystem to be checked before it is mounted during early boot.

> 💡 The `splash` parameter is intentionally omitted because Plymouth is used to provide the boot splash screen. If you want to use the splash screen provided by the `mkinitcpio` preset configured in the following step, you can add `splash` here.

- 🧠 Disable kernel zswap to avoid duplicate swap compression when using zRam as primary swap device

```bash
nvim /etc/cmdline.d/02-zswap.conf
```

- Content:

```bash
zswap.enabled=0
```

### 🧬 Step 16 — Initramfs Preset for Unified Kernel Image (UKI)

- 🔧 Setup mkinitcpio preset to generate a UKI

```bash
nvim /etc/mkinitcpio.d/linux.preset
```

- Content only:

```bash
ALL_kver="/boot/vmlinuz-linux"

PRESETS=('default' 'fallback')

default_uki="/efi/EFI/Linux/arch-linux.efi"

fallback_uki="/efi/EFI/Linux/arch-linux-fallback.efi"
fallback_options="-S autodetect"
```

> 💡 `default_options="--splash=/usr/share/systemd/bootctl/splash-arch.bmp"` is commented out by default and can be uncommented to enable the splash screen, but it may be redundant if **Plymouth** is used.

### 🔐 Step 17 — Secure Boot with sbctl

- 🔑 Create Secure Boot keys

```bash
sbctl create-keys
```

- 📥 Enroll custom keys and micr0$0ft💩 keys

```bash
sbctl enroll-keys -m
```

- 🛠️ Generate the Unified Kernel Image

```bash
mkdir -p /efi/EFI/Linux
mkinitcpio -p linux
```

> ℹ️ Note: TPM2-based disk decryption will be configured after the first reboot to ensure the system is fully initialized and all required services are available.

> 💡 **Automatic UKI signing with `sbctl`**
>
> `sbctl` provides a **`mkinitcpio` post-hook** that automatically signs newly generated UKI files with the enrolled Secure Boot key.
>
> Therefore, no manual `sbctl sign` command is required after running `mkinitcpio`.
>
> - 🔍 Verify that the UKI is correctly signed:
>
> ```bash
> sbctl verify
> ```
>
> - ✍️ If the UKI is not signed, it can be signed manually and added to the `sbctl` database:
>
> ```bash
> sbctl sign -s /efi/EFI/Linux/arch-linux.efi
> ```
>
> The `-s` option saves the file path in the `sbctl` database, allowing it to be included in future automatic signing operations.

- 🧹 Clean up unused initramfs images (Optional)

```bash
rm /boot/initramfs-*.img
```

> 💡 When using a Unified Kernel Image (UKI), the standalone initramfs-*.img files in /boot are no longer used for booting. They may have been generated automatically during the initial system installation with pacstrap.
>
> These leftover images can therefore be safely removed.

### 💻 Step 18 — EFI Boot Entry

- 🧷 Register UKI with UEFI firmware

```bash
efibootmgr --create --disk /dev/nvme0n1 --part 1 --label "Arch Linux" --loader /EFI/Linux/arch-linux.efi --unicode

efibootmgr --create --disk /dev/nvme0n1 --part 1 --label "Arch Linux (Fallback)" --loader /EFI/Linux/arch-linux-fallback.efi --unicode
```

- 🔢 Set UEFI boot order

```bash
efibootmgr -o 0000,0001
```

### 🧠 Step 19 — zRam Setup

- ⚙️ Configure zRam swap device (primary in-memory compressed swap)

```bash
nvim /etc/systemd/zram-generator.conf
```

- Content (balanced gaming + desktop performance):

```bash
[zram0]
zram-size = min(ram / 4, 8 * 1024)
compression-algorithm = zstd
swap-priority = 100
fs-type = swap
```

- 🧮 Configure kernel virtual memory parameters for zRam-based swap behavior

```bash
nvim /etc/sysctl.d/99-vm-zram-parameters.conf
```

- Content (low-latency desktop + gaming responsiveness):

```bash
vm.swappiness = 20
vm.watermark_boost_factor = 0
vm.watermark_scale_factor = 125
vm.page-cluster = 0
```

### 🔄 Step 20 — Encrypted Swap Setup

- 🔐 Add encrypted swap mapping using `/dev/urandom` (secure swap partition via device-mapper)

```bash
nvim /etc/crypttab
```

- Content:

```bash
swap      /dev/vg_system/lv_swap      /dev/urandom      swap,cipher=aes-xts-plain64,sector-size=4096
```

> ⚠️ **Note:** The encrypted swap is identified using the **LVM Logical Volume name** (`/dev/vg_system/lv_swap`) rather than its UUID.
>
> This is intentional because the swap device is recreated and re-encrypted at every boot. Since `mkswap` is used again when the encrypted swap is initialized, a new filesystem UUID is generated each time.
>
> Therefore, the UUID cannot be used directly as a persistent identifier for this swap device. The stable LVM Logical Volume name (`/dev/vg_system/lv_swap`) is used instead.
>
> This avoids the need for an indirect UUID-based method while providing a simple and reliable identifier for the encrypted swap device.

- 📄 Add swap entry with low priority (fallback to zram)

```bash
nvim /etc/fstab
```

- Content:

```bash
#	/dev/vg_system/lv_swap      ENCRYPTED FALLBACK SWAP
/dev/mapper/swap      none      swap      pri=0      0 0
```

### 📦 Step 21 — Pacman Configuration

- 📦 Enable multilib, candy theme & parallel downloads

```bash
nvim /etc/pacman.conf
```

- Content:

```bash
Color
ParallelDownloads = 10
ILoveCandy

[multilib]
Include = /etc/pacman.d/mirrorlist
```

### 🌐 Step 22 — Network Configuration

> 🔀 Choose one network management method depending on your setup
> - ⚙️ `systemd-networkd` → lightweight, minimal, server-friendly, wired only
> - 🖥️ `NetworkManager` → recommended for desktop environments (e.g. KDE Plasma, GNOME) with Wi-Fi support

####  ⚙️ Option A — systemd-networkd (Only Wired, Minimal & Lightweight)

- 📡 Configure wired interface for DHCP, mDNS, and IPv6

```bash
nvim /etc/systemd/network/20-wired.network
```

- Content:

<details>
<summary>📄 <code>20-wired.network</code> content (click to expand)</summary>

```bash
[Match]
Name=eno* ens* enp* eth*

[Link]
RequiredForOnline=routable

[Network]
DHCP=yes
IPv6PrivacyExtensions=yes
MulticastDNS=yes

[DHCPv4]
RouteMetric=100

[IPv6AcceptRA]
RouteMetric=100
```

</details>

####  ⚙️ Option B — NetworkManager (Desktop-Friendly, Wi-Fi Ready)

> 💡 Recommended if you plan to use KDE Plasma, GNOME, or need Wi-Fi support

- 📦 Install NetworkManager

```bash
pacman -Syy networkmanager
```

### 🔌 Step 23 — Basic Packages: Bluetooth, Pacman Cache Service, Reflector, Firewall

- 📦 Install essential tools

```bash
pacman -Syy bluez pacman-contrib reflector firewalld
```

### 🕰️ Step 24 — Time Sync with French NTP Servers

- ⏲️ Set systemd-timesyncd to use French pool servers with iburst

```bash
nvim /etc/systemd/timesyncd.conf
```

- Content:

```bash
[Time]
NTP=0.fr.pool.ntp.org 1.fr.pool.ntp.org 2.fr.pool.ntp.org 3.fr.pool.ntp.org
FallbackNTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
```

### 🚀 Step 25 — I/O Scheduler Tuning for NVMe

- 📉 Disable I/O scheduler on NVMe device to use none (for performance)

```bash
nvim /etc/udev/rules.d/60-schedulers.rules
```

- Content:

```bash
ACTION=="add|change", KERNEL=="nvme[0-9]*", ENV{DEVTYPE}=="disk", ATTR{queue/scheduler}="none"
```

### 🧭 Step 26 — DNS Stub Resolver via systemd-resolved

> ⚠️ Only apply this step if you are using *systemd-networkd* (from Step 23)
> ⏭️ Skip this step if you selected *NetworkManager*

- 🔁 Link stub resolver to `/etc/resolv.conf`

```bash
ln -sf ../run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

### 🌐 Step 27 — Reflector Configuration (Update Mirrorlist)

- 🌍 Optimize pacman mirrors by age, country, and protocol

```bash
nvim /etc/xdg/reflector/reflector.conf
```

- Content:

```bash
--save /etc/pacman.d/mirrorlist
--country France,Germany,Netherlands
--protocol https
--latest 5
--sort age
```

### ⚙️ Step 28 — Enable Key Services (Networking, Bluetooth, Time, Firewall, Trim, Maintenance)

- 🌐 Enable network services (based on your previous choice)

- ⚙️ If using *systemd-networkd*:

```bash
systemctl enable systemd-networkd.service
systemctl enable systemd-resolved.service
```

- 🖥️ If using *NetworkManager*:

```bash
systemctl enable NetworkManager.service
```

- 🔧 Enable essential system services

```bash
systemctl enable bluetooth.service
systemctl enable systemd-timesyncd.service
```

- 🧱 Enable firewall service

```bash
systemctl enable firewalld.service
```

- 🕒 Enable regular TRIM

```bash
systemctl enable fstrim.timer
```

- 🧹 Enable maintenance timers (package cache cleaner & mirrorlist updater)

```bash
systemctl enable paccache.timer
systemctl enable reflector.timer
```

- 🚫 Disable hibernation-related targets (not used / not supported in this setup)

```bash
systemctl mask hibernate.target hybrid-sleep.target
```

> This disables hibernation and hybrid sleep at the systemd level.
> - Swap is encrypted with a non-persistent key, making hibernation unusable  
> - It prevents any accidental suspend-to-disk attempts  
> - It keeps the system configuration cleaner and more explicit
>  
> 💡 On KDE Plasma, this also removes the hibernation option from the power menu, making it cleaner and less confusing.

### 🧰 Step 29 — Change System Editor and Visualiser

- 📝 Define default system editor (used by system tools like systemctl edit, git, etc.)

```bash
nvim /etc/environment
```

- Content:

```bash
EDITOR=nvim
VISUAL=nvim
```

- 🔄 Apply changes immediately (current shell)

```bash
export EDITOR=nvim
```

### 🔑 Step 30 — Configure sudo

- 🛡️ Grant sudo to wheel group

```bash
visudo
```

- Content:

```bash
%wheel ALL=(ALL:ALL) ALL
```

### 🚧 Step 31 — Compilation Optimization (makepkg)

- 🧰 Tune makepkg flags for native arch, use /tmp for build

```bash
nvim /etc/makepkg.conf
```

- Content:

```bash
CFLAGS="-march=native -O2 -pipe ..."
MAKEFLAGS="-j$(nproc)"
BUILDDIR=/tmp/makepkg
```

- 🦀 Optimize Rust build flags

```bash
nvim /etc/makepkg.conf.d/rust.conf
```

- Content:

```bash
RUSTFLAGS="-C opt-level=2 -C target-cpu=native"
```

### 🔇 Step 32 — Disable HDMI Audio

- 🔕 Blacklist HDMI audio module

```bash
nvim /etc/modprobe.d/blacklist.conf
```

- Content:

```bash
blacklist snd_hda_intel
```

### 🔒 Step 33 — Disable Webcam Microphone

- 🎙️ Block Logitech webcam microphone via udev rule

```bash
nvim /etc/udev/rules.d/90-blacklist-webcam-sound.rules
```

- Content:

```bash
SUBSYSTEM=="usb", DRIVER=="snd-usb-audio", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="085c", ATTR{authorized}="0"
```

### ⚡ Step 34 — Allow games Group to Read CPU Power

- 🎮 Grant members of the games group permission to read CPU power (via Intel RAPL interface).

```bash
nvim /etc/udev/rules.d/70-intel-rapl.rules
```

- Content:

```bash
SUBSYSTEM=="powercap", KERNEL=="intel-rapl:0", RUN+="/usr/bin/chgrp games /sys/%p/energy_uj", RUN+="/usr/bin/chmod g+r /sys/%p/energy_uj"
```

> ✅ This ensures users in the `games` group can access CPU energy readings without requiring root privileges — useful for monitoring tools or performance overlays.

### 🔐 Step 35 — Set Root Password

- 🔑 Set root password

```bash
passwd root
```

### 🚪 Step 36 — Exit chroot, Unmount, Reboot into Firmware Setup

- 👋 Exit chroot, unmount and reboot into UEFI/BIOS to check if Secure Boot is enabled

```bash
exit
umount -R /mnt
systemctl reboot --firmware-setup
```

### 🛡️ Step 37 — LUKS TPM2 Key Enrollment

- 🔒 Enroll TPM2 key (PCR 0 = firmware, PCR 7 = Secure Boot state)

```bash
systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7 /dev/nvme0n1p2
```

### 🎮 Step 38 — Shared games directory (multi-user Steam library)

- 🕹️ Allow access and inheritance for users in the `games` group via ACL

```bash
chown root:games /opt/games
chmod 2775 /opt/games
setfacl -dm g:games:rwx /opt/games
```

### 🛡️ Step 39 — Preserve the Previous UKI

To provide a simple recovery mechanism, the currently installed **Unified Kernel Image (UKI)** is preserved before transactions that may trigger a kernel or initramfs regeneration.

The current UKI:

```text
/efi/EFI/Linux/arch-linux.efi
```

is copied to:

```text
/efi/EFI/Linux/arch-linux-previous.efi
```

The pacman transaction then proceeds normally. When the UKI is regenerated, `arch-linux.efi` is replaced with the new version, while `arch-linux-previous.efi` continues to contain the previously working UKI.

The result is:

```text
Arch Linux
└── /EFI/Linux/arch-linux.efi
    └── Current UKI

Arch Linux (Previous)
└── /EFI/Linux/arch-linux-previous.efi
    └── Previous UKI
```

> 💡 The previous UKI is overwritten before each relevant transaction. This means the system always keeps the **previous version of the currently installed UKI**, rather than maintaining multiple kernel versions.

#### 🪝 Create the pacman hook

```bash
nvim /etc/pacman.d/hooks/10-uki_previous.hook
```

- Content:

<details>
<summary>📄 <code>10-uki_previous.hook</code> content (click to expand)</summary>

```bash
## PACMAN PREVIOUS UKI HOOK
## /etc/pacman.d/hooks/10-uki_previous.hook

[Trigger]
Type = Path
Operation = Install
Operation = Upgrade
Operation = Remove
Target = usr/lib/initcpio/*
Target = usr/lib/firmware/*
Target = usr/lib/modules/*/extramodules/
Target = usr/lib/modules/*/vmlinuz
Target = usr/src/*/dkms.conf

[Trigger]
Type = Package
Operation = Install
Operation = Upgrade
Operation = Remove
Target = mkinitcpio
Target = mkinitcpio-git

[Action]
Description = Preserving current UKI as previous...
When = PreTransaction
Exec = /usr/local/sbin/uki_previous.sh
```

</details>

#### ✍️ Create the UKI preservation script

```bash
nvim /usr/local/sbin/uki_previous.sh
```

- Content:

<details>
<summary>📄 <code>uki_previous.sh</code> content (click to expand)</summary>

```bash
#!/bin/bash

## SCRIPT PREVIOUS UKI
##
## /usr/local/sbin/uki_previous.sh
##
## Dependency:
## /usr/local/sbin/efi_bootorder.sh

EFI_CURRENT="/efi/EFI/Linux/arch-linux.efi"
EFI_PREVIOUS="/efi/EFI/Linux/arch-linux-previous.efi"

EFI_DISK="/dev/nvme0n1"
EFI_PARTITION="1"
EFI_LABEL="Arch Linux (Previous)"
EFI_LOADER="/EFI/Linux/arch-linux-previous.efi"

EFI_BOOTORDER="/usr/local/sbin/efi_bootorder.sh"

# Check required dependency
if [ ! -x "$EFI_BOOTORDER" ]; then
    echo "Error: required dependency not found or not executable: $EFI_BOOTORDER" >&2
    exit 1
fi

if [ -f "$EFI_CURRENT" ]; then
    cp -f "$EFI_CURRENT" "$EFI_PREVIOUS"
fi

if [ -f "$EFI_PREVIOUS" ] && ! efibootmgr | grep -q "$EFI_LABEL"; then
    efibootmgr \
        --create-only \
        --disk "$EFI_DISK" \
        --part "$EFI_PARTITION" \
        --label "$EFI_LABEL" \
        --loader "$EFI_LOADER" \
        --unicode

    "$EFI_BOOTORDER"
fi
```

</details>

> 🧠 The hook runs during the `PreTransaction` stage, before the pacman transaction modifies the system. The currently working UKI is therefore preserved before a kernel, firmware, DKMS or initramfs-related update can replace it.

> ⚠️ The **Arch Linux (Previous)** EFI boot entry is created automatically the first time the script successfully creates `arch-linux-previous.efi`. If the entry already exists, no new entry is created.

#### ✍️ Create the EFI Boot reorder script

```bash
nvim /usr/local/sbin/efi_bootorder.sh
```

- Content:

<details>
<summary>📄 <code>efi_bootorder.sh</code> content (click to expand)</summary>

```bash
#!/bin/bash

## SCRIPT EFI BOOT ORDER
##
## /usr/local/sbin/efi_bootorder.sh
##
## Reorder UEFI boot entries numerically.
## This script sorts the existing BootOrder and applies the new order.

set -e

# Get the current BootOrder
BOOT_ORDER=$(efibootmgr | awk -F': ' '/^BootOrder:/ {print $2}')

# Abort if BootOrder could not be retrieved
if [ -z "$BOOT_ORDER" ]; then
    echo "Error: unable to retrieve BootOrder." >&2
    exit 1
fi

# Sort boot entries numerically
BOOT_ORDER=$(echo "$BOOT_ORDER" | tr ',' '\n' | sort | paste -sd, -)

# Apply the new BootOrder
efibootmgr -o "$BOOT_ORDER"
```

#### ✅ Make the script executable

```bash
chmod 750 /usr/local/sbin/uki_previous.sh
chmod 750 /usr/local/sbin/efi_bootorder.sh
```

### 🛡️ Step 39 — Custom Pacman Hook to Backup /efi

- 🪝 Create a hook to automatically backup /efi before critical updates
```bash
nvim /etc/pacman.d/hooks/20-efi_backup.hook
```

- Content:

<details>
<summary>📄 <code>20-efi_backup.hook</code> content (click to expand)</summary>

```bash
## PACMAN EFI BACKUP HOOK
## /etc/pacman.d/hooks/10-efi_backup.hook

[Trigger]
Type = Path
Operation = Install
Operation = Upgrade
Operation = Remove
Target = usr/lib/initcpio/*
Target = usr/lib/firmware/*
Target = usr/lib/modules/*/extramodules/
Target = usr/lib/modules/*/vmlinuz
Target = usr/src/*/dkms.conf

[Trigger]
Type = Package
Operation = Install
Operation = Upgrade
Operation = Remove
Target = mkinitcpio
Target = mkinitcpio-git

[Action]
Description = Backing up /efi...
When = PreTransaction
Exec = /usr/local/sbin/efi_backup.sh
```

</details>

- ✍️ Create the backup script
```bash
nvim /usr/local/sbin/efi_backup.sh
```

- Content:

<details>
<summary>📄 <code>efi_backup.sh</code> content (click to expand)</summary>

```bash
#!/bin/bash

## SCRIPT EFI BACKUP
## /usr/local/sbin/efi_backup.sh

BACKUP_DIR="/boot/efibackup"
BACKUP_COUNT=5

if [ ! -d "$BACKUP_DIR" ]; then
    mkdir -p "$BACKUP_DIR"
fi

if tar -czf "$BACKUP_DIR/efi-$(date +%Y%m%d-%H%M%S).tar.gz" -C / efi; then
    ls -1t "$BACKUP_DIR"/efi-*.tar.gz | \
        tail -n +$((BACKUP_COUNT + 1)) | \
        xargs -r rm --
fi
```

</details>

- ✅ Make it executable
```bash
chmod 750 /usr/local/sbin/efi_backup.sh
```

---

## ❓ FAQ

### ❓ Why no bootloader?

Because **UKI** allows booting the kernel directly from the EFI partition — no need for GRUB or systemd-boot.

### ❓ Can I use this on my laptop?

Yes — it's ideal for modern laptops with TPM2 and Secure Boot enabled.

---

## 🛠 Requirements

- 🖥️ UEFI firmware
- 🧩 TPM 2.0 module
- 🧷 Secure Boot support
- 💿 Recent Arch Linux ISO
- 🌐 Internet access
- 🧮 NVMe drive with 4K physical block size

---

## 📜 License

Licensed under the [MIT License](LICENSE).
Feel free to use, modify, and share!

---

## 👤 Author

Crafted with ❤️ by [joan31](https://github.com/joan31)

> _"Build it clean. Build it solid. Fortress-grade Arch Linux."_
