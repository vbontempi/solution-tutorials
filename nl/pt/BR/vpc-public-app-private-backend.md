---
copyright:
  years: 2019
lastupdated: "2019-04-02"
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
{:important: .important}

# Sub-redes privadas e públicas em uma Nuvem Particular Virtual
{: #vpc-public-app-private-backend}

A IBM aceitará um número limitado de clientes para participar de um programa de Acesso Antecipado à VPC no início de abril de 2019, com o uso expandido sendo aberto nos meses seguintes. Se a sua organização desejar obter acesso ao IBM Virtual Private Cloud, preencha este [formulário de nomeação](https://{DomainName}/vpc){: new_window} e um representante IBM entrará em contato com você sobre as próximas etapas.
{: important}

Este tutorial conduz você na criação de seu próprio {{site.data.keyword.vpc_full}} (VPC) com uma sub-rede pública e privada e uma instância de servidor virtual (VSI) em cada sub-rede. Uma VPC é a sua própria nuvem particular na infraestrutura de nuvem compartilhada com isolamento lógico de outras redes virtuais.

Uma [sub-rede](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#subnet) é uma variação de endereços IP. Ela é ligada a uma única zona e não pode abranger múltiplas zonas ou regiões. Para os propósitos de VPC, a característica importante de uma sub-rede é o fato de que as sub-redes podem ser isoladas umas das outras, bem como estar interconectadas da maneira usual. O isolamento de sub-rede pode ser realizado pelas [Listas de Controle de Acesso](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#access-control-list) (ACLs) de rede que agem como firewalls para controlar o fluxo de pacotes de dados entre sub-redes. Da mesma forma, os [Security Groups](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#security-group) (SGs) agem como firewalls virtuais para controlar o fluxo de pacotes de dados para e de VSIs individuais.

A sub-rede pública é usada para recursos que devem ser expostos para o mundo exterior. Os recursos com acesso restrito que nunca devem ser acessados diretamente do mundo exterior são colocados dentro da sub-rede privada. As instâncias em tal sub-rede podem ser seu banco de dados de back-end ou algum armazenamento secreto que você não deseja que seja publicamente acessível. Você definirá os SGs para permitir ou negar tráfego para as VSIs.
{:shortdesc}

Em resumo, usando VPC, é possível

- criar um software-defined network (SDN),
- isolar cargas de trabalho,
- ter bom controle de tráfego de entrada e de saída.

## Objetivos

{: #objectives}

- Entender os objetos de infraestrutura disponíveis para nuvens particulares virtuais
- Aprender como criar uma nuvem particular virtual, sub-redes e instâncias do servidor
- Saber como aplicar grupos de segurança para proteger o acesso aos servidores

## Serviços usados

{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

![Arquitetura](images/solution40-vpc-public-app-private-backend/Architecture.png)


1. O administrador (DevOps) configura a infraestrutura necessária (VPC, sub-redes, grupos de segurança com regras, VSIs) na nuvem.
2. O usuário da Internet faz uma solicitação de HTTP/HTTPS para o servidor da web no front-end.
3. O front-end solicita recursos privados do back-end seguro e entrega resultados para o usuário.

## Antes de Começar

{: #prereqs}

- Verifique as permissões do usuário. Certifique-se de que sua conta do usuário tenha permissões suficientes para criar e gerenciar recursos VPC. Para obter uma lista de permissões necessárias, consulte [Concedendo permissões necessárias para usuários da VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).

- Você precisa de uma chave SSH para se conectar aos servidores virtuais. Se você não tiver uma chave SSH, consulte as [instruções para criar uma chave](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).

## Criar uma Nuvem Particular Virtual
{: #create-vpc}

Para criar sua própria {{site.data.keyword.vpc_short}},

1. Navegue para a página [Visão geral do VPC](https://{DomainName}/vpc/overview) e clique em **Criar um VPC**.
2. Na seção **Nova nuvem particular virtual**:
   * Insira **vpc-pubpriv** como nome para seu VPC.
   * Selecione um **Grupo de recursos**.
   * Opcionalmente, inclua **Tags** para organizar seus recursos.
3. Selecione **Criar novo padrão (Permitir todos)** como sua lista de controle de acesso (ACL) padrão do VPC.
1. Desmarque SSH e execute ping do **Grupo de segurança padrão**.
4. Em **Nova sub-rede para VPC**:
   * Como um nome exclusivo, insira **vpc-pubpriv-backend-subnet**.
   * Selecione uma localização.
   * Insira o intervalo de IP para a sub-rede em notação CIDR, ou seja, **10.xxx.0.0/24**. Deixe o **Prefixo de endereço** como ele é e selecione o **Número de endereços** como 256.
5. Selecione **Usar padrão de VPC** para sua lista de controle de acesso (ACL) de sub-rede. É possível configurar as regras de entrada e de saída posteriormente.
6. Clique em **Criar nuvem particular virtual** para provisionar a instância.

Se as VSIs conectadas à sub-rede privada precisarem de acesso à Internet para carregar o software, alterne o gateway público para **Conectado** porque conectar um gateway público permitirá que todos os recursos conectados se comuniquem com a Internet pública. Depois que as VSIs tiverem todos os softwares necessários, retorne o gateway público para **Removido** para que a sub-rede não possa atingir a Internet pública.
{: important}

Para confirmar a criação da sub-rede, clique em **Sub-redes** na área de janela à esquerda e aguarde até que o status mude para **Disponível**. É possível criar uma nova sub-rede em **Sub-redes**.

## Criar um grupo de segurança e uma VSI de back-end
{: #backend-subnet-vsi}

Nesta seção, você criará um grupo de segurança e uma instância de servidor virtual para o back-end.

### Criar um grupo de segurança de back-end

Por padrão, um grupo de segurança é criado junto com sua VPC permitindo todo o tráfego de SSH (porta TCP 22) e Ping (ICMP tipo 8) para as instâncias conectadas.

Para criar um novo grupo de segurança para o back-end:  
1. Clique em **Grupos de segurança** em **Rede**e, em seguida, **Novo grupo de segurança**.  
2. Insira **vpc-pubpriv-backend-sg** como nome e selecione a VPC que você criou anteriormente.  
3. Clique em **Criar grupo de segurança**.

Posteriormente, você editará o grupo de segurança para incluir as regras de entrada e saída.

### Criar uma instância de servidor virtual de back-end

Para criar uma instância de servidor virtual na sub-rede recém-criada:

1. Clique na sub-rede de back-end em **Sub-redes**.
2. Clique em **Instâncias conectadas**; em seguida, **Nova instância**.
3. Insira um nome exclusivo e escolha **vpc-pubpriv-backend-vsi**. Em seguida, selecione a VPC que você criou anteriormente e o **Local** como antes.
4. Escolha a imagem **Ubuntu Linux**, clique em **Todos os perfis** e, em **Cálculo**, escolha **c-2x4** com 2 vCPUs e 4 GB de RAM.
5. Para **Chaves SSH**, selecione a chave SSH que você criou anteriormente.
6. Em **Interfaces de rede**, clique no ícone **Editar** ao lado de Grupos de segurança
   * Selecione **vpc-pubpriv-backend-subnet** como a sub-rede.
   * Desmarque o grupo de segurança padrão e marque **vpc-pubpriv-backend-sg** como ativo.
   * Clique em **Salvar**.
7. Clique em  ** Criar instância de servidor virtual **.

## Criar uma sub-rede, um grupo de segurança e uma VSI de front-end
{: #frontend-subnet-vsi}

De forma semelhante ao back-end, você criará uma sub-rede de front-end com a instância de servidor virtual e um grupo de segurança.

### Criar uma sub-rede para o front-end

Para criar uma nova sub-rede para o front-end,

1. Clique em **Sub-redes** em **Rede** na área de janela esquerda > **Nova sub-rede**.
   * Insira **vpc-pubpriv-frontend-subnet** como nome, em seguida, selecione a VPC que você criou.
   * Selecione uma localização.
   * Insira o intervalo de IP para a sub-rede em notação CIDR, ou seja, **10.xxx.1.0/24**. Deixe o **Prefixo de endereço** como ele é e selecione o **Número de endereços** como 256.
1. Selecione **Padrão VPC** para a sua lista de controle de acesso (ACL) de sub-rede. É possível configurar as regras de entrada e de saída posteriormente.
1. Considerando que todas as instâncias do servidor virtual na sub-rede terão um IP flutuante anexado, ele não será necessário para ativar um gateway público para a sub-rede. As instâncias de servidor virtual terão conectividade de Internet por meio de seu IP flutuante.
1. Clique em **Criar sub-rede** para provisioná-la.

### Criar um grupo de segurança de front-end

Para criar um novo grupo de segurança para o front-end:
1. Clique em **Grupos de segurança** em Rede; em seguida, **Novo grupo de segurança**.
2. Insira **vpc-pubpriv-frontend-sg** como o nome e selecione a VPC que você criou anteriormente.
3. Clique em **Criar grupo de segurança**.

### Criar uma instância de servidor virtual de front-end

Para criar uma instância de servidor virtual na sub-rede recém-criada:

1. Clique na sub-rede de frontend em **Sub-redes**.
2. Clique em **Instâncias conectadas**; em seguida, **Nova instância**.
3. Insira um nome exclusivo, **vpc-pubpriv-frontend-vsi**, selecione a VPC que foi criada anteriormente, em seguida, o mesmo **Local** como antes.
4. Selecione a imagem **Ubuntu Linux**, clique em **Todos os perfis** e, em **Cálculo**, escolha **c-2x4** com 2vCPUs e 4 GB de RAM
5. Para **Chaves SSH**, selecione a chave SSH que você criou anteriormente.
6. Em **Interfaces de rede**, clique no ícone **Editar** ao lado de Grupos de segurança
   * Selecione **vpc-pubpriv-frontend-sub-rede** como a sub-rede.
   * Desmarque a segurança e o grupo padrão e ative **vpc-pubpriv-frontend-sg**.
   * Clique em **Salvar**.
   * Clique em  ** Criar instância de servidor virtual **.
7. Aguarde até que o status do VSI mude para **Ligado**. Em seguida, selecione a VSI de front-end **vpc-pubpriv-frontend-vsi**, role para **Interfaces de rede** e clique em **Reservar** em **IP flutuante** para associar um endereço IP público à sua VSI de front-end. Salve o Endereço IP associado em uma área de transferência para referência futura.

## Configurar a conectividade entre o front-end e o back-end
{: #setup-connectivity-frontend-backend}

Com todos os servidores em vigor, nesta seção, você configurará a conectividade para permitir operações regulares entre os servidores de front-end e back-end.

### Configure o grupo de segurança de front-end

1. Navegue para **Grupos de segurança** na seção **Rede**, em seguida, clique em **vpc-pubpriv-frontend-sg**.
2. Primeiro, inclua as regras de **entrada** a seguir usando **Incluir regra**. Elas permitem solicitações de HTTP recebidas e Ping (ICMP).

	<table>
   <thead>
      <tr>
         <td><strong>Origem</strong></td>
         <td><strong>Protocolo</strong></td>
         <td><strong>Valor</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Qualquer - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>De: <strong>80</strong> Para <strong>80</strong></td>
      </tr>
      <tr>
         <td>Qualquer - 0.0.0.0/0</td>
         <td>TCP</td>
         <td>De: <strong>443</strong> Para <strong>443</strong></td>
      </tr>
      <tr>
         <td>Qualquer - 0.0.0.0/0</td>
	      <td>ICMP</td>
	      <td>Tipo: <strong>8</strong>, Código: <strong>Deixar vazio</strong></td>
      </tr>
   </tbody>
   </table>

3. Em seguida, inclua estas regras de **saída**.

   <table>
   <thead>
      <tr>
         <td><strong>Destino:</strong></td>
         <td><strong>Protocolo</strong></td>
         <td><strong>Valor</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Tipo: <strong>Grupo de segurança</strong> - Nome: <strong>vpc-pubpriv-backend-sg</strong></td>
         <td>TCP</td>
         <td>Porta do servidor de back-end, consulte a dica</td>
      </tr>
   </tbody>
   </table>

Aqui estão as portas para serviços típicos de back-end. O MySQL está usando a porta 3306, o PostgreSQL a porta 5432. O Db2 é acessado na porta 50000 ou 50001. O Microsoft SQL Server usa, por padrão, a porta 1433. Uma das muitas [listas com uma porta comum está localizada na Wikipédia](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers).
{:tip }

### Configure o grupo de segurança de back-end
De forma semelhante ao front-end, configure o grupo de segurança para o back-end.

1. Navegue para **Grupos de segurança** na seção **Rede**, em seguida, clique em **vpc-pubpriv-backend-sg**.
2. Inclua a regra de **entrada** a seguir usando **Incluir regra**. Ela permite uma conexão com o serviço de back-end.

   <table>
   <thead>
      <tr>
         <td><strong>Origem</strong></td>
         <td><strong>Protocolo</strong></td>
         <td><strong>Valor</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Tipo: <strong>Grupo de segurança</strong> - Nome: <strong>vpc-pubpriv-frontend-sg</strong></td>
         <td>TCP</td>
         <td>Porta do servidor de back-end</td>
      </tr>
   </tbody>
   </table>


## Instalar software e executar tarefas de manutenção
{: #install-software-maintenance-tasks}

Siga as etapas mencionadas em [acessar instâncias remotas de forma segura com um host bastion](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) para manutenção segura dos servidores usando um host bastion que age como um servidor `jump` e um grupo de segurança de manutenção.


## Remover recursos
{: #remove-resources}

1. No console de gerenciamento de VPC, clique em **IPs flutuantes**, em seguida, no endereço IP para suas VSIs, em seguida, no menu ação, selecione **Liberar**. Confirme se você deseja liberar o endereço IP.
2. Em seguida, alterne para **Instâncias de servidor virtual** e **Excluir** suas instâncias. As instâncias serão excluídas e seus status permanecerão em **Excluindo** por um tempo. Certifique-se de atualizar o navegador de tempos em tempos.
3. Quando os VSIs forem desligados, alterne para **Sub-redes**. Se a sub-rede tiver um gateway público conectado, clique no nome da sub-rede. Nos detalhes da sub-rede, desconecte o gateway público. As sub-redes sem o gateway público podem ser excluídas da página de visão geral. Exclua suas sub-redes.
4. Depois que as sub-redes tiverem sido excluídas, alterne para a guia **VPC** e exclua seu VPC.

Ao usar o console, pode ser necessário atualizar seu navegador para ver informações de status atualizadas depois de excluir um recurso.
{:tip}

## Expandir o tutorial
{: #expand-tutorial}

Deseja incluir ou estender este tutorial? Aqui estão algumas ideias:

- Inclua um [balanceador de carga](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-load-balancer) para distribuir o tráfego de entrada entre múltiplas instâncias.
- Crie uma [rede privada virtual](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn) (VPN) para que seu VPC possa se conectar de forma segura a outra rede privada, como uma rede no local ou outra VPC.


## Conteúdo relacionado
{: #related}

- [Glossário do VPC](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)
- [VPC usando a CLI do IBM Cloud](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli#creating-a-vpc-using-the-ibm-cloud-cli)
- [VPC usando as APIs de REST](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis#creating-a-vpc-using-the-rest-apis)
