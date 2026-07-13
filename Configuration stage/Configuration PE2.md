# Stage Configuration Scripts
## This file contains the complete configuration commands for the PE2 Router
## Device: PE2

---

# PART ONE: THE UNDERLYING BACKBONE CONTROL-PLANE ROUTING 
### OSPF BACKBONE CONFIGURATION

```bash
configure terminal

hostname PE2

interface Loopback0
 ip address 10.0.0.12 255.255.255.255

interface Ethernet0/0
 description Link_to_P2
 ip address 10.0.1.5 255.255.255.252
 ip ospf network point-to-point
 no shutdown

router ospf 1
 router-id 10.0.0.12
 passive-interface Loopback0
 network 10.0.0.12 0.0.0.0 area 0
 network 10.0.1.5 0.0.0.3 area 0

end
write memory
```

# PART TWO: MPLS AND MP-BGP 
### MPLS CONFIGURATION

```bash
configure terminal

mpls label protocol ldp
mpls ldp router-id Loopback0 force
mpls label range 200 299

interface Ethernet0/0
 mpls ip

end
write memory
```

### MP-BGP CONFIGURATION

```bash
configure terminal

router bgp 65000
 bgp router-id 10.0.0.12
 no bgp default ipv4-unicast

 neighbor 10.0.0.11 remote-as 65000
 neighbor 10.0.0.11 update-source Loopback0

 neighbor 10.0.0.13 remote-as 65000
 neighbor 10.0.0.13 update-source Loopback0

 address-family vpnv4
  neighbor 10.0.0.11 activate
  neighbor 10.0.0.11 send-community extended
  neighbor 10.0.0.13 activate
  neighbor 10.0.0.13 send-community extended
 exit-address-family

end
write memory
```

# PART THREE: VRF AND PE-CE ROUTING
### VRF CONFIGURATION 

```bash
configure terminal

ip vrf VERT
 rd 65000:1100
 route-target export 65000:100
 route-target import 65000:100

ip vrf ORANGE
 rd 65000:2000
 route-target export 65000:200
 route-target import 65000:200

end

```

### PE-CE INTERFACE CONFIGURATION

```bash
configure terminal
interface Ethernet0/1
 description Link_to_CE2_VRF_VERT
 ip vrf forwarding VERT
 ip address 172.16.0.5 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 description Link_to_CE3_VRF_ORANGE
 ip vrf forwarding ORANGE
 ip address 172.16.16.1 255.255.255.252
 no shutdown
!
end

```

### PE-CE OSPF CONFIGURATION 

```bash
configure terminal

router ospf 10 vrf VERT
 router-id 172.16.0.5
 domain-id 1.1.1.1
 network 172.16.0.5 0.0.0.3 area 2
 redistribute bgp 65000 subnets

router ospf 20 vrf ORANGE
 router-id 172.16.16.1
 domain-id 2.2.2.2
 network 172.16.16.1 0.0.0.3 area 3
 redistribute bgp 65000 subnets
!
end

```

### BGP-OPSF REDISTRIBUTION

```bash
configure terminal

router bgp 65000
 address-family ipv4 vrf VERT
  redistribute ospf 10 vrf VERT match internal external 1 external 2
 exit-address-family

 address-family ipv4 vrf ORANGE
  redistribute ospf 20 vrf ORANGE match internal external 1 external 2
 exit-address-family

end

```
- 

---