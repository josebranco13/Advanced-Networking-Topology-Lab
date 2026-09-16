# Configuration Documentation - Router CE-BR

**Device:** CE-BR  
**IOS Version:** Cisco IOS 15.2  
**Last Configuration Change:** Wed Sep 16 2026, 01:56:36 UTC  
**Network Role:** Customer Edge (CE) - Branch Site  
**Routing Protocol:** RIPv2  

---

## 1. Executive Summary & Architecture

Router **CE-BR** operates as the **Customer Edge (CE)** gateway for the "BR" branch network:

1. **LAN Default Gateway:** Provides default gateway services (`192.168.20.1/24`) for local devices on the BR site (e.g., `PC-BR`).
2. **WAN Connectivity:** Connects the customer site to provider edge router **R2** over a point-to-point link (`192.168.200.0/30`).
3. **Dynamic Routing (RIPv2):** Exchanges internal branch routes with provider router R2 via RIP Version 2.

---

## 2. Interface and Addressing Table

| Interface | Description | IPv4 Address | IPv6 Address | Function / Protocol | State |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FastEthernet0/0** | Link to R2 | `192.168.200.2/30` | N/A | WAN Uplink / RIPv2 Peering | Active (Half-Duplex) |
| **FastEthernet1/0** | Link to PC-BR | `192.168.20.1/24` | N/A | LAN Gateway (BR Branch) | Active |
| **FastEthernet1/1** | N/A | Unassigned | N/A | None | Disabled (`shutdown`) |
| **FastEthernet2/0** | N/A | Unassigned | N/A | None | Disabled (`shutdown`) |
| **FastEthernet2/1** | N/A | Unassigned | N/A | None | Disabled (`shutdown`) |

---

## 3. Routing Configuration (RIPv2)

- **Protocol:** RIP (Routing Information Protocol)
- **Version:** 2

### 3.1. Active Networks in RIP

| Declared Network | Netmask / Class | Purpose |
| :--- | :--- | :--- |
| **192.168.20.0** | `255.255.255.0` (`/24`) | Advertises local BR LAN subnet |
| **192.168.200.0** | `255.255.255.252` (`/30`) | Exchanges RIP routes with R2 on the WAN link |

---

## 4. Infrastructure Services & Management

- **CEF:** Enabled for IPv4 (`ip cef`) and IPv6 (`ipv6 cef`).
- **IPv6 Unicast Routing:** Enabled (`ipv6 unicast-routing`).
- **Management & Security:** Disabled DNS lookup, HTTP/HTTPS servers deactivated, exec-timeout 0, level 15 privilege on Console/Aux/VTY.

---

## 5. Original Configuration (Cisco IOS)

```cisco
!
!

!
! Last configuration change at 01:56:36 UTC Wed Sep 16 2026
upgrade fpd auto
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname CE-BR
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
no ip icmp rate-limit unreachable
!
!
!
!
!
!
no ip domain lookup
ip cef
ipv6 unicast-routing
ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
!
redundancy
!
!
ip tcp synwait-time 5
! 
!
!
!
!
!
!
!
!
!
interface FastEthernet0/0
 description Link para R2
 ip address 192.168.200.2 255.255.255.252
 duplex half
!
interface FastEthernet1/0
 description Link para PC-BR
 ip address 192.168.20.1 255.255.255.0
 duplex auto
 speed auto
!
interface FastEthernet1/1
 no ip address
 shutdown
 duplex auto
 speed auto
!
interface FastEthernet2/0
 no ip address
 shutdown
 duplex auto
 speed auto
!
interface FastEthernet2/1
 no ip address
 shutdown
 duplex auto
 speed auto
!
router rip
 version 2
 network 192.168.20.0
 network 192.168.200.0
!
ip forward-protocol nd
no ip http server
no ip http secure-server
!
!
!
no cdp log mismatch duplex
!
!
!
control-plane
!
!
!
mgcp profile default
!
!
!
gatekeeper
 shutdown
!
!
line con 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
 stopbits 1
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
 stopbits 1
line vty 0 4
 login
 transport input all
!
!
end
```
