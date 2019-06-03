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

# Acelerar a entrega de arquivos estáticos usando um CDN
{: #static-files-cdn}

Este tutorial conduz você em como hospedar e entregar ativos de website (imagens, vídeos, documentos) e o conteúdo gerado pelo usuário em um {{site.data.keyword.cos_full_notm}} e como usar um [{{site.data.keyword.cdn_full}} (CDN)](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai) para entrega rápida e segura para usuários em todo o mundo.

Os aplicativos da web têm tipos diferentes de conteúdo: conteúdo HTML, imagens, vídeos, folhas de estilo em cascata, arquivos JavaScript, conteúdo gerado pelo usuário. Alguns conteúdos mudam frequentemente, outros não muito, alguns são acessados muito frequentemente por muitos usuários, outros ocasionalmente. À medida que o público para o aplicativo cresce, você pode desejar transferir esses conteúdos para outro componente, liberando recursos para seu aplicativo principal. Você também pode desejar que esses conteúdos sejam entregues por meio de um local próximo aos usuários do aplicativo, onde quer que estejam no mundo.

Há várias razões pelas quais você usaria uma Rede de Entrega de Conteúdo nestas situações:
* o CDN armazenará em cache o conteúdo, puxando o conteúdo da origem (seus servidores) somente se ele não estiver disponível em seu cache ou se tiver expirado;
* com múltiplos data centers em todo o mundo, o CDN servirá o conteúdo em cache pelo local mais próximo para seus usuários;
* executando em um domínio diferente do seu aplicativo principal, o navegador será capaz de carregar mais conteúdo em paralelo - a maioria dos navegadores tem um limite no número de conexões por nome do host.

## Objetivos
{: #objectives}

* Fazer upload de arquivos para um depósito do {{site.data.keyword.cos_full_notm}}.
* Tornar o conteúdo globalmente disponível com uma Rede de Entrega de Conteúdo (CDN).
* Expor arquivos usando um aplicativo da web do Cloud Foundry.

## Serviços usados
{: #services}

Este tutorial usa os produtos a seguir:
   * [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
   * [{{site.data.keyword.cdn_full}}](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura

<p style="text-align: center;">
![Arquitetura](images/solution3/Architecture.png)
</p>

1. O usuário acessa o aplicativo
2. O aplicativo inclui conteúdo distribuído por meio de uma Rede de Entrega de Conteúdo
3. Se o conteúdo não estiver disponível no CDN ou tiver expirado, o CDN puxará o conteúdo da origem.

## Antes de Começar
{: #prereqs}

Entre em contato com o usuário principal de sua conta de Infraestrutura para obter as permissões a seguir:
   * Gerenciar Conta CDN
   * Gerenciar armazenamento
   * Chave de API

Essas permissões são necessárias para que seja possível visualizar e usar os serviços de Armazenamento e CDN.

## Obter o código do aplicativo da web
{: #get_code}

Vamos considerar um aplicativo da web simples com diferentes tipos de conteúdo, como imagens, vídeos e folhas de estilo em cascata. Você armazenará o conteúdo em um depósito de armazenamento e configurará o CDN para usar o depósito como sua origem.

Para iniciar, recupere o código do aplicativo:

   ```sh
   git clone https://github.com/IBM-Cloud/webapp-with-cos-and-cdn
   ```
  {: pre}

## Criar um Object Storage
{: #create_cos}

O {{site.data.keyword.cos_full_notm}} fornece armazenamento em nuvem flexível, com custo reduzido e escalável para dados não estruturados.

1. Acesse o [catálogo](https://{DomainName}/catalog/) no console e selecione [**Object Storage**](https://{DomainName}/catalog/services/cloud-object-storage) na seção Armazenamento.
2. Crie uma nova instância do {{site.data.keyword.cos_full_notm}}
4. No painel de serviço, clique em **Criar depósito**.
5. Configure um nome de depósito exclusivo, como `username-mywebsite` e clique em **Criar**. Evite pontos (.) no nome do depósito.

## Fazer upload de arquivos para um depósito
{: #upload}

Nesta seção, você usará a ferramenta de linha de comandos **curl** para fazer upload de arquivos no depósito.

1. Efetue login no {{site.data.keyword.Bluemix_notm}} por meio da CLI e obtenha um token do IBM Cloud IAM.
   ```sh
   ibmcloud login
   ibmcloud iam oauth-tokens
   ```
   {: pre}
2. Copie o token da saída do comando na etapa anterior.
   ```
   IAM token:  Bearer <token>
   ```
   {: screen}
3. Configure o valor do nome do token e do depósito para uma variável de ambiente para acesso fácil.
   ```sh
   export IAM_TOKEN=<REPLACE_WITH_TOKEN>
   export BUCKET_NAME=<REPLACE_WITH_BUCKET_NAME>
   ```
   {: pre}
4. Faça upload dos arquivos denominados `a-css-file.css`, `a-picture.png` e `a-video.mp4` do diretório de conteúdo do código do aplicativo da web que você transferiu por download anteriormente. Fazer upload dos arquivos para a raiz do depósito.
  ```sh
   cd content
  ```
  {: pre}
  ```sh
   curl -X "PUT" \ "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-picture.png" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: image/png" \
        -T a-picture.png
  ```
  {: pre}
  ```sh
   curl -X "PUT" \ "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-css-file.css" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: text/css" \
        -T a-css-file.css
  ```
  {: pre}
  ```sh
   curl -X "PUT" \ "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-video.mp4" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: video/mp4" \
        -T a-video.mp4
  ```
  {: pre}
5. Visualize seus arquivos em seu painel.
   ![](images/solution3/Buckets.png)
6. Acesse os arquivos por meio de seu navegador usando um link semelhante ao exemplo a seguir:

   `http://s3-api.us-geo.objectstorage.softlayer.net/YOUR_BUCKET_NAME/a-picture.png`

## Tornar os arquivos globalmente disponíveis com um CDN

Nesta seção, você criará um serviço CDN. O serviço CDN distribui o conteúdo onde ele é necessário. Na primeira vez que o conteúdo é solicitado, ele é puxado do servidor host (seu depósito no {{site.data.keyword.cos_full_notm}}) para a rede e permanece lá para que outros usuários o acessem rapidamente sem a latência de rede para atingir o servidor host novamente.

### Criar uma instância do CDN

1. Acesse o catálogo no console e selecione **Rede de Entrega de Conteúdo** na seção Rede. Esse CDN é desenvolvido com Akamai. Clique em **Criar**.
2. No próximo diálogo, configure o **Nome de host** para o CDN para seu domínio customizado. Embora você configure um domínio customizado, ainda é possível acessar o conteúdo do CDN por meio do CNAME fornecido pela IBM. Portanto, se você não planejar usar o domínio customizado, será possível configurar um nome arbitrário.
3. Configure o prefixo **CNAME customizado** para um valor exclusivo.
4. Em seguida, em **Configurar sua origem**, selecione **Object Storage** para configurar o CDN para COS.
5. Configure o **Terminal** para o terminal de API do depósito, como *s3-api.us-geo.objectstorage.softlayer.net*.
6. Deixe **Cabeçalho do host** e **Caminho** vazios. Configure **Nome do depósito** como *your-bucket-name*.
7. Ative as portas HTTP (80) e HTTPS (443).
8. Para **Certificado SSL**, selecione *Certificado SAN DV* se você desejar usar um domínio customizado. Além disso, para acessar o armazenamento por meio do CNAME, selecione a opção **Certificado curinga*.
9. Clique em **Criar**.

### Acessar seu conteúdo por meio do CNAME do CDN

1. Selecione a instância do CDN [na lista](https://{DomainName}/classic/network/cdn).
2. Se você selecionou anteriormente *Certificado SAN DV*, será solicitado que forneça a validação de domínio quando a configuração inicial for concluída. Siga as etapas mostradas ao clicar em **Visualizar validação de domínio**.
3. O painel **Detalhes** mostra o **Nome do host** e o **CNAME** para seu CDN.
4. Acesse seu arquivo com `https://your-cdn-cname.cdnedge.bluemix.net/a-picture.png` ou, se você estiver usando um domínio customizado, `https://your-cdn-hostname/a-picture.png`. Se omitir o nome do arquivo, você deverá ver o S3 ListBucketResult no lugar.

## Implementar o aplicativo do Cloud Foundry

O aplicativo contém uma página da web public/index.html que inclui referências aos arquivos agora hospedados no {{site.data.keyword.cos_full_notm}}. O `app.js` de back-end entrega essa página da web e substitui um item temporário pelo local real de seu CDN. Dessa forma, todos os ativos que são usados pela página da web são entregues pelo CDN.

1. Em um terminal, vá para o diretório no qual o código foi retirado.
   ```
   cd webapp-with-cos-and-cdn
   ```
   {: pre}
2. Envie por push o aplicativo sem iniciá-lo.
   ```
   ibmcloud cf push --no-start
   ```
   {: pre}
3. Configure a variável de ambiente CDN_NAME para que o app possa referenciar o conteúdo do CDN.
   ```
   ibmcloud cf set-env webapp-with-cos-and-cdn CDN_CNAME your-cdn.cdnedge.bluemix.net
   ```
   {: pre}
4. Inicie o app.
   ```
   ibmcloud cf start webapp-with-cos-and-cdn
   ```
   {: pre}
5. Acesse o app com seu navegador da web, a folha de estilo de página, uma figura e um vídeo são carregados por meio do CDN.

![](images/solution3/Application.png)

O uso de um CDN com o {{site.data.keyword.cos_full_notm}} é uma combinação poderosa que permite hospedar arquivos e entregá-los aos usuários de todo o mundo. Também é possível usar o {{site.data.keyword.cos_full_notm}} para armazenar quaisquer arquivos que seus usuários façam upload em seu aplicativo.

## Remover recursos

* Excluir o aplicativo do Cloud Foundry
* Excluir o serviço de Rede de Entrega de Conteúdo
* Exclua o serviço ou depósito do {{site.data.keyword.cos_full_notm}}

## Conteúdo relacionado

[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)

[Gerenciar acesso ao {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-getting-started-with-iam#getting-started-with-iam)

[Introdução ao CDN](https://{DomainName}/docs/infrastructure/CDN?topic=CDN-getting-started#getting-started)
