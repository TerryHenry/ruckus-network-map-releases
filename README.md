# RUCKUS Network Map

A live map of a RUCKUS One venue: switches, APs and every connected client, plus the third-party gear RUCKUS One can't see. It adds spanning-tree, Layer-2 path and health views, with monitoring and alerts on top. It works entirely through the RUCKUS One API and the site's own switches and APs, so **nothing has to be installed on site**.

It ships as a **VM appliance (OVA)**: import it, open the web UI, and it keeps itself up to date from signed releases.

> Downloads are on the [Releases](https://github.com/TerryHenry/ruckus-network-map-releases/releases) page. The source code is private.

---

## Feature sheet

### Topology and clients
- **One map per venue:** switches, stacks, APs, mesh links and the LLDP links between them, laid out top-down from the Internet edge. The view and layout you last used are remembered.
- **Every client:** Wi-Fi and wired, with a filter by VLAN. Wi-Fi clients sit on their AP and wired clients on their switch port.
- **Third-party gear is drawn in too:** routers, firewalls, unmanaged switches, servers, IoT and so on. It comes from LLDP/CDP/FDP neighbor tables, switch MAC tables and ARP, and RUCKUS One's own per-port neighbor data.
- **Hard-to-place devices get placed:** an AP whose switch isn't in RUCKUS One is found through its own LLDP neighbor (Detect runs automatically when needed) or through switch MAC tables. Ports where several MACs appear show an inferred unmanaged switch. Stale links from offline switches are dropped.
- **Internet edge detection:** RUCKUS One's cloud-uplink ports are followed from switch to switch to the real gateway. A switch in that path isn't mistaken for the gateway.
- **Identification:** MAC vendors from an offline copy of the full IEEE OUI registry (MA-L, MA-M and MA-S). Randomized MACs are flagged. Vendor logos are shown. LLDP is used to auto-detect device roles.
- **Identify:** checks open ports, SSH/Telnet banners, web titles, TLS certificates and SNMP sysDescr to guess what an unknown device is.
- **IP memory:** static-IP devices keep their last known IP for 30 days after the source that found it forgets it.
- **Change tracking:** devices added, removed or renamed in RUCKUS One appear on the timeline. A monitored device that is later adopted into RUCKUS One is merged, not duplicated.

### Views
- **Topology, Spanning tree and Floorplan**, switchable from the toolbar.
- **Floorplans** come from RUCKUS One, with APs and rogue APs placed on them.

### Spanning tree
- **Per-VLAN root bridge:** predicted from the switches' configured priorities and bridge MACs. **Read real root (SSH)** reads `show span` / `show 802-1w` from every switch to get the root they actually elected, each switch's root port and cost, and per-VLAN port roles and states.
- **Blocked ports:** shown on the map at the end that is actually discarding.
- **Problems flagged:**
  - mixed STP/RSTP on a VLAN
  - STP turned off on a switch
  - every switch left at the default priority, with the root decided by lowest MAC
  - a non-RUCKUS bridge (e.g. a MikroTik) winning the election
  - switches that disagree about the root
  - port states that don't form a valid tree
- **Untagged-VLAN mismatch detection:** links where each end has a different untagged VLAN, which silently merges two VLANs' spanning trees.

### Layer-2 path trace
- **trace-l2 from any ICX:** runs `trace-l2` on a VLAN and draws every switch-to-switch path on the map, colour-coded with direction arrows. Each hop is matched to its device, including switches that aren't in RUCKUS One.

### Finding devices without a box on site
- **Site sweep:** your APs and switches ping a subnet through RUCKUS One.
  - Addresses already on the map can be skipped.
  - Pings that fail to run are retried on another device, and addresses that don't reply get a second try from a different device.
  - Every error is listed with its cause.
- **IPs matched to MACs after a sweep:** the switches' ARP tables are read over SSH afterwards. Each answered IP gets its MAC, so devices already on the map (static ones included) get their IPs.
- **Investigate an unknown IP:** works out where it's connected (Wi-Fi client, behind an AP, or behind a switch port) and adds it to the map in the right place.
- **Local sweep:** the appliance can also sweep the subnets it's connected to directly.

### Monitoring third-party devices
- **Ping, SNMP v2c, MikroTik RouterOS API and ICX SSH CLI**, each with optional per-device credentials. SSH host keys are pinned on first use.
- **Monitor from anywhere:** "Monitor this device" on any device or client, right-click menus on the map, and bulk start, stop and edit with category filters (wireless, wired, AP, router, …).
- **Neighbors you can monitor next:** each switch's LLDP/FDP/CDP neighbors with their management IPs, ready to add.
- **Read switch tables (SSH):** one click reads an ICX's LLDP/FDP/CDP, ARP and MAC tables using the switch login RUCKUS One already manages.
- **Ping from inside the site:** ping and traceroute from any switch or AP, routed through RUCKUS One.

### Health, alerts and history
- **Link health:** errors and utilization per link, plus VLAN and Layer-3 consistency checks.
- **RUCKUS One alarms, events and incidents** on the map and timeline.
- **Syslog and SNMP trap receivers.**
- **WAN probes run from the site:** the site's own APs and switches test the Internet connection.
- **Alerts and history:** timelines and per-device charts. Alerts go to webhooks (Slack, Mattermost, Google Chat).

### Configuration and audit
- **Switch config archive:** automatic backups, side-by-side diffs and restore.
- **Admin activity log** from RUCKUS One.
- **Audit:** findings across switches, APs and Wi-Fi, including rogue APs, evil twins and stale LLDP. Findings can be acknowledged one at a time or in bulk, with notes.

### Remote tools via RUCKUS One
- **Switch tools:** ping, traceroute, IP route table, MAC address table and DHCP server leases. No route from your computer to the switch is needed.
- **AP tools:** ping and traceroute.

### The appliance
- **Network settings:** hostname, DHCP or a static IP, gateway, DNS and NTP servers, set in the web UI or with `sudo netmap-network` on the console. A change made from the web UI reverts on its own unless confirmed from the new address within 3 minutes.
- **LLDP:** the appliance announces itself to its switch, and the network page shows which switch and port it's plugged into.
- **Signed in-place upgrades:** one click, with automatic rollback. Debian security updates install automatically.
- **Accounts:** an admin password and an optional read-only viewer password.

### Reports and export
- **All-venues overview**
- **PDF report**
- **CSV and PNG export**

---

## The appliance

| | |
|---|---|
| Hypervisors | VMware ESXi / Workstation / Fusion (Intel), VirtualBox |
| Virtual hardware | 2 vCPU, 2 GB RAM, 12 GB disk, one NIC (DHCP or static IP) |
| Network | On the site LAN, with Internet access to reach RUCKUS One. Hostname, static IP, DNS, NTP and LLDP are configurable |
| Access | Web UI over HTTPS (self-signed certificate), with admin and read-only viewer passwords |
| Base OS | Debian 12, with automatic security updates |
| Updates | One-click, signed in-place upgrade |

### Quick start
1. Import `ruckus-network-map-<version>.ova` and power it on. It has 2 vCPU, 2 GB RAM, a 12 GB disk and one NIC using DHCP. Put the NIC on the site LAN.
2. The VM console shows the `https://` address and the **initial password**. The web UI takes only a password, with no username. The console/SSH login `admin` uses the same initial password, and you choose a new one at first login.
3. Open the address. Expect a warning about the self-signed certificate. Sign in, then enter your RUCKUS One API credentials in **Settings**.
4. Set your own web password, or add a read-only viewer password, with `sudo netmap-passwd`. The console password is separate.
5. Optional: set its **hostname, a static IP, gateway, DNS and NTP servers** in **Settings → Appliance network**, or with `sudo netmap-network` on the console. The appliance also runs **LLDP**: its switch sees it as a neighbor, and the page shows which switch port it's plugged into. LLDP can be turned off. A change made from the web UI reverts on its own unless you confirm it from the new address within 3 minutes, so a typo can't strand the appliance.

### Upgrades
**Settings → Updates** shows the installed and latest versions. An admin clicks **Upgrade now**, or runs `sudo netmap-upgrade` on the console. The upgrade:
- downloads the new release and checks its **Ed25519 signature** and SHA-256 against the key built into the appliance;
- swaps the new version in and restarts, keeping settings and history;
- **rolls back automatically** if the new version doesn't come up healthy.

Upgrades started from the web UI never downgrade. Debian security updates install automatically.

---

## What's new
- **0.3.3:** set the appliance hostname. LLDP shows which switch port the appliance is on, and makes it visible to that switch.
- **0.3.2:** static IP, gateway, DNS and NTP settings. Site sweeps retry addresses that don't reply. Upgrades also update the appliance's system files.
- **0.3.1:** after a site sweep, the switches' ARP tables match answered IPs to devices on the map, including static ones.
- **0.3.0:** update check and signed one-click upgrades. Internet-edge fix: a switch in the uplink path isn't treated as the gateway.
- **0.2.x:** first appliance release.

---

## Requirements
- A RUCKUS One **API client**: tenant ID, client ID and secret, created under Administration → Account Management → Settings → Application Tokens. Read-only access covers the map and monitoring.
- **Optional:** SSH to the ICX switches, using the switch login RUCKUS One manages or a per-device login. It's needed for Read switch tables, the real spanning-tree root, trace-l2 and the ARP matching after a sweep.

## Safety
- **Read-only by default:** the app changes something on the network only when you choose an action that does, such as restoring a switch config or taking a manual config backup. Everything it runs on switches is a `show` command, except `trace-l2`, which sends probe packets on the chosen VLAN.
- **Secrets are encrypted at rest** on the appliance.
- **The appliance** keeps admin and viewer roles separate, protect against cross-site requests, send strict security headers and rate-limit sign-in attempts.

## Release files
Each release has:
- `ruckus-network-map-<version>.ova` (+ `.sha256`): the appliance.
- `netmap-app-<version>.tar.gz` (+ `.sig`) and `latest.json` (+ `.sig`): the signed in-place upgrade, used by the appliance's **Upgrade now**.

*RUCKUS is a trademark of its owner. This is an independent tool and isn't affiliated with or endorsed by RUCKUS Networks.*
