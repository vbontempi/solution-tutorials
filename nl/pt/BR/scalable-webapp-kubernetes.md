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

# Aplicativo da web escalável no Kubernetes
{: #scalable-webapp-kubernetes}

Este tutorial conduz você na criação do esqueleto de um aplicativo da web, em executá-lo localmente em um contêiner e, em seguida, implementá-lo em um cluster Kubernetes criado com o [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster). Além disso, você aprenderá como ligar um domínio customizado, monitorar o funcionamento do ambiente e escalar o aplicativo.
{:shortdesc}

Os contêineres são uma maneira padrão de empacotar apps e todas as suas dependências para que seja possível mover facilmente os apps entre ambientes. Ao contrário de máquinas virtuais, os contêineres não empacotam o sistema operacional. Somente o código de app, o tempo de execução, as ferramentas de sistema, as bibliotecas e as configurações são empacotados dentro de contêineres. Os contêineres são mais leves, móveis e eficientes do que máquinas virtuais.

Para desenvolvedores que procuram impulsionar seus projetos, a CLI do {{site.data.keyword.dev_cli_notm}} permite o desenvolvimento e a implementação rápidas de aplicativos gerando aplicativos modelo que podem ser executados imediatamente ou customizados como o iniciador para suas próprias soluções. Além de gerar o código do aplicativo iniciador, a imagem de contêiner do Docker e os ativos do CloudFoundry, os geradores de código usados pela CLI de desenvolvimento e pelo console da web geram arquivos para auxiliar na implementação em ambientes do [Kubernetes](https://kubernetes.io/). Os modelos geram gráficos do [Helm](https://github.com/kubernetes/helm) que descrevem a configuração inicial de implementação do Kubernetes do aplicativo e são facilmente ampliados para criar implementações de multi-imagem ou complexas, conforme necessário.

## Objetivos
{: #objectives}

* Criar o esqueleto de um aplicativo iniciador.
* Implementar o aplicativo no cluster Kubernetes.
* Ligar um domínio customizado.
* Monitore os logs e o funcionamento do cluster.
* Escalar pods do Kubernetes.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

  ![Arquitetura](images/solution2/Architecture.png)
</p>

1. Um desenvolvedor gera um aplicativo iniciador com o {{site.data.keyword.dev_cli_notm}}.
1. Construir o aplicativo produz uma imagem de contêiner do Docker.
1. A imagem é enviada por push para um namespace no {{site.data.keyword.containershort_notm}}.
1. O aplicativo é implementado em um cluster Kubernetes.
1. Os usuários acessam o aplicativo.

## Antes de Começar
{: #prereqs}

* [Configure a CLI do {{site.data.keyword.registrylong_notm}} e seu namespace de registro](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)
* [Instale o {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - Script para instalar o docker, kubectl, helm, ibmcloud cli e os plug-ins necessários
* [Entenda os fundamentos do Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## Criar um cluster Kubernetes
{: #create_kube_cluster}

O {{site.data.keyword.containershort_notm}} fornece ferramentas poderosas, combinando s tecnologias Docker e Kubernetes, uma experiência intuitiva do usuário e a segurança e o isolamento integrados para automatizar a implementação, a operação, o ajuste de escala e o monitoramento de apps conteinerizados em um cluster de hosts de cálculo.

A parte maior deste tutorial pode ser realizada com um cluster **Grátis**. Duas seções opcionais relacionadas ao Ingresso do Kubernetes e ao domínio customizado requerem um cluster **Pago** do tipo **Padrão**.

1. Crie um cluster Kubernetes por meio do [catálogo do {{site.data.keyword.Bluemix}}](https://{DomainName}/containers-kubernetes/launch).

   Para facilidade de uso, verifique os detalhes de configuração, como o número de CPUs, a memória e o número de nós do trabalhador que você obtém com os planos Lite e Padrão.
   {:tip}

   ![Criação do cluster Kubernetes no IBM Cloud](images/solution2/KubernetesClusterCreation.png)
2. Selecione o **Tipo de cluster** e clique em **Criar cluster** para provisionar um cluster Kubernetes.
3.  Verifique o status do **Cluster** e dos **Nós do trabalhador** e aguarde que eles estejam **prontos**.

### Configurar kubectl

Nesta etapa, você configurará kubectl para apontar para o seu cluster recém-criado que avança. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) é uma ferramenta de linha de comandos que você usa para interagir com um cluster Kubernetes.

1. Use `ibmcloud login` para efetuar login interativamente. Forneça a organização (org), a localização e o espaço sob os quais o cluster é criado. É possível reconfirmar os detalhes ao executar o comando `ibmcloud target`.
2. Quando o cluster estiver pronto, recupere a configuração de cluster definindo a variável de ambiente MYCLUSTER para o seu nome do cluster:
   ```bash
   export MYCLUSTER=<CLUSTER_NAME>
   ibmcloud cs cluster-config ${MYCLUSTER}
   ```
   {: pre}
3. Copie e cole o comando **export** para configurar a variável de ambiente KUBECONFIG como instruído. Para verificar se a variável de ambiente KUBECONFIG está configurada adequadamente ou não, execute o comando a seguir: `echo $KUBECONFIG`
4. Verificar se o comando `kubectl` está configurado corretamente
   ```bash
   kubectl cluster-info
   ```
   {: pre}
   ![](images/solution2/kubectl_cluster-info.png)


## Criar um cluster Starter
{: #create_application}

O conjunto de ferramentas `ibmcloud dev` reduz significativamente o tempo de desenvolvimento, gerando iniciadores do aplicativo com todo o código de modelo, construção e configuração necessário para que seja possível iniciar a codificação da lógica de negócios mais rapidamente.

1. Inicie o assistente `ibmcloud dev`.
   ```
   Ibmcloud dev criar
   ```
   {: pre}

1. Selecione `Backend Service / Web App` > `Java - MicroProfile / JavaEE` > `Java Web App with Eclipse MicroProfile and Java EE (Web App)` para criar um iniciador Java. (Para criar um iniciador Node.js, use `Backend Service / Web App` > `Node`> `Node.js Web App with Express.js (Web App)`)
1. Insira um **nome** para seu aplicativo.
1. Selecione o grupo de recursos no qual implementar esse aplicativo.
1. Não inclua serviços adicionais.
1. Não inclua uma cadeia de ferramentas do DevOps, selecione **implementação manual**.

Isso gera um aplicativo iniciador completo com o código e todos os arquivos de configuração necessários para o desenvolvimento e a implementação locais para a nuvem no Cloud Foundry ou Kubernetes. 

<!-- For an overview of the files generated, see [Project Contents Documentation](https://{DomainName}/docs/cloudnative/projects/java_project_contents.html#java-project-files). -->

![](images/solution2/Contents.png)

### Construir o aplicativo

É possível construir e executar o aplicativo, como você normalmente usaria `mvn` para desenvolvimento local java ou `npm` para desenvolvimento de nó.  Também é possível construir uma imagem do docker e executar o aplicativo em um contêiner para assegurar execução consistente localmente e na nuvem. Use as etapas a seguir para construir sua imagem do docker.

1. Assegure-se de que o mecanismo de Docker local esteja iniciado.
   ```
   docker ps
   ```
   {: pre}
2. Mude para o diretório de projeto gerado.
   ```
   cd <project name>
   ```
   {: pre}
3. Construa o aplicativo.
   ```
   ibmcloud dev build
   ```
   {: pre}

   Isso pode levar alguns minutos para ser executado, pois todas as dependências do aplicativo são transferidas por download e uma imagem do Docker, que contém seu aplicativo e todo o ambiente necessário, é construída.

### Executar o aplicativo localmente

1. Execute o contêiner.
   ```
   ibmcloud dev run
   ```
   {: pre}

   Isso usa o mecanismo de Docker local para executar a imagem do docker que você construiu na etapa anterior.
2. Depois que seu contêiner for iniciado, acesse `http://localhost:9080/`. Se você criou um aplicativo Node.js, acesse `http://localhost:3000/`.
   ![](images/solution2/LibertyLocal.png)

## Implementar o aplicativo no cluster usando o gráfico do Helm
{: #deploy}

Nesta seção, você primeiro envia por push a imagem do Docker para o registro de contêiner privado do IBM Cloud e, em seguida, cria uma implementação do Kubernetes apontando para essa imagem.

1. Localize seu **namespace** listando todo o namespace no registro.
   ```sh
   ibmcloud cr namespaces
   ```
   {: pre}
   Se você tiver um namespace, anote o nome para uso posterior. Se você não tiver um, crie-o.
   ```sh
   ibmcloud cr namespace-add <Name>
   ```
   {: pre}
2. Configure as variáveis de ambiente MYNAMESPACE e MYPROJECT para seu namespace e nome do projeto, respectivamente

    ```sh
    export MYNAMESPACE=<NAMESPACE>
    ```
    {: pre}
    ```sh
    export MYPROJECT=<PROJECT_NAME>
    ```
    {: pre}
3. Identifique seu **Registro de contêiner** (por exemplo, us.icr.io) executando `ibmcloud cr info`
4. Configure a variável de ambiente MYREGISTRY para seu registro.
   ```sh
   export MYREGISTRY=<REGISTRY>
   ```
   {: pre}

5. Construa e identifique (`-t`) a imagem do docker
   ```sh
   docker build . -t ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}

6. Envie por push a imagem do docker para seu registro de contêiner no IBM Cloud
   ```sh
   docker push ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}
7. Em um IDE, navegue para **values.yaml** em `chart\YOUR PROJECT NAME` e atualize o valor de **repositório de imagem** apontando para sua imagem no registro de contêiner do IBM Cloud. **Salve** o arquivo.

   Para obter detalhes do repositório de imagem, execute `echo ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}`

8. O [Helm](https://helm.sh/) ajuda a gerenciar aplicativos do Kubernetes por meio de Gráficos do Helm, que ajuda a definir, instalar e fazer upgrade até mesmo do aplicativo do Kubernetes mais complexo. Inicialize o Helm navegando para `chart\YOUR PROJECT NAME` e executando o comando a seguir em seu cluster

   ```bash
   helm init
   ```
   {: pre}
   Para fazer upgrade do helm, execute este comando `helm init --upgrade`
   {:tip}

9. Para instalar um gráfico do Helm, mude para o diretório `chart\YOUR PROJECT NAME` e execute o comando abaixo
  ```sh
  helm install . --name ${MYPROJECT}
  ```
  {: pre}
10. Use `kubectl get service ${MYPROJECT}-service` para seu aplicativo Java e `kubectl get service ${MYPROJECT}-application-service` para o seu aplicativo Node.js para identificar a porta pública na qual o serviço está atendendo. A porta é um número de 5 dígitos (por exemplo, 31569) em `PORT(S)`.
11. Para o IP público do nó do trabalhador, execute o comando abaixo
   ```sh
   ibmcloud cs workers ${MYCLUSTER}
   ```
   {: pre}
12. Acesse o aplicativo em `http://worker-ip-address:portnumber/`.

## Usar o domínio fornecido pela IBM para seu cluster
{: #ibm_domain}

Na etapa anterior, o aplicativo foi acessado com uma porta não padrão. O serviço foi exposto por meio do recurso NodePort do Kubernetes.

Os clusters pagos vêm com um domínio fornecido pela IBM. Isso fornece a você uma melhor opção para expor aplicativos com uma URL adequada e nas portas HTTP/S padrão.

Use o Ingresso para configurar a conexão de entrada de cluster para o serviço.

![Ingresso](images/solution2/Ingress.png)

1. Identifique seu **Domínio do Ingresso** fornecido pela IBM
   ```
   ibmcloud cs cluster-get ${MYCLUSTER}
   ```
   {: pre}
   para localizar
   ```
   Ingress subdomain:	mycluster.us-south.containers.appdomain.cloud
   Ingress secret:		mycluster
   ```
   {: screen}
2. Crie um arquivo de Ingresso `ingress-ibmdomain.yml` apontando para seu domínio com suporte para HTTP e HTTPS. Use o arquivo a seguir como um modelo, substituindo todos os valores agrupados em <> pelos valores apropriados da saída acima. O **service-name** é o nome sob `==> v1/Service` na etapa acima. Ou execute `kubectl get svc` para localizar o nome do serviço do tipo **NodePort**.
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-ibmdomain-http-and-https
   spec:
     tls:
     - hosts:
       -  <nameofproject>.<ingress-sub-domain>
       secretName: <ingress-secret>
     rules:
     - host: <nameofproject>.<ingress-sub-domain>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: codeblock}
3. Implementar o ingresso
   ```sh
   kubectl apply -f ingress-ibmdomain.yml
   ```
   {: pre}
4. Acesse seu aplicativo em `https://<nameofproject>.<ingress-sub-domain>/`

## Usar seu próprio domínio customizado
{: #custom_domain}

Para usar seu domínio customizado, é necessário atualizar seus registros DNS com um registro CNAME apontando para seu domínio fornecido pela IBM ou um registro A apontando para o endereço IP público móvel do Ingresso fornecido pela IBM. Considerando que um cluster pago é fornecido com endereços IP fixos, um registro A é uma boa opção.

Consulte [Usando o controlador de Ingresso com um domínio customizado](https://{DomainName}/docs/containers?topic=containers-ingress#ingress) para obter mais informações.

### Com HTTP

1. Crie um arquivo de Ingresso `ingress-customdomain-http.yml` apontando para seu domínio:
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-customdomain-http
   spec:
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
2. Implementar o ingresso
   ```sh
   kubectl apply -f ingress-customdomain-http.yml
   ```
   {: pre}
3. Acesse seu aplicativo em `http://<customdomain>/`

### Com HTTPS

Se você tentasse acessar seu aplicativo com HTTPS neste momento `https://<customdomain>/`, provavelmente obteria um aviso de segurança do seu navegador da web informando que a conexão não é privada. Você também obteria um 404, pois o Ingresso recém-configurado não saberia como direcionar o tráfego HTTPS.

1. Obtenha um certificado SSL confiável para seu domínio. Você precisará do certificado e da chave:
  https://{DomainName}/docs/containers?topic=containers-ingress#public_inside_3
   É possível usar [Let's Encrypt](https://letsencrypt.org/) para gerar o certificado confiável.
2. Salve o certificado e a chave nos arquivos de formato ascii base64.
3. Crie um segredo do TLS para armazenar o certificado e a chave:
   ```
   kubectl create secret tls my-custom-domain-secret-name --cert=<custom-domain.cert> --key=<custom-domain.key>
   ```
   {: pre}
4. Crie um arquivo de Ingresso `ingress-customdomain-https.yml` apontando para seu domínio:
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-customdomain-https
   spec:
     tls:
     - hosts:
       - <my-custom-domain.com>
       secretName: <my-custom-domain-secret-name>
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
5. Implemente o Ingresso:
   ```
   kubectl apply -f ingress-customdomain-https.yml
   ```
   {: pre}
6. Acesse seu aplicativo em `https://<customdomain>/`.

## Monitorar o funcionamento do aplicativo
{: #monitor_application}

1. Para verificar o funcionamento de seu aplicativo, navegue para [clusters](https://{DomainName}/containers-kubernetes/clusters) para ver uma lista de clusters e clique no cluster criado acima.
2. Clique em **Painel do Kubernetes** para ativar o painel em uma nova guia.
   ![](images/solution2/launch_kubernetes_dashboard.png)
3. Selecione **Nós** na área de janela esquerda, clique no **Nome** dos nós e consulte os **Recursos de alocação** para ver o funcionamento de seus nós.
   ![](images/solution2/KubernetesDashboard.png)
4. Para revisar os logs do aplicativo por meio do contêiner, selecione **Pods**, **pod-name** e **Logs**.
5. Para usar **ssh** no contêiner, identifique seu nome do pod da etapa anterior e execute
   ```sh
   kubectl exec -it <pod-name> -- bash
   ```
   {: pre}

## Escalar os pods do Kubernetes
{: #scale_cluster}

À medida que o carregamento aumenta em seu aplicativo, é possível aumentar manualmente o número de réplicas do pod em sua implementação. As réplicas são gerenciadas por um [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/). Para escalar o aplicativo para duas réplicas, execute o comando a seguir:

   ```sh
 kubectl scale deployment <nameofproject>-deployment --replicas=2
   ```
   {: pre}

Após algum tempo, você verá dois pods para seu aplicativo no painel do Kubernetes (ou com `kubectl get pods`). O controlador de Ingresso no cluster manipulará o balanceamento de carga entre as duas réplicas. O ajuste de escala horizontal também pode ser feito automaticamente.

Consulte a documentação do Kubernetes para ajuste de escala manual e automático:

   * [Escalando uma implementação](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
   * [Ajuste automático de escala do pod horizontal](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## Remover recursos

* Exclua o cluster ou exclua somente os artefatos do Kubernetes criados para o aplicativo se você planejar reutilizar o cluster.

## Conteúdo relacionado

* [IBM Cloud
Kubernetes Service](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
<!-- * [IBM Cloud App Service](https://{DomainName}/docs/cloudnative/index.html#web-mobile) -->
* [Implementação contínua para o Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
