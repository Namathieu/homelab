# IP Plan

This homelab uses static IPs for anything that matters. The goal is that a rebuild doesn’t require hunting for addresses.

## Rules of thumb

- **Static range**: `.100`–`.254` (servers, VMs, infrastructure)
- **Naming**: VM hostname includes the last octet (see `docs/naming.md`)

## Subnet(s)

Fill this in with your real network details:

| Network | CIDR | Gateway | DNS | DHCP scope | Notes |
|---|---|---|---|---|---|
| LAN | `x.x.x.0/24` | `x.x.x.1` |  |  |  |

## Reserved ranges (suggested)

These are not strict requirements—just a pattern that keeps things tidy.

- `.1` — gateway/router
- `.2`–`.99` — DHCP clients (laptops, phones, misc devices)
- `.100`–`.199` — homelab servers / VMs
- `.200`–`.254` — infrastructure / “special” devices (APs, switches, IPMI, etc.)

## Static assignments

| IP (last octet) | Hostname | Role | Notes |
|---:|---|---|---|
| 110 | `110-networking` | Networking VM | Core internal services (DNS/monitoring/tools) |
| 130 | `130-edge` | Edge VM | Reverse proxy / ingress |
| 160 | `160-streaming` | Streaming VM | Media stacks |
|  |  |  |  |

## Quick checks

- If a VM is renamed, update DNS records (and anything that references it).
- If an IP changes, update the hostname to match the last octet.

