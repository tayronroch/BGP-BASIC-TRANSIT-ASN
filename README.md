# BGP-BASIC-TRANSIT-ASN


# Huawei Enterprise Router Lab - Políticas de Roteamento Diferenciadas

Este repositório contém a configuração de um laboratório utilizando roteadores Huawei Enterprise para demonstrar a implementação de políticas de roteamento diferenciadas. O foco é aplicar duas abordagens para o cliente com ASN **65300**:
- **Rota Default Simples:** Entrega exclusiva da rota default (0.0.0.0/0).
- **Rota Parcial:** Anúncio da rota default combinado com o bloco anunciado pelo ASN **65200**.

## Sumário

- [Introdução](#introdução)
- [Objetivos](#objetivos)
- [Arquitetura e Topologia](#arquitetura-e-topologia)
- [Configuração](#configuração)
- [Como Utilizar](#como-utilizar)
- [Contribuições](#contribuições)
- [Licença](#licença)

## Introdução

Neste laboratório, simulamos um ambiente de rede real utilizando roteadores Huawei Enterprise. A configuração demonstra como aplicar políticas de roteamento específicas para atender às necessidades de um cliente, utilizando técnicas de BGP, OSPF/OSPFv3 e route-policies, garantindo o controle refinado do tráfego e a segurança da interconexão.

## Objetivos

- **Segmentação de Políticas de Roteamento:**  
  Implementar dois cenários de anúncio de rotas:
  - **Rota Default Simples:** Exclusivamente 0.0.0.0/0.
  - **Rota Parcial:** Combinação de rota default com o bloco anunciado pelo ASN 65200.
  
- **Validação de Cenários Operacionais:**  
  Simular ambientes com múltiplas VLANs, interfaces e sessões BGP para replicar cenários de operadora e grandes empresas.

## Arquitetura e Topologia

- **Gerenciamento de VLANs:**  
  - VLAN 300: Identifica o PTP do cliente ASN65300.
  - VLANs 4001 e 4002: Utilizadas para interconexão entre roteadores.

- **Protocolos de Roteamento:**  
  - **OSPF/OSPFv3:** Para roteamento interno com o router ID **10.255.255.1**.
  - **BGP:** Para o estabelecimento de peering com ASN 65200 e ASN 65300, com políticas específicas de anúncio de rotas.

- **Interfaces e Conectividade:**  
  - Configuração de interfaces VLAN (Vlanif300, Vlanif4001, Vlanif4002) e interfaces físicas (MEth e GE) em modo trunk para passagem de múltiplas VLANs.

## Configuração

A seguir, um trecho resumido da configuração utilizada:

```plaintext
sysname CORE
#
vlan batch 300 4000 to 4003
#
router id 10.255.255.1
#
vlan 300
 description PTP-ASN65300
#
ospfv3 1
 router-id 10.255.255.1
 import-route direct
 import-route static
 area 0.0.0.0
#
interface Vlanif300
 description PTP-OPERADORA-ASN65200
 ipv6 enable
 ip address 192.168.20.1 255.255.255.252
 ipv6 address 2001:DB8:BEBE:CAFE:BEBE::4/127
#
...
#
bgp 65200
 peer 10.254.254.1 as-number 65200
 peer 10.254.254.1 description EDGE-01
 peer 10.254.254.1 connect-interface LoopBack0
 peer 10.254.254.2 as-number 65200
 peer 10.254.254.2 description EDGE-02
 peer 10.254.254.2 connect-interface LoopBack0
 peer 192.168.20.2 as-number 65300
 peer 192.168.20.2 description OPERADORA-ASN65300
 ...
 ipv4-family unicast
  network 172.16.20.0 255.255.252.0
  peer 192.168.20.2 route-policy PARCIAL-ROUTE-V4-OUT export
  peer 192.168.20.2 default-route-advertise 
#
...
#
route-policy DEFAULT-ROUTE-V4-OUT deny node 10
route-policy PARCIAL-ROUTE-V4-OUT permit node 1
 if-match as-path-filter PARCIAL-ROUTE
route-policy PARCIAL-ROUTE-V4-OUT deny node 2
#
ip as-path-filter DEFAULT-GATEWAY index 10 permit ^$
ip as-path-filter PARCIAL-ROUTE index 10 permit ^$
