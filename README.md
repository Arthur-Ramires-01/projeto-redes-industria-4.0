# Projeto de Redes — Fábrica Inteligente (Indústria 4.0)

Projeto de Redes de Computadores desenvolvido no **Cisco Packet Tracer**, simulando a infraestrutura de rede de uma fábrica inteligente (Indústria 4.0). A rede integra os setores administrativo, de engenharia, de produção e de estoque, além dos servidores de apoio, mantendo cada ambiente segmentado, com roteamento centralizado, endereçamento automático e proteção da rede industrial.

> **Universidade Cruzeiro do Sul** — Análise e Desenvolvimento de Sistemas (noturno)
> Disciplina: Redes de Computadores · Professor: Genivaldo · **Grupo 4**

---

## Sumário

- [Descrição do problema](#descrição-do-problema)
- [Requisitos e como foram atendidos](#requisitos-e-como-foram-atendidos)
- [Topologia](#topologia)
- [Equipamentos](#equipamentos)
- [Plano de endereçamento (VLSM)](#plano-de-endereçamento-vlsm)
- [Protocolos e serviços](#protocolos-e-serviços)
- [Principais configurações](#principais-configurações)
- [Segurança — ACL de proteção da Produção](#segurança--acl-de-proteção-da-produção)
- [Testes realizados](#testes-realizados)
- [Problemas encontrados e soluções](#problemas-encontrados-e-soluções)
- [Como abrir o projeto](#como-abrir-o-projeto)
- [Autores](#autores)

---

## Descrição do problema

Uma indústria precisa integrar os setores administrativo, de engenharia, de produção e de estoque, além dos dispositivos de automação, mantendo cada ambiente separado. A rede de produção deve ficar protegida contra acessos indevidos dos demais setores, e o tráfego desnecessário entre ambientes deve ser reduzido.

## Requisitos e como foram atendidos

| Requisito | Como foi atendido |
|---|---|
| Redes administrativa, engenharia, produção e estoque | VLANs 10, 20, 30 e 40, cada uma com sua sub-rede |
| Servidor de monitoramento | Servidor Web (HTTP + DNS) na VLAN 50 |
| DHCP | Servidor DHCP na VLAN 50, com um pool por setor e DHCP relay no switch núcleo |
| Comunicação Ethernet e switches distribuídos | Um switch de acesso por setor, ligados a um switch núcleo multilayer |
| Roteamento entre sub-redes | Roteamento inter-VLAN no switch multilayer 3650 (SVIs) e roteador de borda |
| Enlace de fibra óptica | Fibra entre o Switch ESTOQUE e o PC Estoque |
| Redução de tráfego desnecessário | Segmentação em VLANs e restrição das VLANs permitidas nos troncos |
| Proteção da rede industrial | ACL estendida que isola a rede de produção |

## Topologia

Topologia em **estrela**: o switch multilayer **SW-CORE** concentra os quatro switches de acesso, os dois servidores e o roteador de borda. O enlace entre o Switch ESTOQUE e o PC Estoque é feito por **fibra óptica**.

```
   [ Servidor DHCP ]        [ Servidor WEB ]        [ R1-BORDA ]
      (VLAN 50)                (VLAN 50)          (roteador de borda)
          |  Gi1/0/1              |  Gi1/0/8            |  Gi1/0/7 (VLAN 99)
          +----------------------+---------------------+
                                 |
                      +---------------------------+
                      |          SW-CORE          |
                      |   Cisco 3650 multilayer   |
                      |  inter-VLAN / relay / ACL |
                      +---------------------------+
                     /        |          |         \
                trunk/   trunk|     trunk|    trunk \
                   /          |          |            \
            [ SW-ADM ]   [ SW-ENG ]  [ SW-PROD ]   [ SW-EST ]
             VLAN 10      VLAN 20     VLAN 30        VLAN 40
                |            |           |              |
              2 PCs        2 PCs       1 PC       1 PC + 1 PC (fibra)
```

## Equipamentos

| Equipamento | Modelo | Qtd. | Função |
|---|---|---|---|
| SW-CORE | Cisco 3650-24PS (multilayer) | 1 | Núcleo, roteamento inter-VLAN, DHCP relay e ACL |
| R1-BORDA | Router-PT | 1 | Roteador de borda |
| SW-ADM, SW-ENG, SW-PROD, SW-EST | Switch-PT | 4 | Switches de acesso dos setores |
| Servidor DHCP | Server-PT | 1 | Distribuição de endereços IP |
| Servidor WEB | Server-PT | 1 | Monitoramento (HTTP) e DNS |
| Estações | PC-PT | 7 | 2 ADM, 2 ENG, 1 PROD, 2 EST |

### Conexões

| Origem | Destino | Meio | Modo da porta |
|---|---|---|---|
| SW-CORE Gi1/0/1 | Servidor DHCP Fa0 | Cobre direto | Access VLAN 50 |
| SW-CORE Gi1/0/2 | SW-PROD Fa1/1 | Cobre direto | Trunk |
| SW-CORE Gi1/0/4 | SW-ADM Fa0/1 | Cobre direto | Trunk |
| SW-CORE Gi1/0/5 | SW-ENG Fa0/1 | Cobre direto | Trunk |
| SW-CORE Gi1/0/6 | SW-EST Fa0/1 | Cobre direto | Trunk |
| SW-CORE Gi1/0/7 | R1-BORDA Fa0/0 | Cobre direto | Access VLAN 99 |
| SW-CORE Gi1/0/8 | Servidor WEB Fa0 | Cobre direto | Access VLAN 50 |
| SW-EST Gig6/1 | PC Estoque Gig0 | **Fibra óptica** | Access VLAN 40 |
| Switches de acesso | Demais PCs | Cobre direto | Access na VLAN do setor |

## Plano de endereçamento (VLSM)

Cada setor recebeu uma rede própria, com o terceiro octeto igual ao número da VLAN. As máscaras foram dimensionadas pela quantidade de dispositivos prevista em cada setor (VLSM).

| VLAN | Setor | Rede | Máscara | Gateway | Faixa útil | Broadcast | Hosts |
|---|---|---|---|---|---|---|---|
| 10 | Administrativo | 192.168.10.0/26 | 255.255.255.192 | 192.168.10.1 | .1 – .62 | .63 | 62 |
| 20 | Engenharia | 192.168.20.0/27 | 255.255.255.224 | 192.168.20.1 | .1 – .30 | .31 | 30 |
| 30 | Produção | 192.168.30.0/25 | 255.255.255.128 | 192.168.30.1 | .1 – .126 | .127 | 126 |
| 40 | Estoque | 192.168.40.0/28 | 255.255.255.240 | 192.168.40.1 | .1 – .14 | .15 | 14 |
| 50 | Servidores | 192.168.50.0/27 | 255.255.255.224 | 192.168.50.1 | .1 – .30 | .31 | 30 |
| 99 | Trânsito core-borda | 192.168.99.0/30 | 255.255.255.252 | — | .1 – .2 | .3 | 2 |

### Endereços fixos

| Dispositivo | Interface | Endereço |
|---|---|---|
| SW-CORE | SVIs Vlan10 a Vlan50 | 192.168.X.1 (X = número da VLAN) |
| SW-CORE | Vlan99 | 192.168.99.1/30 |
| R1-BORDA | Fa0/0 | 192.168.99.2/30 |
| Servidor DHCP | Fa0 | 192.168.50.10/27 |
| Servidor WEB | Fa0 | 192.168.50.20/27 |

As estações recebem endereço por DHCP, a partir do `.10` de cada rede.

### Cálculo das sub-redes

Número de hosts por sub-rede: `2^(32 - prefixo) - 2`.

| Setor | Prefixo | Cálculo | Justificativa |
|---|---|---|---|
| Produção | /25 | 2⁷ − 2 = 126 | Maior setor: PCs, CLPs, sensores e robôs |
| Administrativo | /26 | 2⁶ − 2 = 62 | Estações de escritório |
| Engenharia | /27 | 2⁵ − 2 = 30 | Equipe técnica menor |
| Servidores | /27 | 2⁵ − 2 = 30 | Servidores com espaço para expansão |
| Estoque | /28 | 2⁴ − 2 = 14 | Poucos terminais |
| Trânsito | /30 | 2² − 2 = 2 | Enlace ponto a ponto, só dois endereços |

## Protocolos e serviços

| Protocolo / serviço | Uso no projeto |
|---|---|
| Ethernet (IEEE 802.3) | Enlaces em cobre e fibra |
| IEEE 802.1Q | Troncos entre o núcleo e os switches de acesso |
| STP | Prevenção de laços na camada 2 |
| ARP | Resolução de IP para MAC dentro de cada VLAN |
| IPv4 e ICMP | Endereçamento e testes (ping, tracert) |
| Roteamento estático | Rota padrão no núcleo e rota de retorno no roteador de borda |
| DHCP (UDP 67/68) | Endereçamento automático, com relay (`ip helper-address`) |
| DNS (UDP 53) | Resolução do nome `fabrica.local` |
| HTTP (TCP 80) | Página do servidor de monitoramento |
| CDP | Identificação dos vizinhos nas portas do núcleo |
| ACL estendida | Controle de acesso à rede de produção |

## Principais configurações

### SW-CORE (Cisco 3650)

```
hostname SW-CORE
enable secret ********
service password-encryption
no ip domain-lookup
ip routing

vlan 10
 name ADMINISTRATIVO
(VLANs 20, 30, 40, 50 e 99 criadas da mesma forma)

interface vlan 10
 ip address 192.168.10.1 255.255.255.192
 ip helper-address 192.168.50.10
interface vlan 20
 ip address 192.168.20.1 255.255.255.224
 ip helper-address 192.168.50.10
interface vlan 30
 ip address 192.168.30.1 255.255.255.128
 ip helper-address 192.168.50.10
 ip access-group PROTEGE-PROD out
interface vlan 40
 ip address 192.168.40.1 255.255.255.240
 ip helper-address 192.168.50.10
interface vlan 50
 ip address 192.168.50.1 255.255.255.224
interface vlan 99
 ip address 192.168.99.1 255.255.255.252

interface range GigabitEthernet1/0/4 - 6
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50
interface GigabitEthernet1/0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50
interface GigabitEthernet1/0/1
 switchport mode access
 switchport access vlan 50
interface GigabitEthernet1/0/8
 switchport mode access
 switchport access vlan 50
interface GigabitEthernet1/0/7
 switchport mode access
 switchport access vlan 99

ip route 0.0.0.0 0.0.0.0 192.168.99.2
```

### Switches de acesso (exemplo: SW-ADM)

```
hostname SW-ADM
vlan 10
 name ADMINISTRATIVO
interface FastEthernet0/1
 switchport mode trunk
interface FastEthernet1/1
 switchport mode access
 switchport access vlan 10
interface FastEthernet2/1
 switchport mode access
 switchport access vlan 10
```

Os demais seguem o mesmo padrão: uplink em trunk e portas de estação em access na VLAN do setor. No **SW-PROD** o uplink é a `Fa1/1`. No **SW-EST**, a porta de fibra `Gig6/1` está em access na VLAN 40.

### R1-BORDA

```
hostname R1-BORDA
interface FastEthernet0/0
 ip address 192.168.99.2 255.255.255.252
 no shutdown
ip route 192.168.0.0 255.255.0.0 192.168.99.1
```

### Servidores

**Servidor DHCP** (`192.168.50.10/27`, gateway `192.168.50.1`): serviço DHCP ativo com um pool por VLAN.

| Pool | Gateway | DNS | IP inicial | Máscara |
|---|---|---|---|---|
| ADM | 192.168.10.1 | 192.168.50.20 | 192.168.10.10 | 255.255.255.192 |
| ENG | 192.168.20.1 | 192.168.50.20 | 192.168.20.10 | 255.255.255.224 |
| PROD | 192.168.30.1 | 192.168.50.20 | 192.168.30.10 | 255.255.255.128 |
| EST | 192.168.40.1 | 192.168.50.20 | 192.168.40.10 | 255.255.255.240 |

**Servidor WEB** (`192.168.50.20/27`, gateway `192.168.50.1`): HTTP ativo com a página de monitoramento e DNS ativo com o registro `fabrica.local → 192.168.50.20`.

### Enlace de fibra óptica

No Switch ESTOQUE foi instalado o módulo `PT-SWITCH-NM-1FGE` (porta `Gig6/1`) e no PC Estoque o módulo `PT-HOST-NM-1FGE` (porta `Gig0`). O terminal do estoque fica próximo de empilhadeiras e motores, e a fibra é imune à interferência eletromagnética gerada por esses equipamentos.

## Segurança — ACL de proteção da Produção

A ACL `PROTEGE-PROD`, aplicada na saída da Vlan30, permite que **apenas a Engenharia e os servidores** iniciem comunicação com a Produção. Administrativo e Estoque são bloqueados, reduzindo o risco de um equipamento de escritório comprometido alcançar os controladores da fábrica.

```
ip access-list extended PROTEGE-PROD
 10 permit ip 192.168.20.0 0.0.0.31 192.168.30.0 0.0.0.127
 20 permit ip 192.168.50.0 0.0.0.31 192.168.30.0 0.0.0.127
 30 permit icmp any 192.168.30.0 0.0.0.127 echo-reply
 40 deny ip any 192.168.30.0 0.0.0.127
 50 permit ip any any
```

**Outras medidas:** senha de console e `enable secret` no núcleo, `service password-encryption`, banner de acesso restrito e desativação da porta sem uso (`Gi1/0/3`).

**Melhorias propostas:** `port-security` nas portas de acesso da produção (limitando os MACs aceitos) e acesso remoto por SSH no lugar de Telnet.

## Testes realizados

- **DHCP:** PCs recebem endereço automaticamente a partir do `.10` de cada rede (ex.: PC Produção `192.168.30.10/25`; PC Estoque via fibra `192.168.40.x/28`).
- **Comunicação entre setores:** ping do Administrativo para a Engenharia e da Engenharia para os servidores (DHCP e WEB), com 0% de perda.
- **Tracert:** do Administrativo até o Servidor WEB, passando pelo gateway `192.168.10.1` (SW-CORE).
- **HTTP e DNS:** acesso à página de monitoramento pelo IP `192.168.50.20` e pelo nome `fabrica.local`.
- **ACL:** Engenharia alcança a Produção (permitido); Administrativo é bloqueado (`Destination host unreachable`); contadores da ACL confirmam pacotes permitidos (regra 10) e bloqueados (regra 40).
- **Verificações:** `show vlan brief`, `show ip interface brief`, `show interfaces trunk`, `show ip route` e `show access-lists` no SW-CORE e no R1-BORDA.

## Problemas encontrados e soluções

| Problema | Causa | Solução |
|---|---|---|
| Comandos de interface rejeitados nos scripts | O 3650 usa portas `GigabitEthernet1/0/x` e os Switch-PT usam `Fa0/1`, `Fa1/1`, `Fa2/1`; o 3650 também não aceita `switchport trunk encapsulation dot1q` | Nomes das portas ajustados com base no `show ip interface brief` de cada equipamento |
| CLI travando em "Translating..." ao digitar errado | Comando inválido interpretado como nome de host | `no ip domain-lookup` |
| PC da Produção com endereço `169.254.x.x` | O uplink do SW-PROD estava na `Fa1/1`, mas o trunk tinha sido configurado na `Fa0/1`, onde estava o PC. O CDP acusou *native VLAN mismatch* | Mapeamento das portas com `show cdp neighbors`; `Fa1/1` em trunk e `Fa0/1` em access VLAN 30 |
| Enlace de fibra não subia entre o núcleo e os demais equipamentos | Módulos de fibra incompatíveis entre as portas de uplink do 3650 e os equipamentos genéricos | Fibra entre Switch ESTOQUE e PC Estoque, com módulos do mesmo tipo (1FGE) nas duas pontas |
| Servidor WEB sem responder, apesar do IP correto | Com máscara `/28`, a VLAN 50 ia só até `192.168.50.15`, e o `.20` ficava fora da rede | VLAN 50 redimensionada para `/27` no SW-CORE, nos servidores e na ACL |

## Como abrir o projeto

1. Instale o **Cisco Packet Tracer** (versão recente — o projeto usa o switch multilayer Cisco 3650).
2. Baixe o arquivo [`industria inteligente 4.0.pkt`](./industria%20inteligente%204.0.pkt) deste repositório.
3. Abra-o no Packet Tracer (menu **File → Open**).
4. Use o modo **Simulation** para visualizar o fluxo dos pacotes e testar ping, tracert, DHCP, HTTP e a ACL.

## Documentação completa

O relatório técnico completo está disponível em [`docs/relatorio.md`](./docs/relatorio.md), com todas as seções, configurações e a descrição dos testes.

> Se quiser disponibilizar também o PDF original, arraste o arquivo para a pasta `docs/` pela interface do GitHub (**Add file → Upload files**).

## Autores

Grupo 4 — Análise e Desenvolvimento de Sistemas (noturno), Universidade Cruzeiro do Sul.

| Integrante | RGM |
|---|---|
| Nicolas Henrique | 42647444 |
| Giovani Ramires Cardoso dos Santos | 46104828 |
| Arthur Ramires Quirino | 46092323 |
| Arthur Oliveira Souza | 42941661 |
