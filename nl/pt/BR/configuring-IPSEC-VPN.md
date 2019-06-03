---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:java: #java .ph data-hd-programlang='java'}
{:swift: #swift .ph data-hd-programlang='swift'}
{:ios: #ios data-hd-operatingsystem="ios"}
{:android: #android data-hd-operatingsystem="android"}
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# VPN em uma rede privada segura
{: #configuring-IPSEC-VPN}

A necessidade de criar uma conexão privada entre um ambiente de rede remota e servidores na rede privada do {{site.data.keyword.Bluemix_notm}} é um requisito comum. Geralmente, essa conectividade suporta cargas de trabalho híbridas, transferências de dados, cargas de trabalho privadas ou administração de sistemas no {{site.data.keyword.Bluemix_notm}}. Um túnel Rede Privada Virtual (VPN) de site para site é a abordagem usual para proteger a conectividade entre redes. 

O {{site.data.keyword.Bluemix_notm}} fornece uma série de opções para conectividade do data center de site para site, usando uma VPN na Internet pública ou por meio de uma conexão de rede dedicada privada. 

Consulte [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
para obter mais detalhes sobre links de rede segura dedicada para o {{site.data.keyword.Bluemix_notm}}. Uma VPN na Internet pública fornece uma opção de custo inferior, embora sem garantias de largura da banda. 

Há duas opções adequadas de VPN para conectividade na Internet pública para servidores provisionados no {{site.data.keyword.Bluemix_notm}}:

-	[VPN IPSEC]( https://{DomainName}/catalog/infrastructure/ipsec-vpn)
-	[VPN de dispositivo de roteador virtual](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Este tutorial apresenta a configuração de uma VPN IPSec de site para site usando um Virtual Router Appliance (VRA) para conectar uma sub-rede em um data center do cliente a uma sub-rede protegida na rede privada do {{site.data.keyword.Bluemix_notm}}. 

Este exemplo é construído sobre o tutorial [Isolar cargas de trabalho com uma rede privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure). Ele usa uma VPN IPSec
de site para site, túnel GRE e roteamento estático. Configurações mais complexas de VPN que usam o roteamento dinâmico (BGP, etc) e túneis VTI podem ser localizadas na [documentação suplementar do VRA](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Objetivos
{: #objectives}

- Documentar parâmetros de configuração para VPN IPSec
- Configurar a VPN IPSec em um Virtual Router Appliance
- Rotear o tráfego por meio de um túnel GRE

## Serviços usados
{: #products}

Este tutorial usa os serviços do {{site.data.keyword.Bluemix_notm}} a seguir: 

* [VPN de dispositivo de roteador virtual](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Este tutorial pode incorrer em custos. O VRA está disponível apenas em um plano de precificação mensal.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

  ![Arquitetura](images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png)
</p>

1.	Documentar a configuração de VPN
2.	Criar a VPN IPSec em um VRA
3.	Configuração de VPN e túnel do data center
4.	Criar túnel GRE
5.	Criar rota de IP estática
6.	Configurar firewall 

## Antes de Começar
{: #prereqs}

Este tutorial conecta o gabinete privado seguro no tutorial [Isolar cargas de trabalho com uma rede privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) para o seu data center. Esse tutorial deve ser concluído primeiro.

## Documentar a configuração de VPN
{: #Document_VPN}

Configurar um link de site para site da VPN IPSec entre seu data center e o {{site.data.keyword.Bluemix_notm}} requer coordenação com sua equipe de rede no local para determinar muitos dos parâmetros de configuração, o tipo de túnel e as informações de roteamento de IP. Os parâmetros devem estar em conformidade exata para uma conexão VPN operacional. Geralmente, sua equipe de rede no local especificará a configuração para corresponder aos padrões corporativos acordados e fornecerá o endereço IP necessário do gateway VPN do data center e as variações de endereços de sub-rede que podem ser acessadas.

Antes de começar a configuração da VPN, os endereços IP dos gateways VPN e os intervalos de sub-rede de rede IP devem ser determinados e disponibilizados para a configuração de VPN do data center e para o gabinete de rede privada segura no {{site.data.keyword.Bluemix_notm}}. Eles são ilustrados na figura a seguir, em que a zona de APP no gabinete seguro será conectada por meio do túnel IPSec para sistemas na ‘Sub-rede IP do DC’ no data center do cliente.   

<p style="text-align: center;">

  ![](images/solution36-configuring-IPSEC-VPN/vpn-addresses.png)
</p>

Os parâmetros a seguir devem ser acordados e documentados entre o usuário do {{site.data.keyword.Bluemix_notm}} que está configurando a VPN e a equipe de rede para o data center do cliente. Neste exemplo, os endereços IP de túnel Remoto e Local são configurados para 192.168.10.1 e 192.168.10.2. Qualquer sub-rede arbitrária pode ser usada com a concordância da equipe de rede no local. 

| Item  | Descrição |
|:------ |:--- | 
| &lt;ike group name&gt; | Nome fornecido ao grupo IKE para a conexão. |
| &lt;ike encryption&gt; |Padrão de criptografia IKE acordado a ser usado entre o {{site.data.keyword.Bluemix_notm}} e o data center do cliente, geralmente *aes256*. |
| &lt;ike hash&gt; | Hash IKE acordado entre o {{site.data.keyword.Bluemix_notm}} e o data center do cliente, geralmente *sha1*. |
| &lt;ike-lifetime&gt; | Tempo de vida do IKE do data center do cliente, geralmente *3600*. |
| &lt;esp group name&gt; | Nome dado ao grupo ESP para a conexão. |
| &lt;esp encryption&gt; | Padrão de criptografia ESP acordado entre o {{site.data.keyword.Bluemix_notm}} e o data center do cliente, geralmente *aes256*. |
| &lt;esp hash&gt; | Hash ESP acordado entre o {{site.data.keyword.Bluemix_notm}} e o data center do cliente, geralmente *sha1*. |
| &lt;esp-lifetime&gt; | Tempo de vida do ESP do data center do cliente, geralmente *1800*. |
| &lt;DC VPN Public IP&gt;  | Endereço IP público voltado para a Internet do gateway VPN no data center do cliente. | 
| &lt;VRA Public IP&gt; | Endereço IP público do VRA. |
| &lt;Remote tunnel IP\/24&gt; | Endereço IP designado à extremidade remota do túnel IPSec. Par de endereço IP no intervalo que não entra em conflito com a Nuvem de IPs ou o data center do cliente.   |
| &lt;Local tunnel IP\/24&gt; | Endereço IP designado à extremidade local do túnel IPSec. |
| &lt;DC Subnet/CIDR&gt; | Endereço IP da sub-rede a ser acessada no data center do cliente e no CIDR. |
| &lt;App Zone subnet/CIDR&gt; | Endereço IP da rede e o CIDR da sub-rede de Zona APP do tutorial de criação do VRA. | 
| &lt;Shared-Secret&gt; | Chave de criptografia compartilhada a ser usada entre o {{site.data.keyword.Bluemix_notm}} e o data center do cliente. |

## Configurar VPN IPSec em um VRA
{: #Configure_VRA_VPN}

Para criar a VPN no {{site.data.keyword.Bluemix_notm}}, os comandos e todas as variáveis que precisam mudar são destacados abaixo com &lt; &gt;. As mudanças são identificadas linha por linha, para cada linha que precisa ser mudada. Os valores são provenientes da
tabela. 

1. Use SSH no VRA e insira o modo `[edit]`.
   ```bash
   SSH vyatta@<Endereço IP privado VRA> configure
   ```
   {: codeblock}
2. Crie o grupo Troca de Chave da Internet (IKE).
   ```
   set security vpn ipsec ike-group <ike group name> proposal 1
   set security vpn ipsec ike-group <ike group name> proposal 1 encryption <ike encryption>
   set security vpn ipsec ike-group <ike group name> proposal 1 hash <ike hash>
   set security vpn ipsec ike-group <ike group name> proposal 1 dh-group 2
   set security vpn ipsec ike-group <ike group name> lifetime <ike-lifetime>
   ```
   {: codeblock}
3. Crie o grupo Encapsulating Security Payload (ESP)
   ```
   set security vpn ipsec esp-group <esp group name> proposal 1 encryption <esp encryption>
   set security vpn ipsec esp-group <esp group name> proposal 1 hash <esp hash>
   set security vpn ipsec esp-group <esp group name> lifetime <esp-lifetime>
   set security vpn ipsec esp-group <esp group name> mode tunnel
   set security vpn ipsec esp-group <esp group name> pfs enable
   ```
   {: codeblock}
4. Defina a conexão de site para site
   ```
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication mode pre-shared-secret
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication pre-shared-secret <Shared-Secret>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  connection-type initiate
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  ike-group <ike group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  local-address <VRA Public IP>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  default-esp-group <esp group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1 protocol gre
   commit
   save
   ```
   {: codeblock}

## Configurar a VPN e o túnel do data center
{: #Configure_DC_VPN}

1. A equipe de rede no data center do cliente configurará uma conexão de VPN IPSec idêntica com o &lt;DC VPN Public IP&gt; e o &lt;VRA Public IP&gt; trocados, o endereço de túnel local e remoto e também os parâmetros &lt;DC Subnet/CIDR &gt; e &lt;App Zone subnet/CIDR&gt; trocados. Os comandos específicos de configuração no datacenter do cliente dependerão do fornecedor da VPN.
1. Valide se o endereço IP público do gateway VPN do DC está acessível na Internet antes de continuar:
   ```
   ping <DC VPN Public IP>
   ```
   {: codeblock}
2. Quando a configuração de VPN do data center for concluída, o link IPSec deverá aparecer automaticamente. Verifique se o link foi estabelecido e se o status mostra que há um ou mais túneis IPsec ativos. Verifique com o data center se ambas as extremidades da VPN mostram túneis IPsec ativos.
   ```bash
   show vpn ipsec sa
   show vpn ipsec status
   ```
   {: codeblock}
3. Se o link não tiver sido criado, confirme se os endereços local e remoto foram especificados corretamente e outros parâmetros são como esperado usando o comando de depuração:
   ``` bash
   show vpn debug
   ```
   {: codeblock}

A linha `peer-<DC VPN Public IP>-tunnel-1: ESTABLISHED 5 seconds ago, <VRA Public IP>[500].......` deve estar localizada na saída. Se isso não existir ou mostrar 'CONNECTING', haverá um erro na configuração de VPN.  

## Definir túnel GRE 
{: #Define_Tunnel}

1. Crie o túnel GRE no modo de edição do VRA.
   ```
   set interfaces tunnel tun0 address <Local tunnel IP/24>
   set interfaces tunnel tun0 encapsulation gre
   set interfaces tunnel tun0 mtu 1300
   set interfaces tunnel tun0 local-ip <VRA Public IP>
   set interfaces tunnel tun0 remote-ip <DC VPN Public IP>
   commit
   ```
   {: codeblock}
2. Depois que ambas as extremidades do túnel tiverem sido configuradas, ele deverá aparecer automaticamente. Verifique o estado operacional do túnel por meio da linha de comandos do VRA.
   ```
   show interfaces tunnel
   show interfaces tun0
   ```
   {: codeblock}
   O primeiro comando deve mostrar o túnel com Estado e Link como `u/u` (UP/UP). O segundo comando mostra mais detalhes sobre o túnel e que o tráfego é transmitido e recebido.
3. Valide que o tráfego flui ao longo do túnel
   ```bash
   ping <Remote tunnel IP>
   ```
   {: codeblock}
   As contagens de TX e RX em um `show interfaces tunnel tun0` devem ser vistas para incrementar enquanto há tráfego de `ping`.
4. Se o tráfego não estiver fluindo, os comandos `monitor interface` poderão ser usados para observar o tráfego que é visto em cada interface. A interface `tun0` mostra o tráfego interno no túnel. A interface `dp0bond1` mostrará o fluxo de tráfego encapsulado para e do gateway VPN remoto.
   ```
   monitor interface tunnel tun0 traffic
   monitor interface bonding dp0bond1 traffic 
   ```
   {: codeblock}

Se nenhum tráfego de retorno for visto, a equipe de rede do data center precisará monitorar os fluxos de tráfego nas interfaces de VPN e de túnel no site remoto para localizar o problema. 

## Criar rota de IP estática
{: #Define_Routing}

Crie o roteamento do VRA para direcionar o tráfego para a sub-rede remota por meio do túnel.

1. Crie a rota estática no modo de edição do VRA.
   ```
   set protocols static route <DC Subnet/CIDR>  next-hop <Remote tunnel IP>
   ```
   {: codeblock}   
2. Revise a tabela de roteamento do VRA na linha de comandos do VRA. Neste momento, nenhum tráfego cruzará a rota, já que não existem regras de firewall para permitir o tráfego por meio do túnel. As regras de firewall são necessárias para o tráfego iniciado em qualquer dos lados.
   ```bash
   show ip route
   ```
   {: codeblock}

## Configurar firewall
{: #Configure_firewall}

1. Crie grupos de recursos para tráfego de icmp permitido e portas tcp. 
   ```
   set res group icmp-group icmpgrp type 8
   set res group icmp-group icmpgrp type 11
   set res group icmp-group icmpgrp type 3

   set res group port tcpports port 22
   set res group port tcpports port 80
   set res group port tcpports port 443
   commit
   ```
   {: codeblock}
2. Crie regras de firewall para o tráfego para a sub-rede remota no modo de edição do VRA.
   ```
   set security firewall name APP-TO-TUNNEL default-action drop
   set security firewall name APP-TO-TUNNEL default-log

   set security firewall name APP-TO-TUNNEL rule 100 action accept
   set security firewall name APP-TO-TUNNEL rule 100 protocol tcp
   set security firewall name APP-TO-TUNNEL rule 100 destination port tcpports

   set security firewall name APP-TO-TUNNEL rule 200 protocol icmp
   set security firewall name APP-TO-TUNNEL rule 200 icmp group icmpgrp
   set security firewall name APP-TO-TUNNEL rule 200 action accept
   commit
   ```
   {: codeblock}

   ```
   set security firewall name TUNNEL-TO-APP default-action drop
   set security firewall name TUNNEL-TO-APP default-log

   set security firewall name TUNNEL-TO-APP rule 100 action accept
   set security firewall name TUNNEL-TO-APP rule 100 protocol tcp
   set security firewall name TUNNEL-TO-APP rule 100 destination port tcpports

   set security firewall name TUNNEL-TO-APP rule 200 protocol icmp
   set security firewall name TUNNEL-TO-APP rule 200 icmp group icmpgrp
   set security firewall name TUNNEL-TO-APP rule 200 action accept 
   commit
   ```
   {: codeblock}
2. Criar a Zona para túnel e associe firewalls para tráfego iniciado em qualquer uma das zonas.
   ```
   set security zone-policy zone TUNNEL description "GRE Tunnel"
   set security zone-policy zone TUNNEL default-action drop
   set security zone-policy zone TUNNEL interface tun0

   set security zone-policy zone TUNNEL to APP firewall TUNNEL-TO-APP
   set security zone-policy zone APP to TUNNEL firewall APP-TO-TUNNEL
   commit
   save
   ```
   {: codeblock}
3. Para validar que os firewalls e o roteamento em ambas as extremidades estão configurados corretamente e agora estão permitindo o tráfego ICMP e TCP, execute ping do endereço do gateway da sub-rede remota, primeiro na linha de comandos do VRA e, se bem-sucedido, efetuando login no VSI.
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}
4. Se o ping na linha de comandos do VRA falhar, valide que uma resposta de ping é vista em resposta a uma solicitação de ping na interface do túnel.
   ```
   monitor interface tunnel tun0 traffic
   ```
   {: codeblock}
   Nenhuma resposta indica um problema com as regras de firewall ou o roteamento no data center. Se uma resposta for vista na saída do monitor, mas o comando ping atingir o tempo limite, verifique a configuração das regras de firewall do VRA local.
5. Se o ping do VSI falha, isso indica um problema com as regras de firewall do VRA, roteando a configuração do VRA ou VSI. Conclua a etapa anterior para assegurar que uma solicitação seja enviada e uma resposta seja vista no data center. Monitorar o tráfego na VLAN local e inspecionar os logs de firewall ajudará a isolar o problema para roteamento ou o firewall.
   ```
   monitor interfaces bonding dp0bond0.<VLAN ID>
   show log firewall name APP-TO-TUNNEL
   show log firewall name TUNNEL-TO-APP
   ```
   {: codeblock}

Isso conclui a configuração da VPN por meio do gabinete de rede privada segura. Tutoriais adicionais nesta série ilustram como o gabinete pode acessar serviços na Internet pública.

## Remover recursos
{:removeresources}

Etapas a serem executadas para remover os recursos criados neste tutorial.

O VRA está em um plano pago mensal. O cancelamento não resulta em um reembolso. Sugere-se cancelar apenas se esse VRA não for necessário novamente no próximo mês. Se um cluster de Alta Disponibilidade dual VRA for necessário, esse VRA único poderá ser atualizado na página [Detalhes do gateway](https://{DomainName}/classic/network/gatewayappliances).
{:tip}  

1. Cancelar quaisquer servidores virtuais ou servidores bare-metal
2. Cancelar o VRA
3. Cancele quaisquer VLANs adicionais por chamado de suporte. 

## Conteúdo relacionado
{:related}
- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Sub-redes de IP estático e móvel](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-ips#about-subnets-ips)
- [Documentação do Vyatta](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
- [Brocade Vyatta Network OS IPsec Site-to-Site VPN Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-ipsec-vpn.pdf)
- [Brocade Vyatta Network OS Tunnels Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-tunnels.pdf)
