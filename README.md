# RUCKUS Network Map: releases

Downloads for RUCKUS Network Map. The source code is private.

Each release has:
- `ruckus-network-map-<version>.ova`: the appliance for VMware ESXi / Workstation / Fusion (Intel) or VirtualBox. Its console shows the web address and initial password.
- `netmap-app-<version>.tar.gz` (+ `.sig`) and `latest.json` (+ `.sig`): the signed in-place upgrade, used by **Settings → Updates → Upgrade now** or `sudo netmap-upgrade` on the appliance.

Upgrade bundles are signed with Ed25519 in the release build. The appliance installs only bundles that verify against the key built into it.
