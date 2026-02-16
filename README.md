# 🏠 Homelab Infrastructure Architecture

A self-hosted, modular homelab environment built for media management, cloud services, monitoring, development, and game hosting, powered by ZFS storage and virtualized using Proxmox VE.

## Table of Contents

- [Core Principles](#core-principles)
- [🗂️ Architecture Overview](#️-architecture-overview)
- [💾 Data Layer — ZFS](#-data-layer--zfs)
- [🖥️ Host Layer — Proxmox](#️-host-layer--proxmox)
- [📦 System Layer — LXC Containers](#-system-layer--lxc-containers)
- [🧱 Platform Layer — Virtual Machines](#-platform-layer--virtual-machines)
- [📊 Monitoring Stack](#-monitoring-stack)
- [🔐 Security Model](#-security-model)
- [🚀 Key Design Principles](#-key-design-principles)
- [📈 Future Improvements](#-future-improvements)

## Core Principles

This architecture emphasizes:

- 🔐 Service isolation
- 📦 Logical storage separation
- 🌐 Clean network segmentation
- 🔄 Containerized application management
- 📊 Observability and monitoring
- 🧪 (Generally) Safe experimentation environment

## 🗂️ Architecture Overview

```
Physical Host
└── Proxmox (Hypervisor)
    ├── LXC Containers (System Layer)
    ├── Virtual Machines (Platform Layer)
    └── ZFS Storage (Data Layer)
```

## 💾 Data Layer — ZFS

All persistent storage is managed via ZFS using the primary pool: `tank/`

### ZFS Benefits

- Data integrity (checksums)
- Snapshots
- Efficient replication
- Dataset-level control
- Compression support

### 📁 Dataset Layout

```
tank/
├── media/
│   ├── user/        # personal photos + videos
│   ├── movies/      # feature films, anime movies
│   └── shows/       # episodic series, anime
├── services/
│   ├── infra/       # reverse proxy, auth, orchestration
│   ├── monitoring/  # metrics + dashboards
│   ├── apps/        # productivity & personal cloud
│   ├── media/       # media automation stack
│   ├── downloads/   # torrent & usenet clients
│   └── dev/         # development storage
└── backups/         # snapshots & external backups
```

### 🔎 Design Rationale

- `media/` is isolated for high-capacity streaming workloads
- `services/` separates container data by function for easier backup and migration

## 🖥️ Host Layer — Proxmox

The hypervisor is powered by Proxmox VE, enabling both:

- LXC containers (lightweight system services)
- Full Virtual Machines (service segmentation)

### 🌐 Network Layout

| Device Type | IP Range |
|-------------|----------|
| Router | 192.168.0.1 |
| Network Devices | 192.168.0.2–9 |
| LXC Containers | 192.168.0.11–13 |
| Virtual Machines | 192.168.0.20–24 |

Static addressing ensures predictability and clean reverse proxy routing.

## 📦 System Layer — LXC Containers

Lightweight services that benefit from minimal overhead:

| Service | Role | IP |
|---------|------|-----|
| dns | AdGuard Home (network-wide DNS + ad blocking) | 192.168.0.11 |
| vpn | Tailscale (secure remote access) | 192.168.0.12 |
| smb | SMB/NFS file sharing | 192.168.0.13 |

### Why LXC Here?

- Low resource footprint
- Faster startup
- Direct network integration
- Ideal for infrastructure utilities

## 🧱 Platform Layer — Virtual Machines

Each VM isolates a specific workload domain.

### 🏗️ vm-infra — Infrastructure Services

**IP:** 192.168.0.20  
**Specs:** 2C / 4GB RAM / 64GB

#### Docker Services

- Authentik
- Portainer
- Redis
- Traefik

#### Reverse Proxy Mapping

```
jellyfin.local → 192.168.0.30:8096
```

#### Traefik Responsibilities

- TLS termination
- Internal DNS routing
- Service discovery
- Middleware (auth, rate limits)

### ☁️ vm-apps — Personal Cloud Stack

**IP:** 192.168.0.21  
**Specs:** 4C / 6GB RAM / 64GB

#### Docker Services

- Nextcloud
- Vaultwarden
- Immich

#### Purpose

- File sync & collaboration
- Password management
- Photo/video management
- Mobile-first integration

### 🎬 vm-media — Media Automation Stack

**IP:** 192.168.0.22  
**Specs:** 4C / 6GB RAM / 64GB

#### Docker Services

- Jellyfin
- Sonarr
- Radarr
- Prowlarr
- Gluetun

#### VPN Routing

Sonarr, Radarr, and Prowlarr are configured to use Gluetun's network namespace:

```yaml
network_mode: "container:gluetun"
```

This ensures:

- All indexer & download traffic routes through VPN
- Media streaming remains local
- Clean separation of trusted vs external traffic

### 🎮 vm-games — Game Hosting Platform

**IP:** 192.168.0.23  
**Specs:** 8C / 24GB RAM / 128GB

#### Stack

- Pelican Panel
- Wings daemon

#### Designed for:

- Multiplayer game hosting
- Scalable server instances

### 🧪 vm-dev — Experimental Environment

**IP:** 192.168.0.24  
**Specs:** 2C / 4GB RAM / 64GB

#### Used for:

- Testing new services
- Development experiments
- CI/CD concepts

## 📊 Monitoring Stack

Under `tank/services/monitoring/`:

- Prometheus
- Grafana

### Provides:

- VM resource tracking
- Container metrics
- Network observability
- Long-term performance analytics

## 🔐 Security Model

- Centralized authentication via Authentik
- VPN-protected automation stack
- Internal DNS with ad-blocking
- Reverse proxy entrypoint control
- Service isolation by VM
- ZFS snapshot-based backups

## 🚀 Key Design Principles

### 1️⃣ Isolation by Responsibility

Each VM owns a single functional domain.

### 2️⃣ Storage as a First-Class Layer

ZFS datasets align with service boundaries.

### 3️⃣ Containerization Within Virtualization

Hybrid model:
- Proxmox → isolation
- Docker → portability

### 4️⃣ Scalable & Modular

New service?
- Add Docker container
- Or spin up dedicated VM

## 📈 Future Improvements

- Automated ZFS replication to offsite node
- Infrastructure-as-Code (Ansible/Terraform)
- Kubernetes experimentation in vm-dev
