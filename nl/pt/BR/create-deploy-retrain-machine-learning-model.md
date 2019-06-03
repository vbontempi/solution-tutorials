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

# Construir, implementar, testar e treinar novamente um modelo de aprendizado de máquina preditivo
{: #create-deploy-retrain-machine-learning-model}
Este tutorial conduz você pelo processo de construção de um modelo de aprendizado de máquina preditivo, implementando-o como uma API para ser usado em aplicativos, testando o modelo e treinando novamente o modelo com dados de feedback. Tudo isso está acontecendo em uma experiência de autoatendimento integrada e unificada no IBM Cloud.

Neste tutorial, o **Conjunto de dados de flor Íris** é usado para criar um modelo de aprendizado de máquina para classificar espécies de flores.

Na terminologia do aprendizado de máquina, a classificação é considerada uma instância de aprendizado supervisionado, ou seja, aprender onde um conjunto de treinamento de observações corretamente identificadas está disponível.
{:tip}

{:shortdesc}

<p style="text-align: center;">
  ![](images/solution22-build-machine-learning-model/architecture_diagram.png)
</p>

## Objetivos
{: #objectives}

* Importar dados para um projeto.
* Construir um modelo de aprendizado de máquina.
* Implementar o modelo e testar a API.
* Testar um modelo de aprendizado de máquina.
* Criar uma conexão de dados de feedback para avaliação contínua de aprendizado e modelo.
* Treinar novamente seu modelo.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.sparkl}}](https://{DomainName}/catalog/services/apache-spark)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [{{site.data.keyword.pm_full}}](https://{DomainName}/catalog/services/machine-learning)
* [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)

## Antes de Começar
{: #prereqs}
* O IBM Watson Studio e o Watson Knowledge Catalog são aplicativos que fazem parte do IBM Watson. Para criar uma conta do IBM Watson, inicie inscrevendo-se para um ou ambos os aplicativos.

   Acesse [Testar o IBM Watson](https://dataplatform.ibm.com/registration/stepone) e inscreva-se para os apps do IBM Watson.

## Importar dados para um projeto

{:#import_data_project}

Um projeto é como você organiza seus recursos para alcançar um objetivo específico. Seus recursos do projeto podem incluir dados, colaboradores e ferramentas analíticas, como blocos de notas do Jupyter e modelos de aprendizado de máquina.

É possível criar um projeto para incluir dados e abrir um ativo de dados no refinador de dados para limpar e moldar seus dados.

**Criar um projeto:**

1. Acesse o [catálogo do {{site.data.keyword.Bluemix_short}}](https://{DomainName}/catalog) e selecione [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience?taxonomyNavigation=app-services) sob a seção **IA**. **Crie** o serviço. Clique no botão **Introdução** para ativar o painel do **{{site.data.keyword.DSX_short}}**.
2. Crie um **projeto** > Clique em **Criar projeto** no ladrilho **Padrão**. Inclua um nome, por exemplo `iris_project`, e uma descrição opcional para o projeto.
3. Deixe a caixa de seleção **Restringir quem pode ser um colaborador** desmarcada, visto que não há dados confidenciais.
4. Em **Definir armazenamento**, clique em **Incluir** e escolha um serviço Cloud Object Storage existente ou crie um novo (selecione Plano **Lite** > Criar). Pressione **Atualizar** para ver o serviço criado.
5. Clique em **Criar**. Seu novo projeto é aberto e é possível começar a incluir recursos nele.

**Importar dados:**

Conforme mencionado anteriormente, você usará o **Conjunto de dados Íris**. O conjunto de dados Íris foi usado no documento clássico de R.A. Fisher de 1936, [The Use of Multiple Measurements in Taxonomic Problems](http://rcs.chemometrics.ru/Tutorials/classification/Fisher.pdf) e também pode ser localizado no [UCI Machine Learning Repository](http://archive.ics.uci.edu/ml/). Esse pequeno conjunto de dados é frequentemente usado para testar algoritmos e visualizações de aprendizado de máquina. O objetivo é classificar as flores Íris entre três espécies (Setosa, Versicolor ou Virginica) por meio de medições de comprimento e largura de sépalas e pétalas. O conjunto de dados Íris contém 3 classes de 50 instâncias cada, em que cada classe refere-se a um tipo de planta Íris.
![](images/solution22-build-machine-learning-model/iris_machinelearning.png)


**Faça download de** [iris_initial.csv](https://ibm.box.com/shared/static/nnxx7ozfvpdkjv17x4katwu385cm6k5d.csv), que consiste em 40 instâncias de cada classe. Você usará as 10 instâncias restantes de cada classe para treinar novamente seu modelo.

1. Em **Ativos** em seu projeto, clique no ícone **Localizar e incluir dados** ![Mostra o ícone de localização de dados.](images/solution22-build-machine-learning-model/data_icon.png).
2. Em **Carregar**, clique em **procurar** e faça upload do `iris_initial.csv` transferido por download.
   ![](images/solution22-build-machine-learning-model/find_and_add_data.png)
3. Depois de incluído, você deverá ver `iris_initial.csv` sob a seção **Ativos de dados** do projeto. Clique no nome para ver o conteúdo do conjunto de dados.

## Associar serviços
{:#associate_services}
1. Em **Configurações**, role para **Serviços associados** > clique em **Incluir serviço** > escolha **Spark**.
   ![](images/solution22-build-machine-learning-model/associate_services.png)
2. Selecione o plano **Lite** e clique em **Criar**. Use os valores padrão e clique em **Confirmar**.
3. Clique em **Incluir serviço** novamente e escolha **Watson**. Clique em **Incluir** no ladrilho **Machine Learning** > escolha o plano **Lite** > clique em **Criar**.
4. Deixe os valores padrão e clique em **Confirmar** para provisionar um serviço Machine Learning.

## Construir um modelo de aprendizado de máquina

{:#build_model}

1. Clique em **Incluir no projeto** e selecione **Modelo do Watson Machine Learning**. No diálogo, inclua **iris_model** como nome e uma descrição opcional.
2. Na seção **Serviço Machine Learning**, você deverá ver o serviço Machine Learning que foi associado na etapa acima.
   ![](images/solution22-build-machine-learning-model/machine_learning_model_creation.png)
3. Selecione **Construtor de modelo** como seu tipo de modelo e na seção **Serviço Spark ou ambiente**, escolha o serviço spark criado anteriormente
4. Selecione **Manual** para criar manualmente um modelo. Clique em **Criar**.

   Para o método automático, você depende completamente de automatic data preparation (ADP). Para o método manual, além de algumas funções que são manipuladas pelo transformador ADP, é possível incluir e configurar seus próprios estimadores, que são os algoritmos usados na análise.
   {:tip}

5. Na próxima página, selecione `iris_initial.csv` como seu conjunto de dados e clique em **Avançar**.
6. Na página **Selecionar uma técnica**, com base no conjunto de dados incluído, as colunas Rótulo e as colunas de recurso são pré-preenchidas. Selecione **species (String)** como sua **Coluna de rótulo** e **petal_length (Decimal)** e **petal_width (Decimal)** como suas **Colunas de recurso**.
7. Escolha **Classificação de multiclasse** como sua técnica sugerida.
   ![](images/solution22-build-machine-learning-model/model_technique.png)
8. Para **Divisão de validação**, defina a configuração a seguir:

   **Treino:** 50%,
   **Teste** 25%,
   **Validação:** 25%

9. Clique em **Incluir estimadores** e selecione **Classificador de árvore de decisão**, em seguida, **Incluir**.

   É possível avaliar múltiplos estimadores de uma vez. Por exemplo, é possível incluir **Classificador de árvore de decisão** e **Classificador aleatório de floresta** como estimadores para treinar seu modelo e escolher o melhor ajuste com base na saída de avaliação.
   {:tip}

10. Clique em **Avançar** para treinar o modelo. Depois de ver o status como **Treinado e avaliado**, clique em **Salvar**.
    ![](images/solution22-build-machine-learning-model/trained_model.png)

11. Clique em **Visão geral** para verificar os detalhes do modelo.

## Implementar o modelo e testar a API

{:#deploy_model}

1. Sob o modelo criado, clique em **Implementações** > **Incluir implementação**.
2. Escolha **Serviço da web**. Inclua um nome, por exemplo `iris_deployment`, e uma descrição opcional.
3. Clique em **Salvar**. Na página de visão geral, clique no nome do novo serviço da web. Depois que o status for **DEPLOY_SUCCESS**, será possível verificar o terminal de pontuação, os fragmentos de código em várias linguagens de programação e a Especificação da API em **Implementação**.
4. Clique em **Visualizar especificação da API** para ver e testar os terminais da API do {{site.data.keyword.pm_short}}.
   ![](images/solution22-build-machine-learning-model/machine_learning_api.png)

   Para começar a trabalhar com a API, é necessário gerar um **token de acesso** usando o **nome do usuário** e a **senha** disponíveis na guia **Credenciais de serviço** da instância de serviço do {{site.data.keyword.pm_short}} em [Lista de recursos do {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/). Siga as instruções mencionadas na página de especificação da API para gerar um **token de acesso**.
   {:tip}
5. Para fazer uma predição on-line, use a chamada da API `POST /online`.
   * O `instance_id` pode ser localizado na guia **Credenciais de serviço** do serviço {{site.data.keyword.pm_short}} em [Lista de recursos do {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/).
   * O `deployment_id` e o `published_model_id` estão em **Visão geral** de sua implementação.
   *  Para `online_prediction_input`, use o JSON abaixo

     ```json
     {
     	"fields": ["sepal_length", "sepal_width", "petal_length", "petal_width"],
     	"values": [
     		[5.1, 3.5, 1.4, 0.2]
     	]
     }
     ```
   * Clique em **Testar** para ver a saída JSON.

6. Usando os terminais de API, é possível agora chamar esse modelo por meio de qualquer aplicativo.

## Testar seu modelo

{:#test_model}

1. Em **Teste**, você deverá ver os dados de entrada (dados do Recurso) sendo preenchidos automaticamente.
2. Clique em **Prever** e você deverá ver o **Valor predito para as espécies** em um gráfico.
   ![](images/solution22-build-machine-learning-model/model_predict_test.png)
3. Para entrada e saída JSON, clique nos ícones ao lado da entrada e da saída ativa.
4. É possível mudar os dados de entrada e continuar testando seu modelo.

## Criar uma conexão de dados de feedback

{:#create_feedback_connection}

1. Para a avaliação contínua de aprendizado e modelo, é necessário armazenar os novos dados em algum lugar. Crie um serviço [{{site.data.keyword.dashdbshort}}](https://{DomainName}/catalog/services/db2-warehouse) > plano **Entry** que age como nossa conexão de dados de feedback.
2. Na página **Gerenciar** do {{site.data.keyword.dashdbshort}}, clique em **Abrir**. Na navegação superior, selecione **Carregar**.
3. Clique em **procurar arquivos** em **Meu computador** e faça upload do `iris_initial.csv`. Clique em **Avançar**.
4. Selecione **DASHXXXX**, por exemplo DASH1234, como seu **Esquema** e, em seguida, clique em **Nova tabela**. Nomeie-o como `IRIS_FEEDBACK` e clique em **Avançar**.
5. Os tipos de dados são detectados automaticamente. Clique em **Avançar** e, em seguida, em **Iniciar carregamento**.
   ![](images/solution22-build-machine-learning-model/define_table.png)
6. Um novo destino **DASHXXXX.IRIS_FEEDBACK** é criado.

   Você usará isso na próxima etapa em que treinará novamente o modelo para melhor desempenho e precisão.

## Treinar novamente seu modelo

{:#retrain_model}

1. Retorne para sua [Lista de recursos do {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources) e, sob o serviço {{site.data.keyword.DSX_short}} que você tem usado, clique em **Projetos** > iris_project > **iris-model** (em ativos) > Avaliação.
2. Em **Monitoramento de desempenho**, clique em **Configurar monitoramento de desempenho**.
3. Na página Configurar monitoramento de desempenho,
   * Selecione o serviço Spark. O tipo de predição deve ser preenchido automaticamente.
   * Escolha **weightedPrecision** como sua métrica e configure `0.98` como o limite opcional.
   * Clique em **Criar nova conexão** para apontar para o IBM Db2 Warehouse on Cloud que você criou na seção acima.
   * Selecione a conexão do Db2 Warehouse e, quando os detalhes da conexão forem preenchidos, clique em **Criar**.
    ![](images/solution22-build-machine-learning-model/new_db2_connection.png)
   * Clique em **Selecionar referência de dados de feedback** e aponte para a tabela IRIS_FEEDBACK e clique em **Selecionar**.
    ![](images/solution22-build-machine-learning-model/select_source.png)
   * Na caixa **Contagem de registros necessária para reavaliação**, digite o número mínimo de novos registros para acionar o novo treinamento. Use **10** ou deixe em branco para usar o valor padrão de 1000.
   * Na caixa **Novo treinamento automático**, selecione uma das opções a seguir:
     - Para iniciar o novo treinamento automático sempre que o desempenho do modelo estiver abaixo do limite configurado, selecione **quando o desempenho do modelo estiver abaixo do limite**. Para este tutorial, você escolherá essa opção porque nossa precisão está abaixo do limite (.98).
     - Para proibir o novo treinamento automático, selecione **nunca**.
     - Para iniciar o novo treinamento automático independentemente do desempenho, selecione **sempre**.
   * Na caixa **Implementação automática**, selecione uma das opções a seguir:
     - Para iniciar a implementação automática sempre que o desempenho do modelo for melhor do que a versão anterior, selecione **quando o desempenho do modelo for melhor do que a versão anterior**. Para este tutorial, você escolherá essa opção porque nosso objetivo é melhorar continuamente o desempenho do modelo.
     - Para proibir a implementação automática, selecione **nunca**.
     - Para iniciar a implementação automática independentemente do desempenho, selecione **sempre**.
   * Clique em **Salvar**.
     ![](images/solution22-build-machine-learning-model/configure_performance_monitoring.png)
4. Faça download do arquivo [iris_retrain.csv](https://ibm.box.com/shared/static/96kvmwhb54700pjcwrd9hd3j6exiqms8.csv). Posteriormente, clique em **Incluir dados de feedback**, selecione o arquivo csv transferido por download e clique em **Abrir**.
5. Clique em **Nova avaliação** para iniciar.
   ![](images/solution22-build-machine-learning-model/retraining_model.png)
6. Quando a avaliação for concluída, será possível verificar a seção **Último resultado de avaliação** para o valor **WeightedPrecision** melhorado.

## Remover recursos
{:removeresources}

1. Navegue para [Lista de recursos do {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/) > escolha o Local, Organização e Espaço nos quais você criou os serviços.
2. Exclua os respectivos serviços {{site.data.keyword.DSX_short}}, {{site.data.keyword.sparks}}, {{site.data.keyword.pm_short}}, {{site.data.keyword.dashdbshort}} e {{site.data.keyword.cos_short}} que você criou para este tutorial.

## Conteúdo relacionado
{:related}

- [Visão geral do Watson Studio](https://dataplatform.ibm.com/docs/content/getting-started/overview-ws.html?audience=wdp&context=wdp)
- [Detectar anomalias usando o Machine Learning](https://{DomainName}/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#data_experiencee)
<!-- - [Watson Data Platform Tutorials](https://www.ibm.com/analytics/us/en/watson-data-platform/tutorial/) -->
- [Criação automática de modelo](https://datascience.ibm.com/docs/content/analyze-data/ml-model-builder.html?linkInPage=true)
- [Aprendizado de máquina e IA](https://dataplatform.ibm.com/docs/content/analyze-data/wml-ai.html?audience=wdp&context=wdp)
<!-- - [Watson machine learning client library](https://dataplatform.ibm.com/docs/content/analyze-data/pm_service_client_library.html#client-libraries) -->
