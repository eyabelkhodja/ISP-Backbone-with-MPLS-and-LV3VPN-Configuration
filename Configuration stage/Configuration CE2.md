# Stage Configuration Scripts
## This file contains the complete configuration commands for the CE2 Router
## Device: CE2

---

### CONFIGURATION 

```bash
configure terminal
hostname CE2

interface Loopback1
 ip address 172.16.2.1 255.255.255.0

interface Ethernet0/0
 description Link_to_PE2
 ip address 172.16.0.6 255.255.255.252
 no shutdown

router ospf 1
 network 172.16.2.0 0.0.0.255 area 2
 network 172.16.0.6 0.0.0.3 area 2

end
write memory

```
