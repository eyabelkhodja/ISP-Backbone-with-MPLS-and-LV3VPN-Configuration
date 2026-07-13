#Configuration and verification steps

## Configuration layers, in build order

1. **Core OSPF (area 0)** — loopbacks + core links, `point-to-point` network
   type on all core interfaces to skip DR/BDR election
2. **MPLS + LDP** — `mpls ip` on every core-facing interface, `mpls ldp
   router-id Loopback0 force` on every router
3. **MP-BGP** — full iBGP mesh between PE loopbacks, `address-family vpnv4`,
   `no bgp default ipv4-unicast`, `update-source Loopback0`
4. **VRF definitions** — `ip vrf <name>` with RD + import/export route-targets
5. **PE-CE interfaces** — `ip vrf forwarding <name>` before re-adding the IP
   address (VRF assignment wipes any existing address)
6. **PE-CE OSPF** — one OSPF process per VRF per PE (`router ospf <id> vrf
   <name>`), each with:
   - a matching **domain-id** across every PE serving that VRF, so routes
     re-entering OSPF from BGP show up as inter-area (`O IA`) rather than
     external (`O E2`)
   - `redistribute bgp 65000 subnets` to pull remote-site VPNv4 routes into
     the local CE's OSPF
7. **BGP-side redistribution** — `redistribute ospf <id> vrf <name> match
   internal external 1 external 2` under `address-family ipv4 vrf <name>`,
   to push locally-learned CE routes into BGP regardless of how OSPF
   currently classifies them
8. **CE configuration** — plain (non-VRF) OSPF process per CE, advertising its
   loopback site network and PE-CE link into the matching area

## Verification performed

- **Core OSPF:** full adjacency mesh (P-routers: 3 neighbors each; PEs: 1 core
  neighbor + 2 CE neighbors once VRFs were live), all six loopbacks visible via
  `show ip route ospf`
- **MPLS/LDP:** `show mpls ldp neighbor`, `show mpls forwarding-table`,
  confirmed correct per-router label ranges and PHP (penultimate hop popping)
  behavior on P-adjacent prefixes
- **MP-BGP:** `show bgp vpnv4 unicast all summary` — all PE pairs
  `Established`; `show bgp vpnv4 unicast all` — confirmed correct RD tagging,
  RT-based import, and `?` (incomplete) origin codes consistent with
  redistributed routes
- **VRF routing tables:** `show ip route vrf <name>` on each PE, confirming
  both local (directly connected) and remote (BGP-redistributed, `O IA`
  tagged) site routes present
- **Data plane:** `show ip cef vrf <name> <prefix>` confirming the correct
  stacked label (VPN label + transport label) is imposed for every remote
  prefix
- **End-to-end reachability:** `ping`/`traceroute` between CE loopbacks within
  the same VRF succeed, showing the full label-switched path (transport label
  changing hop-by-hop across the core, VPN label constant end-to-end, popped
  entirely at the final PE-CE hop)
- **VRF isolation:** cross-VRF pings (e.g. CE1 in VERT to CE3 in ORANGE)
  confirmed to fail, proving route-target scoping is correctly enforced and
  no unintended route leaking occurs between customers
