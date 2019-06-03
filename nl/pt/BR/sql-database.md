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

# Banco de dados SQL para dados de nuvem
{: #sql-database}

Este tutorial mostra como provisionar um serviço de banco de dados SQL (relacional), criar uma tabela e carregar um conjunto de dados grande (informações da cidade) no banco de dados. Em seguida, você implementa um app da web "worldcities" para fazer uso desses dados e mostra como acessar o banco de dados de nuvem. O app é escrito em Python usando a [Estrutura Flask](http://flask.pocoo.org/).

![](images/solution5/Architecture.png)

## Objetivos

* Provisionar um banco de dados SQL
* Criar o esquema do banco de dados (tabela)
* Carregar dados
* Conectar o serviço de app e de banco de dados (compartilhar credenciais)
* Monitoramento, segurança, backups e recuperação

## Produtos

Este tutorial usa os produtos a seguir:
   * [{{site.data.keyword.dashdblong_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)

## Antes de Começar
{: #prereqs}

Acesse [GeoNames](http://www.geonames.org/) e faça download e extraia o arquivo [cities1000.zip](http://download.geonames.org/export/dump/cities1000.zip). Ele contém informações sobre cidades com uma população de mais de 1000. Você vai usá-lo como conjunto de dados.

## Provisionar o banco de dados SQL
Inicie a criação de uma instância do serviço **[{{site.data.keyword.dashdbshort_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)**.

![](images/solution5/Catalog.png)

1. Visite o [painel do {{site.data.keyword.Bluemix_short}}](https://{DomainName}). Clique em **Catálogo** na barra de navegação superior.
2. Clique em **Data & Analytics** em Plataforma na área de janela esquerda e selecione **{{site.data.keyword.dashdbshort_notm}}**.
3. Selecione o plano **Entrada** e mude o nome do serviço sugerido para "sqldatabase" (você usará esse nome posteriormente). Selecione um local para a implementação do banco de dados e certifique-se de que a organização e o espaço corretos estejam selecionados.
4. Clique em **Criar**. Depois de um breve momento, você deverá receber uma notificação de sucesso.
5. Na **Lista de recursos**, clique na entrada para seu serviço {{site.data.keyword.dashdbshort_notm}} recém-criado.
6. Clique em **Abrir** para ativar o console do banco de dados. Se for a primeira vez que usar o console, será oferecido a você para fazer um tour.

## Criar uma tabela
Você precisa de uma tabela para conter os dados de amostra. Crie-a usando o console.

1. No console para {{site.data.keyword.dashdbshort_notm}}, clique em **Explorar** na barra de navegação. Isso leva você para uma lista de esquemas existentes no banco de dados.
2. Localize e clique no esquema iniciando com "DASH".
3. Clique em **"+ Nova tabela"** para abrir um formulário para o nome da tabela e suas colunas.
4. Coloque "cidades" como o nome da tabela. Copie as definições de coluna do arquivo [cityschema.txt](https://github.com/IBM-Cloud/cloud-sql-database/blob/master/cityschema.txt) e cole-as na caixa para as colunas e os tipos de dados.
5. Clique em **Criar** para definir a nova tabela.   
   ![](images/solution5/TableCitiesCreated.png)

## Carregar dados
Agora que a tabela "cidades" foi criada, você carregará dados nela. Isso pode ser feito de diferentes maneiras, por exemplo, por meio de sua máquina local ou do armazenamento de objeto de nuvem (COS) com a interface Swift ou Amazon S3, utilizando o serviço de migração [{{site.data.keyword.dwl_full}}](https://{DomainName}/catalog/services/lift). Para este tutorial, você fará upload de dados de sua máquina. Durante esse processo, você adaptará a estrutura de tabela e o formato de dados para corresponder totalmente ao conteúdo do arquivo.

1. Na navegação superior, clique em **Carregar**. Em seguida, em **Seleção de arquivo**, clique em **procurar arquivos** para localizar e selecionar o arquivo "cities1000.txt" transferido por download na primeira seção deste guia.
2. Clique em **Avançar** para chegar à visão geral do esquema. Escolha o esquema iniciando com "DASH" novamente, em seguida, a tabela "CITIES". Clique em **Avançar** novamente.   

   Como a tabela está vazia, ela não faz diferença para anexar ou sobrescrever dados existentes.
   {:tip }
3. Agora, customize como os dados do arquivo "cities1000.txt" são interpretados durante o processo de carregamento. Primeiro, desative "Cabeçalho na primeira linha" porque o arquivo contém somente dados. Em seguida, digite "0x09" como separador. Isso significa que os valores dentro do arquivo são delimitados por tabulação. Por último, selecione "AAAA-MM-DD" como formato de data. Agora, tudo deve ser semelhante a esta captura de tela.    
  ![](images/solution5/LoadTabSeparator.png)
4. Clique em **Avançar** e será oferecido a você para revisar as configurações de carregamento. Concorde e clique em **Iniciar carregamento** para iniciar o carregamento dos dados na tabela "CITIES". O progresso é exibido. Quando os dados forem transferidos por upload, deverá levar somente alguns segundos até que o carregamento seja concluído e algumas estatísticas sejam apresentadas.   
   ![](images/solution5/LoadProgressSteps.png)

## Verificar dados carregados usando SQL
Os dados foram carregados no banco de dados relacional. Não houve nenhum erro, mas é necessário executar alguns testes rápidos de qualquer maneira. Use o editor de SQL integrado para digitar e executar algumas instruções SQL.

1. Na navegação superior, clique em **Executar SQL**.
   Em vez do editor de SQL integrado, é possível usar ferramentas SQL baseadas em nuvem e tradicionais em seu desktop ou máquina servidor com o {{site.data.keyword.dashdbshort_notm}}. As informações de conexão podem ser localizadas no menu de configurações. Algumas ferramentas são até oferecidas para download na seção "Downloads" no menu oferecido atrás do ícone "livro" (representando a documentação e ajuda).
    {:tip }
2. No "Editor de SQL", digite ou copie a consulta a seguir:   
   ```bash
   select count(*) from cities
   ```
   {:codeblock}
   em seguida, pressione o botão **Executar todos**. Na seção de resultados, o mesmo número de linhas que o relatado pelo processo de carregamento deve ser mostrado.   
3. No "Editor de SQL", insira a instrução a seguir em uma nova linha:
   ```
   select countrycode, count(name) from cities
   group by countrycode
   order by 2 desc
   ```
   {:codeblock}   
4. No editor, selecione o texto da instrução acima. Clique no botão **Executar selecionado**. Somente essa instrução deve ser executada agora, retornando algumas estatísticas por país na seção de resultados.

## Implementar o código do aplicativo
O [código pronto para execução para o app de banco de dados está localizado nesse repositório Github](https://github.com/IBM-Cloud/cloud-sql-database). Clone ou faça download do repositório, em seguida, envie-o por push para o IBM Cloud.

1. Clone o repositório Github:
   ```bash
   git clone https://github.com/IBM-Cloud/cloud-sql-database
   cd cloud-sql-database
   ```
2. Envie por push o aplicativo para o IBM Cloud. É necessário estar com login efetuado no local, organização e espaço nos quais o banco de dados foi provisionado. Copie e cole esses comandos uma linha por vez.
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push your-app-name
   ```
3. Depois que o processo de push for concluído, você deverá ser capaz de acessar o app. Nenhuma configuração adicional é necessária. O arquivo `manifest.yml` informa ao IBM Cloud para ligar o app e o serviço de banco de dados denominado "sqldatabase" juntos.

## Segurança, backup e recuperação, monitoramento
O {{site.data.keyword.dashdbshort_notm}} é um serviço gerenciado. A IBM cuida de proteger o ambiente, os backups diários e o monitoramento do sistema. No plano de entrada, o ambiente de banco de dados é uma configuração de diversos locatários com administração reduzida e opções configuradas para usuários. No entanto, se você está usando um dos planos corporativos, há [várias opções para gerenciar usuários, para configurar a segurança do banco de dados adicional](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.security.doc/doc/security.html) e para [monitorar o banco de dados](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.admin.mon.doc/doc/c0001138.html).   

Além das opções de administração tradicionais, o serviço [{{site.data.keyword.dashdbshort_notm}} também oferece uma API de REST para monitoramento, gerenciamento de usuários, utilitários, carregamento, acesso de armazenamento e muito mais](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/connecting/connect_api.html). A interface do Swagger executável dessa API pode ser acessada no menu atrás do ícone "livro" em "APIs de Rest". Algumas ferramentas que podem ser usadas para monitoramento e muito mais, por exemplo, o IBM Data Server Manager, podem até mesmo ser transferidas por download na seção "Downloads" nesse mesmo menu.

## Testar o app
O app para exibir informações de cidade com base no conjunto de dados carregado é reduzido para um mínimo. Ele oferece um formulário de procura para especificar um nome de cidade e algumas cidades pré-configuradas. Eles são convertidos em `/search?name=cityname` (formulário de procura) ou `/city/cityname` (cidades especificadas diretamente). Ambas as solicitações são entregues por meio das mesmas linhas de código em segundo plano. O cityname é passado como valor para uma instrução SQL preparada usando um marcador de parâmetro por motivos de segurança. As linhas são buscadas do banco de dados e passadas para um modelo HTML para renderização.

## Limpeza
Para limpar os recursos usados pelo tutorial, siga estas etapas:
1. Visite a [Lista de recursos do {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources). Localize seu app.
2. Clique no ícone de menu para o app e escolha **Excluir app**. Na janela de diálogo, marque o visto em que você deseja excluir o serviço {{site.data.keyword.dashdbshort_notm}} relacionado.
3. Clique no botão **Excluir**. O app e o serviço de banco de dados são removidos e você é levado de volta para a lista de recursos.

## Expandir o tutorial
Deseja ampliar esse app? Aqui estão algumas ideias:
1. Ofereça uma procura de caracteres curinga nos nomes alternativos.
2. Procure cidades de um país específico e somente dentro de um determinado valor de população.
3. Mude o layout da página, substituindo os estilos CSS e ampliando os modelos.
4. Permita a criação baseada em formulário de novas informações de cidade ou permita atualizações em dados existentes, por exemplo, população.

## Conteúdo relacionado
* Documentação: [IBM Knowledge Center for {{site.data.keyword.dashdbshort_notm}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Perguntas mais frequentes sobre o {{site.data.keyword.Db2_on_Cloud_long_notm}} e o {{site.data.keyword.dashdblong_notm}}](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/managed_service.html) respondendo perguntas relacionadas ao serviço gerenciado, backup de dados, criptografia de dados e segurança e muito mais.
* [Db2 Developer Community Edition grátis](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) para desenvolvedores
* Documentação: [Descrição da API do driver ibm_db Python](https://github.com/ibmdb/python-ibmdb/wiki/APIs)
* [IBM Data Server Manager](https://www.ibm.com/us-en/marketplace/data-server-manager)
