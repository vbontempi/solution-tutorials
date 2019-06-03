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

# Implementar cargas de trabalho isoladas em múltiplos locais e zonas
{: #vpc-multi-region}

A IBM aceitará um número limitado de clientes para participar de um programa de Acesso Antecipado à VPC no início de abril de 2019, com o uso expandido sendo aberto nos meses seguintes. Se a sua organização desejar obter acesso ao IBM Virtual Private Cloud, preencha este [formulário de nomeação](https://{DomainName}/vpc){: new_window} e um representante IBM entrará em contato com você sobre as próximas etapas.
{: important}

Este tutorial conduzirá você nas etapas de configuração de cargas de trabalho isoladas provisionando VPCs em diferentes regiões do IBM Cloud. Regiões com sub-redes e instâncias de servidor virtual (VSIs). Essas VSIs são criadas em múltiplas zonas dentro de uma região para aumentar a resiliência dentro de uma região e globalmente configurando balanceadores de carga com conjuntos de back-end, listeners de front-end e verificações de funcionamento adequadas.

Para o balanceador de carga global, você provisionará um serviço IBM Cloud Internet Services (CIS) do catálogo e, para gerenciar o certificado SSL para todas as solicitações recebidas de HTTPS, o serviço de catálogo do {{site.data.keyword.cloudcerts_long_notm}} será criado e o certificado juntamente com a chave privada será importado.

{:shortdesc}

## Objetivos
{: #objectives}

* Entender o isolamento de cargas de trabalho por meio de objetos de infraestrutura disponíveis para nuvens particulares virtuais.
* Usar um balanceador de carga entre as zonas dentro de uma região para distribuir o tráfego entre os servidores virtuais.
* Usar um balanceador de carga global entre regiões para aumentar a resiliência e reduzir a latência.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.loadbalancer_full}}](https://{DomainName}/vpc/provision/loadBalancer)
- IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
- [{{site.data.keyword.cloudcerts_long_notm}}](https://{DomainName}/catalog/services/cloudcerts)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/estimator/review) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

  ![Arquitetura](images/solution41-vpc-multi-region/Architecture.png)

1. O administrador (DevOps) provisiona VSIs em sub-redes em duas zonas diferentes em uma VPC na região 1 e repete o mesmo em um VPC criado na região 2.
2. O administrador cria um balanceador de carga com um conjunto de back-end de servidores de sub-redes em diferentes zonas da região 1 e um listener de front-end. Repete-se o mesmo na região 2.
3. O administrador provisiona o serviço de serviços da Internet em nuvem com um domínio customizado associado e cria um balanceador de carga global apontando para os balanceadores de carga criados em dois VPCs diferentes.
4. O administrador ativa a criptografia HTTPS, incluindo o certificado SSL de domínio no serviço do gerenciador de certificados.
5. O usuário da Internet faz uma solicitação de HTTP/HTTPS e o balanceador de carga global manipula a solicitação.
6. A solicitação é roteada para os balanceadores de carga, tanto globais quanto locais. A solicitação é, então, preenchida pela instância do servidor disponível.

## Antes de Começar
{: #prereqs}

- Verifique as permissões do usuário. Certifique-se de que sua conta do usuário tenha permissões suficientes para criar e gerenciar recursos VPC. Para obter uma lista de permissões necessárias, consulte [Concedendo permissões necessárias para usuários da VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Você precisa de uma chave SSH para se conectar aos servidores virtuais. Se você não tiver uma chave SSH, consulte as [instruções para criar uma chave](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- O Cloud Internet Services requer que você tenha um domínio customizado para que possa configurar o DNS para que esse domínio aponte para os servidores de nomes do Cloud Internet Services. Se você não tiver um domínio, será possível comprar um de um registrador, como [godaddy.com](http://godaddy.com/).

## Criar VPCs, sub-redes e VSIs
{: #create-infrastructure}

Nesta seção, você criará sua própria VPC na região 1 com sub-redes criadas em duas zonas diferentes da região 1, seguido pelo fornecimento de VSIs.

Para criar seu próprio {{site.data.keyword.vpc_short}} na região 1,

1. Navegue para a página [Visão geral do VPC](https://{DomainName}/vpc/overview) e clique em **Criar um VPC**.
2. Na seção **Nova nuvem particular virtual**:
   * Insira **vpc-region1** como nome para seu VPC.
   * Selecione um **Grupo de recursos**.
   * Opcionalmente, inclua **Tags** para organizar seus recursos.
3. Selecione **Criar novo padrão (Permitir todos)** como sua lista de controle de acesso (ACL) padrão do VPC.
4. Desmarque SSH e ping do **Grupo de segurança padrão** e deixe **acesso clássico** desmarcado.
5. Em **Nova sub-rede para VPC**:
   * Como um nome exclusivo, insira **vpc-region1-zone1-subnet**.
   * Selecione um local (por exemplo, Dallas), vamos chamar isso de **região 1** e uma zona na região 1 (por exemplo, Dallas 1), vamos chamar isso de **zona 1**.
   * Insira o intervalo de IP para a sub-rede em notação CIDR, ou seja, **10.xxx.0.0/24**. Deixe o **Prefixo de endereço** como ele é e selecione o **Número de endereços** como 256.
6. Selecione **Usar padrão de VPC** para sua lista de controle de acesso (ACL) de sub-rede. É possível configurar as regras de entrada e de saída posteriormente.
7. Considerando que todas as instâncias do servidor virtual na sub-rede terão um IP flutuante anexado, ele não será necessário para ativar um gateway público para a sub-rede. As instâncias de servidor virtual terão conectividade de Internet por meio de seu IP flutuante.
8. Clique em **Criar nuvem particular virtual** para provisionar a instância.

Para confirmar a criação da sub-rede, clique em **Sub-redes** na área de janela à esquerda e aguarde até que o status mude para **Disponível**. É possível criar uma nova sub-rede em **Sub-redes**.

### Criar sub-rede na zona 2

1. Clique em **Nova sub-rede**, insira **vpc-region1-zone2-subnet** como um nome exclusivo para sua sub-rede e selecione **vpc-region1** como a VPC.
2. Selecione um local que nós chamamos de região 1 acima (por exemplo, Dallas) e selecione uma zona diferente na região 1 (por exemplo, Dallas 2), vamos chamar a zona selecionada como **zona 2**.
3. Insira o intervalo de IP para a sub-rede em notação CIDR, ou seja, **10.xxx.64.0/24**. Deixe o **Prefixo de endereço** como ele é e selecione o **Número de endereços** como 256.
4. Selecione **Usar padrão de VPC** para sua lista de controle de acesso (ACL) de sub-rede.

### Provisionar VSIs
Depois que o status das sub-redes mudarem para **Disponível**,

1. Clique em **vpc-region1-zone1-subnet** e clique em **Instâncias conectadas**, em seguida, **Nova instância**.
2. Insira um nome exclusivo e escolha **vpc-region1-zone1-vsi**. Em seguida, selecione a VPC que você criou anteriormente e o **Local** juntamente com a **zona** como antes.
3. Escolha qualquer imagem do **Ubuntu Linux**, clique em **Todos os perfis** e, em **Cálculo**, escolha **c-2x4** com 2vCPUs e 4 GB de RAM.
4. Para **Chaves SSH**, selecione a chave SSH que você criou inicialmente.
5. Em **Interfaces de rede**, clique no ícone **Editar** ao lado de Grupos de segurança
   * Verifique se **vpc-region1-zone1-subnet** está selecionado como a sub-rede. Se não, selecione.
   * Clique em **Salvar**.
   * Clique em  ** Criar instância de servidor virtual **.
6.  Aguarde até que o status do VSI mude para **Ligado**. Em seguida, selecione a VSI **vpc-region1-zone1-vsi**, role para **Interfaces de rede** e clique em **Reservar** em **IP flutuante** para associar um endereço IP público à sua VSI. Salve o Endereço IP associado em uma área de transferência para referência futura.
7. **REPITA** as etapas 1 a 6 para provisionar uma VSI na **zona 2** da **região 1**.

Navegue para **VPC** e **Sub-redes** em **Rede** na área de janela esquerda e **REPITA** as etapas acima para provisionar uma nova VPC com sub-redes e VSIs em **region2** seguindo as mesmas convenções de nomenclatura que acima.

## Instalar e configurar o servidor da web nas VSIs
{: #install-configure-web-server-vsis}

Siga as etapas mencionadas em [acessar instâncias remotas de forma segura com um host bastion](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) para manutenção segura dos servidores usando um host bastion que age como um servidor `jump` e um grupo de segurança de manutenção.
{:tip}

Depois de usar SSH com êxito no servidor provisionado na sub-rede da zona 1 da região 1,

1. No prompt, execute os comandos abaixo para instalar o Nginx como seu servidor da web
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   {:codeblock}
2. Verifique o status do serviço Nginx com o comando a seguir:
   ```
   sudo systemctl status nginx
   ```
   {:pre}
   A saída deve mostrar a você que o serviço Nginx está **ativo** e em execução.
3. Você precisará abrir as portas **HTTP (80)** e **HTTPS (443)** para receber o tráfego (solicitações). É possível fazer isso ajustando o Firewall por meio de [UFW](https://help.ubuntu.com/community/UFW) - `sudo ufw enable` e ativando o perfil 'Nginx Full' que inclui regras para ambas as portas:
   ```
   sudo ufw allow 'Nginx Full'
   ```
   {:pre}
4. Para verificar se o Nginx funciona como esperado, abra `http://FLOATING_IP` em seu navegador de escolha e você deverá ver a página de boas-vindas padrão do Nginx.
5. Para atualizar a página html com os detalhes da região e zona, execute o comando a seguir
   ```
   nano /var/www/html/index.nginx-debian.html
   ```
   {:pre}
   Anexe a região e a zona a dizer, por exemplo, _servidor em execução na **zona 1 da região 1**_ para a tag `h1` citando `Welcome to nginx!` e salve as mudanças.
6. Reiniciar o servidor nginx para refletir as mudanças
   ```
   sudo systemctl restart nginx
   ```
   {:pre}

**REPITA** as etapas 1 a 6 para instalar e configurar o servidor da web nas VSIs em sub-redes de todas as zonas e não se esqueça de atualizar o html com as respectivas informações de zona.


## Distribuir tráfego entre zonas com balanceadores de carga
{: #distribute-traffic-with-load-balancers}

Nesta seção, você criará dois balanceadores de carga. Um em cada região para distribuir o tráfego entre múltiplas instâncias do servidor sob as respectivas sub-redes dentro de zonas diferentes.

### Configurar balanceadores de carga

1. Navegue para **Balanceadores de carga** e clique em **Novo balanceador de carga**.
2. Forneça **vpc-lb-region1** como o nome exclusivo, selecione **vpc-region1** como sua nuvem particular virtual seguida pelo grupo de recursos em que a VPC foi criada, Tipo: **Público** e **region1** como a região.
3. Selecione os IPs privados de **zona 1** e **zona 2** da **região 1**.
4. Crie um novo conjunto de back-end de VSIs que ajam como peers iguais para compartilhar o tráfego roteado para o conjunto. Configure os parâmetros com os valores abaixo e clique em **criar**.
	- **Name**: region1-pool
	- **Protocolo**: HTTP
	- **Method**: Round robin
	- **Session stickiness**: None
	- **Health check path**: /
	- **Health protocol**: HTTP
	- **Intervalo(sec)**: 15
	- **Timeout(sec)**: 2
	- **Max retries**: 2
5. Clique em **Anexar** para incluir instâncias do servidor para o region1-pool
   - Selecione o IP privado de **vpc-region1-zone1-subnet**, selecione a instância que você criou e configure 80 como a porta.
   - Clique em **Incluir** e, desta vez, selecione o IP privado de **vpc-region1-zone2-subnet**, selecione a instância e configure 80 como a porta.
   - Clique em **Anexar** para concluir a criação de um conjunto de back-end.
6. Clique em **Novo listener** para criar um novo listener de front-end; Um listener é um processo que verifica solicitações de conexão.
   - **Protocolo**: HTTP
   - **Porta**: 80
   - **Conjunto de back-end**: region1-pool
   - **Maxconnections**: deixe-o vazio e clique em **criar**.
7. Clique em **Criar balanceador de carga** para provisionar um balanceador de carga.

### Testar os balanceadores de carga

1. Aguarde até que o status do balanceador de carga mude para **Ativo**.
2. Abra o **Endereço** em um navegador da web.
3. Atualize a página várias vezes e observe o balanceador de carga atingindo diferentes servidores com cada atualização.
4. **Salve** o endereço para referência futura.

Se você observar, as solicitações não são criptografadas e suportam somente HTTP. Você configurará um certificado SSL e ativará o HTTPS na próxima seção.

**REPITA** as etapas 1 a 7 acima na **região 2**.

## Tráfego seguro dentro da VPC com HTTPS
{: #secure_https}

Antes de incluir um listener HTTPS, é necessário gerar um certificado SSL, verificar a autenticidade de seu domínio customizado, um local para conter o certificado e mapeá-lo para o serviço de infraestrutura.

### Provisionar um serviço CIS e configurar o domínio customizado.

Nesta seção, você criará o serviço IBM Cloud Internet Services (CIS), configurará um domínio customizado apontando-o para servidores de nomes do CIS e, posteriormente, configurará um balanceador de carga global.

1. Navegue para o [Internet Services](https://{DomainName}/catalog/services/internet-services) no catálogo do {{site.data.keyword.Bluemix_notm}}.
2. Configure o nome do serviço e clique em **Criar** para criar uma instância do serviço. É possível usar quaisquer planos de precificação para este tutorial.
3. Quando a instância de serviço for provisionada, configure seu nome de domínio clicando em **Vamos começar** e clique em **Incluir domínio**.
4. Clique em **Próxima etapa**. Quando os servidores de nomes forem designados, configure seu registrador ou provedor de nome de domínio para usar os servidores de nomes listados.
5. Depois de ter configurado seu registrador ou o provedor de DNS, pode ser necessário até 24 horas para que as mudanças entrem em vigor.

   Quando o status do domínio na página Visão geral muda de *Pendente* para *Ativo*, é possível usar o comando `dig <YOUR_DOMAIN_NAME> ns` para verificar se os novos servidores de nomes entraram em vigor.
   {:tip}

É necessário obter um certificado SSL para o domínio e o subdomínio que você planeja usar com o balanceador de carga global. Supondo um domínio como mydomain.com, o balanceador de carga global pode ser hospedado em `lb.mydomain.com`. O certificado precisará ser emitido para lb.mydomain.com.

É possível obter certificados SSL grátis em [Vamos criptografar](https://letsencrypt.org/). Durante o processo, talvez seja necessário configurar um registro de DNS do tipo TXT na interface de DNS do Cloud Internet Services para provar que você é o proprietário do domínio.
{:tip}

Depois de ter obtido o certificado SSL e a chave privada para seu domínio, certifique-se de convertê-los para o formato [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Para converter um Certificado no formato PEM:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
2. Para converter uma Chave privada no formato PEM:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Importar o certificado e autorizar o serviço de balanceador de carga

É possível gerenciar os certificados SSL por meio do IBM Certificate Manager.

1. Crie uma instância do [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) em uma localização suportada.
2. No painel de serviço, use **Importar certificado**:
   * Configure **Nome** para o subdomínio e o domínio customizados, como *lb.mydomain.com*.
   * Procure o **Arquivo de certificado** no formato PEM.
   * Procure o **Arquivo de chave privado** no formato PEM.
   * **Importar**.
3. Crie uma autorização que forneça à instância de serviço do balanceador de carga acesso à instância do gerenciador de certificados que contém o certificado SSL. Você pode gerenciar essa autorização por meio de [Autorizações de identidade e acesso](https://{DomainName}/iam#/authorizations).
  - Clique em **Criar** e escolha **Serviço de infraestrutura** como o serviço de origem
  - **Load Balancer para VPC** como o tipo de recurso
  - **Certificate Manager** como o serviço de destino
  - Designe a função de acesso de serviço **Gravador**.
  - Para criar um balanceador de carga, deve-se conceder a autorização Todas as instâncias de recurso para a instância de recurso de origem. A instância de serviço de destino pode ser **Todas as instâncias** ou pode ser sua instância de recurso específica do gerenciador de certificados.

### Criar um listener HTTPS

Agora, navegue para os [Balanceadores de carga](https://{DomainName}/vpc/network/loadBalancers)

1. Selecione **vpc-lb-region1**
2. Em **Listeners de front-end**, clique em **Novo listener**

   -  **Protocolo**: HTTPS
   -  **Porta**: 443
   -  **Conjunto de back-end**: POOL na mesma região
   -  Escolha o certificado SSL para **lb.YOUR-DOMAIN-NAME**

3. Clique em **Criar** para configurar um listener HTTPS

**REPITA** o mesmo no balanceador de carga de **região 2**.

## Configurar um balanceador de carga global
{: #global-load-balancer}

Nesta seção, você configurará um Global Load Balancer (GLB) distribuindo o tráfego recebido para os balanceadores de carga locais configurados em diferentes regiões do {{site.data.keyword.Bluemix_notm}}.

### Distribuir tráfego entre regiões com um balanceador de carga global
Abra o serviço CIS que você criou navegando para a [Lista de recursos](https://{DomainName}/resources) em serviços.

1. Navegue para **Global Load Balancers** em **Confiabilidade** e clique em **criar balanceador de carga**.
2. Insira **lb.YOUR-DOMAIN-NAME** como seu nome do host e TTL como 60 segundos.
3. Clique em **Incluir conjunto** para definir um conjunto de origem padrão
   - **Nome**: lb-region1
   - **Verificação de funcionamento**: CREATE A NEW HEALTH CHECK
     - **Tipo de monitor**: HTTP
     - **Caminho**: /
     - **Porta**: 80
   - **Região de verificação de funcionamento**: Leste da América do Norte
   - **origens**
     - **nome**: region1
     - **endereço**: ADDRESS OF **REGION1** LOCAL LOAD BALANCER
     - **peso**: 1
     - Clique em **Incluir**

4. **INCLUA** mais um **conjunto de origem** apontando para o balanceador de carga **region2** na região **Europa Ocidental** e clique em **Provisionar 1 Recurso** para provisionar seu balanceador de carga global.

Aguarde até que o status da verificação de **Funcionamento** mude para **Funcional**. Abra o link **lb.YOUR-DOMAIN-NAME** em um navegador de sua escolha para ver o balanceador de carga global em ação.

### Teste de failover
Por enquanto, você deve ter visto que na maior parte do tempo você está atingindo os servidores na **região 1** porque há um peso maior designado em comparação com os servidores na **região 2**. Vamos introduzir uma falha de verificação de funcionamento no conjunto de origem **região 1**,

1. Navegue para [instâncias de servidor virtual](https://{DomainName}/vpc/compute/vs).
2. Clique em **três pontos (...)** próximo aos servidores em execução na **zona 1** da **região 1** e clique em **Parar**.
3. **REPITA** o mesmo para servidores em execução na **zona 2** da **região 1**.
4. Retorne ao GLB no serviço CIS e aguarde até que o status de funcionamento mude para **Crítico**.
5. Agora, quando você atualiza sua URL de domínio, é necessário sempre estar atingindo os servidores na **região 2**.

Não se esqueça de **iniciar** os servidores na zona 1 e zona 2 da região 1
{:tip}

## Remover recursos
{: #removeresources}

- Remover o balanceador de carga global, os conjuntos de origem e as verificações de funcionamento no serviço CIS
- Remova os certificados no serviço do gerenciador de certificados.
- Remova os balanceadores de carga, VSIs, sub-redes e VPCs.
- Em [Lista de recursos](https://{DomainName}/resources), exclua os serviços usados neste tutorial.


## Conteúdo relacionado
{: #related}

* [Usando Load Balancers no IBM Cloud VPC](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc#--beta-using-load-balancers-in-ibm-cloud-vpc)
