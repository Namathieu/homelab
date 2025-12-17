# Homelab

Personal homelab infrastructure, Docker stacks, and automation.

## Goals
- Reproducible setup
- Easy migration to new hardware
- Minimal manual configuration
- Docker-first services
- Clear separation of responsibilities

## Architecture Overview
- Proxmox hypervisor
- Ubuntu Server VMs (template-based)
- Docker + Docker Compose
- Central data disk mounted at /mnt/data

## VM Categories
- Streaming
- Gaming
- Networking
- Edge Proxy

## Data Philosophy
- One central data disk (`/mnt/data`)
- Services consume folders, not disks
- VMs are disposable, data is persistent

## Repository Structure
- `streaming/`  : media-related stacks
- `gaming/`     : game server stacks
- `networking/` : monitoring and internal services
- `edge/`       : reverse proxy and ingress
- `scripts/`    : automation helpers
- `docs/`       : documentation and runbook

## Deployment Model
1. Clone this repository
2. Copy `.env.example` -> `.env`
3. Fill required values
4. Run `docker compose up -d`

## Notes
This repository intentionally excludes:
- Secrets
- Docker volumes
- Media data
