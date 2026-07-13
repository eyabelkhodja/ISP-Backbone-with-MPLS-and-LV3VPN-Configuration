# Stage Configuration Scripts
## This file contains the complete configuration commands for the CE6 Router
## Device: CE6

---

### CONFIGURATION 

```bash
configure terminal
hostname CE6

interface Loopback1
 ip address 172.16.34.1 255.255.255.0

interface Ethernet0/0
 description Link_to_PE1
 ip address 172.16.32.6 255.255.255.252
 no shutdown

router ospf 1
 network 172.16.34.0 0.0.0.255 area 5
 network 172.16.32.6 0.0.0.3 area 5

end
write memory

```
