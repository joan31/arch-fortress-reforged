# 🏰 Arch Fortress: Reforged — Modern, Secure & Minimal Arch Linux Installation Guide

![Linux](https://img.shields.io/badge/OS-Linux-black?style=flat-square&logo=linux&logoColor=white)
![Arch Linux](https://img.shields.io/badge/Distro-Arch-blue?style=flat-square&logo=arch-linux)
![EFI](https://img.shields.io/badge/Firmware-EFI-white?style=flat-square&logo=rocket&logoColor=white)
![UKI](https://img.shields.io/badge/Boot-UKI-purple?style=flat-square&logo=linuxfoundation&logoColor=white)
![LUKS2 + TPM2](https://img.shields.io/badge/Encryption-LUKS2%20%2B%20TPM2-orange?style=flat-square&logo=cryptpad&logoColor=white)
![Secure Boot](https://img.shields.io/badge/Secure%20Boot-Enabled-teal?style=flat-square&logo=socket&logoColor=white)
![LVM](https://img.shields.io/badge/Storage-LVM-darkslategray?style=flat-square&logo=discogs&logoColor=white)
![EXT4](https://img.shields.io/badge/Filesystem-EXT4-deepskyblue?style=flat-square&logo=buffer&logoColor=white)
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
- [📦 Structure](#-structure)
- [🗂️ Disk Layout & Subvolume Architecture](#️-disk-layout--subvolume-architecture)
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
- **EXT4** with LVM (physical volume, volume groupe, logical volumes) :
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

## 📦 Structure

<details>
<summary>📁 <code>arch-fortress/</code> (click to expand)</summary>

```
arch-fortress/
├── LICENSE
└── README.md
```

</details>

---

## 🗂️ Disk Layout & Volume Architecture

> This is the storage layout used by **Arch Fortress: Reforged**, based on a secure and flexible setup combining LUKS2, LVM+EXT4, and EFI boot with UKI.

### 💽 Partition Table (GPT - `/dev/nvme0n1`)

| Partition        | Type              | FS    | Mount Point | Size | Description                         |
|------------------|-------------------|-------|-------------|------|-------------------------------------|
| `/dev/nvme0n1p1` | EFI System (ef00) | FAT32 | `/efi`      | 500M | EFI System Partition (boot via UKI) |
| `/dev/nvme0n1p2` | Linux LUKS (8309) | LUKS2 | (LUKS)      | ~2TB | Encrypted root volume               |

---

### 🔐 Encrypted Volume

- `/dev/nvme0n1p2` is encrypted using **LUKS2 + TPM2**
- Mapped as `/dev/mapper/cryptarch`
- Inside: **LVM** and EXT4 filesystem with multiple logical volumes

---

### 🌳 LVM Structure

| Logical volumes | Mount Point               | Description                         |
|-----------------|---------------------------|-------------------------------------|
| `lv_root`       | `/`                       | Root system                         |
| `lv_home`       | `/home`                   | User data                           |
| `lv_var`        | `/var`                    | Pacman cache                        |
| `lv_log`        | `/var/log`                | System logs                         |
| `lv_tmp`        | `/var/tmp`                | Temporary files                     |
| `lv_cache`      | `/var/cache`              | Cache data                          |
| `lv_virt`       | `/var/lib/libvirt/images` | Virtual machines                    |
| `lv_opt`        | `/opt`                    | Optional game data                  |
| `lv_swap`       | `none`                    | Encrypted swap partition (e.g. 4GB) |

---
