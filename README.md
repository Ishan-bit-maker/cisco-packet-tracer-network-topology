# Network Topology Simulation — Cisco Packet Tracer

A multi-network simulation built in Cisco Packet Tracer featuring two subnets, inter-network routing, centralized DHCP with relay, and 11 end devices. All devices receive IP addresses dynamically and can communicate across subnets.

---

## Topology Overview



```
                         [Router0 - 2811]
                        /                \
               f0/0 (10.1)          f0/1 (20.1)
                  |                       |
            [Switch0 - 2960]        [Switch1 - 2960]
           /     |     |    \       /   |   |   |   \
        PC0    PC1  Laptop0  PC2  PC3  PC4  PC5  Laptop1 Laptop2 Laptop3
        
[Server-PT DHCP] — connected to Switch0
```

---

## IP Addressing Scheme

### Subnet 1 — 192.168.10.0/24
| Device       | Role         | IP Address     |
|--------------|--------------|----------------|
| Router0 f0/0 | Default GW   | 192.168.10.1   |
| Server-PT    | DHCP Server  | 192.168.10.2   |
| PC0          | End Device   | DHCP assigned  |
| PC1          | End Device   | DHCP assigned  |
| PC2          | End Device   | DHCP assigned  |
| Laptop0      | End Device   | DHCP assigned  |

### Subnet 2 — 192.168.20.0/24
| Device       | Role         | IP Address     |
|--------------|--------------|----------------|
| Router0 f0/1 | Default GW   | 192.168.20.1   |
| PC3          | End Device   | DHCP assigned  |
| PC4          | End Device   | DHCP assigned  |
| PC5          | End Device   | DHCP assigned  |
| Laptop1      | End Device   | DHCP assigned  |
| Laptop2      | End Device   | DHCP assigned  |
| Laptop3      | End Device   | DHCP assigned  |

---

## Device Configuration

### Router0 (2811)

```
! Interface facing Subnet 1
interface FastEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.10.2
 no shutdown

! Interface facing Subnet 2
interface FastEthernet0/1
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.10.2
 no shutdown
```

### Server-PT (DHCP Server)

**Pool 1 — Subnet 1**
| Field           | Value           |
|-----------------|-----------------|
| Pool Name       | subnet1         |
| Network         | 192.168.10.0    |
| Subnet Mask     | 255.255.255.0   |
| Default Gateway | 192.168.10.1    |
| DNS Server      | (optional)      |
| Start IP        | 192.168.10.3    |

**Pool 2 — Subnet 2**
| Field           | Value           |
|-----------------|-----------------|
| Pool Name       | subnet2         |
| Network         | 192.168.20.0    |
| Subnet Mask     | 255.255.255.0   |
| Default Gateway | 192.168.20.1    |
| DNS Server      | (optional)      |
| Start IP        | 192.168.20.2    |

### Switch0 / Switch1 (2960)
Default configuration — no VLANs, all ports in access mode on VLAN 1.

### End Devices (PC0–PC5, Laptop0–Laptop3)
All set to **DHCP** under Desktop → IP Configuration.

---

## How DHCP Relay Works

Subnet 2 has no local DHCP server. When a device on Switch1 sends a DHCP broadcast, Router0's `ip helper-address` on f0/1 forwards it as a unicast packet to the DHCP server at `192.168.10.2`. The server responds with a lease from the subnet2 pool, and the router relays it back.

```
PC3 broadcasts DHCP Discover
    → Router0 f0/1 receives it
    → Forwarded unicast to 192.168.10.2
    → Server replies with 192.168.20.x lease
    → Router0 relays back to PC3
```

---

## Verification

### Check all interfaces are up
```
Router0# show ip interface brief
```

### Verify routing table
```
Router0# show ip route
```

### Test end-to-end connectivity
```
! From PC0 (192.168.10.x), ping a device on Subnet 2
ping 192.168.20.x

! From PC3 (192.168.20.x), ping the DHCP server
ping 192.168.10.2
```

---

## Features Demonstrated

- Static IP assignment on router interfaces
- Centralized DHCP server serving two subnets
- DHCP relay (`ip helper-address`) for cross-subnet lease delivery
- Inter-network routing via a single 2811 router
- Layer 2 switching with 2960 switches
- End-to-end connectivity verified across all 11 devices

---

## Files

| File | Description |
|------|-------------|
| `topology.pkt` | Cisco Packet Tracer project file |
| `README.md` | This document |
