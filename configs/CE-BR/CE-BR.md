# Documentação de Configuração - Roteador CE-BR

**Dispositivo:** CE-BR  
**Versão do IOS:** Cisco IOS 15.2  
**Data da Última Alteração:** 16 de Setembro de 2026, 01:56:36 UTC  
**Função na Rede:** Customer Edge (CE)  
**Protocolo de Roteamento:** RIPv2  

---

## 1. Resumo Executivo e Arquitetura

O roteador **CE-BR** atua como equipamento **Customer Edge (CE)** para a infraestrutura do site/filial "BR". As suas principais funções incluem:

1. **Gateway da Rede Local (LAN):** Provê o gateway padrão (`192.168.20.1/24`) para os dispositivos locais da rede BR (ex.: `PC-BR`).
2. **Conectividade de Borda (WAN):** Interconecta a rede do cliente ao roteador de borda do provedor (**R2**) através de um enlace ponto-a-ponto de rede `192.168.200.0/30`.
3. **Roteamento Dinâmico (RIPv2):** Troca informações de rotas dinamicamente com o roteador R2 utilizando o protocolo **RIP Versão 2**.

---

## 2. Tabela de Interfaces e Endereçamento

| Interface | Descrição | Endereço IPv4 | Endereço IPv6 | Função / Protocolo | Estado |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FastEthernet0/0** | Link para R2 | `192.168.200.2/30` | N/A | Uplink WAN / Peering RIPv2 | Ativa (Half-Duplex) |
| **FastEthernet1/0** | Link para PC-BR | `192.168.20.1/24` | N/A | Gateway LAN BR | Ativa |
| **FastEthernet1/1** | N/A | Sem endereço | N/A | Nenhum | Desativada (`shutdown`) |
| **FastEthernet2/0** | N/A | Sem endereço | N/A | Nenhum | Desativada (`shutdown`) |
| **FastEthernet2/1** | N/A | Sem endereço | N/A | Nenhum | Desativada (`shutdown`) |

---

## 3. Configuração de Roteamento (RIPv2)

- **Protocolo:** RIP (Routing Information Protocol)
- **Versão:** 2

### 3.1. Redes Anunciadas / Ativas no RIP

| Rede Declarada | Máscara / Classe | Finalidade |
| :--- | :--- | :--- |
| **192.168.20.0** | `255.255.255.0` (`/24`) | Anúncio do segmento da LAN do site BR |
| **192.168.200.0** | `255.255.255.252` (`/30`) | Troca de rotas RIP com o roteador R2 no enlace WAN |

---

## 4. Serviços de Infraestrutura e Gestão

- **Cisco Express Forwarding (CEF):** Ativado para IPv4 (`ip cef`) e IPv6 (`ipv6 cef`).
- **Roteamento Unicast IPv6:** Ativado a nível global (`ipv6 unicast-routing`).
- **Resolução de Nomes:** Desativada (`no ip domain lookup`).
- **Serviços Web:** Servidores HTTP e HTTPS desativados (`no ip http server`, `no ip http secure-server`).
- **Acesso e Linhas de Gestão:**
  - `line con 0`, `line aux 0`, `line vty 0 4`: Configurados com `exec-timeout 0 0`, privilégio nível 15 e `logging synchronous`.

---

## 5. Configuração Original (Cisco IOS)

```cisco
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
redundancy
!
!
ip tcp synwait-time 5
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
