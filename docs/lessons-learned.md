# Lessons Learned

Real problems encountered while building and running this lab, with cause and fix.

## 1. Loopback DNAT did not work on the host kernel

**Symptom:** Services reachable by IP from other machines were *not* reachable via `localhost:PORT` on the host, unlike the previous workstation setup.

**Cause:** DNAT (destination NAT) to loopback does not behave as expected on this host kernel.

**Fix:** Replaced DNAT with a `socat` port-forwarding service that maps host loopback ports to the correct container IPs. The service is defined in a small config file and enabled as a systemd unit.

**Lesson:** Don't assume network behavior carries over between kernels. When NAT semantics differ, a userspace forwarder is a reliable fallback. Also learned that an invisible `localhost` expectation can silently break automation that assumes it.

## 2. Pi-hole on a busy Docker host caused DNS outages

**Symptom:** Restarting or recreating the main Docker host briefly took down DNS for the whole house.

**Cause:** Pi-hole shared a container with the primary workload, so any host maintenance affected the resolver.

**Fix:** Migrated Pi-hole and its DNS-over-HTTPS resolver to a **dedicated container** with its own IP, so DNS is independent of the main Docker host.

**Lesson:** Separate stateful, always-needed infrastructure (DNS, VPN) from the container that changes most often. Blast radius is an architectural decision, not an afterthought.

## 3. Unprivileged container could not read its own database

**Symptom:** After migrating Pi-hole to a new unprivileged container, it reported that it could not read the gravity database.

**Cause:** UID mapping in unprivileged LXC made the database files appear owned by `nobody`, so the service could not open them.

**Fix:** Ran the container as **privileged**, matching the UID of the shared storage.

**Lesson:** LXC UID mapping matters when a container shares storage with other containers. Privileged mode is a deliberate trade-off — document *why* it is needed instead of treating it as a default.

## 4. Host-level disk tools could not run inside the container

**Symptom:** A disk-health (SMART) monitoring tool failed inside its container.

**Cause:** `CAP_SYS_RAWIO` is blocked by the LXC kernel, and raw block devices are not exposed inside the container.

**Fix:** Run the SMART collector on the **host** and keep only the UI in a container.

**Lesson:** Hardware-level access belongs on the host. Not everything can — or should — be containerized.

## 5. Dynamic public IP broke VPN clients

**Symptom:** Remote WireGuard clients stopped connecting after the ISP changed the public IP.

**Cause:** Client configs hard-coded the public IP, which is dynamic.

**Fix:** Switched clients to a **provider dynamic-DNS hostname** that always resolves to the current IP. Client configs never contain a raw public IP.

**Lesson:** Never hard-code a dynamic value in a config. A stable name is the correct abstraction, and it makes the setup resilient to something you do not control.
