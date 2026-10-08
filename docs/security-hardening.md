# Security Hardening

## SSH

- **Key-based authentication only** for administrative access. Password login is disabled for external access.
- A single dedicated key pair is used per operator machine.
- The management host does not expose SSH to the internet.

TODO: confirm whether `PasswordAuthentication no` and `PermitRootLogin prohibit-password` are set on every host.

## Firewall

- The VPN gateway enforces an `nftables` **kill-switch**: if the tunnel is down, client traffic is dropped rather than leaking to the ISP.
- Default-deny for unknown traffic; explicit allow rules for LAN access and the encrypted exit.
- Host services are bound to loopback where possible and forwarded to containers only where needed.

## Least Privilege

- Administrative API tokens are read-only by default; privileged actions are done over SSH as root on the host.
- Database and service credentials are kept in separate secret files, never in version control.
- Containers run with the minimum privileges required. One container had to run privileged because an unprivileged container could not read its own database (UID mapping); this is a known trade-off, not a default.

## Updates

- Proxmox host and containers are updated on a regular cadence.
- **Container images are pinned** where an automatic upgrade would break behavior.
- One service (a file-sync platform) performs **automatic schema upgrades** on start; its image is pinned and the web UI is not opened until containers report healthy.

## Logging

- Systemd journal storage is capped to limit disk growth.
- Host OOM is mitigated by `earlyoom`, which triggers recovery before the kernel kills critical processes.

## Secret Handling

- No secrets are stored in this repository.
- Placeholder values are used for all hostnames, addresses, and identifiers.
- `.env.example` files document required variables with dummy values.

## Backup Integrity

- Backups are written to **two independent physical disks**.
- A monthly integrity `check` verifies the restic repositories.

## Known Gaps / TODO

- TODO: Add full-disk encryption for backup repositories.
- TODO: Add off-site (3-2-1) copy.
- TODO: Rotate any previously exposed API keys and remove them from scripts.
- TODO: Enable and verify `fail2ban` on all SSH-exposed hosts.
