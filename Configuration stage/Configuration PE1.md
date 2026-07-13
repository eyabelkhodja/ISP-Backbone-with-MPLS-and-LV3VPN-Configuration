# Stage Configuration Scripts
## This file contains the complete configuration commands for the PE1 Router
## Device: PE1

---

# PART ONE: THE UNDERLYING BACKBONE CONTROL-PLANE ROUTING 
### OSPF BACKBONE CONFIGURATION

```bash
configure terminal

hostname PE1

interface Loopback0
 ip address 10.0.0.11 255.255.255.255

interface Ethernet0/2
 description Link_to_P1
 ip address 10.0.1.1 255.255.255.252
 ip ospf network point-to-point
 no shutdown

router ospf 1
 router-id 10.0.0.11
 passive-interface Loopback0
 network 10.0.0.11 0.0.0.0 area 0
 network 10.0.1.1 0.0.0.3 area 0

end
write memory
```

# PART TWO: MPLS AND MP-BGP 
### MPLS CONFIGURATION

```bash
configure terminal

mpls label protocol ldp
mpls ldp router-id Loopback0 force
mpls label range 100 199

interface Ethernet0/2
 mpls ip

end
write memory
```

### MP-BGP CONFIGURATION

```bash
configure terminal

router bgp 65000
 bgp router-id 10.0.0.11
 no bgp default ipv4-unicast

 neighbor 10.0.0.12 remote-as 65000
 neighbor 10.0.0.12 update-source Loopback0

 neighbor 10.0.0.13 remote-as 65000
 neighbor 10.0.0.13 update-source Loopback0

 address-family vpnv4
  neighbor 10.0.0.12 activate
  neighbor 10.0.0.12 send-community extended
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
 rd 65000:1000
 route-target export 65000:100
 route-target import 65000:100

ip vrf ROUGE
 rd 65000:3100
 route-target export 65000:300
 route-target import 65000:300

end

```

### PE-CE INTERFACE CONFIGURATION

```bash
configure terminal

interface Ethernet0/0
 description Link_to_CE1_VRF_VERT
 ip vrf forwarding VERT
 ip address 172.16.0.1 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 description Link_to_CE6_VRF_ROUGE
 ip vrf forwarding ROUGE
 ip address 172.16.32.5 255.255.255.252
 no shutdown
!
end

```

### PE-CE OSPF CONFIGURATION 

```bash
configure terminal

router ospf 10 vrf VERT
 router-id 172.16.0.1
 domain-id 1.1.1.1
 network 172.16.0.1 0.0.0.3 area 1
 redistribute bgp 65000 subnets

router ospf 30 vrf ROUGE
 router-id 172.16.32.5
 domain-id 3.3.3.3
 network 172.16.32.5 0.0.0.3 area 6
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

 address-family ipv4 vrf ROUGE
  redistribute ospf 30 vrf ROUGE match internal external 1 external 2
 exit-address-family

end

```
- 

---