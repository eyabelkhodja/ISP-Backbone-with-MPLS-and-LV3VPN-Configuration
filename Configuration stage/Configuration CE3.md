# Stage Configuration Scripts
## This file contains the complete configuration commands for the CE3 Router
## Device: CE3

---

### CONFIGURATION 

```bash
configure terminal
hostname CE3

interface Loopback1
 ip address 172.16.17.1 255.255.255.0

interface Ethernet0/0
 description Link_to_PE2
 ip address 172.16.16.2 255.255.255.252
 no shutdown

router ospf 1
 network 172.16.17.0 0.0.0.255 area 3
 network 172.16.16.2 0.0.0.3 area 3

end
write memory

```
