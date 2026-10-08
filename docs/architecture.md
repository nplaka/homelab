# Architecture

## Host

| Property | Value |
|---|---|
| Hypervisor | Proxmox VE 9.x |
| Hostname | `pve` |
| Management IP | `192.0.2.10` (placeholder) |
| Kernel | 7.0.x-pve |

## Containers (LXC)

The environment separates workloads by role. Each LXC container has its own IP, resource limits, and mount points.

| ID | Name | IP (placeholder) | vCPU | RAM | Role |
|---|---|---|---|---|---|
| 100 | `orion` | `192.0.2.11` | 4 | 8 GB | Automation / agent host |
| 101 | `docker-core` | `192.0.2.12` | 4 | 2 GB | Nextcloud + Immich |
| 102 | `docker-projects` | `192.0.2.13` | 4 | 4 GB | Docker stack (main) |
| 103 | `hermes` | `192.0.2.16` | TODO | TODO | Tooling host |
| 104 | `vpn-gateway` | `192.0.2.14` | TODO | TODO | WireGuard gateway |
| 105 | `pihole-dns` | `192.0.2.15` | 2 | 1 GB | Pi-hole + DoH resolver |

### Why separate containers

- **Blast radius:** restarting the busy Docker host does not take down DNS or the VPN.
- **Resource control:** each workload is capped independently.
- **Clear ownership:** photos/files/DNS/VPN each live in one place.

## Storage Layout

| Device | Size | Purpose |
|---|---|---|
| NVMe (`nvme0n1`) | ~476 GB | Proxmox root + LVM-thin (container disks) |
| HDD (`sda`) | ~320 GB | Backup repository #1 |
| 2× HDD (`sdb`/`sdc`) | 2× ~1.8 TB | ZFS mirror `tank` (primary data) |
| External (`sdd`) | ~1 TB | Backup repository #2 |

- Primary data (Docker projects, media) lives on the **ZFS mirror** `tank`.
- Docker containers never run from the NVMe; only the Docker image store stays there for speed.
- Two backup repositories on **two different physical disks** provide redundancy.

## Docker Stack (CT102 / `docker-projects`)

| Service | Purpose |
|---|---|
| Homepage | Service dashboard |
| Portainer | Container management UI |
| Jellyfin | Media server |
| Uptime-Kuma | Uptime monitoring |
| Grafana | Metrics dashboards |
| Prometheus | Metrics collection |
| Backrest | restic backup UI |
| Blackbox Exporter | Endpoint probing |
| Pi-hole | DNS (migrated to CT105) |
| n8n | Workflow automation |
| cAdvisor | Container metrics |
| Node Exporter | Host metrics |

> Some services were split into dedicated containers over time. See [docs/monitoring.md](docs/monitoring.md) and the lessons in [docs/lessons-learned.md](docs/lessons-learned.md).

## Host Services (systemd)

| Unit | Function |
|---|---|
| `cloudflared.service` | Outbound tunnel for select services |
| `orion-port-proxy.service` | `socat` TCP forwards from loopback to containers |
| `earlyoom.service` | OOM protection |
| `journald` limits | Bounded log storage |

> The port-proxy exists because loopback DNAT does not work on this host kernel; 16 host ports are forwarded with `socat` so `localhost:PORT` resolves to the right container.

## Placeholders Used

- Management and container IPs → `192.0.2.x`
- Tunnel ID → `TUNNEL_ID`
- Domain → `example.com`

TODO: confirm exact vCPU/RAM for CT103 and CT104.
