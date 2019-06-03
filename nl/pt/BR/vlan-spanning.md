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

# Vinculando redes privadas seguras na rede IBM
{: #vlan-spanning}

À medida que necessidade de alcance global e de operações 24-7 do aplicativo da web aumenta, a necessidade de hospedar serviços em múltiplos data centers em nuvem aumenta. Os data centers em múltiplos locais fornecem resiliência no caso de uma falha geográfica e também trazem cargas de trabalho mais próximas de usuários globalmente distribuídos, reduzindo a latência e aumentando o desempenho percebido. A [rede do {{site.data.keyword.Bluemix_notm}}](https://www.ibm.com/cloud/data-centers/) permite que os usuários vinculem as cargas de trabalho hospedadas em redes privadas seguras entre data centers e locais.

Este tutorial apresenta a configuração de uma conexão de IP roteada privadamente na rede privada do {{site.data.keyword.Bluemix_notm}} entre duas redes privadas seguras hospedadas em diferentes data centers. Todos os recursos são de propriedade de uma conta do {{site.data.keyword.Bluemix_notm}}. Ele usa o tutorial [Isolar cargas de trabalho com uma rede privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) para implementar duas redes privadas que são vinculadas de forma segura na rede privada do {{site.data.keyword.Bluemix_notm}} usando o serviço [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning).
{:shortdesc}

## Objetivos
{: #objectives}

- Vincular redes seguras dentro de uma conta do {{site.data.keyword.Bluemix_notm}} IaaS
- Configurar regras de firewall para acesso de site para site 
- Configurar roteamento entre sites

## Serviços usados
{: #products}

Este tutorial usa os serviços do {{site.data.keyword.Bluemix_notm}} a seguir: 
* [Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [Ampliação de VLAN]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)

Este tutorial pode incorrer em custos. O VRA está disponível apenas em um plano de precificação mensal.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

  ![Arquitetura](images/solution43-vlan-spanning/vlan-spanning.png)
</p>


1. Implementar redes privadas seguras
2. Configurar VLAN Spanning
3. Configurar roteamento de IP entre redes privadas
4. Configurar regras de firewall para acesso ao site remoto

## Antes de Começar
{: #prereqs}

Este tutorial é baseado no tutorial, [Isolar cargas de trabalho com uma rede privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). Esse tutorial e seus pré-requisitos devem ser revisados antes de começar. 

## Configurar sites de rede privada segura
{: #private_network}

O tutorial [Isolar cargas de trabalho com uma rede privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) é utilizado duas vezes para implementar redes privadas em dois data centers diferentes. Não há nenhuma restrição em que dois data centers podem ser utilizados, exceto observar o impacto da latência em qualquer tráfego ou cargas de trabalho que se comunicarão entre os sites. 

O tutorial [Isolar cargas de trabalho com uma rede privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) pode ser seguido sem mudança para cada data center selecionado, registrando as informações a seguir para etapas posteriores. 

| Item  | Data center 1 | Data center 2 |
|:------ |:--- | :--- |
| Datacenter |  |  |
| Endereço IP público do VRA | <DC1 VRA Public IP Address> | <DC2 VRA Public IP Address> |
| Endereço IP privado do VRA | <DC1 VRA Private IP Address> | <DC2 VRA Private IP Address> |
| Sub-rede privada do VRA e CIDR |  |  |
| ID da VLAN privada | &lt;DC1 Private VLAN ID&gt;  | &lt;DC2 Private VLAN ID&gt; |
| Endereço IP privado da VSI | <DC1 VSI Private IP Address> | <DC2 VSI Private IP Address> |
| Sub-rede da zona APP e CIDR | <DC1 APP zone subnet/CIDR> | <DC2 APP zone subnet/CIDR> |

1. Continue com a página Detalhes do gateway para cada VRA por meio da página [Dispositivos de gateway](https://{DomainName}/classic/network/gatewayappliances).  
2. Localize a seção VLANs do gateway e clique na [VLAN]( https://{DomainName}/classic/network/vlans) do gateway na rede **Privada** para visualizar os detalhes da VLAN. O nome deve conter o ID, `bcrxxx`, que representa 'roteador de cliente de back-end' e estar no formato `nnnxx.bcrxxx.xxxx`.
3. O VRA provisionado será visto sob a seção **Dispositivos*. Na seção **Sub-redes**, anote o endereço IP de sub-rede privada do VRA e o CIDR (/26). A sub-rede será do tipo primário com 64 IPs. Esses detalhes serão necessários posteriormente para a configuração de roteamento. 
4. Mais uma vez na página Detalhes do gateway, localize a seção **VLANs associadas** e clique na [VLAN]( https://{DomainName}/classic/network/vlans) na rede **Privada** que foi associada para criar a rede segura e a zona APP. 
5. A VSI provisionada será vista sob a seção **Dispositivos*. Na seção **Sub-redes**, anote o endereço IP de sub-rede da VSI e o CIDR (/26), pois eles serão necessários para a configuração de roteamento. Essa VLAN e sub-rede são identificadas como a zona APP nas configurações de firewall do VRA e são registradas como a &lt;APP Zone subnet/CIDR&gt;.


## Configurar VLAN Spanning 
{: #configure-vlan-spanning}

Por padrão, os servidores (e VRAs) em VLANs e data centers diferentes não podem se comunicar entre si na rede privada. Nestes tutoriais, dentro de um único data center, os VRAs são usados para vincular VLANs e sub-redes com roteamento de IP clássico e firewalls para criar uma rede privada para comunicação do servidor em VLANs. Embora eles possam se comunicar no mesmo data center, nesta configuração, os servidores pertencentes à mesma conta do {{site.data.keyword.Bluemix_notm}} não podem se comunicar entre os data centers. 

O serviço [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning) eleva essa restrição de comunicação entre as VLANs e sub-redes que **NÃO** estão associadas a VRAs. Deve ser observado que mesmo quando o VLAN Spanning está ativado, as VLANs associadas a VRAs podem se comunicar somente por meio de seu VRA associado, conforme determinado pela configuração de firewall e roteamento do VRA. Quando o VLAN Spanning é ativado, os VRAs pertencentes a uma conta do {{site.data.keyword.Bluemix_notm}} são conectados na rede privada e podem se comunicar. 

As VLANs não associadas às redes privadas seguras criadas pelos VRAs são 'abrangidas', permitindo a interconexão dessas VLANs 'não associadas' nos data centers. Isso inclui as VLANs do Gateway do VRA (trânsito) pertencentes à mesma conta do IBM Cloud em diferentes data centers. Permitindo, consequentemente, que os VRAs se comuniquem entre os data centers quando o VLAN Spanning está ativado. Com a conectividade de VRA para VRA, a configuração de firewall e de roteamento do VRA permite que os servidores dentro das redes seguras se conectem. 

Ative o VLAN Spanning:

1. Continue com a página [VLANs]( https://{DomainName}/classic/network/vlans).
2. Selecione a guia **Abranger** na parte superior da página
3. Selecione o botão de opções ‘Ativar’ do VLAN Spanning. Levará alguns minutos para que a mudança de rede seja concluída.
4. Confirme se os dois VRAs podem agora se comunicar:

   Efetue login no VRA do data center 1 e execute ping do VRA do data center 2

   ```
   SSH vyatta@<DC1 VRA Private IP Address>
   ping <DC2 VRA Private IP Address>
   ```
   {: codeblock}

   Efetue login no VRA do data center 2 e execute ping do VRA do data center 1
   ```
   SSH vyatta@<DC2 VRA Private IP Address>
   ping <DC1 VRA Private IP Address>
   ```
   {: codeblock}

## Configurar o roteamento de IP do VRA 
{: #vra_routing}

Crie o roteamento do VRA em cada data center para ativar as VSIs nas zonas APP em ambos os data centers para se comunicar. 

1. Crie a rota estática no data center 1 para a sub-rede privada da zona APP no data center 2, no modo de edição do VRA.
   ```
   ssh vyatta@<DC1 VRA Private IP Address>
   conf
   set protocols static route <DC2 APP zone subnet/CIDR>  next-hop <DC2 VRA Private IP>
   commit
   ```
   {: codeblock}
2. Crie a rota estática no data center 2 para a sub-rede privada da zona APP no data center 1, no modo de edição do VRA.
   ```
   ssh vyatta@<DC2 VRA Private IP Address>
   conf
   set protocols static route <DC1 APP zone subnet/CIDR>  next-hop <DC1 VRA Private IP>
   commit
   ```
   {: codeblock}
2. Revise a tabela de roteamento do VRA na linha de comandos do VRA. Neste momento, as VSIs não podem se comunicar, pois não existem regras de firewall de zona APP para permitir o tráfego entre as duas sub-redes de Zona APP. As regras de firewall são necessárias para o tráfego iniciado em qualquer dos lados.
   ```
   show ip route
   ```
   {: codeblock}

A nova rota para permitir que a zona APP se comunique por meio da rede privada IBM será agora vista. 

## Configuração de firewall do VRA
{: #vra_firewall}

As regras de firewall da zona APP existentes são configuradas somente para permitir o tráfego para e dessa sub-rede para os serviços do {{site.data.keyword.Bluemix_notm}} na rede privada do {{site.data.keyword.Bluemix_notm}} e para acesso público à Internet por meio de NAT. Outras sub-redes associadas a VSIs nesse VRA, ou em outros data centers, são bloqueadas. A próxima etapa é atualizar o grupo de recursos `ibmprivate` associado à regra de firewall APP-TO-INSIDE para permitir acesso explícito à sub-rede no outro data center. 

1. No modo de comando de edição do VRA do data center 1, inclua o <DC2 APP zone subnet>/CIDR no grupo de recursos `ibmprivate`
   ```
   set resources group address-group ibmprivate address <DC2 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
2. No modo de comando de edição do VRA do data center 2, inclua o <DC1 APP zone subnet>/CIDR no grupo de recursos `ibmprivate`
   ```
   set resources group address-group ibmprivate address <DC1 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
3. Verifique se as VSIs em ambos os data centers podem agora se comunicar
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}

   Se as VSIs não puderem se comunicar, siga as instruções no tutorial [Isolar cargas de trabalho com uma rede privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) para monitorar o tráfego nas interfaces e revisar os logs de firewall. 

## Remover recursos
{: #removeresources}

Etapas a serem executadas para remover os recursos criados neste tutorial. 

O VRA está em um plano pago mensal. O cancelamento não resulta em um reembolso. Sugere-se cancelar apenas se esse VRA não for necessário novamente no próximo mês. Se um cluster de Alta Disponibilidade dual VRA for necessário, esse VRA único poderá ser atualizado na página [Detalhes do gateway](https://{DomainName}/classic/network/gatewayappliances).
{:tip}

1. Cancelar quaisquer servidores virtuais ou servidores bare-metal
2. Cancelar o VRA
3. Cancele quaisquer VLANs adicionais por chamado de suporte. 


## Ampliar o tutorial

Este tutorial pode ser usado em conjunto com o
tutorial [VPN em uma rede privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#vpn-into-a-secure-private-network) para vincular ambas as redes seguras a uma rede remota de usuários em uma VPN IPSec. Os links de VPN podem ser estabelecidos para ambas as redes seguras para maior resiliência de acesso à plataforma {{site.data.keyword.Bluemix_notm}} IaaS. Observe que a IBM não permite o roteamento do tráfego do usuário entre os data centers de cliente na rede privada IBM. A configuração de roteamento para evitar loops de rede está além do escopo deste tutorial. 


## Material relacionado
{:related}

1. O Virtual Routing and Forwarding (VRF) é uma alternativa para o uso do VLAN Spanning para conectar redes em uma Conta do {{site.data.keyword.Bluemix_notm}}. O VRF é obrigatório para todos os clientes usando [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview). [Visão geral do Virtual Routing and Forwarding (VRF) no IBM Cloud](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)
2. [A rede IBM Cloud](https://www.ibm.com/cloud/data-centers/)
