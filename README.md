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
- [🗂️ DDisk Layout & Volume Architecture](#️-disk-layout--volume-architecture)
- [🔧 Mount Options Summary](#-mount-options-summary)
- [🚀 Automatic Installation (WIP)](#-automatic-installation-wip)
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
- 🧵 **zRAM** enabled for fast compressed in-memory swap
- 💾 Encrypted **swap partition** as zRam fallback
  - Uses a transient encryption key generated at boot from `/dev/urandom`  
  - ⚠️ Hibernation is not possible (non-persistent encryption key)

---

## ⚙️ Features

### 🔐 Security
- Full `/` system encryption with **LUKS2 + TPM2**
- Fallback passphrase support
- Secure Boot ready with signed kernels

### 🧊 Filesystem
- **Ext4** with LVM (physical volume, volume groupe, logical volumes) :
  - `lv_root`, `lv_home`, `lv_var`, etc.
- **zRam** enabled to provide fast compressed RAM-based swap
- Encrypted **swap partition**

### ⚙️ Boot Process
- **No bootloader** (no GRUB, no systemd-boot)
- EFI directly loads a **signed Unified Kernel Image (UKI)**
- UKI built with `mkinitcpio`, containing:
  - Kernel
  - Initramfs
  - Kernel cmdline

### 🧠 Init System
- `mkinitcpio` using:
  - `systemd`, `sd-vconsole`, `sd-encrypt`
- No legacy hooks like `udev`, `usr`, `resume`, `keymap`, `consolefont`, `encrypt`
- Faster, cleaner, future-proof boot

### 🛟 Automatic EFI Backup
- The `/efi` (ESP) is automatically backed up to `/.efibck`
- Useful for system recovery

---

## 📦 Project Structure

<details>
<summary>📁 <code>arch-fortress/</code> (click to expand)</summary>

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

| Logical Volumes Name | Logical Volumes Mapper           | Logical Volumes Devices   | Mount Point               | Description                      |
|----------------------|----------------------------------|---------------------------|---------------------------|----------------------------------|
| `lv_root`            | `/dev/mapper/vg_system-lv_root`  | `/dev/vg_system/lv_root`  | `/`                       | Root system                      |
| `lv_home`            | `/dev/mapper/vg_system-lv_home`  | `/dev/vg_system/lv_home`  | `/home`                   | User data                        |
| `lv_var`             | `/dev/mapper/vg_system-lv_var`   | `/dev/vg_system/lv_var`   | `/var`                    | Variable system data             |
| `lv_log`             | `/dev/mapper/vg_system-lv_log`   | `/dev/vg_system/lv_log`   | `/var/log`                | System logs                      |
| `lv_tmp`             | `/dev/mapper/vg_system-lv_tmp`   | `/dev/vg_system/lv_tmp`   | `/var/tmp`                | Temporary files                  |
| `lv_cache`           | `/dev/mapper/vg_system-lv_cache` | `/dev/vg_system/lv_cache` | `/var/cache`              | Application and package caches   |
| `lv_virt`            | `/dev/mapper/vg_system-lv_virt`  | `/dev/vg_system/lv_virt`  | `/var/lib/libvirt/images` | Virtual machine images           |
| `lv_opt`             | `/dev/mapper/vg_system-lv_opt`   | `/dev/vg_system/lv_opt`   | `/opt`                    | Optional third-party software    |
| `lv_games`           | `/dev/mapper/vg_system-lv_games` | `/dev/vg_system/lv_games` | `/opt/games`              | Games and game libraries         |
| `lv_srv`             | `/dev/mapper/vg_system-lv_srv`   | `/dev/vg_system/lv_srv`   | `/srv`                    | Server data                      |
| `lv_swap`            | `/dev/mapper/vg_system-lv_swap`  | `/dev/vg_system/lv_swap`  | `[SWAP]`                  | Encrypted swap volume (e.g. 4GB) |

---

🧠 **Why this layout?**

This architecture is designed to provide:

- 🔒 **Security** through isolated mount points and dedicated mount options.
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

LVM Logical Volumes (inside /dev/mapper/cryptarch):

```
┌───────────────────────────────────────────────────────────────────────────┐
│ lv_root   → /                                      ← Root filesystem      │
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

Boot process:

```
[ EFI Firmware ]
    ↓
[ UKI Image (.efi) in /efi ]
    ↓
[ systemd (init) in initramfs ]
    ↓
[ Unlock LUKS via TPM2 ]
    ↓
[ Mount BTRFS subvolumes ]
    ↓
[ Boot into secure, modern Arch Fortress 🔐🛡️ ]
```

---

