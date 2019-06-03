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

# Analisar e visualizar dados abertos com o Apache Spark
{: #big-data-analytics-spark}

Neste tutorial, você analisará e visualizará conjuntos de dados abertos usando o {{site.data.keyword.DSX_full}}, um Jupyter Notebook e o Apache Spark. Você iniciará combinando dados que descrevem o crescimento populacional, a expectativa de vida e os códigos ISO do país em um único quadro de dados. Para descobrir insights, você usará uma biblioteca do Python chamada Pixiedust para consultar e visualizar dados em uma variedade de maneiras.

<p style="text-align: center;">

  ![](images/solution23/Architecture.png)
</p>

## Objetivos
{: #objectives}

* Implementar o Apache Spark e o {{site.data.keyword.DSX_short}} no IBM Cloud
* Trabalhar com um Notebook Jupyter e um kernel Python
* Importar, transformar, analisar e visualizar conjuntos de dados

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
   * {{site.data.keyword.sparkl}}
   * {{site.data.keyword.DSX_full}}
   * {{site.data.keyword.cos_full_notm}}

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Configuração de serviço e ambiente
Inicie provisionando os serviços usados neste tutorial e crie um projeto no {{site.data.keyword.DSX_short}}.

É possível provisionar serviços para o {{site.data.keyword.Bluemix_short}} por meio da [Lista de recursos](https://{DomainName}/resources) e do [catálogo](https://{DomainName}/catalog/). Como alternativa, o {{site.data.keyword.DSX_short}} permite criar ou incluir serviços Data & Analytics existentes por meio de suas configurações de painel e projeto.
{:tip}

1. No [catálogo do {{site.data.keyword.Bluemix_short}}](https://{DomainName}/catalog), navegue para a seção **IA**. Crie o serviço **{{site.data.keyword.DSX_short}}**. Clique no botão **Introdução** para ativar o painel do **{{site.data.keyword.DSX_short}}**.
2. No painel, clique no ladrilho **Criar um projeto** > selecione **Padrão** > Criar projeto. No campo **Nome**, insira `1stProject` como o nome. É possível deixar a descrição vazia.
3. No lado direito da página, é possível **Definir o armazenamento**. Se você já tiver provisionado o armazenamento, selecione uma instância na lista. Se não estiver, clique em **Incluir** e siga as instruções na nova guia do navegador. Depois que a criação de serviço estiver pronta, clique em **Atualizar** para ver o novo serviço.
4. Clique no botão **Criar** para criar o projeto. Você será redirecionado para a página de visão geral do projeto.  
   ![](images/solution23/NewProject.png)
5. Na página de visão geral, clique em **Configurações**.
6. Na seção **Serviços associados**, clique em **Incluir serviço** e selecione **Spark** no menu. Na tela resultante, é possível escolher uma instância de serviço do Spark existente ou criar uma nova.

## Criar e preparar um bloco de notas
O [Jupyter Notebook](http://jupyter.org/) é um aplicativo da web de software livre que permite criar e compartilhar documentos que contêm código em tempo real, equações, visualizações e texto de narrativa. Os blocos de notas e outros recursos são organizados em projetos.
1. Clique na guia **Ativos**, role para baixo até a seção **Blocos de notas** e clique em **Novo bloco de notas**.
2. Use o bloco de notas **Em branco**. Insira `MyNotebook` como o **Nome**.
3. No menu **Selecionar tempo de execução**, escolha a instância do Spark que você incluiu nas configurações do projeto. Mantenha a **Linguagem** padrão como **Python 3.5**.
4. Clique em **Criar bloco de notas** para concluir o processo.
5. O campo no qual você insere texto e comandos é chamado de **Célula**. Copie o código a seguir para a célula vazia para importar o [pacote **Pixiedust**](https://pixiedust.github.io/pixiedust/use.html). Execute a célula clicando no ícone **Executar** na barra de ferramentas ou pressionando **Shift + Enter** no teclado.
   ```Python
   import pixiedust
   ```
   {:codeblock}
   ![](images/solution23/FirstCell_ImportPixiedust.png)

Se você nunca trabalhou com os Jupyter Notebooks, clique no ícone **Docs** no menu superior direito. Navegue para **Analisar dados**, em seguida, para a [seção **Blocos de notas**](https://dataplatform.ibm.com/docs/content/analyze-data/notebooks-parent.html?context=analytics) para aprender mais sobre [blocos de notas e suas partes](https://dataplatform.ibm.com/docs/content/analyze-data/parts-of-a-notebook.html?context=analytics&linkInPage=true).
{:tip}

## Carregar dados
Em seguida carregue três conjuntos de dados abertos e torne-os disponíveis dentro do bloco de notas. A biblioteca **Pixiedust** permite [carregar arquivos **CSV** facilmente usando uma URL](https://pixiedust.github.io/pixiedust/loaddata.html).

1.  Copie a linha a seguir para a próxima célula vazia em seu bloco de notas, mas não a execute ainda.
   ```Python
   df_pop = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
2. Em outra guia do navegador, acesse a seção [Comunidade](https://dataplatform.ibm.com/community?context=analytics). Em **Conjuntos de dados**, procure [**População total por país**](https://dataplatform.ibm.com/exchange/public/entry/view/889ca053a19986a4445839358a91963e) e clique no ladrilho. Na parte superior direita, clique no ícone de **link** para obter um URI de acesso. Copie o URI e substitua o texto **YourAccessURI** na célula do bloco de notas pelo link. Clique no ícone **Executar** na barra de ferramentas ou **Shift + Enter**.
3. Repita a etapa para outro conjunto de dados. Copie a linha a seguir para a próxima célula vazia em seu bloco de notas.
   ```Python
   df_life = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
4. Na outra guia do navegador com os **Conjuntos de dados**, procure [**Expectativa de vida à nascença por país no total de anos**](https://dataplatform.ibm.com/exchange/public/entry/view/f15be429051727172e0d0c226e2ce895). Obtenha o link novamente e use-o para substituir **YourAccessURI** na célula do bloco de notas e **Executar** para iniciar o processo de carregamento.
5. Para o último dos três conjuntos de dados, carregue uma lista de nomes de países e seus códigos ISO de uma coleção de conjuntos de dados abertos no Github. Copie o código para a próxima célula de bloco de notas vazia e execute-o.
   ```Python
     df_countries = pixiedust.sampleData('https://raw.githubusercontent.com/datasets/country-list/master/data.csv')
   ```
   {:codeblock}

A lista de códigos de país será usada posteriormente para simplificar a seleção de dados usando um código do país em vez do nome exato do país.

## Transformar dados
Depois que os dados estiverem disponíveis, transforme-os um pouco e combine os três conjuntos em um único quadro de dados.
1. O bloco de código a seguir redefinirá o quadro de dados para os dados populacionais. Isso é feito com uma instrução SQL que renomeia as colunas. Uma visualização é, então, criada e o esquema impresso. Copie esse código para a próxima célula vazia e execute-o.
   ```Python
   sqlContext.registerDataFrameAsTable(df_pop, "PopTable")
   df_pop = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Population FROM PopTable")
   df_pop.createOrReplaceTempView('population')
   df_pop.printSchema()
   ```
   {:codeblock}
2. Repita o mesmo para os dados de Expectativa de vida. Em vez de imprimir o esquema, esse código imprime as primeiras 10 linhas.  
   ```Python
   sqlContext.registerDataFrameAsTable(df_life, "lifeTable")
   df_life = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Life FROM lifeTable")
   df_life = df_life.withColumn("Life", df_life["Life"].cast("double"))
   df_life.createOrReplaceTempView('life')
   df_life.show(10)
   ```
   {:codeblock}

3. Repita a transformação do esquema para os dados do país.
   ```Python
   sqlContext.registerDataFrameAsTable(df_countries, "CountryTable")
   df_countries = sqlContext.sql("SELECT `Name` as Country, Code as CountryCode FROM CountryTable")
   df_countries.createOrReplaceTempView('countries')
   ```
   {:codeblock}

4. Os nomes de colunas agora são mais simples e os mesmos entre os conjuntos de dados, que podem ser combinados em um único quadro de dados. Execute uma junção **externa** nos dados de expectativa de vida e de população. Em seguida, na mesma instrução, junção **interna** join para incluir os códigos de país. Tudo será ordenado por país e ano. A saída define o quadro de dados **df_all**. Utilizando uma junção interna, os dados resultantes contêm somente os países que estão localizados na lista ISO. Esse processo remove as entradas regionais e outras dos dados.
   ```Python
   df_all = df_life.join(df_pop, ['Country', 'Year'], 'outer').join(df_countries, ['Country'], 'inner').orderBy(['Country', 'Year'], ascending=True)
   df_all.show(10)
   ```
   {:codeblock}

5. Mude o tipo de dados de **Ano** para um número inteiro.
   ```Python
   df_all = df_all.withColumn("Year", df_all["Year"].cast("integer"))
   df_all.printSchema()
   ```
   {:codeblock}

Seus dados combinados estão prontos para serem analisados.

## Analisar dados
Nesta parte, use [Pixiedust para visualizar os dados em diferentes gráficos](https://pixiedust.github.io/pixiedust/displayapi.html). Inicie comparando a expectativa de vida para alguns países.

1. Copie o código na próxima célula vazia e execute-o.
   ```Python
   df_all.createOrReplaceTempView('l2')
   dfl2=spark.sql("SELECT Life, Country, Year FROM l2 where CountryCode in ('CN','DE','FR','IN','US')")
   display(dfl2)
   ```
   {:codeblock}
2. Uma tabela de rolagem é mostrada. Clique no ícone de gráfico diretamente sob o bloco de códigos e selecione **Gráfico de linha**. Um diálogo pop-up com o **Pixiedust: opções de gráfico de linha** aparecerá. Insira um **Título do gráfico** como "Comparação de expectativa de vida". Nos **Campos** oferecidos, arraste **Ano** para a caixa **Chaves**, **Vida** para a área **Valores**. Insira **1000** para **N° de linhas para exibir**. Pressione **OK** para ter o gráfico de linha plotado. No lado direito, certifique-se de que **mapplotlib** esteja selecionado como **Renderizador**. Clique no seletor **Agrupar por** e escolha **País**. Um gráfico semelhante ao seguinte será mostrado.
   ![](images/solution23/LifeExpectancy.png)

3. Crie um gráfico com foco no ano de 2010. Copie o código na próxima célula vazia e execute-o.
   ```Python
   df_all.createOrReplaceTempView('life2010')
   df_life_2010=spark.sql("SELECT Life, Country FROM life2010 WHERE Year=2010 AND Life is not NULL ")
   display(df_life_2010)
   ```
   {:codeblock}
4. No seletor de gráfico, escolha **Mapa**. No diálogo de configuração, arraste **País** para a área **Chaves**. Mova **Vida** para a caixa **Valores**. Semelhante ao primeiro gráfico, aumente o **N° de linhas para exibir** para **1000**. Pressione **OK** para plotar o mapa. Escolha **brunel** como **Renderizador**. Um mapa mundial colorido em relação à expectativa de vida é mostrado. É possível usar o mouse para aumentar o zoom no mapa.
   ![](images/solution23/LifeExpectancyMap2010.png)

## Expandir o tutorial
Abaixo estão algumas ideias e sugestões para aprimorar esse tutorial.
* Criar e visualizar uma consulta mostrando a taxa de expectativa de vida relativa ao crescimento populacional para um país de sua escolha
* Calcular e visualizar as taxas de crescimento populacional por país em um mapa mundial
* Carregar e integrar dados adicionais por meio do catálogo de conjuntos de dados
* Exportar os dados combinados para um arquivo ou banco de dados

## Conteúdo relacionado
{:related}
Abaixo são fornecidos os links relacionados aos tópicos abordados neste tutorial.
* [Watson Data Platform](https://dataplatform.ibm.com): use o Watson Data Platform para colaborar e construir aplicativos mais inteligentes. Visualize e descubra rapidamente os insights de seus dados e colabore entre as equipes.
* [PixieDust](https://www.ibm.com/cloud/pixiedust): ferramenta de produtividade de software livre para Jupyter Notebooks
* [Class.ai cognitivo](https://cognitiveclass.ai/): cursos de ciência de dados e de computação cognitiva
* [IBM Watson Data Lab](https://ibm-watson-data-lab.github.io/): coisas que fizemos com dados, portanto, você pode também
* [Serviço Analytics Engine](https://{DomainName}/catalog/services/analytics-engine): desenvolva e implemente aplicativos analíticos usando o Apache Spark e o Apache Hadoop de software livre
