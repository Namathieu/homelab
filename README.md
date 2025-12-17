# Homelab

Personal homelab configurations and notes. The goal is to keep everything easy to rebuild, easy to migrate, and hard to forget.

## What lives where
- `streaming/` — media and streaming stacks
- `gaming/` — game servers and related tooling
- `networking/` — core internal services (DNS, monitoring, utilities)
- `edge/` — ingress, reverse proxy, TLS, and anything internet-facing
- `scripts/` — small helper scripts
- `docs/` — reference docs (inventory, IP plan, naming)

## Principles
- Treat VMs as disposable; treat data as persistent.
- Keep configuration in Git; keep secrets out of Git.
- Prefer simple, repeatable setups over hand-tuned snowflakes.

## Architecture (high level)
- Proxmox as the hypervisor
- Ubuntu Server VMs (template-based)
- Docker + Docker Compose for services
- Persistent data under `/mnt/data`

## IPs & naming
- Services use static IPs (typically `.100`–`.254`).
- VM names include the last octet of the IP.

Examples:
- `160-streaming`
- `110-networking`
- `130-edge`

## How I deploy things
Each folder is meant to be self-contained. The usual pattern is:
1. Create a VM from the Ubuntu template
2. Make `/mnt/data` available if the stack needs persistence
3. Clone this repo
4. Create a local `.env` (don’t commit it)
5. Run the stack (usually `docker compose up -d`)

## What’s intentionally not in Git
- Secrets (`.env`, credentials, tokens)
- Runtime data (Docker volumes, backups, media)
