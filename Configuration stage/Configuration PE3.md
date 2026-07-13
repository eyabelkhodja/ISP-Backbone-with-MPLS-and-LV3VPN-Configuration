# Stage Configuration Scripts
## This file contains the complete configuration commands for the PE3 Router
## Device: PE3

---

# PART ONE: THE UNDERLYING BACKBONE CONTROL-PLANE ROUTING 
### OSPF BACKBONE CONFIGURATION

```bash
configure terminal

hostname PE3

interface Loopback0
 ip address 10.0.0.13 255.255.255.255

interface Ethernet0/0
 description Link_to_P3
 ip address 10.0.1.9 255.255.255.252
 ip ospf network point-to-point
 no shutdown

router ospf 1
 router-id 10.0.0.13
 passive-interface Loopback0
 network 10.0.0.13 0.0.0.0 area 0
 network 10.0.1.9 0.0.0.3 area 0

end
write memory
```

# PART TWO: MPLS AND MP-BGP 
### MPLS CONFIGURATION

```bash
configure terminal

mpls label protocol ldp
mpls ldp router-id Loopback0 force
mpls label range 300 399

interface Ethernet0/0
 mpls ip

end
write memory
```

### MP-BGP CONFIGURATION

```bash
configure terminal

router bgp 65000
 bgp router-id 10.0.0.13
 no bgp default ipv4-unicast

 neighbor 10.0.0.12 remote-as 65000
 neighbor 10.0.0.12 update-source Loopback0

 neighbor 10.0.0.11 remote-as 65000
 neighbor 10.0.0.11 update-source Loopback0

 address-family vpnv4
  neighbor 10.0.0.12 activate
  neighbor 10.0.0.12 send-community extended
  neighbor 10.0.0.11 activate
  neighbor 10.0.0.11 send-community extended
 exit-address-family

end
write memory
```

# PART THREE: VRF AND PE-CE ROUTING
### VRF CONFIGURATION 

```bash
configure terminal

ip vrf ORANGE
 rd 65000:2100
 route-target export 65000:200
 route-target import 65000:200

ip vrf ROUGE
 rd 65000:3000
 route-target export 65000:300
 route-target import 65000:300

end

```

### PE-CE INTERFACE CONFIGURATION

```bash
configure terminal
interface Ethernet0/1
 description Link_to_CE4_VRF_ORANGE
 ip vrf forwarding ORANGE
 ip address 172.16.16.5 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 description Link_to_CE5_VRF_ROUGE
 ip vrf forwarding ROUGE
 ip address 172.16.32.1 255.255.255.252
 no shutdown
!
end

```

### PE-CE OSPF CONFIGURATION 

```bash
configure terminal

router ospf 20 vrf ORANGE
 router-id 172.16.16.5
 domain-id 2.2.2.2
 network 172.16.16.5 0.0.0.3 area 4
 redistribute bgp 65000 subnets

router ospf 30 vrf ROUGE
 router-id 172.16.32.1
 domain-id 3.3.3.3
 network 172.16.32.4 0.0.0.3 area 5
 redistribute bgp 65000 subnets
!
end

```

### BGP-OPSF REDISTRIBUTION

```bash
configure terminal

router bgp 65000
 address-family ipv4 vrf ORANGE
  redistribute ospf 20 vrf ORANGE match internal external 1 external 2
 exit-address-family

 address-family ipv4 vrf ROUGE
  redistribute ospf 30 vrf ROUGE match internal external 1 external 2
 exit-address-family

end

```
- 

---