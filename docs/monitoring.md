# Monitoring

## Tools

| Tool | Function |
|---|---|
| Prometheus | Metrics collection and time-series storage |
| Grafana | Dashboards and visualization |
| Uptime-Kuma | Uptime checks and status page |
| cAdvisor | Container resource metrics |
| Node Exporter | Host metrics (CPU, RAM, disk, network) |
| Blackbox Exporter | Endpoint probing (HTTP/TCP/ICMP) |
| Homepage | Service dashboard / landing page |

## Metrics Collected

| Source | Metrics |
|---|---|
| Node Exporter | CPU, memory, disk I/O, filesystem, network |
| cAdvisor | Per-container CPU, memory, network |
| Blackbox | Endpoint availability and latency |
| Uptime-Kuma | HTTP/TCP uptime and response time |

## Dashboards

- **Grafana** — infrastructure and container metrics.
- **Homepage** — service catalog with status widgets and quick links.
- **Uptime-Kuma** — human-readable up/down overview.

## Alerts

TODO: document the exact alert rules and where notifications are delivered.

- TODO: confirm alert channel (e.g., chat notification) and thresholds.
- TODO: confirm whether backup job failures trigger an alert.

## Known Gaps / TODO

- TODO: Centralize log aggregation.
- TODO: Add alerting for disk space thresholds.
- TODO: Document dashboard ownership and refresh intervals.
