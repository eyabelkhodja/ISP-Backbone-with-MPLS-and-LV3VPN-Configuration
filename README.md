# ISP-Backbone-with-MPLS-and-LV3VPN-Configuration

A service-provider backbone lab built in PNetLab, simulating an ISP core offering
Layer 3 VPN services to three customers over a shared MPLS/BGP infrastructure.

## Overview

- **Platform:** PNetLab, using Cisco IOL images (chosen over Dynamips for full
  physical emulation of RAM/CPU/interfaces, and lightweight footprint for a
  12-node topology)
- **Autonomous System:** 65000
- **Core IGP:** OSPF (single area design was extended to a per-customer area
  scheme on PE-CE links — see below)
- **MPLS signaling:** LDP
- **VPN control plane:** MP-BGP (VPNv4 address family), full iBGP mesh — no
  route reflector needed given the small number of well-meshed PEs
- **PE-CE IGP:** OSPF, uniformly, to keep the lab focused on VRF/MPLS design
  rather than IGP redistribution complexity
- **Link type:** Ethernet throughout (preferred over serial for bandwidth,
  cost, and negligible impact at these speeds)

## Topology

**Core (P) routers:** P1, P2, P3 — full triangle mesh
**Provider edge (PE) routers:** PE1, PE2, PE3 — each single-homed to one P router
**Customer edge (CE) routers:** CE1–CE6, two per PE, one per VRF

```
PE1 → P1        (single link)
PE2 → P2        (single link)
PE3 → P3        (single link)
P1 ↔ P2 ↔ P3    (full triangle core)

PE1 — CE1 (VRF VERT)   PE1 — CE6 (VRF ROUGE)
PE2 — CE2 (VRF VERT)   PE2 — CE3 (VRF ORANGE)
PE3 — CE4 (VRF ORANGE) PE3 — CE5 (VRF ROUGE)
```

## Addressing plan

### Loopbacks (also OSPF/LDP/BGP router-ids)

| Router | Loopback0 |
|---|---|
| P1 | 10.0.0.1/32 |
| P2 | 10.0.0.2/32 |
| P3 | 10.0.0.3/32 |
| PE1 | 10.0.0.11/32 |
| PE2 | 10.0.0.12/32 |
| PE3 | 10.0.0.13/32 |

### Core links (OSPF area 0, `ip ospf network point-to-point`, `mpls ip`)

| Link | Subnet |
|---|---|
| P1–P2 | 10.0.2.0/30 |
| P1–P3 | 10.0.2.4/30 |
| P2–P3 | 10.0.2.8/30 |
| PE1–P1 | 10.0.1.0/30 |
| PE2–P2 | 10.0.1.4/30 |
| PE3–P3 | 10.0.1.8/30 |

### PE-CE links (per VRF, own OSPF process + area per CE)

| Link | VRF | Subnet | CE OSPF area |
|---|---|---|---|
| PE1–CE1 | VERT | 172.16.0.0/30 | 1 |
| PE2–CE2 | VERT | 172.16.0.4/30 | 2 |
| PE2–CE3 | ORANGE | 172.16.16.0/30 | 3 |
| PE3–CE4 | ORANGE | 172.16.16.4/30 | 4 |
| PE3–CE5 | ROUGE | 172.16.32.0/30 | 5 |
| PE1–CE6 | ROUGE | 172.16.32.4/30 | 6 |

### Customer site networks (CE Loopback1, simulating the customer LAN)

| CE | VRF | Site network |
|---|---|---|
| CE1 | VERT | 172.16.1.0/24 |
| CE2 | VERT | 172.16.2.0/24 |
| CE3 | ORANGE | 172.16.17.0/24 |
| CE4 | ORANGE | 172.16.18.0/24 |
| CE5 | ROUGE | 172.16.33.0/24 |
| CE6 | ROUGE | 172.16.34.0/24 |

> **Note:** loopback interfaces force OSPF to advertise a `/32` regardless of the
> configured mask. Each CE loopback needs `ip ospf network point-to-point` to
> get OSPF to advertise the intended `/24` instead.

## VRF design

| VRF | Route-target | PEs serving it |
|---|---|---|
| VERT | 65000:100 | PE1, PE2 |
| ORANGE | 65000:200 | PE2, PE3 |
| ROUGE | 65000:300 | PE1, PE3 |

| PE | VRF | RD |
|---|---|---|
| PE1 | VERT | 65000:1000 |
| PE2 | VERT | 65000:1100 |
| PE2 | ORANGE | 65000:2000 |
| PE3 | ORANGE | 65000:2100 |
| PE3 | ROUGE | 65000:3000 |
| PE1 | ROUGE | 65000:3100 |

RDs are unique per PE (to avoid VPNv4 best-path collisions when the same prefix
is announced from two locations); RTs are shared per VRF to control which PEs
import a given customer's routes.

## MPLS label ranges

Per-router label ranges assigned via `mpls label range` for readability when
inspecting label-switched paths:

| Router | Range |
|---|---|
| P1 | 1000–1999 |
| P2 | 2000–2999 |
| P3 | 3000–3999 |
| PE1 | 100–199 |
| PE2 | 200–299 |
| PE3 | 300–399 |
