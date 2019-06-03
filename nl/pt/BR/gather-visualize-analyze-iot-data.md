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

# Reunir, visualizar e analisar dados do IoT
{: #gather-visualize-analyze-iot-data}
Este tutorial conduz você na configuração de um dispositivo IoT, reunindo dados no {{site.data.keyword.iot_short_notm}}, explorando dados e criando visualizações e, em seguida, usando serviços avançados de aprendizado de máquina para analisar dados e detectar anomalias nos dados históricos.
{:shortdesc}

## Objetivos
{: #objectives}

* Configurar o IoT Simulator.
* Enviar dados de coleção para o {{site.data.keyword.iot_short_notm}}.
* Criar visualizações.
* Analisar os dados gerados pelo dispositivo e detectar anomalias.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.iot_full}}](https://{DomainName}/catalog/services/internet-of-things-platform)
* [Aplicativo Node.js ](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience) com serviço Spark e {{site.data.keyword.cos_full_notm}}
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)


Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

   ![](images/solution16/Architecture.png)
</p>

* Os dispositivos enviam dados do sensor para o {{site.data.keyword.iot_full}} usando o protocolo MQTT
* Os dados históricos são exportados para um banco de dados {{site.data.keyword.cloudant_short_notm}}
* O {{site.data.keyword.DSX_short}} puxa dados desse banco de dados
* Os dados são analisados e visualizados por meio de um bloco de notas do Jupyter

