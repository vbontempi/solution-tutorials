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

# Entrega de aplicativo da web por meio de uma rede privada segura
{: #web-app-private-network}

A hospedagem de aplicativos da web é um padrão de implementação comum para nuvem pública, em que os recursos podem ser escalados on demand para atender às demandas de uso de longo e curto prazo. A segurança para as cargas de trabalho do aplicativo é um pré-requisito fundamental para complementar a resiliência e a escalabilidade oferecidas pela nuvem pública. 

Este tutorial o conduz você na criação de um aplicativo da web escalável e seguro, voltado para a Internet, hospedado na rede privada protegida usando um virtual router appliance (VRA), VLANs, NAT e firewalls. O aplicativo inclui um balanceador de carga, dois servidores de aplicativos da web e um servidor de banco de dados MySQL. Ele combina três tutoriais para ilustrar como os aplicativos da web podem ser implementados de forma segura na plataforma {{site.data.keyword.Bluemix_notm}} IaaS usando a rede clássica.
{:shortdesc}

## Objetivos
{: #objectives}

- Criar Virtual Servers para instalar o PHP e o MySQL
- Provisionar um Load Balancer para distribuir solicitações para os servidores de aplicativos
- Implementar um Virtual Router Appliance (VRA) para criar uma rede segura
- Definir VLANs e sub-redes de IP 
- Proteger a rede com regras de firewall
- Conversão de Endereço de Rede de Origem (SNAT) para implementação do aplicativo

## Serviços usados
{: #products}

Este tutorial usa os serviços do {{site.data.keyword.Bluemix_notm}} a seguir: 

* [VPN de dispositivo de roteador virtual](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)
* [Load Balancer]( https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)

Este tutorial pode incorrer em custos. O VRA está disponível apenas em um plano de precificação mensal.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

  ![Arquitetura](images/solution42-web-app-private-network/web-app-private.png)
</p>

1.	Configurar a rede privada segura
2.	Configurar o acesso NAT para implementação do aplicativo
3.	Implementar app da web escalável e balanceador de carga

## Antes de Começar
{: #prereqs}

Este tutorial utiliza três tutoriais existentes, que são implementados em sequência. Todos os três devem ser revistos antes de começar:

-	[Isolar cargas de trabalho com uma rede privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 
-	[Configurar NAT para acesso da Internet por uma rede segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)
-	[Usar Servidores virtuais para construir app da web altamente disponível e escalável]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)



## Configurar a rede privada segura
{: #private_network}

Ambientes de rede privada isolados e seguros são centrais para o modelo de segurança do aplicativo IaaS na nuvem pública. Firewalls, VLANs, roteamento e VPNs são todos os componentes necessários na criação de ambientes privados isolados.
A primeira etapa é criar o gabinete de rede privada segura dentro do qual o app da web será implementado.  

- [Isolar cargas de trabalho com uma rede privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)

Este tutorial pode ser seguido sem mudança. Em uma etapa posterior, três máquinas virtuais serão implementadas na zona de APP como servidores da web Nginx e um banco de dados MySQL. 

## Configurar o NAT para implementação do aplicativo seguro
{: #nat_config}

A instalação de aplicativos de software livre requer acesso seguro à Internet para acessar os repositórios de origem. Para proteger os servidores na rede privada segura de serem expostos na Internet pública, o NAT de Origem é usado, no qual o endereço de origem é ofuscado e as regras de firewall são usadas para proteger as solicitações de repositório do aplicativo de saída. Todas as solicitações de entrada são negadas. 

- [Configurar NAT para acesso da Internet por uma rede segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)

Este tutorial pode ser seguido sem mudança. Na próxima etapa, a configuração NAT será usada para acessar os módulos Nginx e MySQL necessários.  


## Implementar app da web escalável e balanceador de carga
{: #scalable_app}

Uma instalação do Wordpress no Nginx e MySQL, com um Load Balancer, é usada para ilustrar como um aplicativo da web escalável e resiliente pode ser implementado na rede privada segura 

Este tutorial conduzirá você nesse cenário com a criação de um balanceador de carga do {{site.data.keyword.Bluemix_notm}}, dois servidores de aplicativos da web e um servidor de banco de dados MySQL. Os servidores são implementados na zona APP na rede privada segura para fornecer a separação de firewall de outras cargas de trabalho e a rede pública. 

- [Usar Servidores virtuais para construir app da web altamente disponível e escalável]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)

Há três mudanças neste tutorial:

1.	Os servidores virtuais usados neste tutorial são implementados na VLAN e na sub-rede protegida pela zona de firewall do APP atrás do VRA.
2. Especifique &lt;Private VLAN ID&gt; ao pedir os servidores virtuais. Consulte as instruções [Pedir o primeiro servidor virtual](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#order_virtualserver) no tutorial [Isolar cargas de trabalho com uma rede privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) para obter detalhes sobre como especificar o &lt;Private VLAN ID&gt; ao pedir um servidor virtual. Além disso, lembre-se de selecionar sua chave SSH transferida por upload anteriormente no tutorial para permitir acesso aos servidores virtuais. 
3. É altamente recomendado **NÃO** usar o serviço de armazenamento de arquivo para este tutorial devido ao desempenho fraco de rsync ao copiar os arquivos Wordpress para o armazenamento compartilhado. Isso não afeta o tutorial geral. As etapas relacionadas à criação do armazenamento de arquivo e à configuração de montagens podem ser ignoradas para os servidores de aplicativos e o db. Como alternativa, todas as etapas de [Instalar e configurar o aplicativo PHP nos servidores de aplicativos](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application) precisam ser executadas em ambos os servidores de aplicativos.
   Antes de concluir as etapas em [Instalar e configurar o aplicativo PHP nos servidores de aplicativos](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application), primeiro crie o diretório `/mnt/www/` em ambos os servidores de aplicativos. Esse diretório era criado originalmente na seção de armazenamento de arquivo agora removida. 

   ```sh
   mkdir /mnt/www
   ```

No final desta etapa, o balanceador de carga deve estar em um estado funcional e o site do Wordpress acessível na Internet. Os servidores virtuais que constituem o aplicativo da web são protegidos contra acesso externo por meio da Internet pelo firewall do VRA e o único acesso é por meio do balanceador de carga. Para um ambiente de produção, a proteção DDoS e o Web Application Firewall (WAF) também devem ser considerados quando fornecidos pelo [{{site.data.keyword.Bluemix_notm}} Internet Services](https://{DomainName}/catalog/services/internet-services).


## Remover recursos
{: #removeresources}

Etapas a serem executadas para remover os recursos criados neste tutorial. 

O VRA está em um plano pago mensal. O cancelamento não resulta em um reembolso. Sugere-se cancelar apenas se esse VRA não for necessário novamente no próximo mês. 
{:tip}  

1. Cancelar quaisquer servidores virtuais ou servidores bare-metal
2. Cancelar o VRA
3. Cancele quaisquer VLANs adicionais por chamado de suporte.
4. Excluir o balanceador de carga
5. Excluir os serviços File Storage

## Expandir o tutorial 

1. Neste tutorial, somente dois servidores virtuais são provisionados inicialmente como a camada de app, mais servidores podem ser incluídos automaticamente para manipular a carga adicional. A [Escala automática]( https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-getting-started-with-auto-scaling#create-an-autoscale-group) fornece a capacidade de automatizar o processo de ajuste de escala manual associado à inclusão ou remoção de servidores virtuais para suportar seus aplicativos de negócios.

2. Proteja separadamente os dados do usuário incluindo uma segunda VLAN privada e um IP de sub-rede no VRA para criar uma zona DATA para hospedar o servidor de banco de dados MySQL. Configure regras de firewall para permitir somente o tráfego de IP do MySQL na porta 3306 de entrada da zona APP para a zona DATA. 

