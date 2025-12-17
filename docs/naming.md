# Naming

These conventions keep the lab easy to understand at a glance and help avoid “what box is that?” problems later.

## VM hostnames

Format:

`<last-octet>-<role>`

Examples:
- `110-networking`
- `130-edge`
- `160-streaming`

Rules:
- The number is the **last octet of the VM’s static IP**.
- Keep `<role>` short and obvious (use the repo folder name when it fits).
- One VM should have one main job (edge, networking, streaming, etc.).

## Folder names

Top-level folders are broad categories. Keep stack folders inside them named after the service:

- `streaming/<service>/`
- `gaming/<service>/`
- `networking/<service>/`
- `edge/<service>/`

## Docker / Compose naming

General guidelines (pick a consistent pattern and stick to it):
- Compose project name: use the service name (or `role-service` if you run multiple copies)
- Container names: avoid hard-coding unless you need stable names for external integrations
- Networks/volumes: prefer project-scoped names so stacks don’t collide

## Common examples

- Reverse proxy: `edge/traefik/` or `edge/nginx-proxy-manager/`
- Monitoring: `networking/monitoring/`
- Media stack: `streaming/media/` or split into `streaming/plex/`, `streaming/arr/`, etc.