## Antes de Começar
{: #prereqs}

[{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - Execute o script para instalar o ibmcloud cli e os plug-ins necessários

## Criar o IoT Platform
{: #iot_starter}

Para começar, você criará o serviço Internet of Things Platform - O hub que pode gerenciar dispositivos, conectar de forma segura e **coletar dados** e disponibilizar os dados históricos para visualizações e aplicativos.

1. Acesse o [**Catálogo do {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e selecione [**Internet of Things Platform**](https://{DomainName}/catalog/services/internet-of-things-platform) na seção **Internet of Things**.
2. Insira `IoT demo hub` como o nome do serviço, clique em **Criar** e **Ativar** o painel.
3. No menu lateral, selecione **Segurança > Segurança de conexão** e escolha **TLS opcional** em **Regra padrão** > **Nível de segurança** e clique em **Salvar**.
4. No menu lateral, selecione **Dispositivos** > **Tipos de dispositivo** e **+ Incluir tipo de dispositivo**.
5. Insira `simulator` como o **Nome** e clique em **Avançar** e **Pronto**.
6. Em seguida, clique em **Registrar dispositivos**
7. Escolha `simulator` para **Selecionar tipo de dispositivo existente** e, em seguida, insira `phone` para **ID do dispositivo**.
8. Clique em **Avançar** até que a tela **Segurança do dispositivo** (na guia Segurança) seja exibida.
9. Insira um valor para o **Token de autenticação**, por exemplo: `myauthtoken` e clique em **Avançar**.
10. Depois de clicar em **Pronto**, suas informações de conexão serão exibidas. Mantenha essa guia aberta.

O IoT Platform está agora configurado para iniciar o recebimento de dados. Os dispositivos precisarão enviar seus dados para o IoT Platform com o Tipo de dispositivo, ID e Token especificados.

## Criar simulador de dispositivo
{: #confignodered}
Em seguida, você implementará um aplicativo da web Node.js e o visitará em seu telefone, o qual se conectará e enviará o acelerômetro do dispositivo e os dados de orientação para o IoT Platform.

1. Clone o repositório Github:
   ```bash
   git clone https://github.com/IBM-Cloud/iot-device-phone-simulator
   cd iot-device-phone-simulator
   ```
2. Abra o código em um IDE de sua escolha e mude os valores `name` e `host` no arquivo **manifest.yml** para um valor exclusivo.
3. Envie por push o aplicativo para o {{site.data.keyword.Bluemix_notm}}.
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push
   ```
4. Em alguns minutos, seu aplicativo será implementado e você deverá ver uma URL semelhante a `<UNIQUE_NAME>.mybluemix.net`
5. Visite essa URL em seu telefone usando um navegador.
6. Insira as informações de conexão de sua guia do Painel do IoT em **Credenciais do dispositivo** e clique em **Conectar**.
7. Seu telefone iniciará a transmissão de dados. De volta à **guia IBM {{site.data.keyword.iot_short_notm}}**, verifique se há novas entradas na seção **Eventos recentes**.
   ![](images/solution16/recent_events_with_phone.png)

## Exibir dados ativos no IBM {{site.data.keyword.iot_short_notm}}
{: #createcards}
Em seguida, você criará uma placa e cartões para exibir dados do dispositivo no painel. Para obter mais informações sobre as placas e
os cartões, veja [Visualizando dados em tempo real usando placas e cartões](https://console.ng.bluemix.net/docs/services/IoT/data_visualization.html).

### Crie uma placa
{: #createboard}

1. Abra o **Painel do IBM {{site.data.keyword.iot_short_notm}}**.
2. Selecione **Placas** no menu esquerdo e, em seguida, clique em **Criar nova placa**.
3. Insira um nome para a placa, `Simuladores` como um exemplo, e clique em **Avançar** e, em seguida, **Enviar**.  
4. Selecione a placa que você acabou de criar para abri-la.

### Exibir dados do dispositivo
{: #cardtemp}
1. Clique em **Incluir novo cartão** e, em seguida, selecione o tipo de cartão **Gráfico de linha**, que está localizado na seção Dispositivos.
2. Selecione o seu dispositivo na lista e, em seguida, clique em **Avançar**.
3. Clique em **Conectar novo conjunto de dados**.
4. Na página Criar cartão de valor, selecione ou insira os valores a seguir e clique em **Avançar**.
   - Evento: sensorData
   - Propriedade: ob
   - Nome: OrientationBeta
   - Tipo: Valor flutuante
   - Mín: -180
   - Máx: 180
5. Na página Visualização de cartão, selecione **L** para o tamanho do gráfico de linha e clique em **Avançar** > **Enviar**
6. O cartão aparece no painel e inclui um gráfico de linha dos dados de temperatura em tempo real.
7. Use seu navegador do telefone celular para ativar o simulador novamente e incline lentamente o telefone para frente e para trás.
8. De volta à **guia IBM {{site.data.keyword.iot_short_notm}}**, você deverá ver o gráfico sendo atualizado.
   ![](images/solution16/board.png)

## Armazenar dados históricos no {{site.data.keyword.cloudant_short_notm}}
1. Acesse o [**Catálogo do {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e crie um novo [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) denominado `iot-db`.
2. Em **Conexões**:
   1. **Criar conexão**
   1. Selecione o local, a organização e o espaço do Cloud Foundry nos quais um alias para o serviço {{site.data.keyword.cloudant_short_notm}} deve ser criado.
   1. Expanda o nome de espaço na tabela **Local de conexão** e use o botão **Conectar** ao lado de **iot demo hub** para criar um alias para o serviço {{site.data.keyword.cloudant_short_notm}} nesse espaço. 
   1. Conecte e remonte o app.
3. Abra o **Painel do IBM {{site.data.keyword.iot_short_notm}}**.
4. Selecione **Extensões** no menu à esquerda e, em seguida, clique em **Configuração** em **Armazenamento de dados históricos**.
5. Selecione o banco de dados {{site.data.keyword.cloudant_short_notm}} `iot-db`.
6. Insira `devicedata` para **Nome do banco de dados** e clique em **Pronto**.
7. Uma nova janela deve carregar o prompt para autorização. Se você não vir esta janela, desative seu bloqueador de pop-up e atualize a página.

Os dados do dispositivo são agora salvos no {{site.data.keyword.cloudant_short_notm}}. Depois de alguns minutos, ative o painel do {{site.data.keyword.cloudant_short_notm}} para ver seus dados.

![](images/solution16/cloudant.png)

## Detectar anomalias usando o Machine Learning
{: #data_experience}

Nesta seção, você usará o Jupyter Notebook que está disponível no serviço IBM {{site.data.keyword.DSX_short}} para carregar seus dados móveis históricos e detectar anomalias usando o z-score. O *z-score* é uma pontuação padrão que indica quantos desvios padrão um elemento tem da média

### Criar um novo projeto
1. Acesse o [**Catálogo do {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e, em **IA**, selecione [**{{site.data.keyword.DSX_short}}**](https://{DomainName}/catalog/services/data-science-experience).
2. **Crie** o serviço e ative seu painel clicando em **Introdução**
3. Crie um Projeto > Selecione **Ciência de dados** > Clique em **Criar projeto** e insira `Detectar anomalia` como o **Nome** do projeto.
4. Deixe a caixa de seleção **Restringir quem pode ser um colaborador** desmarcada, visto que não há dados confidenciais.
5. Em **Definir armazenamento**, clique em **Incluir** e escolha um serviço **Cloud Object Storage** existente ou crie um novo (selecione plano **Lite** > Criar). Pressione **Atualizar** para ver o serviço criado.
6. Clique em **Criar**. Seu novo projeto é aberto e é possível começar a incluir recursos nele.

### Conexão com o {{site.data.keyword.cloudant_short_notm}} para dados

1. Clique em **Ativos** > **+ Incluir no projeto** > **Conexão**  
2. Selecione o **iot-db**{{site.data.keyword.cloudant_short_notm}} no qual os dados do dispositivo são armazenados.
3. Faça verificação cruzada das **Credenciais** e, em seguida, clique em **Criar**.

### Criar um serviço Apache Spark

1. Clique em **Serviços** na barra de navegação superior > Serviços de cálculo.
2. Clique em **Incluir serviço**.
   1. Clique em **Incluir** em **Apache Spark**.
   1. Selecione o plano **Lite**.
   1. Clique em **Criar**.
3. Selecione uma organização e um espaço, mude o nome do serviço, se desejar, e **Confirme**.
1. Navegue para o projeto `Detectar anomalia` por meio de **Projetos**.
1. Nas **Configurações**, role para **Serviços associados**.
1. Clique em **Incluir serviço**, selecione **Spark**.
1. Selecione a instância do **Apache Spark** criada anteriormente.

### Criar um bloco de notas do Jupyter (ipynb)
1. Clique em **+ Incluir no projeto** e inclua um novo **bloco de notas**.
2. Insira `Anomaly-detection-notebook` para o **Nome**.
3. Insira `https://github.com/IBM-Cloud/iot-device-phone-simulator/raw/master/anomaly-detection/Anomaly-detection-watson-studio-python3.ipynb` na **URL do bloco de notas**.
4. Selecione o serviço **Apache Spark** associado anteriormente como o tempo de execução.
5. Crie o **Bloco de notas**. Configure `Python 3.5 com Spark 2.1` como seu Kernel. Verifique se o notebook é criado com metadados e código.
   ![Jupyter Notebook Watson Studio](images/solution16/jupyter_notebook_watson_studio.png)
   Para atualizar, **Kernel** > Mudar kernel. Para **Confiar** no bloco de notas, **Arquivo** > Confiar no bloco de notas.
   {:tip}

### Executar o bloco de notas e detectar anomalias   
1. Selecione a célula que inicia com `!pip install --upgrade pixiedust` e, em seguida, clique em **Executar** ou **Ctrl + Enter** para executar o código.
2. Quando a instalação estiver concluída, reinicie o kernel Spark clicando no
ícone **Reiniciar Kernel**.
3. Na próxima célula de código, Importe suas credenciais do {{site.data.keyword.cloudant_short_notm}} para essa célula concluindo as etapas a seguir:
   * Clique em ![](images/solution16/data_icon.png)
   * Selecione a guia **Conexões**.
   * Clique em **Inserir no código**. Um dicionário chamado _credentials_1_ é criado com suas credenciais do {{site.data.keyword.cloudant_short_notm}}. Se o nome não for especificado como _credentials_1_, renomeie o dicionário para `credentials_1`. O `credentials_1` é usado nas células restantes.
4. Na célula com o nome do banco de dados (`dbName`), insira o nome do banco de dados {{site.data.keyword.cloudant_short_notm}} que é a origem dos dados, por exemplo, *iotp_yourWatsonIoTProgId_DBName_Year-month-day*. Para visualizar dados de diferentes dispositivos, mude os valores de `deviceId` e `deviceType` adequadamente.
   É possível localizar o banco de dados exato, navegando para sua instância do {{site.data.keyword.cloudant_short_notm}} **iot-db** que você criou anteriormente > Ativar painel.
   {:tip}
5. Salve o bloco de notas e execute cada célula de código uma após outra ou execute todas (**Célula** > Executar todas) e, no final do bloco de notas, você deverá ver anomalias para dados de movimentação do dispositivo (oa, ob e og).
   É possível mudar o intervalo de tempo de interesse para o horário desejado do dia. Procure os valores `start` e `end`.
   {:tip}
   ![Jupyter Notebook DSX](images/solution16/anomaly_detection_watson_studio.png)
6. Junto com a detecção de anomalias, as descobertas ou lições chave desta seção são
    * Uso de Spark para preparar os dados para visualização.
    * Uso de Pandas para visualização de dados
    * Gráficos de barras, histogramas para dados do dispositivo.
    * Correlação entre dois sensores por meio da matriz de Correlação.
    * Um plot de caixa para cada sensor de dispositivo, produzido com a função de plot Pandas.
    * Plots de densidade por meio do Kernel density estimation (KDE).
    ![](images/solution16/density_plots_sensor_data.png)

## Remover recursos
{:removeresources}

1. Navegue para [Lista de recursos](https://{DomainName}/resources/) > escolha o Local, Organização e Espaço nos quais você criou o app e os serviços. Em **Apps do Cloud Foundry**, exclua o App Node.JS que você criou acima.
2. Em **Serviços**, exclua o respectivos serviços Internet of Things Platform, Apache Spark, {{site.data.keyword.cloudant_short_notm}} e {{site.data.keyword.cos_full_notm}} que você criou para este tutorial.

## Conteúdo relacionado
{:related}

* [Construir, implementar, testar e treinar novamente um modelo de aprendizado de máquina preditivo](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#build-deploy-test-and-retrain-a-predictive-machine-learning-model)
* Visão geral do [IBM {{site.data.keyword.DSX_short}}](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-ws.html)
* [Jupyter Notebook](https://github.com/IBM-Cloud/iot-device-phone-simulator/blob/master/anomaly-detection/Anomaly-detection-DSX.ipynb) de detecção de anomalia
* [Entendendo o z-score](https://en.wikipedia.org/wiki/Standard_score)
* Desenvolvendo soluções de IoT cognitivo para detecção de anomalias usando uma [Série de 5 posts](https://developer.ibm.com/series/iot-anomaly-detection-deep-learning/) de deep learning
