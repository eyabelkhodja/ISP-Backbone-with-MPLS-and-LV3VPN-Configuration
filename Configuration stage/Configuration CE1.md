# Stage Configuration Scripts
## This file contains the complete configuration commands for the CE1 Router
## Device: CE1

---

### CONFIGURATION 

```bash
configure terminal
hostname CE1

interface Loopback1
 ip address 172.16.1.1 255.255.255.0

interface Ethernet0/0
 description Link_to_PE1
 ip address 172.16.0.2 255.255.255.252
 no shutdown

router ospf 1
 network 172.16.1.0 0.0.0.255 area 1
 network 172.16.0.0 0.0.0.3 area 1

end
write memory

```
