---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-23"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Implementar uma pilha LAMP usando o Terraform
{: #infrastructure-as-code-terraform}

O [Terraform](https://www.terraform.io/) permite criar, mudar e melhorar a infraestrutura de forma segura e previsível. Ele é uma ferramenta de software livre que codifica as APIs em arquivos de configuração declarativos que podem ser compartilhados entre membros da equipe, tratados como código, editados, revisados e com versão.

Neste tutorial, você usará uma configuração de amostra para provisionar um servidor virtual **L**inux, com o servidor da web **A**pache, o **M**ySQL e o servidor **P**HP denominado como pilha **LAMP**. Em seguida, você atualizará a configuração para incluir o serviço {{site.data.keyword.cos_full_notm}} e escalar os recursos para ajustar o ambiente (memória, CPU e tamanho do disco). Conclua excluindo todos os recursos criados pela configuração.

## Objetivos
{: #objectives}

* Configurar o Terraform e o {{site.data.keyword.Bluemix_notm}} Provider for Terraform.
* Usar o Terraform para criar, atualizar, escalar e finalmente destruir uma configuração de pilha LAMP.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.virtualmachinesshort}}
](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

  ![Diagrama de arquitetura](images/solution10/architecture-2.png)
</p>

1. Um conjunto de arquivos do Terraform é criado para descrever a configuração da pilha LAMP.
1. `terraform` é chamado por meio do diretório de configuração.
1. `terraform` chama a API do {{site.data.keyword.cloud_notm}} para provisionar os recursos.

## Antes de Começar
{: #prereqs}

Entre em contato com o usuário principal da infraestrutura para obter as permissões a seguir:
- Rede (para incluir **Uplink de rede pública e privada**)
- Chave de API

## Pré-requisitos

{: #prereq}

Instale o **Terraform** por meio do [instalador](https://www.terraform.io/intro/getting-started/install.html) ou use [Homebrew](https://brew.sh/) no macOS executando o comando: `brew install terraform`

No **Windows**, siga estas etapas para concluir a configuração do terraform:
   1. Copie os arquivos do zip transferido por download para `C:\terraform` (crie uma pasta `terraform`).
   2. Abra o prompt de comandos como um administrador e configure o PATH para usar binários do terraform.

      ```
      set PATH=%PATH%;C:\terraform
      ```
      {:pre}

## Configurar o {{site.data.keyword.Bluemix_notm}} Provider for Terraform
{: #setup}

Para suportar uma abordagem multinuvem, o Terraform funciona com os provedores. Um provedor é responsável por entender as interações da API e expor os recursos. Nesta seção, você configurará a CLI para especificar o local do provedor {{site.data.keyword.Bluemix_notm}}.

1. Verifique a instalação do Terraform executando `terraform` em seu terminal ou janela de prompt de comandos. Você deverá ver uma lista de `Common commands`.

  ```
  terraform
  ```

2. Faça download do plug-in [{{site.data.keyword.Bluemix_notm}} Provider](https://github.com/IBM-Cloud/terraform-provider-ibm/releases) apropriado para seu sistema e extraia o archive. Você deverá ver o arquivo de plug-in binário `terraform-provider-ibm_VERSION`.

3. Para sistemas não Windows, crie um diretório `.terraform.d/plugins` no diretório inicial de seu usuário e coloque o arquivo binário nele. Use os comandos a seguir para referência.

  ```
  mkdir -p $HOME/.terraform.d/plugins
  mv $HOME/Downloads/terraform-provider-ibm_VERSION $HOME/.terraform.d/plugins/
  ```

    No **Windows**, o arquivo precisa ser colocado em `terraform.d/plugins` sob o diretório "Application Data" do usuário.

  - Execute os comandos abaixo em uma [Configuração do provedor](https://www.terraform.io/docs/configuration/providers.html) do prompt de comandos
   ```
  MD %USERPROFILE%\AppData\terraform.d\plugins
   ```
  ```
   MOVE PATH_TO_UNZIPPED_PROVIDER_FILE\terraform-provider-ibm_VERSION.exe  %USERPROFILE%\AppData\terraform.d\plugins
  ```
   - Ative o **Windows Powershell** (Iniciar + R > Powershell) e execute o comando a seguir para criar o arquivo `terraform.rc`
   ```
    echo > $env:APPDATA\terraform.rc
   ```
   No primeiro prompt, insira o conteúdo abaixo
   ```
    # ~/.terraformrc providers {
        ibm = "PATH_TO_YOUR_APPDATA_PLUGINS/terraform-provider-ibm_VERSION.exe"
    }
   ```
        O PATH_TO_YOUR_APPDATA_PLUGINS deve ser um caminho absoluto com barra (/). Por exemplo, `C:/Users/VMac/AppData/terraform.d/plugins/terraform-provider-ibm.exe`
        {: tip}

  - Clique em Enter para sair do prompt.

## Preparar a configuração do terraform

{: #terraformconfig}

Nesta seção, você aprenderá os fundamentos de uma configuração do terraform usando uma configuração do Terraform de amostra fornecida pelo {{site.data.keyword.Bluemix_notm}}.

1. Visite https://github.com/IBM-Cloud/LAMP-terraform-ibm e **Bifurque** sua própria cópia para sua conta.
2. Clone sua bifurcação localmente:
   ```bash
   git clone https://github.com/YOUR_USER_NAME/LAMP-terraform-ibm
   ```
3. Inspecione os arquivos de configuração
   - [install.yml](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/install.yml) - contém configurações de instalação do servidor, aqui é possível incluir todos os scripts relacionados à instalação do servidor para o que instalar no servidor. Consulte `phpinfo();` injetado nesse arquivo.
   - [provider.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/provider.tf) - contém as variáveis relacionadas ao provedor no qual o nome do usuário do provedor e a chave de API são necessários.
   - [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) -contém as configurações do servidor para implementar a VM com variáveis especificadas.
   - [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) - contém o nome do usuário e a chave de API do **SoftLayer**, Chave de API do {{site.data.keyword.Bluemix_notm}} e seus nomes de espaço/organização. Essas credenciais podem ser incluídas nesse arquivo para melhores práticas para evitar a reinserção dessas credenciais por meio da linha de comandos toda vez ao implementar o servidor. Nota: NÃO publique esse arquivo com suas credenciais.
4. Abra o arquivo [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) em um IDE de sua escolha e modifique o arquivo incluindo sua chave **SSH pública**. Isso será usado para acessar a VM criada por essa configuração. Para obter informações sobre a criação de chaves SSH, siga as instruções [neste link](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html). Para copiar a chave pública para sua área de transferência, é possível executar o comando abaixo em seu terminal.

     ```bash
     pbcopy < ~/.ssh/id_rsa.pub
     ```
     Esse comando copiará o SSH para sua área de transferência. Em seguida, será possível passá-lo para o [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) sob a variável padrão `ssh_key` ao redor da linha 69.

    No **Windows**, faça download, instale, ative o [Git Bash](https://git-scm.com/download/win) e execute o comando abaixo para copiar a chave SSH pública para sua área de transferência.

    ```
     clip < ~/.ssh/id_rsa.pub
    ```

5. Abra o arquivo [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) com seu IDE, modifique o arquivo incluindo todas as credenciais listadas. A inclusão dessas credenciais nesses arquivos significa que não será necessário reinserir essas credenciais toda vez que a execução do terraform se aplicar. Deve-se incluir todas as cinco credenciais listadas no arquivo [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) para concluir o restante deste tutorial.

## Criar um servidor de pilha LAMP por meio da configuração do terraform
{: #Createserver}
Nesta seção, você aprenderá como criar um servidor de pilha LAMP por meio da amostra de configuração do terraform. A configuração é usada para provisionar uma instância de máquina virtual e instalar o **A**pache, o **M**ySQL (**M**ariaDB) e o **P**HP nessa instância.

1. Navegue para a pasta do repositório clonado.
   ```bash
    cd LAMP-terraform-ibm
   ```
   {: pre}
2. Inicialize a configuração do terraform. Isso também instalará o plug-in `terraform-provider-ibm_VERSION`.
   ````bash
    terraform init
   ````
   {: pre}
3. Aplique a configuração do terraform. Isso criará os recursos definidos na configuração.
   ```
    terraform apply
   ```
   {: pre}
   Você deverá ver uma saída semelhante àquela abaixo.![URL do controle de fonte](images/solution10/created.png)
4. Em seguida, vá para a sua [lista de dispositivos de infraestrutura](https://{DomainName}/classic/devices) para verificar se o servidor foi criado.![URL do controle de fonte](images/solution10/configuration.png)

## Inclua o serviço {{site.data.keyword.cos_full_notm}} e escale os recursos

{: #modify}

Nesta seção, você examinará como escalar o recurso do servidor virtual e incluir um serviço [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) em seu ambiente de infraestrutura.

1. Edite o arquivo `vm.tf` para aumentar o seguinte e salve o arquivo.
 - Aumentar número de núcleos da CPU para 4 núcleos
 - Aumentar a RAM (memória) para 4096
 - Aumentar o tamanho do disco para 100 GB

2. Em seguida, inclua um novo serviço [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage). Para fazer isso, crie um novo arquivo e nomeie-o como **ibm-cloud-object-storage.tf**. Inclua os fragmentos de código abaixo no arquivo recém-criado. Os fragmentos de código abaixo criam um nome de variável para o nome da organização e o nome do espaço, em seguida, esses dois nomes de variável são usados para recuperar o guid de espaço no qual foi necessário criar o serviço. Ele configura o nome do serviço {{site.data.keyword.cos_full_notm}} como `lamp_objectstorage`, em seguida, você precisa de um guid de espaço, um nome completo do serviço e um tipo de plano. O código abaixo criará um plano premium, já que é um plano de pagamento conforme o uso, de qualquer maneira. Também é possível usar o plano Lite, mas observe que o plano Lite é limitado a somente um serviço por conta.

   ```
   variable "org_name" {
     description = "Enter your IBM Cloud org name, you can get your org name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   variable "space_name" {
     description = "Enter your IBM Cloud space name, you can get your space name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   data "ibm_space" "space" {
     space = "${var.space_name}"
     org   = "${var.org_name}"
   }

   # a cloud object storage
   resource "ibm_service_instance" "objectstorage" {
     name       = "lamp_objectstorage"
     space_guid = "${data.ibm_space.space.id}"
     service    = "cloud-object-storage"

     # you can only have one Lite plan per account so let's use the Premium - it is pay-as-you-go
     plan = "Premium"
   }
   ```
   {: pre}
   **Nota:** o rótulo "lamp_objectstorage", nós o procuraremos posteriormente nos logs para garantir que o {{site.data.keyword.cos_full_notm}} tenha sido criado com êxito.

3. Inicialize a configuração do terraform novamente, executando:

   ```bash
    terraform init
   ```
   {: pre}

4. Aplique as mudanças do terraform executando:
   ```bash
    terraform apply
   ```
   {: pre}
   **Nota:** depois de executar o comando terraform apply com êxito, você deverá ver um novo arquivo `terraform.tfstate` incluído em seu diretório. Esse arquivo contém a confirmação de implementação completa para manter o controle do que você aplicou pela última vez e quaisquer modificações futuras em sua configuração. Se esse arquivo for removido ou perdido, você perderá suas configurações de implementação do terraform.

## Verificar a VM e o {{site.data.keyword.cos_short}}
{: #verifyvm}

Nesta seção, você verificará a VM e o {{site.data.keyword.cos_short}} para certificar-se de que ela tenha sido criada com êxito.

**Verificar VM**

1. No menu do lado esquerdo, clique em **Infraestrutura** para visualizar a lista de dispositivos de servidor virtual.
2. Clique em **Dispositivos** -> **Lista de dispositivos** para localizar o servidor criado. Seu dispositivo do servidor deve ser listado.
3. Clique no servidor para visualizar mais informações sobre a configuração do servidor. Examinando a captura de tela abaixo, é possível ver que o servidor foi criado com êxito. ![URL do controle de fonte](images/solution10/configuration.png)
4. Em seguida, vamos testar o servidor no navegador da web. Abra o endereço IP público do servidor no navegador da web. Você deverá ver a página de instalação padrão do servidor, como a seguir.![URL do controle de fonte](images/solution10/LAMP.png)


**Verificar o {{site.data.keyword.cos_full_notm}}**

1. No **Painel do {{site.data.keyword.Bluemix_notm}}**, você deverá ver que uma instância do serviço {{site.data.keyword.cos_full_notm}} foi criada para você e está pronta para uso. ![object-storage](images/solution10/ibm-cloud-object-storage.png)

   Mais informações sobre o {{site.data.keyword.cos_full_notm}} podem ser localizadas [aqui](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage).

## Remover recursos
{: #deleteresources}

Exclua os recursos usando o comando a seguir:
   ```bash
   terraform destroy
   ```
   {: pre}

**Nota:** para excluir recursos, você precisaria de permissões de administrador de Infraestrutura. Se você não tiver uma conta de superusuário administrador, solicite para cancelar os recursos usando o painel de infraestrutura. É possível solicitar para cancelar um dispositivo no painel de infraestrutura sob os dispositivos. ![object-storage](images/solution10/rm.png)

## Conteúdo relacionado

- [Terraform](https://www.terraform.io/)
- [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
- [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
- [Acelerar a entrega de arquivos estáticos usando um CDN - {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)

