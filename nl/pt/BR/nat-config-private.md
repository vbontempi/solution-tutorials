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

# Configurar o NAT para acesso à Internet por meio de uma rede privada
{: #nat-config-private}

No mundo de hoje de aplicativos e serviços de TI baseados na web, poucos aplicativos existem no isolamento. Os desenvolvedores habituaram-se a obter acesso aos serviços na Internet, quer seja código do aplicativo de software livre e atualizações ou serviços de ‘terceiros’ fornecendo funcionalidade de aplicativo por meio de APIs de REST. O mascaramento de Conversão de Endereço de Rede (NAT) é uma abordagem comumente usada para proteger o acesso ao serviço hospedado pela Internet de redes privadas. No mascaramento de NAT, os endereços IP privados são convertidos no endereço IP da interface pública de saída em um relacionamento muitos para um, protegendo o endereço IP privado do acesso público.  

Este tutorial apresenta a configuração do mascaramento de Conversão de Endereço de Rede (NAT) em um Virtual Router Appliance (VRA) para conectar uma sub-rede segura na rede privada do {{site.data.keyword.Bluemix_notm}}. Ela é construída no tutorial [Isolar cargas de trabalho com uma rede privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure), incluindo uma configuração de NAT de Origem (SNAT), em que o endereço de origem é ofuscado e as regras de firewall são usadas para proteger o tráfego de saída. As configurações de NAT mais complexas podem ser localizadas na [documentação suplementar do VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Objetivos
{: #objectives}

-	Configurar a Conversão de Endereço de Rede de Origem (SNAT) em um Virtual Router Appliance (VRA)
-	Configurar regras de firewall para acesso à Internet

## Serviços usados
{: #products}

Este tutorial usa os serviços do {{site.data.keyword.Bluemix_notm}} a seguir: 

* [VPN de dispositivo de roteador virtual](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Este tutorial pode incorrer em custos. O VRA está disponível apenas em um plano de precificação mensal.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

  ![Arquitetura](images/solution35-nat-config-private/vra-nat.png)
</p>

1.	Documente os serviços da Internet necessários.
2.	Configure o NAT.
3.	Crie a zona de firewall da Internet e as regras.

## Antes de Começar
{: #prereqs}

Este tutorial ativa os hosts no gabinete de rede privada segura criado pelo tutorial [Isolar cargas de trabalho com uma rede privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) para acessar serviços da Internet pública. Esse tutorial deve ser concluído primeiro. 

## Documentar serviços da Internet
{: #Document_outbound}

A primeira etapa é identificar os serviços que serão acessados na Internet pública e documentar as portas que devem ser ativadas para o tráfego de saída e de entrada correspondente na Internet. Essa lista de portas será necessária para as regras de firewall em uma etapa posterior. 

Nesse exemplo, somente as portas http e https são ativadas porque elas cobrem uma maioria de requisitos. Os serviços DNS e NTP são fornecidos por meio da rede privada do {{site.data.keyword.Bluemix_notm}}. Se esses e outros serviços, como SMTP (Porta 25) ou MySQL (Porta 3306) forem necessários, regras de firewall adicionais serão necessárias. As duas regras básicas de porta são:

-	Porta 80 (http)
-	Porta 443 (https)

Verifique se o serviço de terceiro suporta a whitelisting de endereços de origem. Se sim, o endereço IP público do VRA será necessário para configurar o serviço de terceiro para limitar o acesso ao serviço. 


## Mascaramento de NAT para Internet 
{: #NAT_Masquerade}

Siga as instruções aqui para configurar o acesso externo à Internet para hosts na zona APP usando mascaramento de NAT. 

1.	Use SSH para VRA e insira o modo \[edit\] (configuração).
   ```bash
   SSH vyatta@<Endereço IP privado VRA> configure
   ```
   {: codeblock}
2.	Crie as regras de SNAT no VRA, especificando o mesmo `<Subnet Gateway IP>/<CIDR>` conforme determinado para a sub-rede/VLAN da zona APP no tutorial de fornecimento do VRA anterior. 
   ```
   set service nat source rule 1000 description 'pass traffic to the internet'
   set service nat source rule 1000 outbound-interface 'dp0bond1'
   set service nat source rule 1000 source address <Subnet Gateway IP>/<CIDR>
   set service nat source rule 1000 translation address masquerade
   commit
   save
   ```
   {: codeblock}

## Criar firewalls
{: #Create_firewalls}

1.	Criar regras de firewall 
   ```
   set security firewall name APP-TO-OUTSIDE default-action drop
   set security firewall name APP-TO-OUTSIDE description 'APP traffic to the Internet'
   set security firewall name APP-TO-OUTSIDE default-log

   set security firewall name APP-TO-OUTSIDE rule 90 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 90 action accept
   set security firewall name APP-TO-OUTSIDE rule 90 destination port 80

   set security firewall name APP-TO-OUTSIDE rule 100 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 100 action accept
   set security firewall name APP-TO-OUTSIDE rule 100 destination port 443

   set security firewall name APP-TO-OUTSIDE rule 200 protocol icmp
   set security firewall name APP-TO-OUTSIDE rule 200 icmp type 8
   set security firewall name APP-TO-OUTSIDE rule 200 action accept
   commit
   ```
   {: codeblock}

## Criar zona e aplicar regras
{: #Create_zone}

1.	Crie a zona OUTSIDE para controlar o acesso à Internet externa.
   ```
   set security zone-policy zone OUTSIDE default-action drop
   set security zone-policy zone OUTSIDE interface dp0bond1
   set security zone-policy zone OUTSIDE description 'External internet'
   ```
   {: codeblock}
2.	Designar firewalls para controlar o tráfego para e da Internet.
   ```
   set security zone-policy zone APP to OUTSIDE firewall APP-TO-OUTSIDE 
   commit
   save
   ```
   {: codeblock}
3.	Valide se a VSI na zona APP pode agora acessar serviços na Internet. Efetue login na VSI local usando SSH, use ping e curl para validar o acesso de icmp e de tcp a sites na Internet.  
   ```bash
   ssh root@<VSI Private IP>
   ping 8.8.8.8
   curl www.google.com
   ```
   {: codeblock}

## Remover recursos
{:removeresources}
Etapas a serem executadas para remover os recursos criados neste tutorial. 

O VRA está em um plano pago mensal. O cancelamento não resulta em um reembolso. Sugere-se cancelar apenas se esse VRA não for necessário novamente no próximo mês. Se um cluster de Alta Disponibilidade dual VRA for necessário, esse VRA único poderá ser atualizado na página [Detalhes do gateway](https://{DomainName}/classic/network/gatewayappliances).
{:tip}  

1. Cancelar quaisquer servidores virtuais ou servidores bare-metal
2. Cancelar o VRA
3. Cancele quaisquer VLANs adicionais por chamado de suporte. 

## Material relacionado
{:related}

-	[Conversão de Endereço de Rede do VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#network-address-translation-nat-) 
-	[Mascaramento de NAT]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-setting-up-nat-rules-on-vyatta-5400#one-to-many-nat-rule-masquerade-)
-	[Documentação suplementar do VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).

