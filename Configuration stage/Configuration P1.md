# Stage Configuration Scripts
## This file contains the complete configuration commands for the P1 Router
## Device: P1

---

# PART ONE: THE UNDERLYING BACKBONE CONTROL-PLANE ROUTING 
### OSPF BACKBONE CONFIGURATION

```bash
configure terminal

hostname P1

interface Loopback0
 ip address 10.0.0.1 255.255.255.255

interface Ethernet0/0
 description Link_to_PE1
 ip address 10.0.1.2 255.255.255.252
 ip ospf network point-to-point
 no shutdown

interface Ethernet0/1
 description Link_to_P3
 ip address 10.0.2.5 255.255.255.252
 ip ospf network point-to-point
 no shutdown

interface Ethernet0/2
 description Link_to_P2
 ip address 10.0.2.1 255.255.255.252
 ip ospf network point-to-point
 no shutdown

router ospf 1
 router-id 10.0.0.1
 passive-interface Loopback0
 network 10.0.0.1 0.0.0.0 area 0
 network 10.0.1.0 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.3 area 0
 network 10.0.2.4 0.0.0.3 area 0

end
write memory
```

# PART TWO: MPLS AND MP-BGP 
### MPLS CONFIGURATION

```bash
configure terminal

mpls label protocol ldp
mpls ldp router-id Loopback0 force
mpls label range 1000 1999

interface Ethernet0/0
 mpls ip

interface Ethernet0/1
 mpls ip

interface Ethernet0/2
 mpls ip

end
write memory
```

