# Stage Configuration Scripts
## This file contains the complete configuration commands for the CE4 Router
## Device: CE4

---

### CONFIGURATION 

```bash
configure terminal
hostname CE4

interface Loopback1
 ip address 172.16.18.1 255.255.255.0

interface Ethernet0/0
 description Link_to_PE3
 ip address 172.16.16.6 255.255.255.252
 no shutdown

router ospf 1
 network 172.16.18.0 0.0.0.255 area 4
 network 172.16.16.6 0.0.0.3 area 4

end
write memory

```
