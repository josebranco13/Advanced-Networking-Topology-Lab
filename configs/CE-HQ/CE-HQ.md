# Documentação de Configuração - Roteador CE-HQ

**Dispositivo:** CE-HQ  
**Versão do IOS:** Cisco IOS 15.2  
**Data da Última Alteração:** 16 de Setembro de 2026, 02:32:48 UTC  
**Função na Rede:** Customer Edge (CE) - Headquarter  
**Sistema Autónomo do Cliente (AS):** 65010  

---

## 1. Resumo Executivo e Arquitetura

O roteador **CE-HQ** atua como equipamento **Customer Edge (CE)** para a sede principal (Headquarter - "HQ"). As suas principais funções incluem:

1. **Gateway da Rede Local (LAN HQ):** Provê o gateway padrão (`192.168.10.1/24`) para os equipamentos da LAN da sede (ex.: `PC-HQ`).
2. **Conectividade Exterior eBGP:** Estabelece sessão eBGP com o roteador de borda do provedor (**R1**, AS 2121) no enlace de WAN `192.168.100.0/30`.
3. **Anúncio de Prefixo Local:** Anuncia a sub-rede interna da sede (`192.168.10.0/24`) para o BGP do provedor através da sua AS privada (`65010`).

---

## 2. Tabela de Interfaces e Endereçamento

| Interface | Descrição | Endereço IPv4 | Endereço IPv6 | Função / Protocolo | Estado |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FastEthernet0/0** | Link para R1 | `192.168.100.2/30` | N/A | Uplink WAN / eBGP Peering | Ativa (Half-Duplex) |
| **FastEthernet1/0** | Link para PC-HQ | `192.168.10.1/24` | N/A | Gateway LAN HQ | Ativa |
| **FastEthernet1/1** | N/A | Sem endereço | N/A | Nenhum | Desativada (`shutdown`) |
| **FastEthernet2/0** | N/A | Sem endereço | N/A | Nenhum | Desativada (`shutdown`) |
| **FastEthernet2/1** | N/A | Sem endereço | N/A | Nenhum | Desativada (`shutdown`) |

---

## 3. Configuração BGP (AS 65010)

- **Processo BGP:** `65010` (AS Privado)
- **Router ID:** `192.168.100.2`

### 3.1. Peering Exterior (eBGP) com R1 (AS 2121)

| Vizinho | Endereço IP | AS Remoto | Família | Estado / Ativação |
| :--- | :--- | :--- | :--- | :--- |
| **R1 (IPv4)** | `192.168.100.1` | 2121 | IPv4 Unicast | Ativo |

### 3.2. Redes Anunciadas pelo AS 65010
O roteador injeta o prefixo da sua LAN local na tabela BGP:
- **IPv4 Network:** `192.168.10.0` (Class C / `/24`)

---

## 4. Serviços de Infraestrutura e Gestão

- **Cisco Express Forwarding (CEF):** Ativado para IPv4 (`ip cef`) e IPv6 (`ipv6 cef`).
- **Roteamento Unicast IPv6:** Ativado a nível global (`ipv6 unicast-routing`).
- **Resolução de Nomes:** Desativada (`no ip domain lookup`).
- **Serviços Web:** Servidores HTTP e HTTPS desativados (`no ip http server`, `no ip http secure-server`).
- **Linhas de Gestão e Acesso:**
  - `line con 0`, `line aux 0`, `line vty 0 4`: Configurados com `exec-timeout 0 0`, privilégio nível 15 e `logging synchronous`.

---

## 5. Configuração Original (Cisco IOS)

```cisco
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
redundancy
!
!
ip tcp synwait-time 5
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
