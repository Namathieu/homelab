# Inventory

This is the “what runs where” page. Keep it updated whenever you add/remove hardware, create a VM, or move a service.

## Hardware

### Proxmox host(s)

| Name | Model/CPU/RAM | Storage | Notes |
|---|---|---|---|
| `pve-01` |  |  |  |

### Storage layout

- **OS / containers**: SSD (keep it disposable)
- **Persistent data**: single HDD mounted at `/mnt/data`

Recommended folders under `/mnt/data` (adjust as needed):
- `/mnt/data/media`
- `/mnt/data/backups`
- `/mnt/data/volumes` (bind mounts / shared Docker data)

## Virtual machines

Guidelines:
- Ubuntu Server VMs (template-based)
- One VM = one main responsibility (streaming, networking, edge, etc.)
- VM names include the **last octet** of their static IP

| VM name | Category | OS | IP | Data needed | Notes |
|---|---|---|---|---|---|
| `110-networking` | Networking | Ubuntu Server | `x.x.x.110` | Optional | Monitoring, DNS, internal tools |
| `130-edge` | Edge | Ubuntu Server | `x.x.x.130` | Optional | Reverse proxy / ingress / TLS |
| `160-streaming` | Streaming | Ubuntu Server | `x.x.x.160` | Yes (`/mnt/data`) | Media stacks |
|  |  |  |  |  |  |

## Services

Track services at a level that makes migrations easy: name, where it runs, and where the data lives.

| Service | Runs on (VM) | Stack location | Data location | Notes |
|---|---|---|---|---|
|  |  |  |  |  |

## Backups

Keep this simple and explicit:
- What is backed up (folders, volumes, configs)
- Where it lands (`/mnt/data/backups`, NAS, cloud, etc.)
- How often
- How to restore (one or two lines)

