# CMPG 325 – Computer Networks Individual Project

**Client:** Platinum Sound Recording Studio (Rustenburg) · **Industry:** Entertainment
**Student:** SEDZE, TK (52827097) · **Project ID:** CMPG325-2026-121 · **Client ID:** CLI-121
**Assigned feature:** SSH (secure device management) · **Change request:** CR-14 (after-hours contractor wireless)

## Status

| Milestone | Due | Status |
|---|---|---|
| 1 – Client Design Review | 28 Aug 2026 | Submitted |
| 2 – Client Implementation Review | 02 Oct 2026 | Build complete, all tests passed |
| Final submission | 16 Oct 2026 | In progress (report, video) |

## Network overview

A segmented network built in Cisco Packet Tracer on the assigned block `192.168.52.0/24`, using VLSM, a Layer 3 core switch for inter-VLAN routing, and an edge router for NAT to the internet.

```
INTERNET-SRV ── ISP            (simulated, outside client scope)
                 │
              EDGE-R1           NAT overload, summary route
                 │ /30
              CORE-SW           SVIs, DHCP, GUEST-IN ACL
   ┌──────┬──────┼──────┬──────────┐
SW-ADMIN SW-STUDIO SW-PROD FILE-SRV GUEST-AP
 VLAN10   VLAN20   VLAN30  VLAN40    VLAN50
   │        │        │               ┆ (wireless)
 2 PCs    2 PCs    2 PCs          CONTRACTOR-1
```

## IP addressing plan

| VLAN | Purpose | Subnet | Gateway | Addressing |
|---|---|---|---|---|
| 10 | Admin / Management | 192.168.52.0/27 | .1 | DHCP (.10+); switch SVIs .2–.4 |
| 20 | Recording Studios | 192.168.52.32/27 | .33 | DHCP (.38+) |
| 30 | Production / Editing | 192.168.52.64/27 | .65 | DHCP (.70+) |
| 40 | Servers | 192.168.52.96/28 | .97 | Static (FILE-SRV .98) |
| 50 | Guest wireless (CR-14) | 192.168.52.112/28 | .113 | DHCP (.116+) |
| – | Router ↔ core link | 192.168.52.128/30 | – | .129 (EDGE-R1), .130 (CORE-SW) |
| – | Reserved for growth | 192.168.52.132–.255 | – | – |

## What was implemented

- **VLANs and trunks** – three access switches, each trunking only its own VLAN plus VLAN 10; unused ports shut down.
- **Inter-VLAN routing** – SVIs on the 3560 core switch with `ip routing`; routed /30 uplink to the edge router.
- **DHCP** – four pools on the core switch, with gateway and management addresses excluded.
- **NAT / internet** – PAT on EDGE-R1 to a simulated ISP; one summary static route back to the internal VLANs.
- **Centralised storage** – FILE-SRV in VLAN 40, reachable over FTP from staff VLANs.
- **CR-14 contractor wireless** – WPA2 SSID `PSR-Contractors` in VLAN 50; extended ACL `GUEST-IN` allows DHCP, DNS, HTTP/S and ping to the internet and denies all internal traffic.
- **SSH (assigned feature)** – SSHv2 on all five infrastructure devices, Telnet disabled, VTY access limited to VLAN 10 by `access-class`, local authentication, idle timeouts and a login banner.

## Testing summary

| # | Test | Expected | Result |
|---|---|---|---|
| 1 | VLAN and trunk configuration | Accepted, trunks up | Pass |
| 2 | Core SVIs | Vlan10–50 up/up | Pass |
| 3 | DHCP in all user VLANs | Correct pool and gateway | Pass |
| 4 | Inter-VLAN routing (Admin → Studio) | Reply, TTL=127 | Pass |
| 5 | Internet reachability | Reply, TTL=125 | Pass |
| 6 | DNS and web (`www.example.com`) | Page loads | Pass |
| 7 | NAT overload | 192.168.52.x → 203.0.113.2 | Pass |
| 8 | File server (Production → FILE-SRV) | Ping and FTP login | Pass |
| 9 | Guest reaches internal (before ACL) | Reply – baseline | Pass |
| 10 | Guest blocked from internal (after ACL) | Dropped, 8 ACL matches | Pass |
| 11 | Guest internet (after ACL) | DNS, HTTP, ping permitted | Pass |
| 12 | SSHv2 enabled | `show ip ssh` → version 2.0 | Pass |
| 13 | SSH from VLAN 10 | Encrypted login (aes128-cbc) | Pass |
| 14 | Telnet | Closed, no login prompt | Pass |
| 15 | SSH from VLAN 20 | Connection refused | Pass |

Screenshots for every test are in [`/testing`](testing/), numbered to match the Milestone 2 Implementation Review in [`/docs`](docs/).

## Limitations and deviations

- **Time-based ACL:** Packet Tracer does not support `time-range`, so CR-14's after-hours window cannot be simulated. Isolation is demonstrated in the simulation; the real-IOS `time-range CONTRACTOR-HOURS` configuration is documented in the Implementation Review.
- **SSH scope:** extended from the router and core (Milestone 1) to the access switches as well, so that no infrastructure device accepts cleartext management.
- **ISP and internet server** are a test harness only and use RFC 5737 documentation address ranges.

## Repository layout

```
/configs    running-config of each infrastructure device (passwords redacted)
/testing    evidence screenshots 01–21
/docs       Client Design Review and Implementation Review
52827097_SEDZE_CMPG325-2026-121.pkt
README.md
```
