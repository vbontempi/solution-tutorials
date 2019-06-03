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

# Acessar instâncias remotas de forma segura com um host bastion
{: #vpc-secure-management-bastion-server}

A IBM aceitará um número limitado de clientes para participar de um programa de Acesso Antecipado à VPC no início de abril de 2019, com o uso expandido sendo aberto nos meses seguintes. Se a sua organização desejar obter acesso ao IBM Virtual Private Cloud, preencha este [formulário de nomeação](https://{DomainName}/vpc){: new_window} e um representante IBM entrará em contato com você sobre as próximas etapas.
{: important}

Este tutorial conduzirá você na implementação de um host bastion para acessar instâncias remotas de forma segura em uma nuvem particular virtual. O host bastion é uma instância que é provisionada em uma sub-rede pública e pode ser acessada por meio de SSH. Depois de configurado, o host bastion age como um servidor de **salto** permitindo conexão segura com instâncias provisionadas em uma sub-rede privada.

Para reduzir a exposição de servidores dentro da VPC, você criará e usará um host bastion. As tarefas administrativas nos servidores individuais serão executadas usando SSH, com proxy por meio do bastion. O acesso aos servidores e o acesso regular à Internet por meio dos servidores, por exemplo, para a instalação de software, serão permitidos somente com um grupo de segurança de manutenção especial conectado a esses servidores.
{:shortdesc}

## Objetivos
{: #objectives}

* Aprenda como configurar um host bastion e grupos de segurança com regras
* Gerenciar servidores de forma segura por meio do host bastion

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:  

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)  
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

  ![Arquitetura](images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png)

1. Depois de configurar a infraestrutura necessária (sub-redes, grupos de segurança com regras, VSIs) na nuvem, o administrador (DevOps) se conecta (SSH) ao host bastion usando a chave SSH privada.
2. O administrador designa a um grupo de segurança de manutenção as regras de saída adequadas.
3. O administrador se conecta (SSH) de forma segura ao endereço IP privado da instância por meio do host bastion para instalar ou atualizar qualquer software necessário, por exemplo, um servidor da web
4. O usuário da Internet faz uma solicitação de HTTP/HTTPS para o servidor da web.

## Antes de Começar
{: #prereqs}

- Verifique as permissões do usuário. Certifique-se de que sua conta do usuário tenha permissões suficientes para criar e gerenciar recursos VPC. Para obter uma lista de permissões necessárias, consulte [Concedendo permissões necessárias para usuários da VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Você precisa de uma chave SSH para se conectar aos servidores virtuais. Se você não tiver uma chave SSH, consulte as [instruções para criar uma chave](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- O tutorial supõe que você esteja incluindo o host bastion em uma [nuvem particular virtual](https://{DomainName}/vpc/network/vpcs) existente. **Se você não tiver uma nuvem particular virtual em sua conta, crie uma antes de continuar com as próximas etapas.**

## Criar um host bastion
{: #create-bastion-host}

Nesta seção, você criará e configurará um host bastion juntamente com um grupo de segurança em uma sub-rede separada.

### Criar uma sub-rede
{: #create-bastion-subnet}

1. Clique em **Sub-redes** em **Rede** na área de janela esquerda; em seguida, **Nova sub-rede**.  
   * Insira **vpc-secure-bastion-subnet** como o nome, em seguida, selecione a VPC que você criou.  
   * Selecione um local e uma zona.  
   * Insira o intervalo de IP para a sub-rede em notação CIDR, ou seja, **10.xxx.0.0/24**. Deixe o **Prefixo de endereço** como ele é e selecione o **Número de endereços** como 256.
1. Selecione **Padrão VPC** para a sua lista de controle de acesso (ACL) de sub-rede. É possível configurar as regras de entrada e de saída posteriormente.
1. Alterne o **Gateway público** para **Conectado**. 
1. Clique em **Criar sub-rede** para provisioná-la.

### Criar e configurar o grupo de segurança de bastion

Vamos criar um grupo de segurança e configurar regras de entrada para sua VSI de bastion.

1. Navegue para **Grupos de segurança** e clique em **Novo grupo de segurança**. Insira **vpc-secure-bastion-sg** como o nome e selecione seu VPC. 
2. Agora, crie as regras de entrada a seguir clicando em **Incluir regra** na seção de entrada. Elas permitem acesso SSH e Ping (ICMP).
 
	**Regra de entrada:**
	<table>
	   <thead>
	      <tr>
	         <td><strong>Origem</strong></td>
	         <td><strong>Protocolo</strong></td>
	         <td><strong>Valor</strong></td>
	      </tr>
	   <tbody>
	      <tr>
	         <td>Qualquer - 0.0.0.0/0</td>
	         <td>TCP</td>
	         <td>De: <strong>22</strong> Para <strong>22</strong></td>
	      </tr>
         <tr>
            <td>Qualquer - 0.0.0.0/0</td>
	         <td>ICMP</td>
	         <td>Tipo: <strong>8</strong>, Código: <strong>Deixar vazio</strong></td>
         </tr>
	   </tbody>
	</table>

   Para aprimorar ainda mais a segurança, o tráfego de entrada pode ser restrito à rede da empresa ou a uma rede doméstica típica. É possível executar `curl ipecho.net/plain ; echo` para obter o endereço IP externo de sua rede e usá-lo no lugar.
   {:tip }

### Criar uma instância de bastion
Com a sub-rede e o grupo de segurança já em vigor, em seguida, crie a instância de servidor virtual de bastion.

1. Em **Sub-redes** na área de janela esquerda, selecione **vpc-secure-bastion-subnet**.
2. Clique em **Instâncias conectadas** e provisione uma **Nova instância** chamada **vpc-secure-vsi** em sua própria VPC. Selecione Ubuntu Linux como sua imagem e **c-2x4** (2 vCPUs e 4 GB de RAM) como seu perfil.
3. Selecione um **Local** e certifique-se de usar posteriormente o mesmo local novamente.
4. Para criar uma nova **Chave SSH**, clique em **Nova chave**
   * Insira **vpc-ssh-key** como o nome da chave.
   * Deixe a **Região** no estado em que se encontra.
   * Copie o conteúdo de sua chave SSH local existente e cole-o em **Chave pública**.  
   * Clique em **Incluir chave SSH**.
5. Em **Interfaces de rede**, clique no ícone **Editar** ao lado de Grupos de segurança 
   * Certifique-se de que **vpc-secure-subnet** esteja selecionado como a sub-rede.
   * Desmarque o grupo de segurança padrão e marque **vpc-secure-bastion-sg**.
   * Clique em **Salvar**.
6. Clique em  ** Criar instância de servidor virtual **.
7. Depois que a instância estiver ligada, clique em **vpc-secure-bastion-vsi** e **reserve** um IP flutuante.

### Testar seu bastion

Depois que o endereço IP flutuante do bastion estiver ativo, tente se conectar a ele usando **ssh**:

   ```sh
   ssh -i ~/.ssh/<PRIVATE_KEY> root@<BASTION_FLOATING_IP_ADDRESS>
   ```
   {:pre}

## Configurar um grupo de segurança com regras de acesso de manutenção
{: #maintenance-security-group}

Com acesso ao bastion funcionando, continue e crie o grupo de segurança para tarefas de manutenção, como instalar e atualizar o software.

1. Navegue para **Grupos de segurança** e provisione um novo grupo de segurança chamado **vpc-secure-maintenance-sg** com as regras de saída abaixo

   <table>
   <thead>
      <tr>
         <td><strong>Destino:</strong></td>
         <td><strong> Protocolo </strong></td>
         <td><strong>Valor</strong> </td>
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
         <td>Qualquer - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>De: <strong>53</strong> Para <strong>53</strong></td>
      </tr>
      <tr>
         <td>Qualquer - 0.0.0.0/0</td>
         <td>UDP</td>
         <td>De: <strong>53</strong> Para <strong>53</strong></td>
      </tr>
   </tbody>
   </table>

   As solicitações do servidor DNS são endereçadas à porta 53. O DNS usa TCP para a transferência de Zona e UDP para consultas de nome regulares (primárias) ou reversas. As solicitações de HTTP estão na porta 80 e 443.
   {:tip }

2. Em seguida, inclua esta regra de **entrada** que permite acesso SSH por meio do host bastion.

   <table>
	   <thead>
	      <tr>
	         <td><strong>Origem</strong></td>
	         <td><strong> Protocolo </strong></td>
	         <td><strong>Valor</strong> </td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>Tipo: <strong>Grupo de segurança</strong> - Nome: <strong>vpc-secure-bastion-sg</strong></td>
	         <td>TCP</td>
	         <td>De: <strong>22</strong> Para <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>

3. Crie o grupo de segurança.
4. Navegue para **Todos os grupos de segurança para VPC**, em seguida, selecione **vpc-secure-sg**.
5. Finalmente, edite o grupo de segurança e inclua a regra de **saída** a seguir.

   <table>
	   <thead>
	      <tr>
	         <td><strong>Destino:</strong></td>
	         <td><strong> Protocolo </strong></td>
	         <td><strong>Valor</strong> </td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>Tipo: <strong>Grupo de segurança</strong> - Nome: <strong>vpc-secure-maintenance-sg</strong></td>
	         <td>TCP</td>
	         <td>De: <strong>22</strong> Para <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>


## Usar o host bastion para acessar outras instâncias na VPC
{: #bastion-host-access-instances}

Nesta seção, você criará uma sub-rede privada com a instância de servidor virtual e um grupo de segurança. Por padrão, qualquer sub-rede criada em uma VPC é privada.

Se você já tiver instâncias de servidor virtual em seu VPC ao qual deseja se conectar, será possível ignorar as próximas três seções e iniciar a [inclusão de suas instâncias de servidor virtual no grupo de segurança de manutenção](#add-vsi-to-maintenance).

### Criar uma sub-rede
{: #create-private-subnet}

Para criar uma nova sub-rede,

1. Clique em **Sub-redes** em **Rede** na área de janela esquerda; em seguida, **Nova sub-rede**.  
   * Insira **vpc-secure-private-subnet** como o nome, em seguida, selecione a VPC que você criou.  
   * Selecione uma localização.  
   * Insira o intervalo de IP para a sub-rede em notação CIDR, ou seja, **10.xxx.1.0/24**. Deixe o **Prefixo de endereço** como ele é e selecione o **Número de endereços** como 256.
1. Selecione **Padrão VPC** para a sua lista de controle de acesso (ACL) de sub-rede. É possível configurar as regras de entrada e de saída posteriormente.
1. Alterne o **Gateway público** para **Conectado**. 
1. Clique em **Criar sub-rede** para provisioná-la.

### Criar um grupo de segurança

Para criar um novo grupo de segurança:  
1. Clique em **Grupos de segurança** em Rede; em seguida, **Novo grupo de segurança**.  
2. Insira **vpc-secure-private-sg** como o nome e selecione a VPC que você criou anteriormente.   
3. Clique em **Criar grupo de segurança**.  

### Criar uma instância de servidor virtual

Para criar uma instância de servidor virtual na sub-rede recém-criada:

1. Clique na sub-rede privada em **Sub-redes**.
2. Clique em **Instâncias conectadas**; em seguida, **Nova instância**.
3. Insira um nome exclusivo, **vpc-secure-private-vsi**, selecione a VPC que você criou anteriormente, em seguida, o mesmo **Local** como antes.
4. Selecione a imagem **Ubuntu Linux**, clique em **Todos os perfis** e, em **Cálculo**, escolha **c-2x4** com 2vCPUs e 4 GB de RAM
5. Para **Chaves SSH**, selecione a chave SSH que você criou anteriormente para o bastion.
6. Em **Interfaces de rede**, clique no ícone **Editar** ao lado de Grupos de segurança   
   * Selecione **vpc-secure-private-subnet** como a sub-rede.  
   * Desmarque a segurança e o grupo padrão e ative **vpc-secure-private-sg**.  
   * Clique em **Salvar**.  
7. Clique em  ** Criar instância de servidor virtual **.  


### Incluir servidores virtuais no grupo de segurança de manutenção
{: #add-vsi-to-maintenance}

Para trabalho administrativo nos servidores, é necessário associar os servidores virtuais específicos ao grupo de segurança de manutenção. A seguir, você ativará a manutenção, efetuará login no servidor privado, atualizará as informações do pacote de software, em seguida, desassociará o grupo de segurança novamente.

Vamos ativar o grupo de segurança de manutenção para o servidor.

1. Navegue para **Grupos de segurança** e selecione o grupo de segurança **vpc-secure-maintenance-sg**.  
2. Clique em **Interfaces anexadas**; em seguida, **Editar interfaces**.  
3. Expanda as instâncias de servidor virtual e ative a seleção ao lado de **primário** na coluna **Interfaces**.
4. Clique em **Salvar** para que as mudanças sejam aplicadas.

### Conectar-se à instância

Para usar SSH em uma instância usando seu **IP privado**, você usará o host bastion como seu **host de salto**.

1. Obtenha o endereço IP privado de uma instância de servidor virtual em **Instâncias de servidor virtual**.
2. Use o comando ssh com `-J` para efetuar login no servidor com o endereço de **IP flutuante** do bastion que você usou anteriormente e o endereço de **IP privado** do servidor mostrado em **Interfaces de rede**.

   ```sh
   ssh -J root@<BASTION_FLOATING_IP_ADDRESS> root@<PRIVATE_IP_ADDRESS>
   ```
   {:pre}
   
   A sinalização `-J` é suportada no OpenSSH versão 7.3+. O `-J` de versões mais antigas não está disponível. Nesse caso, a maneira mais segura e direta é usar o modo de encaminhamento de stdio do ssh (`-W`) para "gerar bounce" da conexão por meio de um host bastion. Por exemplo, `ssh -o ProxyCommand="ssh -W %h:%p root@<BASTION_FLOATING_IP_ADDRESS" root@<PRIVATE_IP_ADDRESS>`
   {:tip }

### Instalar software e executar tarefas de manutenção

Depois de conectado, é possível instalar o software no servidor virtual na sub-rede privada ou executar tarefas de manutenção.

1. Primeiro, atualize as informações do pacote de software:
   ```sh
   apt-get update
   ```
   {:pre}
2. Instale o software desejado, por exemplo, Nginx ou MySQL ou IBM Db2.

Quando pronto, desconecte-se do servidor com o comando `exit`. 

Para permitir solicitações de HTTP/HTTPS do usuário da Internet, designe um **IP flutuante** à VSI na sub-rede privada e abra as portas necessárias (80 - HTTP e 443 - HTTPS) por meio das regras de entrada no grupo de segurança de VSI privada.
{:tip}

### Desativar o grupo de segurança de manutenção

Depois de terminar de instalar o software ou executar a manutenção, será necessário remover os servidores virtuais do grupo de segurança de manutenção para mantê-los isolados.

1. Navegue para **Grupos de segurança** e selecione o grupo de segurança **vpc-secure-maintenance-sg**.  
2. Clique em **Interfaces anexadas**; em seguida, **Editar interfaces**.  
3. Expanda as instâncias de servidor virtual e desmarque a seleção ao lado de **primário** na coluna **Interfaces**.
4. Clique em **Salvar** para que as mudanças sejam aplicadas.

## Remover recursos
{: #removeresources}

1. Alterne para **Instâncias de servidor virtual** e **Exclua** suas instâncias. As instâncias serão excluídas e seus status permanecerão em **Excluindo** por um tempo. Certifique-se de atualizar o navegador de tempos em tempos.
2. Depois que as VSIs desaparecerem, alterne para **Sub-redes** e exclua suas sub-redes.
4. Depois que as sub-redes tiverem sido excluídas, alterne para a guia **Nuvens particulares virtuais** e exclua seu VPC.

Ao usar o console, pode ser necessário atualizar seu navegador para ver informações de status atualizadas depois de excluir um recurso.
{:tip}

## Conteúdo relacionado
{: #related}

* [Sub-redes privadas e públicas em uma Nuvem Particular Virtual](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend)
