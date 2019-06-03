---
copyright:
  years: 2019
lastupdated: "2019-05-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}
{:important: .important}

# Usar um gateway VPC/VPN para acesso no local seguro e privado para recursos em nuvem
{: #vpc-site2site-vpn}

A IBM aceitará um número limitado de clientes para participar de um programa de Acesso Antecipado à VPC no início de abril de 2019, com o uso expandido sendo aberto nos meses seguintes. Se a sua organização desejar obter acesso ao IBM Virtual Private Cloud, preencha este [formulário de nomeação](https://{DomainName}/vpc){: new_window} e um representante IBM entrará em contato com você sobre as próximas etapas.
{: important}

A IBM oferece várias maneiras de ampliar de forma segura uma rede de computadores no local com recursos no IBM Cloud. Isso permite que você se beneficie da elasticidade de provisionar servidores quando precisar deles e removê-los quando não forem mais necessários. Além disso, é possível conectar de forma fácil e segura seus recursos no local aos serviços do {{site.data.keyword.cloud_notm}}.

Este tutorial conduz você na conexão de um gateway Rede Privada Virtual (VPN) no local para uma VPN de nuvem criada dentro de uma VPC (um gateway VPC/VPN). Primeiro, você criará uma nova {{site.data.keyword.vpc_full}} (VPC) e os recursos associados, como sub-redes, Listas de Controle de Acesso (ACLs) à rede, Grupos de Segurança e Instância de Servidor Virtual (VSI).
O gateway VPC/VPN estabelecerá um link de site para site do [IPsec](https://en.wikipedia.org/wiki/IPsec) para uma VPN de gateway no local. Os protocolos IPsec e [Troca de Chave da Internet](https://en.wikipedia.org/wiki/Internet_Key_Exchange), IKE, são padrões abertos comprovados para comunicação segura.

Para demonstrar ainda mais o acesso seguro e privado, você implementará um microsserviço em uma VSI para acessar o {{site.data.keyword.cos_short}} (COS), representando um aplicativo de linha de negócios.
O serviço COS tem um terminal direto que pode ser usado para ingresso/egresso privado sem custo quando todo o acesso está dentro da mesma região do {{site.data.keyword.cloud_notm}}. Um computador no local acessará o microsserviço do COS. Todo o tráfego fluirá por meio da VPN e, portanto, privadamente por meio do {{site.data.keyword.cloud_notm}}.

Há muitas soluções de VPN no local populares para gateways de site para site disponíveis. Este tutorial utiliza o Gateway de VPN [strongSwan](https://www.strongswan.org/) para conectar ao gateway VPC/VPN. Para simular um data center no local, você instalará o gateway strongSwan em uma VSI no {{site.data.keyword.cloud_notm}}.

{:shortdesc}
Em resumo, usando uma VPC, é possível

- conectar seus sistemas no local a serviços e cargas de trabalho em execução no {{site.data.keyword.cloud_notm}},
- assegurar a conectividade privada e de baixo custo para o COS,
- conectar seus sistemas baseados em nuvem a serviços e cargas de trabalho em execução no local.

## Objetivos
{: #objectives}

* Acessar um ambiente de nuvem particular virtual por meio de um data center no local ou uma nuvem particular (virtual).
* Atingir de forma segura os serviços de nuvem usando terminais de serviço privado.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.vpn_full}}](https://{DomainName}/vpc/provision/vpngateway)
- [{{site.data.keyword.cos_full}}](https://{DomainName}/catalog/services/cloud-object-storage)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.
Embora não haja encargos de rede para acessar o COS por meio do microsserviço neste tutorial, os encargos de rede padrão para acesso à VPC serão incorridos.

## Arquitetura
{: #architecture}

O diagrama a seguir mostra a nuvem particular virtual que contém um servidor de aplicativos. O servidor de aplicativos hospeda um microsserviço que faz interface com o serviço {{site.data.keyword.cos_short}}. Uma rede no local (simulada) e o ambiente de nuvem virtual são conectados por meio de gateways VPN.

![Arquitetura](images/solution46-vpc-vpn/ArchitectureDiagram.png)

1. A infraestrutura (VPC, Sub-redes, Security Groups com regras, ACL de rede e VSIs) é configurada usando um script fornecido.
2. O microsserviço faz interface com o {{site.data.keyword.cos_short}} por meio de um terminal direto.
3. Um Gateway VPC/VPN é provisionado para expor o ambiente de nuvem particular virtual para a rede no local.
4. O software de gateway IPsec de software livre Strongswan é usado no local para estabelecer a conexão VPN com o ambiente de nuvem.

## Antes de Começar
{: #prereqs}

- Instale todas as ferramentas necessárias de linha de comandos (CLI) [seguindo estas etapas](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview). Você precisa do plug-in de infraestrutura da CLI opcional.
- Efetue login no {{site.data.keyword.cloud_notm}} por meio da linha de comandos. Consulte [Introdução à CLI](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud-cli) para obter detalhes.
- Verifique as permissões do usuário. Certifique-se de que sua conta do usuário tenha permissões suficientes para criar e gerenciar recursos VPC. Para obter uma lista de permissões necessárias, consulte [Concedendo permissões necessárias para usuários da VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).
- Você precisa de uma chave SSH para se conectar aos servidores virtuais. Se você não tiver uma chave SSH, consulte as [instruções para criar uma chave](/docs/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- Instale o [**jq**](https://stedolan.github.io/jq/download/). Ele é usado pelos scripts fornecidos para processar a saída JSON.

## Implementar um servidor de aplicativos virtual em uma nuvem particular virtual
A seguir, você fará download dos scripts para configurar um ambiente de VPC de linha de base e código para um microsserviço para fazer interface com o {{site.data.keyword.cos_short}}. Depois disso, você provisionará o serviço {{site.data.keyword.cos_short}} e configurará a linha de base.

### Obtenha o código
{: #setup}
O tutorial usa scripts para implementar uma linha de base de recursos de infraestrutura antes de criar os gateways VPN. Esses scripts e o código para o microsserviço estão disponíveis em um repositório GitHub.

1. Obtenha o código do aplicativo:
   ```sh
   git clone https://github.com/IBM-Cloud/vpc-tutorials
   ```
   {: codeblock}
2. Acesse os scripts para este tutorial mudando para **vpc-tutorials**, em seguida, **vpc-site2site-vpn**:
   ```sh
   cd vpc-tutorials/vpc-site2site-vpn
   ```
   {: codeblock}

### Criar serviços
Nesta seção, você efetuará login no {{site.data.keyword.cloud_notm}} na CLI e criará uma instância do {{site.data.keyword.cos_short}}.

1. Verifique se você seguiu as etapas de pré-requisito de login
    ```sh
    ibmcloud target
    ```
    {: codeblock}
2. Crie uma instância do [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) usando um plano **padrão** ou **lite**.
   ```sh
   ibmcloud resource service-instance-create vpns2s-cos cloud-object-storage lite global
   ```
   {: codeblock}
   
   Observe que somente uma instância lite pode ser criada por conta. Se você já tiver uma instância do {{site.data.keyword.cos_short}}, será possível reutilizá-la.
   {: tip}

3. Crie uma chave de serviço com a função **Leitor**:
   ```sh
   ibmcloud resource service-key-create vpns2s-cos-key Reader --instance-name vpns2s-cos
   ```
   {: codeblock}
4. Obtenha os detalhes da chave de serviço no formato JSON e armazene-os em um novo arquivo **credentials.json** no subdiretório **vpc-app-cos**. O arquivo será usado posteriormente pelo app.
   ```sh
   ibmcloud resource service-key vpns2s-cos-key --output json > vpc-app-cos/credentials.json
   ```
   {: codeblock}
   

### Criar recursos de linha de base da Nuvem Particular Virtual
{: #create-vpc}
O tutorial fornece um script para criar os recursos de linha de base necessários para este tutorial, ou seja, o ambiente de início. O script pode gerar esse ambiente em uma VPC existente ou criar uma nova VPC.

A seguir, crie esses recursos configurando e, em seguida, executando um script de configuração. O script incorpora a configuração de um host bastion, conforme discutido em [acessar as instâncias remotas de forma segura com um host bastion](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server).

1. Copie o arquivo de configuração de amostra em um arquivo a ser usado:

   ```sh
   cp config.sh.sample config.sh
   ```
   {: codeblock}

2. Edite o arquivo **config.sh** e adapte as configurações para seu ambiente. É necessário mudar o valor de **SSHKEYNAME** para o nome ou lista separada por vírgula de nomes de chaves SSH (consulte "Antes de iniciar"). Modifique as configurações de **ZONE** diferentes para corresponder à sua região de nuvem. Todas as outras variáveis podem ser mantidas no estado em que se encontram.
3. Para criar os recursos em uma nova VPC, execute o script como a seguir:

   ```sh
   ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

   Para reutilizar uma VPC existente, passe seu nome para o script desta maneira. Substitua **YOUR_EXISTING_VPC** pelo nome real de VPC.
   ```sh
   REUSE_VPC=YOUR_EXISTING_VPC ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

4. Isso resultará na criação dos recursos a seguir, incluindo os recursos relacionados ao bastion:
   - 1 VPC (opcional)
   - 1 gateway público
   - 3 sub-redes dentro da VPC
   - 4 grupos de segurança com regras de ingresso e egresso
   - 3 VSIs: vpns2s-onprem-vsi (floating-ip é ONPREM_IP), vpns2s-cloud-vsi (floating-ip é VSI_CLOUD_IP) e vpns2s-bastion (floating-ip é BASTION_IP_ADDRESS)

   Anote para uso posterior os valores retornados para **BASTION_IP_ADDRESS**, **VSI_CLOUD_IP**, **ONPREM_IP**, **CLOUD_CIDR** e **ONPREM_CIDR**. A saída também é armazenada no arquivo **network_config.sh**. O arquivo pode ser usado para configuração automatizada.

### Criar o gateway e a conexão de Rede Privada Virtual
A seguir, você incluirá um gateway VPN e uma conexão associada com a sub-rede com a VSI do aplicativo.

1. Navegue para a página [Visão geral do VPC](https://{DomainName}/vpc/overview) e, em seguida, clique em **VPNs** na guia de navegação e em **Novo gateway VPN** no diálogo. No formulário **Novo gateway VPN para VPC**, insira **vpns2s-gateway** como o nome. Certifique-se de que a VPC, o grupo de recursos e o **vpns2s-cloud-subnet** como sub-rede corretos estejam selecionados.
2. Deixe **Nova conexão VPN para VPC** ativado. Insira **vpns2s-gateway-conn** como o nome.
3. Para o **Endereço do gateway de peer**, use o endereço IP flutuante de **vpns2s-onprem-vsi** (ONPREM_IP). Digite **20_PRESHARED_KEY_KEEP_SECRET_19** como **Chave pré-compartilhada**.
4. Para **Sub-redes locais**, use as informações fornecidas para **CLOUD_CIDR**, para **Sub-redes de peer**, aquelas para **ONPREM_CIDR**.
5. Deixe as configurações em **Detecção de peer inativo** no estado em que se encontram. Clique em **Criar gateway VPN** para criar o gateway e uma conexão associada.
6. Aguarde que o gateway VPN se torne disponível (talvez seja necessário atualizar a tela).
7. Anote o endereço **IP do gateway** designado como **GW_CLOUD_IP**. 

### Criar o gateway de Rede Privada Virtual no local
Em seguida, você criará o gateway VPN no outro site, no ambiente no local simulado. Você usará o software IPsec [strongSwan](https://strongswan.org/) baseado em software livre.

1. Conecte-se à VSI "no local" **vpns2s-onprem-vsi** usando ssh. Execute o seguinte e substitua **ONPREM_IP** pelo endereço IP retornado anteriormente.

   ```sh
   ssh root@ONPREM_IP
   ```
   {:pre}

   Dependendo de seu ambiente, talvez seja necessário usar `ssh -i <path to your private key file> root@ONPREMP_IP`.
   {:tip}

2. Em seguida, na máquina **vpns2s-onprem-vsi**, execute os comandos a seguir para atualizar o gerenciador de pacotes e para instalar o software strongSwan.

   ```sh
   apt-get update
   ```
   {:pre}
   
   ```sh
   apt-get install strongswan
   ```
   {:pre}

3. Configure o arquivo **/etc/sysctl.conf** incluindo três linhas em sua extremidade. Copie o seguinte e execute-o:

   ```sh
   cat >> /etc/sysctl.conf << EOF
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.send_redirects = 0
   EOF
   ```
   {:codeblock}

4. Em seguida, edite o arquivo **/etc/ipsec.secrets**. Inclua a linha a seguir para configurar os endereços IP de origem e de destino e a chave pré-compartilhada configurada anteriormente. Substitua **ONPREM_IP** com o valor conhecido do IP flutuante do vpns2s-onprem-vsi. Substitua o **GW_CLOUD_IP** pelo endereço IP conhecido do gateway VPN VPC.

   ```
   ONPREM_IP GW_CLOUD_IP : PSK "20_PRESHARED_KEY_KEEP_SECRET_19"
   ```
   {:pre}

5. O último arquivo que você precisa configurar é **/etc/ipsec.conf**. Inclua o bloco de códigos a seguir no final desse arquivo. Substitua **ONPREM_IP**, **ONPREM_CIDR**, **GW_CLOUD_IP** e **CLOUD_CIDR** com os respectivos valores conhecidos.

   ```sh
   # basic configuration
   config setup
      charondebug="all"
      uniqueids=yes
      strictcrlpolicy=no
   
   # connection to vpc/vpn datacenter 
   # left=onprem / right=vpc
   conn tutorial-site2site-onprem-to-cloud
      authby=secret
      left=%defaultroute
      leftid=ONPREM_IP
      leftsubnet=ONPREM_CIDR
      right=GW_CLOUD_IP
      rightsubnet=CLOUD_CIDR
      ike=aes256-sha2_256-modp1024!
      esp=aes256-sha2_256!
      keyingtries=0
      ikelifetime=1h
      lifetime=8h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start
   ```
   {:pre}

6. Reinicie o gateway VPN e, em seguida, verifique seu status executando: reinício ipsec

   ```sh
   ipsec restart
   ```
   {:pre}
   ```sh
   ipsec status
   ```
   {:pre}

   Ele deve relatar que uma conexão foi estabelecida. Manter o terminal e a conexão ssh para esta máquina abertos.

## Testar a conectividade
É possível testar a conexão VPN de site para site usando SSH ou implementando o microsserviço que faz interface com o {{site.data.keyword.cos_short}}.

### Testar usando ssh
Para testar se a conexão VPN foi estabelecida com êxito, use o ambiente no local simulado como proxy para efetuar login no servidor de aplicativos baseado em nuvem. 

1. Em um novo terminal, execute o comando a seguir depois de substituir os valores. Ele usa o host strongSwan como o host de salto para se conectar por meio de VPN ao endereço IP privado do servidor de aplicativos.

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
  
2. Uma vez conectado com êxito, feche a conexão ssh.

3. No terminal de VSI "no local", pare o gateway VPN:
   ```sh
   ipsec stop
   ```
   {:pre}
4. Na janela de comando da etapa 1), tente estabelecer a conexão novamente:

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
   O comando não deve ser bem-sucedido porque a conexão VPN não está ativa e, portanto, não há nenhum link direto entre os ambientes simulados no local e de nuvem.

   Observe que, dependendo dos detalhes da implementação, na realidade, essa conexão ainda é bem-sucedida. A razão é que a conectividade intra-VPC é suportada entre as zonas. Se você implementasse a VSI simulada no local em outra VPC ou em [{{site.data.keyword.virtualmachinesfull}}](/docs/vsi?topic=virtual-servers-about-public-virtual-servers), a VPN seria necessária para o acesso bem-sucedido.
   {:tip}
   
5. No terminal de VSI "no local", inicie o gateway VPN novamente:
   ```sh
   ipsec start
   ```
   {:pre}
 

### Testar usando um microsserviço
É possível testar a conexão VPN de trabalho acessando um microsserviço na VSI de nuvem por meio da VSI no local.

1. Copie o código para o app de microsserviço de sua máquina local para a VSI de nuvem. O comando usa a bastion como host de salto para a VSI de nuvem. Substitua **BASTION_IP_ADDRESS** e **VSI_CLOUD_IP** adequadamente.
   ```sh
   scp -r  -o "ProxyJump root@BASTION_IP_ADDRESS"  vpc-app-cos root@VSI_CLOUD_IP:vpc-app-cos
   ```
   {:pre}
2. Conecte-se à VSI de nuvem, usando novamente o bastion como o host de salto.
   ```sh
   ssh -J root@BASTION_IP_ADDRESS root@VSI_CLOUD_IP
   ```
   {:pre}
3. No VSI de nuvem, mude para o diretório de código:
   ```sh
   cd vpc-app-cos
   ```
   {:pre}
4. Instale o Python e o PIP do gerenciador de pacotes Python.
   ```sh
   apt-get update; apt-get install python python-pip
   ```
   {:pre}
5. Instale os pacotes Python necessários usando **pip**.
   ```sh
   pip install -r requirements.txt
   ```
   {:pre}
6. Inicie o app:
   ```sh
   python browseCOS.py
   ```
   {:pre}
7. No terminal de VSI "no local", acesse o serviço. Substitua VSI_CLOUD_IP adequadamente.
   ```sh
   curl VSI_CLOUD_IP/api/bucketlist
   ```
   {:pre}
   O comando deve retornar um objeto JSON.

## Remover recursos
{: #remove-resources}

1. No console de gerenciamento de VPC, clique em **VPNs**. No menu Ação no gateway VPN, selecione **Excluir** para remover o gateway.
2. Em seguida, clique em **IPs flutuantes** na navegação e, em seguida, no endereço IP para suas VSIs. No menu Ação, selecione **Liberar**. Confirme se você deseja liberar o endereço IP.
3. Em seguida, alterne para **Instâncias de servidor virtual** e **Excluir** suas instâncias. As instâncias serão excluídas e seus status permanecerão em **Excluindo** por um tempo. Certifique-se de atualizar o navegador de tempos em tempos.
4. Quando os VSIs forem desligados, alterne para **Sub-redes**. Se a sub-rede tiver um gateway público conectado, clique no nome da sub-rede. Nos detalhes da sub-rede, desconecte o gateway público. As sub-redes sem o gateway público podem ser excluídas da página de visão geral. Exclua suas sub-redes.
5. Depois que as sub-redes tiverem sido excluídas, alterne para **VPC** e exclua sua VPC.

Ao usar o console, pode ser necessário atualizar seu navegador para ver informações de status atualizadas depois de excluir um recurso.
{:tip}

## Expandir o tutorial 
{: #expand-tutorial}

Deseja incluir ou estender este tutorial? Aqui estão algumas ideias:

- Inclua um [balanceador de carga](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc) para distribuir o tráfego de microsserviço de entrada entre múltiplas instâncias.
- Implemente o [aplicativo em um servidor público, seus dados e serviços em um host privado](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend).


## Conteúdo relacionado
{: #related}

- [Glossário do VPC](/docs/vpc?topic=vpc-vpc-glossary)
- [Referência do plug-in da CLI do IBM Cloud para VPC](/docs/infrastructure-service-cli-plugin?topic=infrastructure-service-cli-vpc-reference)
- [VPC usando as APIs de REST](/docs/infrastructure/vpc/example-code.html)
- Tutorial da solução: [Acessar instâncias remotas de forma segura com um host bastion](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)
