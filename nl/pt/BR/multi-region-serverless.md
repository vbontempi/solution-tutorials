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

# Implementar apps serverless em múltiplas regiões
{: #multi-region-serverless}

Este tutorial mostra como configurar o IBM Cloud Internet Services e o {{site.data.keyword.openwhisk_short}} para implementar apps serverless em múltiplas regiões.

As plataformas computacionais Serverless fornecem aos desenvolvedores uma maneira rápida de construir APIs sem servidores. O {{site.data.keyword.openwhisk}} suporta a geração automática de API de REST para ações, ações de transformação em terminais HTTP e a capacidade de ativar a autenticação de API segura. Esse recurso é útil não somente para expor APIs para consumidores externos, mas também para construir aplicativos de microsserviços.

O {{site.data.keyword.openwhisk_short}} está disponível em várias localizações do {{site.data.keyword.cloud_notm}}. Para aumentar a resiliência e reduzir a latência de rede, os aplicativos podem implementar seu back-end em várias localizações. Em seguida, com o IBM Cloud Internet Services (CIS), os desenvolvedores podem expor um ponto de entrada único encarregado de distribuir o tráfego para o back-end funcional mais próximo.

## Objetivos
{: #objectives}

* Implementar ações do {{site.data.keyword.openwhisk_short}}.
* Expor ações por meio do {{site.data.keyword.APIM}} com um domínio customizado.
* Distribua o tráfego entre múltiplos locais com o Cloud Internet Services.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
* [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-svcs)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

O tutorial considera um aplicativo da web público com um back-end implementado com o {{site.data.keyword.openwhisk_short}}. Para reduzir a latência de rede e evitar indisponibilidade, o aplicativo é implementado em múltiplos locais. Dois locais são configurados no tutorial.

<p style="text-align: center;">

  ![Arquitetura](images/solution44-multi-region-serverless/Architecture.png)
</p>

1. Os usuários acessam o aplicativo. A solicitação passa pelo Internet Services.
2. O Internet Services redireciona os usuários para o back-end de API funcional mais próximo.
3. O {{site.data.keyword.cloudcerts_short}} fornece a API com seu certificado SSL. O tráfego é criptografado de ponta a ponta.
4. A API é implementada com o {{site.data.keyword.openwhisk_short}}.

## Antes de Começar
{: #prereqs}

1. O Cloud Internet Services requer que você tenha um domínio customizado para que possa configurar o DNS para que esse domínio aponte para os servidores de nomes do Cloud Internet Services. Se você não tiver um domínio, será possível comprar um de um registrador, como [godaddy.com](http://godaddy.com).
1. Instale todas as ferramentas necessárias de linha de comandos (CLI) [seguindo estas etapas](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).

## Configurar um domínio customizado

A primeira etapa é criar uma instância do IBM Cloud Internet Services (CIS) e apontar seu domínio customizado para servidores de nomes do CIS.

1. Navegue para o [Internet Services](https://{DomainName}/catalog/services/internet-services) no catálogo do {{site.data.keyword.Bluemix_notm}}.
1. Configure o nome do serviço e clique em **Criar** para criar uma instância do serviço. É possível usar quaisquer planos de precificação para este tutorial.
1. Quando a instância de serviço for provisionada, configure seu nome de domínio clicando em **Vamos começar** e clique em **Incluir domínio**.
1. Clique em **Próxima etapa**. Quando os servidores de nomes forem designados, configure seu registrador ou provedor de nome de domínio para usar os servidores de nomes listados.
1. Depois de ter configurado seu registrador ou o provedor de DNS, pode ser necessário até 24 horas para que as mudanças entrem em vigor.

   Quando o status do domínio na página Visão geral muda de *Pendente* para *Ativo*, é possível usar o comando `dig <your_domain_name> ns` para verificar se os novos servidores de nomes entraram em vigor.
   {:tip}

### Obter um certificado para o domínio customizado

A exposição das ações do {{site.data.keyword.openwhisk_short}} por meio de um domínio customizado exigirá uma conexão HTTPS segura. Você deve obter um certificado SSL para o domínio e subdomínio que planeja usar com o back-end sem servidor. Supondo um domínio como *mydomain.com*, as ações poderiam ser hospedadas em *api.mydomain.com*. O certificado precisará ser emitido para *api.mydomain.com*.

É possível obter certificados SSL grátis em [Vamos criptografar](https://letsencrypt.org/). Durante o processo, talvez seja necessário configurar um registro de DNS do tipo TXT na interface de DNS do Cloud Internet Services para provar que você é o proprietário do domínio.
{:tip}

Depois de ter obtido o certificado SSL e a chave privada para seu domínio, certifique-se de convertê-los para o formato [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Para converter um Certificado no formato PEM:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
1. Para converter uma Chave privada no formato PEM:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Importar o certificado para um repositório central

1. Crie uma instância do [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) em uma localização suportada.
1. No painel de serviço, use **Importar certificado**:
   * Configure **Nome** para o subdomínio e domínio customizados, como *api.mydomain.com*.
   * Procure o **Arquivo de certificado** no formato PEM.
   * Procure o **Arquivo de chave privado** no formato PEM.
   * **Importar**.

## Implementar ações em múltiplos locais

Nesta seção, você criará ações, as exporá como uma API e mapeará o domínio customizado para a API com um certificado SSL armazenado no {{site.data.keyword.cloudcerts_short}}.

<p style="text-align: center;">

  ![Arquitetura de API](images/solution44-multi-region-serverless/api-architecture.png)
</p>

A ação **doWork** implementa uma de suas operações de API. A ação **healthz** será usada posteriormente na verificação se sua API estiver funcional. Poderia ser um no-op simplesmente retornando *OK* ou poderia fazer uma verificação mais complexa como executar ping nos bancos de dados ou outros serviços críticos requeridos por sua API.

As três seções a seguir precisarão ser repetidas para cada local no qual você deseja hospedar o back-end do aplicativo. Para este tutorial, é possível selecionar *Dallas (us-sul)* e *Londres (eu-gb)* como destinos.

### Definir ações

1. Acesse [{{site.data.keyword.openwhisk_short}} / Ações](https://{DomainName}/openwhisk/actions).
1. Alterne para o local de destino e selecione uma organização e espaço no qual implementar as ações.
1. Crie uma ação
   1. Configure **Nome** como **doWork**.
   1. Configure **Pacote de fechamento** como **default**.
   1. Configure **Tempo de execução** para a versão mais recente do **Node.js**.
   1. **Criar**.
1. Mude o código de ação para:
   ```js
   function main(params) {
     msg = "Hello, " + params.name + " from " + params.place;
     return { greeting:  msg, host: params.__ow_headers.host };
   }
   ```
   {: codeblock}
1. **Salvar**
1. Crie outra ação para ser usada como verificação de funcionamento para nossa API:
   1. Configure **Nome** como **healthz**.
   1. Configure **Pacote de fechamento** como **default**.
   1. Configure **Tempo de execução** como o **Node.js** mais recente.
   1. **Criar**.
1. Mude o código de ação para:
   ```js
   function main(params) {
     return { ok: true };
   }
   ```
   {: codeblock}
1. **Salvar**

### Expor as ações com uma API gerenciada

A próxima etapa envolve a criação de uma API gerenciada para expor suas ações.

1. Acesse [{{site.data.keyword.openwhisk_short}}/API](https://{DomainName}/openwhisk/apimanagement).
1. Crie uma nova API do {{site.data.keyword.openwhisk_short}} gerenciada:
   1. Configure **Nome da API** como **API do app**.
   1. Configure **Caminho base** como **/api**.
1. Crie uma operação:
   1. Configure **Caminho** como **/do**.
   1. Configure **Verbo** para **GET**.
   1. Configure **Pacote** para **padrão**.
   1. Configure **Ação** como **doWork**.
   1. ** Criar **
1. Crie outra operação:
   1. Configure **Caminho** como **/healthz**.
   1. Configure **Verbo** para **GET**.
   1. Configure **Pacote** para **padrão**.
   1. Configure **Ação** como **healthz**.
   1. ** Criar **
1. **Salve** a API

### Configure o domínio customizado para a API gerenciada

A criação de uma API gerenciada fornece um terminal padrão como `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`. Nesta seção, você configurará esse terminal para ser capaz de manipular solicitações vindas de seu subdomínio customizado, o domínio que mais tarde será configurado no IBM Cloud Internet Services.

1. Acesse [APIs / Domínios customizados](https://{DomainName}/apis/domains).
1. No seletor **Região**, selecione o local de destino.
1. Localize o domínio customizado vinculado à organização e ao espaço no qual você criou as ações e a API gerenciada. Clique em **Mudar configurações** no menu Ação.
1. Anote o valor **Default domain / alias**.
1. Marque **Aplicar domínio customizado**
   1. Configure **Nome de domínio** para o domínio que você usará com o CIS Global Load Balancer, como *api.mydomain.com*.
   1. Selecione a instância {{site.data.keyword.cloudcerts_short}} que contém o certificado.
   1. Selecione o certificado para o domínio.
1. Acesse o painel de sua instância do **Cloud Internet Services**, em **Confiabilidade/DNS**, crie um novo **Registro TXT do DNS**:
   1. Configure **Nome** para seu subdomínio customizado, como **api**.
   1. Configure **Conteúdo** para o **Domínio/alias padrão**
   1. Salve o registro
1. Salve as configurações de domínio customizado. O diálogo verificará a existência do registro TXT do DNS.

   Se o registro TXT não for localizado, talvez seja necessário esperar que ele seja propagado e tentar salvar novamente as configurações. O registro TXT do DNS poderá ser removido quando as configurações tiverem sido aplicadas.
   {: tip}

Repita as seções anteriores para configurar mais locais.

## Distribuir tráfego entre locais

**Neste estágio, você tem ações de configuração em múltiplos locais**, mas não há nenhum ponto de entrada único para atingi-las. Nessa seção, você configurará um Global Load Balancer (GLB) para distribuir o tráfego entre os locais.

<p style="text-align: center;">

  ![Arquitetura do global load balancer](images/solution44-multi-region-serverless/glb-architecture.png)
</p>

### Criar uma verificação de funcionamento

Os Serviços da Internet chamarão regularmente esse terminal para verificar o funcionamento do back-end.

1. Acesse o painel de sua instância do IBM Cloud Internet Services.
1. Em **Confiabilidade/Global Load Balancers**, crie uma verificação de funcionamento:
   1. Configure **Tipo de monitor** como **HTTPS**.
   1. Configure **Caminho** como **/api/healthz**.
   1. **Provisionar o recurso**.

### Criar conjuntos de origem

Criando um conjunto por local, é possível configurar posteriormente rotas geográficas em seu global load balancer para redirecionar os usuários para o local mais próximo. Outra opção seria criar um único conjunto com todos os locais e deixar que o balanceador de carga percorra as origens no conjunto.

Para cada local:
1. Crie um conjunto de origem.
1. Configure **Nome** como **app-&lt;location&gt;**, como _app-Dallas_.
1. Selecione a verificação de funcionamento criada antes.
1. Configure **Região de verificação de funcionamento** para uma região próxima ao local em que o {{site.data.keyword.openwhisk_short}} está implementado.
1. Configure **Nome de origem** como **app-&lt;location&gt;**.
1. Configure **Endereço de origem** para o domínio/alias padrão para a API gerenciada (como _5d3ffd1eb6.us-south.apiconnect.appdomain.cloud_).
1. **Provisionar o recurso**.

### Criar um global load balancer

1. Crie um balanceador de carga.
1. Configure **Nome do host do balanceador** como **api.mydomain.com**.
1. Inclua os conjuntos de origem regional.
1. **Provisionar o recurso**.

Depois de algum tempo, acesse `https://api.mydomain.com/api/do?name=John&place=Earth`. Isso deve responder com a função em execução no primeiro conjunto funcional.

### Testar o failover

Para testar o failover, uma verificação de funcionamento do conjunto deve falhar para que o GLB seja redirecionável para o próximo conjunto funcional. Para simular uma falha, é possível modificar a função de verificação de funcionamento para fazer com que ela falhe.

1. Acesse [{{site.data.keyword.openwhisk_short}} / Ações](https://{DomainName}/openwhisk/actions).
1. Selecione o primeiro local configurado no GLB.
1. Edite a função `healthz` e mude sua implementação para `throw new Error()`.
1. Salve.
1. Aguarde até que a verificação de funcionamento seja executada para esse conjunto de origem.
1. Obtenha `https://api.mydomain.com/api/do?name=John&place=Earth` novamente, agora ele deve redirecionar para a outra origem funcional.
1. Reverta as mudanças de código para voltar a uma origem funcional.

## Remover recursos
{: #removeresources}

### Remover recursos CIS

1. Remova o GLB.
1. Remova os conjuntos de origem.
1. Remova as verificações de funcionamento.

### Remover ações

1. Remover [APIs](https://{DomainName}/openwhisk/apimanagement)
1. Remover [ações](https://{DomainName}/openwhisk/actions)

## Conteúdo relacionado
{: #related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Clusters Kubernetes multiregion resilientes e seguros com o Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)
