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

# Aplicativo da web PHP em uma pilha LAMP
{: #lamp-stack}

Este tutorial conduz você na criação de um servidor virtual Ubuntu **L**inux com o servidor da web **A**pache, banco de dados **M**ySQL e script **P**HP. Essa combinação de software, mais comumente chamada de pilha LAMP, é muito popular e muitas vezes usada para entregar websites e aplicativos da web. Usando o {{site.data.keyword.BluVirtServers}}, você implementará rapidamente sua pilha LAMP com monitoramento integrado e varredura de vulnerabilidade. Para ver o servidor LAMP em ação, você instalará e configurará o sistema de gerenciamento de conteúdo do [WordPress](https://wordpress.org/) grátis e de software livre.

## Objetivos

* Fornecer um servidor LAMP em minutos
* Aplicar a versão mais recente do Apache, MySQL e PHP
* Hospedar um website ou blog instalando e configurando o WordPress
* Utilizar o monitoramento para detectar indisponibilidades e desempenho lento
* Avaliar vulnerabilidades e proteger do tráfego indesejado

## Serviços usados

Este tutorial usa os tempos de execução e serviços a seguir:

* [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura

![Diagrama de arquitetura](images/solution4/Architecture.png)

1. O usuário final acessa o servidor LAMP e os aplicativos usando um navegador da web

## Antes de Começar

{: #prereqs}

1. Entre em contato com o administrador de infraestrutura para obter as permissões a seguir.
  * Permissão de rede necessária para concluir o **Uplink de rede pública e privada**

### Configurar o acesso VPN

1. [Assegure-se de que seu acesso VPN seja permitido](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Você deve ser um **Usuário principal** para ativar o acesso VPN ou contatar o usuário principal para obter acesso.
     {:tip}
2. Obtenha suas credenciais de Acesso à VPN em [sua página de usuário sob a lista Usuários](https://{DomainName}/iam#/users).
3. Efetue login na VPN por meio da [interface da web](https://www.softlayer.com/VPN-Access) ou use um cliente VPN para [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) ou [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Para o cliente VPN, use o FQDN de um único ponto de acesso VPN do data center na [página de acesso da web VPN](https://www.softlayer.com/VPN-Access), no formato *vpn.xxxnn.softlayer.com* como o Endereço do gateway.
   {:tip}

## Criar serviços

Nesta seção, você provisionará um servidor virtual público com uma configuração fixa. Os {{site.data.keyword.BluVirtServers_short}} podem ser implementados em uma questão de minutos por meio de imagens de servidor virtual em locais geográficos específicos. Os servidores virtuais geralmente abordam picos de demanda após os quais eles podem ser suspensos ou desligados para que o ambiente de nuvem se ajuste perfeitamente às suas necessidades de infraestrutura.

1. Em seu navegador, acesse a página de catálogo de [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group).
2. Selecione **Servidor virtual público** e clique em **Criar**.
3. Em **Imagem**, selecione a versão mais recente do **LAMP** sob **Ubuntu**. Embora ele venha pré-instalado com o Apache, MySQL e PHP, você reinstalará o PHP e o MySQL com a versão mais recente.
4. Em **Interface de rede**, selecione a opção **Uplink de rede pública e privada**.
5. Revise as outras opções de configuração e clique em **Provisionar** para criar seu servidor virtual.
   ![Configure o servidor virtual](images/solution4/ConfigureVirtualServer.png)

Depois que o servidor for criado, você verá as credenciais de login do servidor. Embora seja possível se conectar por meio de SSH usando o endereço IP público do servidor, é recomendado acessar o servidor por meio da Rede Privada e para desativar o acesso SSH na rede pública.

1. Siga [estas etapas](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-restricting-ssh-access-on-a-public-network#restricting-ssh-access-on-a-public-network) para proteger a máquina virtual e para desativar o acesso SSH na rede pública.
1. Usando seu nome do usuário, senha e endereço IP privado, conecte-se ao servidor com SSH.
   ```sh
   sudo ssh root@<Private-IP-Address>
   ```
   {: pre}

  É possível localizar o endereço IP privado do servidor e a senha no painel.
  {:tip}

  ![Servidor virtual criado](images/solution4/VirtualServerCreated.png)

## Reinstalar o Apache, o MySQL e o PHP

É aconselhável atualizar a pilha LAMP com as correções de segurança mais recentes e correções de erro periodicamente. Nesta seção, você executará comandos para atualizar as origens de pacote Ubuntu e reinstalar o Apache, MySQL e PHP com a versão mais recente. Observe o acento circunflexo (^) no final do comando.

```sh
sudo apt update && sudo apt install lamp-server^
```
{: pre}

Uma opção alternativa é fazer upgrade de todos os pacotes com `sudo apt-get update && sudo apt-get dist-upgrade`.
{:tip}

## Verificar a instalação e a configuração

Nesta seção, você verificará se o Apache, o MySQL e o PHP estão atualizados e em execução na imagem do Ubuntu. Você também implementará as configurações de segurança recomendadas para MySQL.

1. Verifique o Ubuntu abrindo o endereço IP público no navegador. Você deverá ver a página de boas-vindas do Ubuntu.
   ![Verificar o Ubuntu](images/solution4/VerifyUbuntu.png)
2. Verifique se a porta 80 está disponível para o tráfego da web executando o comando a seguir.
   ```sh
   sudo netstat -ntlp | grep LISTEN
   ```
   {: pre}
   ![Verificar a porta](images/solution4/VerifyPort.png)
3. Revise as versões do Apache, MySQL e PHP instaladas usando os comandos a seguir.
  ```sh
  apache2 -v
  ```
  {: pre}
  ```sh
  mysql -V
  ```
  {: pre}
  ```sh
   php -v
  ```
   {: pre}
4. Execute o script a seguir para proteger o banco de dados MySQL.
  ```sh
  mysql_secure_installation
  ```
  {: pre}
5. Insira a senha raiz do MySQL e defina as configurações de segurança para seu ambiente. Quando estiver pronto, saia do prompt mysql digitando `\q`.
  ```sh
  mysql -u root -p
  ```
  {: pre}

  O nome do usuário e a senha padrão do MySQL são raiz e raiz.
  {:tip}
6. Além disso, é possível criar rapidamente uma página de informações do PHP com o comando a seguir.
   ```sh
   sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
   ```
   {: pre}
7. Visualize a página de informações do PHP que você criou: abra um navegador e acesse `http://{YourPublicIPAddress}/info.php`. Substitua o endereço IP público de seu servidor virtual. Ele será semelhante à imagem a seguir.

![Informações do PHP](images/solution4/PHPInfo.png)

### Instalar e configurar o WordPress

Experimente sua pilha LAMP instalando um aplicativo. As etapas a seguir instalam a plataforma WordPress de software livre, que é frequentemente usada para criar websites e blogs. Para obter mais informações e configurações para instalação de produção, consulte a [Documentação do WordPress](https://codex.wordpress.org/Main_Page).

1. Execute o comando a seguir para instalar o WordPress.
   ```sh
   sudo apt install wordpress
   ```
   {: pre}
2. Configure o WordPress para usar o MySQL e o PHP. Execute o comando a seguir para abrir um editor de texto e criar o arquivo `/etc/wordpress/config-localhost.php`.
   ```sh
   sudo sensible-editor /etc/wordpress/config-localhost.php
   ```
   {: pre}
3. Copie as linhas a seguir para o arquivo substituindo *yourPassword* por sua senha do banco de dados MySQL e deixando os outros valores sem mudança. Salve e saia do arquivo usando `Ctrl + X`.
   ```php
   <?php
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'wordpress');
   define('DB_PASSWORD', 'yourPassword');
   define('DB_HOST', 'localhost');
   define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
   ?>
   ```
   {: pre}
4. Em um diretório ativo, crie um arquivo de texto `wordpress.sql` para configurar o banco de dados WordPress.
   ```sh
   sudo sensible-editor wordpress.sql
   ```
   {: pre}
5. Inclua os comandos a seguir substituindo sua senha do banco de dados por *yourPassword* e deixando os outros valores sem mudança. Em seguida, salve o arquivo.
   ```sql
   CREATE DATABASE wordpress;
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*
   TO wordpress@localhost
   IDENTIFIED BY 'yourPassword';
   FLUSH PRIVILEGES;
   ```
   {: pre}
6. Execute o comando a seguir para criar o banco de dados.
   ```sh
   cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
   ```
   {: pre}
7. Depois que o comando for concluído, exclua o arquivo `wordpress.sql`. Mova a instalação do WordPress para a raiz do documento do servidor da web.
   ```sh
   sudo ln -s /usr/share/wordpress /var/www/html/wordpress
   sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
   ```
   {: pre}
8. Conclua a configuração do WordPress e publique na plataforma. Abra um navegador e acesse `http://{yourVMPublicIPAddress}/wordpress`. Substitua o endereço IP público de sua VM. Ele deve ser semelhante à imagem a seguir.
   ![Site do WordPress em execução](images/solution4/WordPressSiteRunning.png)

## Configurar domínio

Para usar um nome de domínio existente com seu servidor LAMP, atualize o registro A para apontar para o endereço IP público do servidor virtual.
É possível visualizar o endereço IP público do servidor por meio do painel.

## Monitoramento e uso do servidor

Para assegurar a disponibilidade do servidor e a melhor experiência do usuário, o monitoramento deve ser ativado em cada servidor de produção. Nessa seção, você explorará as opções que estão disponíveis para monitorar seu servidor virtual e entender o uso do servidor em qualquer momento.

### Monitoramento do servidor

Dois tipos de monitoramento básico estão disponíveis: SERVICE PING e SLOW PING.

* **SERVICE PING** verifica se o tempo de resposta do servidor é igual a 1 segundo ou menos
* **SLOW PING** verifica se o tempo de resposta do servidor é igual a 5 segundos ou menos

Como o SERVICE PING é incluído por padrão, inclua o monitoramento SLOW PING com as etapas a seguir.

1. No painel, selecione seu servidor na lista de dispositivos e, em seguida, clique na guia **Monitoramento**.
   ![Monitoramento de ping lento](images/solution4/SlowPing.png)
2. Clique em **Gerenciar monitores**.
3. Inclua a opção de monitoramento **SLOW PING** e clique em **Incluir monitor**. Selecione seu endereço IP público para o endereço IP.
   ![Incluir monitoramento de ping lento](images/solution4/AddSlowPing.png)

  **Nota**: os monitores duplicados com as mesmas configurações não são permitidos. Somente um monitor por configuração pode ser criado.

Se uma resposta não for recebida no prazo atribuído, um alerta será enviado para o endereço de e-mail na conta do {{site.data.keyword.Bluemix_notm}}.
    ![Dois monitoramentos](images/solution4/TwoMonitoring.png)

### Uso do servidor

Selecione a guia **Uso** para entender a memória do servidor atual e o uso da CPU.
  ![Uso do servidor](images/solution4/ServerUsage.png)

## Segurança do servidor

O {{site.data.keyword.BluVirtServers}} fornece várias opções de segurança, como varredura de vulnerabilidade e firewalls de complemento.

### Scanner de vulnerabilidade

O scanner de vulnerabilidade varre o servidor em busca de quaisquer vulnerabilidades relacionadas ao servidor. Para executar uma varredura de vulnerabilidade no servidor, siga as etapas abaixo.

1. No painel, selecione seu servidor e, em seguida, clique na guia **Segurança**.
2. Clique em **Varrer** para iniciar a varredura.
3. Depois que a varredura for concluída, clique em **Varredura concluída** para visualizar o relatório de varredura.
   ![Dois monitoramentos](images/solution4/Vulnerabilities.png)
4. Revise quaisquer vulnerabilidades relatadas.
   ![Dois monitoramentos](images/solution4/VulnerabilityResults.png)

### Firewalls

Outra maneira de proteger o servidor é incluindo um firewall. Os firewalls fornecem uma camada de segurança essencial: evitando que o tráfego indesejado atinja seus servidores, reduzindo a probabilidade de um ataque e permitindo que os recursos do servidor sejam dedicados para seu uso desejado. As opções de firewall são provisionadas on demand sem interrupções de serviço.

Os firewalls estão disponíveis como um recurso complementar para todos os servidores na rede pública de Infraestrutura. Como parte do processo de pedido, é possível selecionar um hardware específico do dispositivo ou um firewall de software para fornecer proteção. Como alternativa, é possível implementar dispositivos de firewall dedicados no ambiente e implementar o servidor virtual em uma VLAN protegida. Para obter mais informações, consulte [Firewalls](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-getting-started-with-hardware-firewall-dedicated#getting-started).

## Remover recursos

Para remover seu servidor virtual, conclua as etapas a seguir.

1. Efetue login no [{{site.data.keyword.slportal}}](https://{DomainName}/classic/devices).
2. No menu **Dispositivos**, selecione **Lista de dispositivos**.
3. Clique em **Ações** para o servidor virtual que você deseja remover e selecione **Cancelar**.

## Conteúdo relacionado

* [Implementar uma pilha LAMP usando o Terraform](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
