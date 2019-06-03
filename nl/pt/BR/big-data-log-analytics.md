---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Logs de big data com análise de dados de fluxo e SQL
{: #big-data-log-analytics}

Neste tutorial, você construirá um pipeline de análise do log projetado para coletar, armazenar e analisar registros de log para suportar requisitos regulamentares ou auxiliar a descoberta de informações. Essa solução alavanca vários serviços disponíveis no {{site.data.keyword.cloud_notm}}: {{site.data.keyword.messagehub}}, {{site.data.keyword.cos_short}}, SQL Query e {{site.data.keyword.streaminganalyticsshort}}. Um programa o ajudará simulando a transmissão de mensagens de log do servidor da web de um arquivo estático para o {{site.data.keyword.messagehub}}.

Com o {{site.data.keyword.messagehub}}, o pipeline é escalado para receber milhões de registros de log de uma variedade de produtores. Aplicando o {{site.data.keyword.streaminganalyticsshort}}, os dados do log do podem ser inspecionados em tempo real para integrar processos de negócios. As mensagens de log também podem ser facilmente redirecionadas para armazenamento de longo prazo usando o {{site.data.keyword.cos_short}}, no qual os desenvolvedores, a equipe de suporte e os auditores podem trabalhar diretamente com dados usando o SQL Query.

Embora este tutorial se concentre na análise do log, ele é aplicável a outros cenários: os dispositivos IoT com armazenamento limitado podem similarmente transmitir mensagens para o {{site.data.keyword.cos_short}} ou os profissionais de marketing podem segmentar e analisar eventos do cliente ao longo de propriedades digitais com o SQL Query.
{:shortdesc}

## Objetivos

{: #objectives}

* Entender o sistema de mensagens de publicação-assinatura do Apache Kafka
* Armazenar dados do log para requisitos de auditoria e conformidade
* Monitorar logs para criar processos de manipulação de exceção
* Conduzir análise forense e estatística nos dados do log

## Serviços usados

{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:

* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/message-hub)
* [{{site.data.keyword.sqlquery_short}}](https://{DomainName}/catalog/services/sql-query)
* [{{site.data.keyword.streaminganalyticsshort}}](https://{DomainName}/catalog/services/streaming-analytics)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura

{: #architecture}

<p style="text-align: center;">

  ![Arquitetura](images/solution31/Architecture.png)
</p>

1. O aplicativo gera eventos de log para o {{site.data.keyword.messagehub}}
2. O evento de log é interceptado e analisado pelo {{site.data.keyword.streaminganalyticsshort}}
3. O evento de log é anexado a um arquivo CSV localizado no {{site.data.keyword.cos_short}}
4. O auditor ou a equipe de suporte emite a tarefa SQL
5. O SQL Query é executado no arquivo de log no {{site.data.keyword.cos_short}}
6. O conjunto de resultados é armazenado no {{site.data.keyword.cos_short}} e entregue ao auditor e à equipe de suporte

## Antes de Começar

{: #prereqs}

* [Instalar Git](https://git-scm.com/)
* [Instale a CLI do {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* [Instale o Node.js](https://nodejs.org)
* [Faça download do cliente Kafka 0.10.2.X](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)

## Criar serviços

{: #setup}

Nesta seção, você criará os serviços necessários para executar a análise de eventos de log gerados por seus aplicativos.

Esta seção usa a linha de comandos para criar instâncias de serviço. Como alternativa, é possível fazer o mesmo por meio da página de serviço no catálogo usando os links fornecidos.
{: tip}

1. Efetue login no {{site.data.keyword.cloud_notm}} por meio da linha de comandos e tenha como destino sua conta do Cloud Foundry. Consulte [Introdução à CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh ibmcloud login
    ```
    {: pre}
    ```sh ibmcloud target --cf
    ```
    {: pre}
2. Crie uma instância Lite do [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
    ```sh
    ibmcloud resource service-instance-create log-analysis-cos cloud-object-storage \
    lite global
    ```
    {: pre}
3. Crie uma instância Lite do [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create log-analysis-sql sql-query lite \
    us-south
    ```
    {: pre}
4. Crie uma instância Padrão do [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams).
    ```sh
    ibmcloud service create messagehub standard log-analysis-hub
    ```
    {: pre}

## Criar um tópico do sistema de mensagens e um depósito do {{site.data.keyword.cos_short}}

{: #topics}

Comece criando um tópico do {{site.data.keyword.messagehub}} e um depósito do {{site.data.keyword.cos_short}}. Os tópicos definem onde os aplicativos entregam mensagens em sistemas de mensagens de publicação/assinatura. Depois que as mensagens forem recebidas e processadas, elas serão armazenadas em um arquivo localizado em um depósito do {{site.data.keyword.cos_short}}.

1. Em seu navegador, acesse a instância de serviço `log-analysis-hub` por meio de [Recursos](https://{DomainName}/resources?search=log-analysis).
2. Clique no botão **+** para criar um tópico.
3. Insira o **Nome do tópico** `webserver` e clique no botão **Criar tópico**.
4. Clique em **Credenciais de serviço** e no botão **Nova credencial**.
5. No diálogo resultante, digite `webserver-flow` como o **Nome** e clique no botão **Incluir**.
6. Clique em **Visualizar credenciais** e copie as informações para um local seguro. Elas serão usadas na próxima seção.
7. De volta na [Lista de recursos](https://{DomainName}/resources?search=log-analysis), selecione a instância de serviço `log-analysis-cos`.
8. Clique em **Criar depósito**.
    * Insira um **Nome** exclusivo para o depósito.
    * Selecione **Região cruzada** para **Resiliência**.
    * Selecione **us-geo** como o **Local**.
    * Clique em **Criar depósito**.

## Criar uma fonte de fluxo do Streams

{: #streamsflow}

Nesta seção, você iniciará a configuração de um fluxo do Streams que recebe mensagens de log. O serviço {{site.data.keyword.streaminganalyticsshort}} é desenvolvido com o {{site.data.keyword.streamsshort}}, que pode analisar milhões de eventos por segundo, ativando os tempos de resposta de submilissegundos e a tomada de decisão instantânea.

1. Em seu navegador, acesse [Watson Data Platform](https://dataplatform.ibm.com).
2. Selecione o botão ou ladrilho **Novo projeto**, em seguida, o ladrilho **Básico** e clique em **OK**.
    * Insira o **Nome** `webserver-logs`.
    * A opção **Armazenamento** deve ser configurada como `log-analysis-cos`. Se não, selecione a instância de serviço.
    * Clique no botão **Criar**.
3. Na página resultante, selecione a guia **Configurações** e verifique o **Streams Designer** em **Ferramentas**. Conclua clicando no botão **Salvar**.
4. Clique no botão **Incluir no projeto**, em seguida, **Fluxo do Streams** na barra de navegação superior.
    * Clique em **Associar uma instância do IBM Streaming Analytics com um plano baseado em contêiner**.
    * Crie uma nova instância do {{site.data.keyword.streaminganalyticsshort}}, selecionando o botão de opções **Lite** e clicando em **Criar**. Não selecione Lite VM.
    * Forneça o **Nome do serviço** como `log-analysis-sa` e clique em **Confirmar**.
    * Digite o **Nome** do fluxo do Streams como `webserver-flow`.
    * Conclua clicando em **Criar**.
5. Na página resultante, selecione o ladrilho **{{site.data.keyword.messagehub}}**.
    * Clique em **Incluir conexão** e selecione sua instância do {{site.data.keyword.messagehub}} `log-analysis-hub`. Se você não vir sua instância listada, selecione a opção **IBM {{site.data.keyword.messagehub}}**. Insira manualmente os **Detalhes da conexão** que você obteve das **Credenciais de serviço** na seção anterior. **Nomeie** a conexão `webserver-flow`.
    * Clique em **Criar** para criar a conexão.
    * Selecione `webserver` na lista suspensa **Tópico**.
    * Selecione **Iniciar com a primeira nova mensagem** na lista suspensa **Deslocamento inicial**.
    * Clique em **Continuar**.
6. Deixe a página **Visualizar dados** aberta; ela será usada na próxima seção.

## Usando as ferramentas do console Kafka com o {{site.data.keyword.messagehub}}

{: #kafkatools}

O `webserver-flow` está atualmente inativo e aguardando mensagens. Nesta seção, você configurará as ferramentas de console do Kafka para trabalhar com o {{site.data.keyword.messagehub}}. As ferramentas de console do Kafka permitem que você produza mensagens arbitrárias por meio do terminal e envie-as para o {{site.data.keyword.messagehub}}, que acionará o `webserver-flow`.

1. Faça download e descompacte o arquivo ZIP do [cliente Kafka 0.10.2.X](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz).
2. Mude o diretório para `bin` e crie um arquivo de texto denominado `message-hub.config` com o conteúdo a seguir.
    ```sh
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER" password="PASSWORD";
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    ssl.protocol=TLSv1.2
    ssl.enabled.protocols=TLSv1.2
    ssl.endpoint.identification.algorithm=HTTPS
    ```
    {: pre}
3. Substitua `USER` e `PASSWORD` em seu arquivo `message-hub.config` pelos valores `user` e `password` vistos em **Credenciais de serviço** na seção anterior. Salve `message-hub.config`.
4. No diretório `bin`, execute o comando a seguir. Substitua `KAFKA_BROKERS_SASL` pelo valor `kafka_brokers_sasl` visto em **Credenciais de serviço**. Um exemplo é fornecido.
    ```sh
    ./kafka-console-producer.sh --broker-list KAFKA_BROKERS_SASL \
    --producer.config message-hub.config --topic webserver
    ```
    {: pre}
    ```sh
    ./kafka-console-producer.sh --broker-list \
    kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093 \
    --producer.config message-hub.config --topic webserver
    ```
5. A ferramenta de console do Kafka está aguardando entrada. Copie e cole a mensagem de log abaixo para o terminal. Pressione `enter` para enviar a mensagem de log para o {{site.data.keyword.messagehub}}. Observe que as mensagens enviadas também são exibidas na página **Visualizar dados** de `webserver-flow`.
    ```javascript { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
![Página de visualização](images/solution31/preview_data.png)

## Criar um destino de fluxo do Streams

{: #streamstarget}

Nesta seção, você concluirá a configuração do fluxo do Streams definindo um destino. O destino será usado para armazenar mensagens de log recebidas no depósito do {{site.data.keyword.cos_short}} criado anteriormente. O processo de armazenamento e anexação de mensagens de log recebidas em um arquivo será feito automaticamente pelo {{site.data.keyword.streaminganalyticsshort}}.

1. Na **Página de visualização** de `webserver-flow`, clique no botão **Continuar**.
2. Selecione o ladrilho **{{site.data.keyword.cos_full_notm}}** como um destino.
    * Clique em **Incluir conexão** e selecione `log-analysis-cos`.
    * Clique em **Criar**.
    * Insira o **Caminho do arquivo** `/YOUR_BUCKET_NAME/http-logs_%TIME.csv`. Substitua `YOUR_BUCKET_NAME` pelo que foi usado na primeira seção.
    * Selecione **csv** na lista suspensa **Formato**.
    * Marque a caixa de seleção **Linha de cabeçalho da coluna**.
    * Selecione **Tamanho do arquivo** na lista suspensa **Política de criação de arquivo**.
    * Configure o limite para 100 MB inserindo `102400` na caixa de texto **Tamanho do arquivo (KB)**.
    * Clique em **Continuar**.
3. Clique em **Salvar**.
4. Clique no botão de reprodução **>** para **Iniciar o fluxo do Streams**.
5. Depois que o fluxo for iniciado, novamente, envie múltiplas mensagens de log por meio da ferramenta de console do Kafka. É possível observar como as mensagens chegam visualizando o `webserver-flow` no Streams Designer.
    ```javascript { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
6. Retorne ao seu depósito no {{site.data.keyword.cos_short}}. Um novo arquivo `log.csv` existirá depois que mensagens suficientes tiverem inserido o fluxo ou o fluxo for reiniciado.

![webserver-flow](images/solution31/flow.png)

## Incluir comportamento condicional nos fluxos Streams

{: #streamslogic}

Até agora, o fluxo do Streams é um canal simples - mover mensagens do {{site.data.keyword.messagehub}} para o {{site.data.keyword.cos_short}}. Mais do que provável, as equipes desejarão saber os eventos de interesse em tempo real. Por exemplo, as equipes individuais podem se beneficiar de alertas quando eventos HTTP 500 (erro do aplicativo) ocorrem. Nesta seção, você incluirá uma lógica condicional no fluxo para identificar códigos HTTP 200 (OK) e não HTTP 200.

1. Use o botão de lápis para **Editar o fluxo do Streams**.
2. Crie um nó de filtro que manipule respostas HTTP 200.
    * Na paleta **Nós**, arraste o nó **Filtro** de **PROCESSAMENTO E ANÁLISE DE DADOS** para a tela.
    * Digite `OK` na caixa de texto de nome, que contém atualmente a palavra `Filter`.
    * Insira a instrução a seguir na área de texto **Expressão de condição**.
      ```sh
      responseCode == 200
      ```
      {: pre}
    * Com o mouse, desenhe uma linha da saída do nó **{{site.data.keyword.messagehub}}** (lado direito) para a entrada do nó **OK** (lado esquerdo).
    * Na paleta **Nós**, arraste o nó **Depurar** localizado em **DESTINOS** para a tela.
    * Conecte o nó **Depurar** ao nó **OK** desenhando uma linha entre os dois.
3. Repita o processo para criar um filtro `Not OK` usando os mesmos nós e a instrução de condição a seguir.
    ```sh
    responseCode >= 300
    ```
    {: pre}
4. Clique no botão de reprodução para **Salvar e executar o fluxo do Streams**.
5. Se solicitado, clique no link para **executar a nova versão**.

![Designer de fluxo](images/solution31/flow_design.png)

## Aumentando o carregamento de mensagem

{: #streamsload}

Para visualizar a manipulação condicional em seu fluxo do Streams, você aumentará o volume de mensagem enviado para o {{site.data.keyword.messagehub}}. O programa Node.js fornecido simula um fluxo realista de mensagens para o {{site.data.keyword.messagehub}} com base no tráfego para o servidor da web. Para demonstrar a escalabilidade de {{site.data.keyword.messagehub}} e {{site.data.keyword.streaminganalyticsshort}}, você aumentará o rendimento de mensagens de log.

Esta seção usa [node-rdkafka](https://www.npmjs.com/package/node-rdkafka). Consulte a página npmjs para obter instruções de resolução de problemas se a instalação do simulador falhar. Se os problemas persistirem, será possível pular para a próxima seção e fazer upload manualmente dos dados.

1. Faça download e descompacte o arquivo ZIP de log [Jul 01 to Jul 31, ASCII format, 20.7 MB gzip compressed](http://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz) da NASA.
2. Clone e instale o simulador de log do [IBM-Cloud no GitHub](https://github.com/IBM-Cloud/kafka-log-simulator).
    ```sh
    git clone https://github.com/IBM-Cloud/kafka-log-simulator.git
    ```
    {: pre}
3. Mude para o diretório do simulador e execute os comandos a seguir para configurar o simulador e produzir mensagens do evento de log. Substitua `LOGFILE` pelo arquivo transferido por download. Substitua `BROKERLIST` e `APIKEY` pelas **Credenciais de serviço** correspondentes usadas anteriormente. Um exemplo é fornecido.
    ```sh npm install
    ```
    ```sh
    npm run build
    ```
    ```sh
    node dist/index.js --file LOGFILE --parser httpd --broker-list BROKERLIST \
    --api-key APIKEY --topic webserver --rate 100
    ```
    {: pre}
    ```sh
    node dist/index.js --file /Users/ibmcloud/Downloads/NASA_access_log_Jul95 \
    --parser httpd --broker-list \
    "kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093" \
    --api-key Np15YZKN3SCdABUsOpJYtpue6jgJ7CwYgsoCWaPbuyFbdM4R \
    --topic webserver --rate 100
    ```
4. Em seu navegador, retorne para seu `webserver-flow` depois que o simulador começar a produzir mensagens.
5. Pare o simulador depois de um número desejado de mensagens ter passado pelas ramificações condicionais usando `control+C`.
6. Experimente com o ajuste de escala do {{site.data.keyword.messagehub}}, aumentando ou diminuindo o valor `--rate`.

O simulador atrasará o envio da próxima mensagem com base no tempo decorrido no log do servidor da web. A configuração de `--rate 1` envia eventos em tempo real. A configuração de `--rate 100` significa que para cada 1 segundo de tempo decorrido no log do servidor da web, um atraso de 10 ms entre as mensagens é usado.
{: tip}

![Carregamento de fluxo configurado para 10](images/solution31/flow_load_10.png)

## Investigando dados do log usando SQL Query

{: #sqlquery}

Dependendo do número de mensagens enviadas pelo simulador, o arquivo de log no {{site.data.keyword.cos_short}} certamente cresceu no tamanho do arquivo. Agora você agirá como um investigador respondendo a perguntas de auditoria ou conformidade, combinando o SQL Query com seu arquivo de log. O benefício de usar o SQL Query é que o arquivo de log está diretamente acessível; nenhuma transformação ou servidor de banco de dados adicional é necessário.

Se você preferir não aguardar que o simulador envie todas as mensagens de log, faça upload do [arquivo CSV completo](https://ibm.box.com/s/dycyvojotfpqvumutehdwvp1o0fptwsp) para o {{site.data.keyword.cos_short}} para iniciar imediatamente.
{: tip}

1. Acesse a instância de serviço `log-analysis-sql` na [Lista de recursos](https://{DomainName}/resources?search=log-analysis). Selecione **Abrir IU** para ativar o SQL Query.
2. Insira o SQL a seguir na área de texto **Digitar SQL aqui...**
    ```sql
    -- What are the top 10 web pages on NASA from July 1995?
    -- Which mission might be significant?
    SELECT REQUEST, COUNT(REQUEST)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE REQUEST LIKE '%.htm%'
    GROUP BY REQUEST
    ORDER BY 2 DESC
    LIMIT 10
    ```
    {: pre}
3. Recupere a URL de SQL do Objeto por meio do arquivo de logs.
    * Na [Lista de recursos](https://{DomainName}/resources?search=log-analysis), selecione a instância de serviço `log-analysis-cos`.
    * Selecione o depósito que você criou anteriormente.
    * Clique no menu overflow no arquivo `http-logs_TIME.csv` e selecione **URL de SQL do objeto**.
    * **Copie** a URL para a área de transferência.
4. Atualize a cláusula `FROM` com sua URL de SQL do Objeto e clique em **Executar**.
5. O resultado pode ser visto na guia **Resultado**. Embora algumas páginas - como a página inicial do Centro Espacial Kennedy - sejam esperadas, uma missão é bastante popular no momento.
6. Selecione a guia **Detalhes da consulta** para visualizar informações adicionais, como o local em que o resultado foi armazenado no {{site.data.keyword.cos_short}}.
7. Tente os pares de pergunta e resposta a seguir, incluindo-os individualmente na área de texto **Digite SQL aqui...**.
    ```sql
    -- Who are the top 5 viewers?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY HOST
    ORDER BY 2 DESC
    LIMIT 5
    ```
    {: pre}

    ```sql
    -- Which viewer has suspicious activity based on application failures?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 500
    GROUP BY HOST
    ORDER BY 2 DESC;
    ```
    {: pre}

    ```sql
    -- Which requests showed a page not found error to the user?
    SELECT DISTINCT REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 404
    ```
    {: pre}

    ```sql
    -- What are the top 10 largest files?
    SELECT DISTINCT REQUEST, BYTES
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE BYTES > 0
    ORDER BY CAST(BYTES as Integer) DESC
    LIMIT 10
    ```
    {: pre}

    ```sql
    -- What is the distribution of total traffic by hour?
    SELECT SUBSTRING(TIMESTAMP, 13, 2), COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY 1
    ORDER BY 1 ASC
    ```
    {: pre}

    ```sql
    -- Why did the previous result return an empty hour?
    -- Hint, find the malformed hostname.
    SELECT HOST, REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE SUBSTRING(TIMESTAMP, 13, 2) == ''
    ```
    {: pre}

As cláusulas FROM não são limitadas a um único arquivo. Use `cos://us-geo/YOUR_BUCKET_NAME/` para executar consultas SQL em todos os arquivos no depósito.
{: tip}

## Expandir o tutorial

{: #expand}

Parabéns, você construiu um pipeline de análise de log com o {{site.data.keyword.cloud_notm}}. Abaixo estão sugestões adicionais para aprimorar sua solução.

* Use destinos adicionais no Streams Designer para armazenar dados no [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) ou executar código no [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* Siga o tutorial [Construir um data lake usando o Object Storage](https://{DomainName}/docs/tutorials?topic=solution-tutorials-smart-data-lake#build-a-data-lake-using-object-storage) para incluir um painel para dados do log
* Integre sistemas adicionais ao {{site.data.keyword.messagehub}} usando o [{{site.data.keyword.appconserviceshort}}](https://{DomainName}/catalog/services/app-connect).

## Remover serviços

{: #removal}

Na [Lista de recursos](https://{DomainName}/resources?search=log-analysis), use o item de menu **Excluir** ou **Excluir serviço** no menu overflow para remover as instâncias de serviço a seguir.

* log-analysis-sa
* log-analysis-hub
* log-analysis-sql
* log-analysis-cos

## Conteúdo relacionado

{:related}

* [Apache Kafka](https://kafka.apache.org/)
