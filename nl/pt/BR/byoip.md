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

# Bring Your Own IP Address
{: #byoip}

Bring Your own IP (BYOIP) é um requisito frequente em que é desejável conectar redes de cliente existentes à infraestrutura provisionada no {{site.data.keyword.Bluemix_notm}}. O intento é normalmente para minimizar a mudança para a configuração de roteamento de rede de clientes e operações com a adoção de um único espaço de endereço IP baseado no esquema de endereçamento IP existente de clientes.

Este tutorial apresenta uma breve visão geral de padrões de implementação BYOIP que podem ser usados com o {{site.data.keyword.Bluemix_notm}} e uma árvore de decisão para identificar o padrão apropriado ao tornar real o gabinete seguro conforme descrito no tutorial [Isolar cargas de trabalho com uma rede privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). A configuração pode requerer entrada adicional de sua equipe de rede no local, suporte técnico do {{site.data.keyword.Bluemix_notm}} ou Serviços IBM.

{:shortdesc}

## Objetivos
{: #objectives}

* Entender os padrões de implementação BYOIP
* Selecionar o padrão de implementação para o {{site.data.keyword.Bluemix_notm}}.

## Endereçamento IP do {{site.data.keyword.Bluemix_notm}}

O {{site.data.keyword.Bluemix_notm}} utiliza uma série de variações de endereços privados, mais especificamente 10.0.0.0/8, e, em alguns casos, esses intervalos podem entrar em conflito com as redes de cliente existentes. Onde existem conflitos de endereço, há uma série de padrões que suportam o BYOIP para permitir a interoperabilidade com a rede do IBM Cloud.

-	Conversão de endereço de rede
-	Tunelamento GRE (Generic Routing Encapsulation)
-	Tunelamento GRE com alias de IP
-	Rede de sobreposição virtual

A opção de padrão é determinada pelos aplicativos planejados para serem hospedados no {{site.data.keyword.Bluemix_notm}}. Há dois aspectos chave, a sensibilidade do aplicativo para a implementação de padrão e a extensão de sobreposição de variações de endereços entre a rede de cliente e o {{site.data.keyword.Bluemix_notm}}. Considerações adicionais também se aplicarão se houver um intento de usar uma conexão de rede privada dedicada para o {{site.data.keyword.Bluemix_notm}}. A documentação para o [{{site.data.keyword.BluDirectLink}}
](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link) é uma leitura recomendada para todos os usuários que consideram o BYOIP. Para usuários do {{site.data.keyword.BluDirectLink}}, a orientação associada deve ser seguida em deferência às informações apresentadas aqui.

## Visão geral dos padrões de implementação
{: #patterns_overview}

1. **NAT**: conversão de endereço NAT no roteador do cliente no local. Execute o NAT no local para converter o esquema de endereçamento do cliente para os endereços IP designados pelo {{site.data.keyword.Bluemix_notm}} para serviços IaaS provisionados.  
2. **Tunelamento GRE**: o esquema de endereçamento é unificado pelo roteamento do tráfego de IP sobre um túnel GRE entre o {{site.data.keyword.Bluemix_notm}} e a rede no local, normalmente por meio de VPN. Esse é o cenário ilustrado [neste tutorial](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN). 

   Há dois subpadrões dependendo do potencial para sobreposição de espaço de endereço.
     * Sem sobreposição de endereço: quando não há nenhuma sobreposição de endereço de variações de endereços e risco de conflito entre as redes.
     * Sobreposição de endereço parcial: quando o cliente e os espaços de endereço IP do {{site.data.keyword.Bluemix_notm}} usam a mesma variação de endereços e há potencial para sobreposição e conflito. Nesse caso, os endereços de sub-rede do cliente são escolhidos, os quais não se sobrepõem na rede privada do {{site.data.keyword.Bluemix_notm}}.

3. Tunelamento GRE + Alias de IP
O esquema de endereçamento é unificado pelo roteamento do tráfego de IP sobre um túnel GRE entre a rede no local e os endereços IP de alias no local designados aos servidores no {{site.data.keyword.Bluemix_notm}}. Esse é um caso especial do cenário ilustrado [neste tutorial](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN). Uma interface adicional e um alias de IP para uma sub-rede IP compatível são criados nos servidores virtuais e bare metal provisionados no {{site.data.keyword.Bluemix_notm}}, suportados pela configuração de roteamento apropriada no VRA.

4. A rede de sobreposição virtual
[Nuvem Particular Virtual (VPC) da {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) suporta BYOIP para ambientes totalmente virtuais no {{site.data.keyword.Bluemix_notm}}. Ela pode ser considerada como uma alternativa para o gabinete de rede privada segura descrito [neste tutorial](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure).

Como alternativa, considere uma solução, como o VMware NSX, que implementa uma rede de sobreposição virtual em uma camada na rede do [{{site.data.keyword.Bluemix_notm}}. Todos os endereços BYOIP na sobreposição virtual são independentes de variações de endereços de rede do [{{site.data.keyword.Bluemix_notm}}. Consulte [Introdução ao VMware e [{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vmware?topic=VMware-getting-started-tutorial#getting-started-with-vmware-and-ibm-cloud).

## Árvore de decisão de padrão
{: #decision_tree}

A árvore de decisão aqui pode ser usada para determinar o padrão de implementação apropriado. 

<p style="text-align: center;">

  ![](images/solution37-byoip/byoipdecision.png)
</p>

As notas a seguir fornecem orientação adicional:

### O NAT é problemático para seus aplicativos?
{: #nat_consideration}

Há dois casos distintos a seguir nos quais o NAT pode ser problemático. Nesses casos, o NAT não deve ser usado. 

- Alguns aplicativos, como a comunicação de domínio do Microsoft AD e os aplicativos P2P, poderiam ter problemas técnicos com o NAT.
- Nos casos em que servidores desconhecidos precisam se comunicar com o [{{site.data.keyword.Bluemix_notm}} ou centenas de conexões bidirecionais entre o [{{site.data.keyword.Bluemix_notm}} e os servidores no local são necessárias. Nesse caso, não é possível configurar todo o mapeamento na tabela do roteador/NAT do cliente porque não é possível identificar antecipadamente o mapeamento.


### Sem sobreposição de endereço
{: #no-overlap}

O 10.0.0.0/8 é usado na rede no local? Quando não existe nenhuma sobreposição de endereço entre a rede no local e a rede privada do [{{site.data.keyword.Bluemix_notm}}, o tunelamento GRE conforme descrito [neste tutorial](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN) pode ser usado entre o IBM Cloud e no local para evitar a necessidade de conversão NAT. Isso requer uma revisão de uso de endereço de rede com a equipe de rede no local. 

### Sobreposição de endereço parcial
{: #partial_overlap}

Se algum do intervalo 10.0.0.0/8 estiver em uso na rede no local, haverá sub-redes sem sobreposição disponíveis na rede do [{{site.data.keyword.Bluemix_notm}}? Revise o uso de endereço de rede existente com a equipe de rede no local e entre em contato com as vendas técnicas do [{{site.data.keyword.Bluemix_notm}} para identificar as redes sem sobreposição disponíveis. 

### O aliasing de IP é problemático?
{: #ip_alias}

Se não existir nenhum endereço com segurança de sobreposição, o aliasing de IP poderá ser implementado em servidores virtuais e bare metal implementados no gabinete de rede privada segura. O aliasing de IP designa múltiplos endereços de sub-rede em uma ou mais interfaces de rede em cada servidor. 

O aliasing de IP é uma técnica comumente usada, embora seja recomendado revisar as configurações de servidor e de aplicativo para determinar se elas funcionam bem em configurações multi-home e de alias de IP.  

A configuração de roteamento adicional no VRA será necessária para criar rotas dinâmicas (por exemplo, BGP) ou rotas estáticas para as sub-redes BYOIP. 

## Conteúdo relacionado
{: #related}

- [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
- [Nuvem Particular Virtual (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure#about-ibm-cloud-virtual-private-cloud-vpc-infrastructure)
