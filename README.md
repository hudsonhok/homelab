# 🏠 Home Server Setup

[![Proxmox](https://img.shields.io/badge/Proxmox-VE-orange?logo=proxmox)](https://www.proxmox.com)  
[![ZFS](https://img.shields.io/badge/ZFS-Storage-blue?logo=linux)](https://openzfs.org)  
[![Jellyfin](https://img.shields.io/badge/Jellyfin-Media%20Server-purple?logo=jellyfin)](https://jellyfin.org)  
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

This repository documents the setup, configuration, and management of my **dedicated Proxmox-based home server**.  
It serves as a central hub for self-hosted services, media management, and network-level tools.  

---

## 📑 Table of Contents
- [Hardware](#-hardware)
- [Proxmox Configuration](#-proxmox-configuration)
- [Services & Applications](#-services--applications)
- [Directory Structure](#-directory-structure)
- [Future Plans](#-future-plans--to-do)
- [Notes](#-notes)

---

## 💻 Hardware

| Component      | Details |
|----------------|---------|
| **Case**       | Jonsbo N10 |
| **CPU**        | Intel i5-13500 |
| **Motherboard**| ROG Strix B660-i |
| **RAM**        | G. Skill Ripjaws S5 32GB DDR5 6000MHz CL36 |
| **Cooling**    | Thermalright AXP90-36 |
| **System Disk**| Samsung 980 Pro 1TB NVMe SSD (ext4) |
| **Storage Pool** | 4 × 2TB Samsung PM863 SATA SSDs (ZFS RAID-Z1 `tank`, 8TB usable) |
| **PSU**        | Enhance ENP-8345L 450W Flex ATX Modular |

---

## ⚙️ Proxmox Configuration

- **Base Storage**:
  - `local-lvm`: Root disks, Docker volumes
  - `tank`: ZFS RAID-Z1 (8TB usable) for media storage (`media-dataset`)

- **LXC Containers**:
  <details>
  <summary><b>media</b></summary>

  - Root disk on `local-lvm`  
  - Docker folder on `local-lvm`  
  - 4TB mount from `tank/media-dataset`  

  </details>

  <details>
  <summary><b>adguard</b></summary>

  - Runs **AdGuard Home** for DNS & ad-blocking  
  - Unprivileged container  

  </details>

- **Virtual Machines (VMs)**:
  <details>
  <summary><b>ARR Stack</b></summary>

  - Radarr, Sonarr, Lidarr  
  - Jellyfin for media streaming  
  - Other automation services  

  </details>

---

## 📡 Services & Applications

### 🔹 Network & DNS
- **AdGuard Home** (LXC: `adguard`) – DNS-level ad blocking and network filtering  

### 🔹 Media & Entertainment
- **Jellyfin** – Media server for movies, shows, and music  
- **Radarr / Sonarr / Lidarr** – Automated library management  
- *(Planned)* qBittorrent / SABnzbd for downloads  

### 🔹 Storage
- **ZFS Pool (`tank`)** – 8TB usable RAID-Z1 array with snapshots & redundancy  

---

## 📂 Directory Structure

```plaintext
/tank
  └── media-dataset/          # Main media storage
  └── gameserver-dataset/     # Main game server storage

/local-lvm - for VM root disks and CT volumes for basic service files and configurations
