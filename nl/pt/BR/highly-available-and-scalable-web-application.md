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

# Usar Virtual Servers para construir um app da web altamente disponível e escalável
{: #highly-available-and-scalable-web-application}

Incluir mais servidores em um aplicativo é um padrão comum para manipular a carga adicional. Outro aspecto chave para aumentar a disponibilidade e a resiliência do aplicativo é implementar o aplicativo em múltiplas zonas ou locais com replicação de dados e balanceamento de carga.

Este tutorial conduz você por um cenário com a criação de:

- Dois servidores de aplicativos da web em Dallas.
- Cloud Load Balancer, para balancear a carga do tráfego de dois servidores dentro de um local.
- Um servidor de banco de dados MySQL.
- Um armazenamento de arquivo durável para armazenar arquivos e backups de aplicativo.
- Configure o segundo local com as mesmas configurações da primeira vez e, em seguida, inclua o Cloud Internet Services para apontar o tráfego para o local funcional se uma cópia falhar.

## Objetivos
{: #objectives}

* Criar {{site.data.keyword.virtualmachinesshort}} para instalar o PHP e o MySQL
* Usar o {{site.data.keyword.filestorage_short}} para persistir arquivos de aplicativo e backups de banco de dados
* Provisionar um {{site.data.keyword.loadbalancer_short}} para distribuir solicitações para os servidores de aplicativos
* Ampliar a solução incluindo um segundo local para melhor resiliência e disponibilidade mais alta

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

O aplicativo é um front-end do PHP simples, um blog do Wordpress, com um banco de dados MySQL. Vários servidores front-end manipulam as solicitações.

<p style="text-align: center;">
  ![Diagrama de arquitetura](images/solution14/Architecture.png)
</p>

1. O usuário se conecta ao aplicativo.
2. O {{site.data.keyword.loadbalancer_short}} seleciona um dos servidores funcionais para manipular a solicitação.
3. O servidor eleito acessa os arquivos de aplicativo armazenados em um armazenamento de arquivo compartilhado.
4. O servidor também puxa informações do banco de dados e, finalmente, renderiza a página para o usuário.
5. Em um intervalo regular, o conteúdo do banco de dados é submetido a backup. Um servidor de banco de dados de espera está disponível no caso de o principal falhar.

## Antes de Começar
{: #prereqs}

### Configurar o acesso VPN

Neste tutorial, o balanceador de carga é a porta frontal para os usuários do aplicativo. As {{site.data.keyword.virtualmachinesshort}} não precisam estar visíveis na Internet pública. Portanto, elas podem ser provisionadas com somente um endereço IP privado e você usará a conexão VPN para trabalhar nos servidores.

1. [Assegure-se de que seu acesso VPN seja permitido](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Você deve ser um **Usuário principal** para ativar o acesso VPN ou contatar o usuário principal para obter acesso.
     {:tip}
2. Obtenha suas credenciais de Acesso VPN selecionando seu usuário na [Lista de usuários](https://{DomainName}/iam#/users).
3. Efetue login na VPN por meio da [interface da web](https://www.softlayer.com/VPN-Access) ou use um cliente VPN para [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) ou [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

É possível escolher ignorar esta etapa e tornar todos os seus servidores visíveis na Internet pública (embora mantê-los privados forneça um nível adicional de segurança). Para torná-los públicos, selecione **Uplink de rede pública e privada** ao provisionar {{site.data.keyword.virtualmachinesshort}}.
{: tip}

### Verificar permissões da conta

Entre em contato com o usuário principal da infraestrutura para obter as permissões a seguir:
- **Rede** para que seja possível criar {{site.data.keyword.virtualmachinesshort}} com **Uplink de rede pública e privada** (essa permissão não será necessária se você usar a VPN para se conectar aos servidores)

## Provisionar um servidor para o banco de dados
{: #database_server}

Nesta seção, você configura um servidor para agir como o banco de dados principal.

1. Acesse o catálogo no console do {{site.data.keyword.Bluemix}} e selecione [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) na seção Infraestrutura.
2. Selecione **Virtual Server público** e, em seguida, clique em **Criar**.
3. Configure o servidor com o seguinte:
   - Configure **Nome** como **db1**
   - Selecione um local no qual provisionar o servidor. **Todos os outros servidores e recursos criados neste tutorial precisarão ser criados no mesmo local.**
   - Selecione a imagem **Ubuntu Minimal**
   - Mantenha o tipo de cálculo padrão. O tutorial foi testado com o menor tipo, mas deve funcionar com qualquer tipo.
   - Em **Discos de armazenamento conectados**, selecione o disco de inicialização de 25 GB.
   - Em **Interface de rede**, selecione a opção **Uplink de rede privada de 100 Mbps**.

     Se você não configurou o Acesso VPN, selecione a opção **Uplink de rede pública e privada de 100 Mbps**.
     {: tip}
   - Revise as outras opções de configuração e clique em **Provisionar** para provisionar o servidor.

      ![Configurar o servidor virtual](images/solution14/db-server.png)

   Nota: o processo de fornecimento pode levar de 2 a 5 minutos para que o servidor esteja pronto para uso. Depois que o servidor for criado, você localizará as credenciais do servidor na página de detalhes do servidor em **Dispositivos > Lista de dispositivos**. Para usar SSH no servidor, você precisa do endereço IP público e privado do servidor, o nome do usuário e a senha (clique na seta ao lado do nome do dispositivo).
   {: tip}

## Instalar e configurar o MySQL
{: #mysql}

O servidor não vem com um banco de dados. Nesta seção, você instalará o MySQL no servidor.

### Instalar o MySQL

1. Conecte-se ao servidor usando SSH:
   ```sh
   ssh root@<Private-OR-Public-IP-Address>
   ```

   Lembre-se de conectar-se ao cliente VPN com o [endereço do site](https://www.softlayer.com/VPN-Access) correto com base no **Local** de seu servidor virtual.
   {:tip}
2. Instalar o MySQL:
   ```sh
   apt-get update
   apt-get -y install mysql-server
   ```

   Você pode ser solicitado a fornecer uma senha. Leia as instruções sobre o console mostradas.
   {:tip}
3. Execute o script a seguir para ajudar a proteger o banco de dados MySQL:
   ```sh
   mysql_secure_installation
   ```

   Você pode ser solicitado a fornecer algumas opções. Escolha sabiamente com base em seus requisitos.
   {:tip}

### Criar um banco de dados para o aplicativo

1. Efetue login no MySQL e crie um banco de dados chamado `wordpress`:
   ```sh
   mysql -u root -p
   CREATE DATABASE wordpress;
   ```

2. Conceda acesso ao banco de dados, substitua database-username e database-password pelos que você configurou anteriormente.

   ```
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO database-username@'%' IDENTIFIED BY 'database-password';
   ```
   ```
   FLUSH PRIVILEGES;
   ```

3. Verifique se o banco de dados foi criado usando:

   ```sh
   show databases;
   ```

4. Saia do banco de dados usando:

   ```sh
   exit
   ```

5. Anote o nome do banco de dados, o usuário e a senha. Você precisará deles ao configurar os servidores de aplicativos.

### Tornar o servidor MySQL visível para outros servidores na rede

Por padrão, o MySQL atende somente na interface local. Os servidores de aplicativos precisarão se conectar ao banco de dados, portanto, a configuração do MySQL precisa ser mudada para atender nas interfaces de rede privada.

1. Edite o arquivo my.cnf usando `nano /etc/mysql/my.cnf` e inclua estas linhas:
   ```
   [mysqld]
   bind-address    = 0.0.0.0
   ```

2. Saia e salve o arquivo usando Ctrl + X.

3. Reinicie o MySQL:

   ```sh
   systemctl restart mysql
   ```

4. Confirme que o MySQL está atendendo em todas as interfaces, executando o comando a seguir:
   ```sh
   netstat --listen --numeric-ports | grep 3306
   ```

## Criar um armazenamento de arquivo para backups de banco de dados
{: #database_backup}

Há muitas maneiras nas quais os backups podem ser feitos e armazenados quando se trata de MySQL. Este tutorial usa uma entrada crontab para fazer dump do conteúdo do banco de dados em disco. Os arquivos de backup serão armazenados em um armazenamento de arquivo. Obviamente, esse é um mecanismo de backup simples. Se planejar gerenciar seu próprio servidor de banco de dados MySQL em um ambiente de produção, você desejará [implementar uma das estratégias de backup descritas na documentação do MySQL](https://dev.mysql.com/doc/refman/5.7/en/backup-and-recovery.html).

### Criar o armazenamento de arquivo
{: #create_for_backup}

1. Acesse o catálogo no console do {{site.data.keyword.Bluemix}} e selecione [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
2. Clique em **Criar**
3. Configure o serviço com o seguinte:
   - Configure **Tipo de armazenamento** para **Endurance**
   - Selecione o mesmo **Local** que aquele em que você criou o servidor de banco de dados
   - Selecione um método de faturamento
   - Em **Pacotes de armazenamento**, selecione **0,25 IOPS/GB**
   - Em **Tamanho do armazenamento**, selecione **20 GB**
   - Mantenha o **Tamanho do espaço de captura instantânea** para **0 GB**
   - Clique em Continuar para criar o serviço.

### Autorizar o servidor de banco de dados a usar o armazenamento de arquivo

Antes que um servidor virtual possa montar um File Storage, ele precisa ser autorizado.

1. Selecione o File Storage recém-criado na [lista de itens existentes](https://{DomainName}/classic/storage/file).
2. Em **Hosts autorizados**, clique em **Autorizar host** e selecione o servidor virtual (banco de dados) (escolha **Dispositivos** > Servidor virtual como tipo de dispositivo > Digite o nome do servidor).

### Monte o armazenamento de arquivo para backups de banco de dados

O File Storage pode ser montado como uma unidade NFS no servidor virtual.

1. Instale as bibliotecas do cliente NFS:
   ```sh
   apt-get -y install nfs-common
   ```

2. Crie um arquivo chamado `/etc/systemd/system/mnt-database.mount`, executando o comando a seguir
   ```bash
   touch /etc/systemd/system/mnt-database.mount
   ```

3. Edite o mnt-database.mount usando:
   ```
   nano /etc/systemd/system/mnt-database.mount
   ```
4. Inclua o conteúdo a seguir no arquivo mnt-database.mount e substitua `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` de `What` pelo **Ponto de montagem** do armazenamento de arquivo (por exemplo, *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). É possível obter a URL do **Ponto de montagem** sob o serviço de armazenamento de arquivo criado.
   ```
   [Unit] Descrição = Montagem para armazenamento de contêiner

   [Mount]
   What=CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT
   Where=/mnt/database
   Type=nfs
   Options=vers=3,sec=sys,noauto

   [Install] WantedBy = multi-user.target
   ```
   Use Ctrl + X para salvar e sair da janela nano
   {: tip}

5. Criar o ponto de montagem
  ```sh
  mkdir /mnt/database
  ```

6. Montar o armazenamento
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-database.mount
   ```

7. Verificar se a montagem foi concluída com êxito
   ```sh
   montagem
   ```
   As últimas linhas devem listar a montagem do File Storage. Se este não for o caso, use `journalctl -xe` para depurar a operação de montagem.
   {: tip}

### Configurar um backup em intervalo regular

1. Crie o shell script `/root/dbbackup.sh` (use `touch` e `nano`) com os comandos a seguir, substituindo `CHANGE_ME` pela senha do banco de dados especificada anteriormente:
   ```sh
   #!/bin/bash
   mysqldump -u root -p CHANGE_ME --all-databases --routines | gzip > /mnt/datamysql/backup-`date '+%m-%d-%Y-%H-%M-%S'`.sql.gz
   ```
2. Certifique-se de que o arquivo seja executável
   ```sh
   chmod 700 /root/dbbackup.sh
   ```
3. Edite o crontab
   ```sh
   crontab -e
   ```
4. Para fazer com que o backup seja executado todos os dias às 23h, configure o conteúdo para o seguinte, salve o arquivo e feche o editor
   ```
   0 23 * * * /root/dbbackup.sh
   ```

## Provisionar dois servidores para o aplicativo PHP
{: #app_servers}

Nesta seção, você criará dois servidores de aplicativos da web.

1. Acesse o catálogo no console do {{site.data.keyword.Bluemix}} e selecione o serviço [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) na seção Infraestrutura.
2. Selecione **Virtual Server público** e, em seguida, clique em **Criar**.
3. Configure o servidor com o seguinte:
   - Configure **Nome** como **app1**
   - Selecione o mesmo local em que você provisionou o servidor de banco de dados
   - Selecione a imagem **Ubuntu Minimal**
   - Mantenha o tipo de cálculo padrão.
   - Em **Discos de armazenamento conectados**, selecione 25 GB como seu disco de inicialização.
   - Em **Interface de rede**, selecione a opção **Uplink de rede privada de 100 Mbps**.

     Se você não configurou o Acesso VPN, selecione a opção **Uplink de rede pública e privada de 100 Mbps**.
     {: tip}
   - Revise as outras opções de configuração e clique em **Provisionar** para provisionar o servidor.
    ![Configurar o servidor virtual](images/solution14/db-server.png)
4. Repita as etapas 1-3 para provisionar outro servidor virtual denominado **app2**

## Criar um armazenamento de arquivo para compartilhar arquivos entre os servidores de aplicativos
{: shared_storage}

Esse armazenamento de arquivo é usado para compartilhar os arquivos de aplicativo entre os servidores *app1* e *app2*.

### Criar o armazenamento de arquivo
{: #create_for_sharing}

1. Acesse o catálogo no console do {{site.data.keyword.Bluemix}} e selecione [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
2. Clique em **Criar**
3. Configure o serviço com o seguinte:
   - Configure **Tipo de armazenamento** para **Endurance**
   - Selecione o mesmo **Local** que aquele em que você criou os servidores de aplicativos
   - Selecione um método de faturamento
   - Em **Pacotes de armazenamento**, selecione **2 IOPS/GB**
   - Em **Tamanho do armazenamento**, selecione **20 GB**
   - Em **Tamanho do espaço de captura instantânea**, selecione **20 GB**
   - Clique em Continuar para criar o serviço.

### Configurar capturas instantâneas regulares

As [Capturas instantâneas](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) fornecem a você uma opção conveniente para proteger seus dados sem impacto no desempenho. Além disso, é possível replicar capturas instantâneas para outro data center.

1. Selecione o File Storage na [lista de itens existentes](https://{DomainName}/classic/storage/file)
2. Em **Planejamentos de captura instantânea**, edite o planejamento de captura instantânea. O planejamento pode ser definido como a seguir:
   1. Inclua uma captura instantânea por hora, configure o minuto para 30 e mantenha as últimas 24 capturas instantâneas
   2. Inclua uma captura instantânea diária, configure o horário para 23h e mantenha as últimas 7 capturas instantâneas
   3. Inclua uma captura instantânea semanal, configure o horário como 13h e mantenha as últimas 4 capturas instantâneas e clique em Salvar.
    ![Fazer backup de capturas instantâneas](images/solution14/snapshots.png)

### Autorizar os servidores de aplicativos a usarem o armazenamento de arquivo

1. Em **Hosts autorizados**, clique em **Autorizar host** para autorizar os servidores de aplicativos (app1 e app2) a usarem esse armazenamento de arquivo.

### Montar o armazenamento de arquivo

Repita as etapas a seguir em cada servidor de aplicativos (app1 e app2):

1. Instale as bibliotecas do cliente NFS
   ```sh
   apt-get update
   apt-get -y install nfs-common
   ```
2. Crie um arquivo usando `touch /etc/systemd/system/mnt-www.mount` e edite usando `nano /etc/systemd/system/mnt-www.mount` com o conteúdo a seguir, substituindo `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` de `What` pelo **Ponto de montagem** para o armazenamento de arquivo (por exemplo, *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). É possível localizar os pontos de montagem na [lista de volumes de armazenamento de arquivo](https://{DomainName}/classic/storage/file)
   ```
   [Unit] Descrição = Montagem para armazenamento de contêiner

   [Mount]
   What=CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT
   Where=/mnt/www
   Type=nfs
   Options=vers=3,sec=sys,noauto

   [Install] WantedBy = multi-user.target
   ```
3. Criar o ponto de montagem
   ```sh
   mkdir /mnt/www
   ```
4. Montar o armazenamento
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-www.mount
   ```
5. Verificar se a montagem foi concluída com êxito
   ```sh
   montagem
   ```
   As últimas linhas devem listar a montagem do File Storage. Se este não for o caso, use `journalctl -xe` para depurar a operação de montagem.
   {: tip}

Finalmente, todas as etapas relacionadas à configuração dos servidores podem ser automatizadas usando um [script de fornecimento](https://{DomainName}/docs/vsi?topic=virtual-servers-managing-a-provisioning-script#managing-a-provisioning-script) ou [capturando uma imagem](https://{DomainName}/docs/infrastructure/image-templates?topic=image-templates-getting-started#creating-an-image-template).
{: tip}

## Instalar e configurar o aplicativo PHP nos servidores de aplicativos
{: #php_application}

Este tutorial configura um blog do Wordpress. Todos os arquivos Wordpress serão instalados no armazenamento de arquivo compartilhado para que ambos os servidores de aplicativos possam acessá-los. Antes de instalar o Wordpress, um servidor da web e um tempo de execução do PHP precisam ser configurados.

### Instalar o nginx e o PHP

Repita as etapas a seguir em cada servidor de aplicativos:

1. Instale o nginx
   ```sh
   apt-get update
   apt-get -y install nginx
   ```
2. Instale o PHP e o cliente mysql
   ```sh
   apt-get -y install php-fpm php-mysql
   ```
3. Parar o serviço PHP e o nginx
   ```sh
   systemctl stop php7.2-fpm
   systemctl stop nginx
   ```
4. Substitua o conteúdo usando `nano /etc/nginx/sites-available/default` pelo seguinte:
   ```sh
   server {
          listen 80 default_server;
          listen [::]:80 default_server;

          root /mnt/www/html;

          index index.php;

          server_name _;

          location = /favicon.ico {
                  log_not_found off;
                  access_log off;
          }

          location = /robots.txt {
                  allow all;
                  log_not_found off;
                  access_log off;
          }

          location / {
                  # following https://codex.wordpress.org/Nginx
                  try_files $uri $uri/ /index.php?$args;
          }

          # pass the PHP scripts to the local FastCGI server
          location ~ \.php$ {
                  include snippets/fastcgi-php.conf;
                  fastcgi_pass unix:/run/php/php7.2-fpm.sock;
          }

          location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                  expires max;
                  log_not_found off;
          }

          # deny access to .htaccess files, if Apache's document root
          # concurs with nginx's one
          location ~ /\.ht {
                  deny all;
          }
   }
   ```
5. Crie uma pasta `html` dentro do diretório `/mnt/www` em um dos dois servidores de aplicativos usando
   ```sh
   cd  /mnt
   mkdir /www/html
   cd www
   mkdir html
   ```

### Instalar e configurar o WordPress

Como o Wordpress será instalado na montagem do File Storage, você precisará somente fazer as etapas a seguir em um dos servidores. Vamos selecionar **app1**.

1. Recuperar arquivos de instalação do Wordpress

   Se o seu servidor de aplicativos tiver um link de rede pública, será possível fazer download diretamente dos arquivos Wordpress de dentro do servidor virtual:

   ```sh
   apt-get install curl
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

   Se o servidor virtual tiver somente um link de rede privada, será necessário recuperar os arquivos de instalação de outra máquina com acesso à Internet e copiá-los para o servidor virtual. Supondo que você tenha recuperado os arquivos de instalação do Wordpress por meio de https://wordpress.org/latest.tar.gz, será possível copiá-los para o servidor virtual com `scp`:

   ```sh
   scp latest.tar.gz root@PRIVATE_IP_ADDRESS_OF_THE_SERVER:/tmp
   ```
   Substitua `latest` pelo nome do arquivo transferido por download do website do wordpress.
   {: tip}

   em seguida, use ssh para o servidor virtual e mude para o diretório `tmp`

   ```sh
   cd /tmp
   ```

2. Extraia os arquivos de instalação

   ```
   tar xzvf latest.tar.gz
   ```

3. Prepare os arquivos Wordpress
   ```sh
   cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   mkdir /tmp/wordpress/wp-content/upgrade
   ```

4. Copie os arquivos para o armazenamento de arquivo compartilhado
   ```sh
   rsync -av -P /tmp/wordpress/. /mnt/www/html
   ```

5. Configure permissões
   ```sh
   chown -R www-data:www-data /mnt/www/html
   find /mnt/www/html -type d -exec chmod g+s {} \;
   chmod g+w /mnt/www/html/wp-content
   chmod -R g+w /mnt/www/html/wp-content/themes
   chmod -R g+w /mnt/www/html/wp-content/plugins
   ```

6. Chame o serviço da web a seguir e injete o resultado em `/mnt/www/html/wp-config.php` usando `nano`
   ```sh
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   Se o seu servidor virtual não tiver um link de rede pública, será possível simplesmente abrir https://api.wordpress.org/secret-key/1.1/salt/ em seu navegador da web.

7. Configure as credenciais do banco de dados usando `nano /mnt/www/html/wp-config.php`, atualize as credenciais do banco de dados:

   ```
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'database-username');
   define('DB_PASSWORD', 'database-password');
   define('DB_HOST', 'database-server-ip-address');

   ```
   O Wordpress está configurado. Para concluir a instalação, é necessário acessar a interface com o usuário do Wordpress.

Em ambos os servidores de aplicativos, inicie o servidor da web e o tempo de execução do PHP:
7. Inicie o serviço executando os comandos a seguir

   ```sh
   systemctl start php7.2-fpm
   systemctl start nginx
   ```

Acesse a instalação do Wordpress em `http://YourAppServerIPAddress/` usando o endereço IP privado (se você estiver passando pela conexão VPN) ou o endereço IP público de *app1* ou *app2*.
![Configurar o servidor virtual](images/solution14/wordpress.png)

Se você configurou os servidores de aplicativos com somente um link de rede privada, não será possível instalar os plug-ins, temas ou upgrades do Wordpress diretamente do console administrativo do Wordpress. Será necessário fazer upload dos arquivos por meio da interface com o usuário do Wordpress.
{: tip}

## Provisionar um servidor balanceador de carga na frente dos servidores de aplicativos
{: #load_balancer}

Neste ponto, temos dois servidores de aplicativos com endereços IP separados. Eles poderão até não ser visíveis na Internet pública, se você escolher provisionar somente Uplink de rede privada. A inclusão de um balanceador de carga na frente desses servidores tornará o aplicativo público. O balanceador de carga também ocultará a infraestrutura subjacente para os usuários. O Load Balancer monitorará o funcionamento dos servidores de aplicativos e despachará as solicitações recebidas para servidores funcionais.

1. Acesse o catálogo para criar um [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/ibm-cloud-load-balancer)
2. Na etapa **Plano**, selecione o mesmo data center que *app1* e *app2*
3. Em **Configurações de rede**,
   1. Selecione a mesma sub-rede que aquela em que *app1* e *app2* foram provisionados
   2. Use o conjunto do sistema IBM padrão para o IP público do balanceador de carga.
4. Em **Básico**,
   1. Nomeie o balanceador de carga, por exemplo, **app-lb-1**
   2. Mantenha a configuração do protocolo padrão, por padrão, o balanceador de carga está configurado para HTTP.
      O protocolo SSL é suportado com seus próprios certificados. Consulte [Importar seus certificados SSL no balanceador de carga](https://{DomainName}/docs/infrastructure/ssl-certificates?topic=ssl-certificates-accessing-ssl-certificates#accessing-ssl-certificates)
      {: tip}
5. Em **Instâncias do servidor**, inclua os servidores *app1* e *app2*
6. Revise e crie para concluir o assistente.

### Mude a configuração do Wordpress para usar a URL do balanceador de carga

A configuração do Wordpress precisa ser mudada para usar o endereço do Load Balancer. De fato, o Wordpress mantém uma referência para [a URL do blog e injeta esse local nas páginas](https://codex.wordpress.org/Settings_General_Screen). Se você não mudar essa configuração, o Wordpress redirecionará os usuários para os servidores de back-end diretamente, deste modo, efetuará bypass do Load Balancer ou não funcionará de forma alguma se os servidores tiverem somente um endereço IP privado.

1. Localize o endereço do Load Balancer em sua página de detalhes. É possível localizar o Load Balancer que você criou em [Rede/Balanceamento de carga/Local](https://{DomainName}/classic/network/loadbalancing/cloud).

   Também é possível usar seu próprio nome de domínio com o Load Balancer, incluindo um registro CNAME apontando para o endereço do Load Balancer em sua configuração de DNS.
   {: tip}
2. Registre-se como administrador no blog do Wordpress por meio da URL *app1* ou *app2*
3. Em Configurações/Geral, configure o Endereço do Wordpress (URL) e o Endereço do site (URL) para o endereço do Load Balancer
4. Salve as configurações. O Wordpress deve redirecionar para o endereço do Load Balancer.
   Pode levar algum tempo antes que o endereço do Load Balancer se torne ativo devido à propagação de DNS.
   {: tip}

### Testar o comportamento do Load Balancer

O Load Balancer está configurado para verificar o funcionamento dos servidores e redirecionar os usuários somente para servidores funcionais. Para entender como o Load Balancer está funcionando, é possível

1. Observar os logs nginx em *app1* e *app2* com:
   ```sh
   tail -f /var/log/nginx/*.log
   ```

   Você já deverá ver os pings regulares no Load Balancer para verificar o funcionamento do servidor.
   {: tip}
2. Acesse o Wordpress por meio do endereço do Load Balancer e certifique-se de forçar um recarregamento permanente da página. Observe nos logs nginx que *app1* e *app2* estão servindo conteúdo para a página. O Load Balancer está redirecionando o tráfego para ambos os servidores conforme o esperado.

3. Pare o nginx em *app1*
   ```sh
   systemctl nginx stop
   ```

4. Depois de algum tempo, recarregue a página Wordpress, observe que todas as ocorrências estão indo para *app2*.

5. Pare o nginx em *app2*.

6. Recarregue a página Wordpress. O Load Balancer retornará um erro, já que não há servidor funcional.

7. Reinicie o nginx em *app1*
   ```sh
   systemctl nginx start
   ```

8. Quando o Load Balancer detectar *app1* como funcional, ele redirecionará o tráfego para esse servidor.

## Ampliar a solução com um 2º local (opcional)
{: #secondregion}

Para aumentar a resiliência e a disponibilidade, é possível ampliar a configuração da infraestrutura com um segundo local e ter seu aplicativo em execução em dois locais.

Com a implementação de um segundo local, a arquitetura será semelhante a esta.

<p style="text-align: center;">

  ![Diagrama de arquitetura](images/solution14/Architecture2.png)
</p>

1. Os usuários acessam o aplicativo por meio do IBM Cloud Internet Services (CIS).
2. O CIS roteia o tráfego para uma local funcional.
3. Dentro de uma localização, um balanceador de carga redireciona o tráfego para um servidor.
4. O aplicativo acessa o banco de dados.
5. O aplicativo armazena e recupera ativos de mídia de um armazenamento de arquivo.

Para implementar essa arquitetura, é necessário fazer o seguinte no local dois:

- Repita todas as etapas anteriores acima no novo local.
- Configure uma replicação de banco de dados entre os dois servidores MySQL nos locais.
- Configure o IBM Cloud Internet Services para distribuir o tráfego entre os locais para servidores funcionais, conforme descrito [neste outro tutorial](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis).

## Remover recursos
{: #removeresources}

1. Excluir o balanceador de carga
2. Cancele *db1*, *app1* e *app2*
3. Exclua os dois serviços do File Storage
4. Se um segundo local estiver configurado, exclua todos os recursos e o Cloud Internet Services.

## Conteúdo relacionado
{: #related}

- O conteúdo estático servido por seu aplicativo pode se beneficiar de uma Rede de Entrega de Conteúdo na frente do Load Balancer para reduzir a carga em seus servidores de back-end. Consulte [Acelerar a entrega de arquivos estáticos usando um CDN - Object Storage](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn) para um tutorial que implementa uma Rede de Entrega de Conteúdo.
- Neste tutorial, nós provisionamos dois servidores, mais servidores podem ser incluídos automaticamente para manipular a carga adicional. A [Escala automática](https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-about-auto-scale#about-auto-scale) fornece a capacidade de automatizar o processo de ajuste de escala manual associado à inclusão ou remoção de servidores virtuais para suportar seus aplicativos de negócios.
- Para aumentar a disponibilidade e as opções de recuperação de desastre, o File Storage pode ser configurado para executar [capturas instantâneas automáticas regulares](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) do conteúdo e [replicação](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-replication#working-with-replication) para outro data center.
