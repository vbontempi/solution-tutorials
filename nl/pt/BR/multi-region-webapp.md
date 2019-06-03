---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Proteger o aplicativo da web em múltiplas regiões
{: #multi-region-webapp}

Este tutorial conduz você na criação, proteção, implementação e balanceamento de carga de um aplicativo do Cloud Foundry em múltiplas regiões usando um pipeline do [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery).

Os apps ou partes de seus apps terão indisponibilidades, isso é um fato. Pode ser um problema em seu código, uma manutenção planejada afetando os recursos usados por seu app, uma falha no hardware desativando uma zona, um local, um data center no qual seu app está hospedado. Qualquer uma dessas situações poderá acontecer e você terá que estar preparado. Com o {{site.data.keyword.Bluemix_notm}}, é possível implementar seu aplicativo em [múltiplos locais](https://{DomainName}/docs/overview?topic=overview-whatis-platform#ov_intro_reg) para aumentar a resiliência do aplicativo. E com seu aplicativo agora em execução em múltiplos locais, também é possível redirecionar o tráfego do usuário para o local mais próximo para reduzir a latência.

## Objetivos

* Implementar um aplicativo do Cloud Foundry em múltiplos locais com o {{site.data.keyword.contdelivery_short}}.
* Mapear um domínio customizado para o aplicativo.
* Configurar o balanceamento de carga global para seu aplicativo multilocal.
* Ligar um certificado SSL a seu aplicativo.
* Monitorar o desempenho do aplicativo.

## Serviços usados

Este tutorial usa os tempos de execução e serviços a seguir:
* App do Cloud Foundry [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) para DevOps
* [Internet services](https://{DomainName}/catalog/services/internet-services)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura

Este tutorial envolve um cenário ativo/ativo no qual duas cópias do aplicativo são implementadas em dois locais diferentes e as duas cópias estão entregando solicitações do cliente de uma maneira round-robin. A configuração do DNS apontará automaticamente para o local funcional se uma cópia falhar.

<p style="text-align: center;">

   ![Arquitetura](./images/solution1/Architecture.png)
</p>

## Criar um aplicativo Node.js
{: #create}

Inicie criando um aplicativo iniciador Node.js que é executado em um ambiente do Cloud Foundry.

1. Clique em **Catálogo do [](https://{DomainName}/catalog/)** no console do {{site.data.keyword.Bluemix_notm}}.
2. Clique em **Apps do Cloud Foundry** sob a categoria **Plataforma** e selecione **[{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)**.
     ![](images/solution1/SDKforNodejs.png)
3. Insira um **nome exclusivo** para seu aplicativo, que também será o seu nome do host, por exemplo: myusername-nodeapp. E clique em **Criar**.
4.  Depois que o aplicativo for iniciado, clique no link **Visitar URL** na página **Visão geral** para ver seu aplicativo EM TEMPO REAL em uma nova guia.

![HelloWorld](images/solution1/HelloWorld.png)

Ótimo início! Você tem o seu próprio aplicativo iniciador Node.js em execução no {{site.data.keyword.Bluemix_notm}}.

Em seguida, vamos enviar por push o código-fonte de seu aplicativo para um repositório e implementar suas mudanças automaticamente.

## Configurar o controle de fonte e o {{site.data.keyword.contdelivery_short}}
{: #devops}

Nesta etapa, você configura um repositório de controle de fonte git para armazenar seu código e, em seguida, cria um pipeline, que implementa qualquer mudança de código automaticamente.

1. Na área de janela esquerda de seu aplicativo recém-criado, selecione **Visão geral** e role para localizar **{{site.data.keyword.contdelivery_short}}**. Clique em **Ativar**.

   ![HelloWorld](images/solution1/Enable_Continuous_Delivery.png)
2. Mantenha as opções padrão e clique em **Criar**. Agora você deverá ter uma **cadeia de ferramentas** padrão criada.

   ![HelloWorld](images/solution1/DevOps_Toolchain.png)
3. Selecione o ladrilho **Git** em **Código**. Em seguida, você é direcionado para a página do repositório git.
4. Se você ainda não tiver configurado as chaves SSH, deverá ver uma barra de notificação na parte superior com instruções. Siga as etapas abrindo o link **incluir uma chave SSH** em uma nova guia ou, se você desejar usar HTTPS em vez de SSH, siga as etapas clicando em **criar um token de acesso pessoal**. Lembre-se de salvar a chave ou o token para referência futura.
5. Selecione SSH ou HTTPS e copie a URL do git. Clone a origem para sua máquina local.
   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   **Nota:** se for solicitado um nome do usuário, forneça seu nome do usuário do git. Para a senha, use uma **chave SSH** existente ou um **token de acesso pessoal** ou aquele criado na etapa anterior.
6. Abra o repositório clonado em um IDE de sua escolha e navegue para `public/index.html`. Agora, vamos atualizar o código. Tente mudar "Hello World" para outra coisa.
7. Execute o aplicativo localmente executando os comandos um após outro
   `npm install`, `npm build`, `npm start` e visite `localhost:<port_number>` em seu navegador.
  **<port_number >** conforme exibido no console.
8. Envie por push a mudança para seu repositório com três etapas simples: inclua, confirme e envie por push.
   ```bash
   git add public/index.html git commit -m "my first changes" git push origin master
   ```
9. Acesse a cadeia de ferramentas que você criou anteriormente e clique no bloco **Delivery Pipeline**.
10. Confirme se você vê o estágio **BUILD** e **DEPLOY**.
    ![HelloWorld](images/solution1/DevOps_Pipeline.png)
11. Aguarde até que o estágio **DEPLOY** seja concluído.
12. Clique na **url** do aplicativo sob o resultado da Última execução para visualizar suas mudanças em tempo real.

Continue fazendo mudanças adicionais em seu aplicativo e confirme periodicamente suas mudanças em seu repositório git. Se você não vir a atualização do aplicativo, verifique os logs dos estágios DEPLOY e BUILD de seu pipeline.

## Implementar em outro local
{: #deploy_another_region}

Em seguida, implementaremos o mesmo aplicativo em um local do {{site.data.keyword.Bluemix_notm}} diferente. Podemos usar a mesma cadeia de ferramentas, mas incluiremos outro estágio DEPLOY para manipular a implementação do aplicativo em outro local.

1. Navegue para a **Visão geral** do Aplicativo e role para localizar **Visualizar cadeia de ferramentas**.
2. Selecione **Delivery Pipeline** em Entregar.
3. Clique no **Ícone de engrenagem** no estágio **DEPLOY** e selecione **Clonar estágio**.
   ![HelloWorld](images/solution1/CloneStage.png)
4. Renomeie o estágio para "Implementar no Reino Unido" e selecione **JOBS**.
5. Mude **Região do IBM Cloud** para **Londres - https://api.eu-gb.bluemix.net**. Crie um **espaço** se você não tiver um.
6. Mude **Implementar script** para `cf push "${CF_APP}" -d eu-gb.mybluemix.net`

   ![HelloWorld](images/solution1/DeployToUK.png)
7. Clique em **Salvar** e execute o novo estágio clicando no **botão Reproduzir**.

## Registrar um domínio customizado com o IBM Cloud Internet Services

{: #domain_cis}

O IBM [Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) é uma plataforma uniforme para configurar e gerenciar o Sistema de Nomes de Domínio (DNS), o Global Load Balancing (GLB), o Web Application Firewall (WAF) e a proteção contra o Distributed Denial of Service (DDoS) para aplicativos da web. Ele fornece um serviço da Internet rápido, altamente funcional, confiável e seguro para clientes que executam seus negócios no IBM Cloud com três recursos principais para aprimorar seu fluxo de trabalho: segurança, confiabilidade e desempenho.  

Ao implementar um aplicativo do mundo real, provavelmente você desejará usar seu próprio domínio em vez do domínio mybluemix.net fornecido pela IBM. Nesta etapa, depois de ter um domínio customizado, é possível usar os servidores DNS fornecidos pelo IBM Cloud Internet Services.

1. Compre um domínio por meio de um escrivão, como [http://godaddy.com](http://godaddy.com).
2. Navegue para o [Internet Services](https://{DomainName}/catalog/services/internet-services) no catálogo do {{site.data.keyword.Bluemix_notm}}.
2. Insira um nome de serviço e clique em **Criar** para criar uma instância do serviço.
3. Quando a instância de serviço for provisionada, configure seu nome de domínio e clique em **Incluir domínio**.
4. Quando os servidores de nomes forem designados, configure seu registrador ou provedor de nome de domínio para usar os servidores de nomes listados.
5. Depois de ter configurado seu registrador ou o provedor de DNS, pode ser necessário até 24 horas para que as mudanças entrem em vigor.
  Quando o status do domínio na página Visão geral muda de *Pendente* para *Ativo*, é possível usar o comando `dig <your_domain_name> ns` para verificar se os servidores de nomes do IBM Cloud entraram em vigor.
  {:tip}

## Incluir o Global Load Balancing no aplicativo

{: #add_glb}

Nesta seção, você usará o Global Load Balancer (GLB) no IBM Cloud Internet Services para gerenciar o tráfego entre múltiplos locais. O GLB utiliza um conjunto de origem que permite que o tráfego seja distribuído para múltiplas origens.

### Antes de criar um GLB, crie uma verificação de funcionamento para o GLB.

1. No aplicativo Cloud Internet Services, navegue para **Confiabilidade** > **Global Load Balancer** e, na parte inferior da página, clique em **Criar verificação de funcionamento**.
2. Insira o caminho que você deseja monitorar, por exemplo, `/`, e selecione um tipo (HTTP ou HTTPS). Geralmente, é possível criar um terminal de funcionamento dedicado. Clique em **Provisionar 1 instância**.
   ![Verificação de funcionamento](images/solution1/health_check.png)

### Depois disso, crie um conjunto de origem com duas origens.

1. Clique em **Criar conjunto**.
2. Insira um nome para o conjunto, selecione a verificação de funcionamento recém-criada e uma região que esteja próxima ao local do seu aplicativo node.js.
3. Insira um nome para a primeira origem e o nome do host para o aplicativo em Dallas `<your_app>.mybluemix.net`.
4. Da mesma forma, inclua outra origem com o endereço de origem apontando para o aplicativo em Londres `<your_app>.eu-gb.mybluemix.net`.
5. Clique em **Provisionar 1 instância**.
   ![Conjunto de origem](images/solution1/origin_pool.png)

### Criar um Global Load Balancer (GLB). 

1. Clique em **Criar balanceador de carga**.
2. Insira um nome para o Global Load Balancer. Esse nome também fará parte de sua URL do aplicativo universal (`http://<glb_name>.<your_domain_name>`), independentemente da localização.
3. Clique em **Incluir conjunto** e selecione o conjunto de origem recém-criado.
4. Clique em **Provisionar 1 instância**.
   ![Global Load Balancer](images/solution1/load_balancer.png)

Nesse estágio, o GLB é configurado, mas os aplicativos do Cloud Foundry ainda não estão prontos para responder às solicitações do nome de domínio do GLB configurado. Para concluir a configuração, você atualizará os aplicativos com rotas usando o domínio customizado.

## Configurar o domínio customizado e as rotas para seu aplicativo

{: #add_domain}

Nesta etapa, você mapeará o nome de domínio customizado para o terminal seguro para o local do {{site.data.keyword.Bluemix_notm}} no qual seu aplicativo está em execução.

1. Na barra de menus, clique em **Gerenciar** e, em seguida, **Conta**: [Conta](https://{DomainName}/account).
2. Na página da conta, navegue para as **Orgs do Cloud Foundry** do aplicativo e selecione **Domínios** na coluna Ações.
3. Clique em **Incluir um domínio** e insira seu nome de domínio customizado adquirido do escrivão.
4. Selecione o local correto e clique em **Salvar**.
5. Da mesma forma, inclua o nome de domínio customizado para Londres.
6. Retorne para a [Lista de recursos](https://{DomainName}/resources) do {{site.data.keyword.Bluemix_notm}}, navegue para **Apps do Cloud Foundry** e clique no aplicativo em Dallas, clique em **Rota** > **Editar rotas** e clique em **Incluir rota**.
   ![Incluir uma rota](images/solution1/ApplicationRoutes.png)
7. Insira o nome do host do GLB que você configurou anteriormente no campo **Inserir host (opcional)** e selecione o domínio customizado recém-incluído. Clique em **Salvar**.
8. Da mesma forma, configure o domínio e as rotas para o aplicativo em Londres.

Neste ponto, é possível visitar seu aplicativo com a URL `<glb_name>.<your_domain_name>` e o Global Load Balancer distribui automaticamente o tráfego para seus aplicativos multilocal. É possível verificar isso parando seu aplicativo em Dallas, mantendo o aplicativo em Londres ativo e acessando o aplicativo por meio do Global Load Balancer.

Embora isso funcione neste momento, já que configuramos a entrega contínua nas etapas anteriores, a configuração pode ser sobrescrita quando outra construção é acionada. Para tornar essas mudanças persistentes, volte para as cadeias de ferramentas e modifique o arquivo *manifest.yml*:

1. Na [Lista de recursos](https://{DomainName}/resources) do {{site.data.keyword.Bluemix_notm}}, navegue para **Apps do Cloud Foundry** e clique no aplicativo em Dallas, navegue para a **Visão geral** do Aplicativo e role para localizar **Visualizar a cadeia de ferramentas**.
2. Selecione o ladrilho Git em Código.
3. Selecione *manifest.yml*.
4. Clique em **Editar** e inclua rotas customizadas. Substitua as configurações originais de domínio e host por `Routes`.

   ```
   aplicativos: - caminho:.
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <glb_name>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}
  
5. Confirme as mudanças e certifique-se de que as construções para ambos os locais sejam bem-sucedidas.  

## Alternativa: mapear o domínio customizado para o domínio do sistema do IBM Cloud

É possível que você não deseje utilizar um Global Load Balancer na frente de seus aplicativos multilocal, mas precise mapear o nome de domínio customizado para o terminal seguro para o local do {{site.data.keyword.Bluemix_notm}} no qual seu aplicativo está em execução.

Com o aplicativo Cloud Internet Services, execute as etapas a seguir para configurar os registros `CNAME` para seu aplicativo:

1. No aplicativo Cloud Internet Services, navegue para **Confiabilidade** > **DNS**.
2. Selecione **CNAME** na lista suspensa **Tipo**, digite um alias para seu aplicativo no campo Nome e a URL do aplicativo no campo de nome de domínio. O aplicativo `<your_app>.mybluemix.net` em Dallas pode ser mapeado para um CNAME `<your_app>`.
3. Clique em **Incluir registro**. Alterne o PROXY para ON para aprimorar a segurança de seu aplicativo.
4. Da mesma forma, configure o registro `CNAME` para o terminal em Londres.
   ![Registros CNAME](images/solution1/cnames.png)

Ao usar outro domínio padrão diferente de `mybluemix.net`, como `cf.appdomain.cloud` ou `cf.cloud.ibm.com`, certifique-se de usar o [respectivo domínio do sistema](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#mapcustomdomain).
{:tip}

Se você estiver usando um provedor DNS diferente, as etapas para configurar o registro CNAME variam dependendo de seu provedor DNS. Por exemplo, se você estiver
usando o GoDaddy, siga a orientação [Ajuda de domínios](https://www.godaddy.com/help/add-a-cname-record-19236) a partir do GoDaddy.

Para que os aplicativos do Cloud Foundry sejam acessíveis por meio do domínio customizado, será necessário incluir o domínio customizado na [lista de domínios na organização do Cloud Foundry em que os aplicativos são implementados](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#updatingapps). Uma vez pronto, será possível incluir as rotas nos manifests do aplicativo:

   ```
   aplicativos: - caminho:.
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <your_app>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}

## Ligar certificado SSL ao seu aplicativo
{: #ssl}

1. Obtenha um certificado SSL. Por exemplo, é possível comprar em https://www.godaddy.com/web-security/ssl-certificate ou gerar um grátis em https://letsencrypt.org/.
2. Navegue para a **Visão geral** do Aplicativo > **Rotas** > **Gerenciar domínios**.
3. Clique no botão de upload do Certificado SSL e faça upload do certificado.
5. Acesse seu aplicativo com https em vez de http.

## Monitorar desempenho do aplicativo
{: #monitor}

Permite verificar o funcionamento de seu aplicativo multilocal.

1. No painel do aplicativo, selecione **Monitoramento**.
2. Clique em **Visualizar todos os testes**
   ![](images/solution1/alert_frequency.png)

O Availability Monitoring executa testes sintéticos de locais em todo o mundo, ininterruptamente, para detectar e corrigir proativamente problemas de desempenho antes de os usuários serem afetados. Se você configurou uma rota customizada para seu aplicativo, mude a definição de teste para acessar seu aplicativo por meio de seu domínio customizado.

## Remover recursos

* Excluir a cadeia de ferramentas
* Exclua os dois aplicativos do Cloud Foundry implementados nos dois locais
* Excluir o GLB, os conjuntos de origem e a verificação de funcionamento
* Excluir a configuração do DNS
* Excluir a instância de Serviços da Internet

## Conteúdo relacionado

[Incluindo um banco de dados Cloudant](https://{DomainName}/docs/services/Cloudant/tutorials?topic=cloudant-creating-an-ibm-cloudant-instance-on-ibm-cloud#creating-an-ibm-cloudant-instance-on-ibm-cloud)

[Ajuste automático de escala de aplicativos do Cloud Foundry](https://{DomainName}/docs/services/Auto-Scaling?topic=services/Auto-Scaling-get-started#get-started)

[Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
