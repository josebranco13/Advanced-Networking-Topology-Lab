# Configuration Documentation - Router CE-HQ

**Device:** CE-HQ  
**IOS Version:** Cisco IOS 15.2  
**Last Configuration Change:** Wed Sep 16 2026, 02:32:48 UTC  
**Network Role:** Customer Edge (CE) - Headquarter  
**Customer Autonomous System (AS):** 65010  

---

## 1. Executive Summary & Architecture

Router **CE-HQ** acts as the **Customer Edge (CE)** device for the headquarters site (HQ):

1. **HQ LAN Gateway:** Acts as the local default gateway (`192.168.10.1/24`) for headquarters workstations and servers (e.g., `PC-HQ`).
2. **eBGP External Peering:** Establishes eBGP session with provider edge router **R1** (AS 2121) across WAN link `192.168.100.0/30`.
3. **Local Prefix Advertisement:** Injects the HQ local subnet (`192.168.10.0/24`) into the provider's BGP network using private AS `65010`.

---

## 2. Interface and Addressing Table

| Interface | Description | IPv4 Address | IPv6 Address | Function / Protocol | State |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FastEthernet0/0** | Link to R1 | `192.168.100.2/30` | N/A | WAN Uplink / eBGP Peering | Active (Half-Duplex) |
| **FastEthernet1/0** | Link to PC-HQ | `192.168.10.1/24` | N/A | LAN Gateway (HQ Site) | Active |
| **FastEthernet1/1** | N/A | Unassigned | N/A | None | Disabled (`shutdown`) |
| **FastEthernet2/0** | N/A | Unassigned | N/A | None | Disabled (`shutdown`) |
| **FastEthernet2/1** | N/A | Unassigned | N/A | None | Disabled (`shutdown`) |

---

## 3. BGP Configuration (AS 65010)

- **BGP Process:** `65010` (Private AS)
- **Router ID:** `192.168.100.2`

### 3.1. External Peering (eBGP) with R1 (AS 2121)

| Neighbor | IP Address | Remote AS | Address Family | Status |
| :--- | :--- | :--- | :--- | :--- |
| **R1 (IPv4)** | `192.168.100.1` | 2121 | IPv4 Unicast | Active |

### 3.2. Advertised Prefixes
- **IPv4 Network:** `192.168.10.0` (`/24`)

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
! Last configuration change at 02:32:48 UTC Wed Sep 16 2026
upgrade fpd auto
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname CE-HQ
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
 description Link para R1
 ip address 192.168.100.2 255.255.255.252
 duplex half
!
interface FastEthernet1/0
 description Link para PC-HQ
 ip address 192.168.10.1 255.255.255.0
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
router bgp 65010
 bgp router-id 192.168.100.2
 bgp log-neighbor-changes
 network 192.168.10.0
 neighbor 192.168.100.1 remote-as 2121
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
