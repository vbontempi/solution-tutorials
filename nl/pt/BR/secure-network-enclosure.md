---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-08"
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

# Isolar cargas de trabalho com uma rede privada segura
{: #secure-network-enclosure}

A necessidade de ambientes de rede privada isolados e seguros é central para o modelo de implementação do aplicativo IaaS na nuvem pública. Firewalls, VLANs, roteamento e VPNs são todos os componentes necessários na criação de ambientes privados isolados. Esse isolamento permite que máquinas virtuais e servidores bare metal sejam implementados de forma segura em topologias complexas de aplicativo multicamada, enquanto prova a proteção contra riscos na Internet pública.  

Este tutorial destaca como um [Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-faqs-for-ibm-virtual-router-appliance#what-is-vra-) (VRA) pode ser configurado no {{site.data.keyword.Bluemix_notm}} para criar uma rede privada segura (gabinete). O VRA Gateway Appliance fornece um único pacote autogerenciado, um firewall, gateway VPN, Conversão de Endereço de Rede (NAT) e roteamento de classificação corporativa. Neste tutorial, um VRA é usado para mostrar como um ambiente de rede isolado e fechado pode ser criado no {{site.data.keyword.Bluemix_notm}}. Dentro desse gabinete, as topologias do aplicativo podem ser criadas, usando as tecnologias familiares e reconhecidas de roteamento de IP, VLANs, sub-redes de IP, regras de firewall, servidores virtuais e bare metal.  

{:shortdesc}

Este tutorial é um ponto de início para a rede clássica no {{site.data.keyword.Bluemix_notm}} e não deve ser considerado um recurso de produção no estado em que se encontra. Os recursos adicionais que podem ser considerados são:
* [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link#get-started-with-ibm-cloud-direct-link)
* [Dispositivos de firewall de hardware](https://{DomainName}/docs/infrastructure/fortigate-10g?topic=fortigate-10g-exploring-firewalls#exploring-firewalls)
* [VPN IPSec](https://{DomainName}/catalog/infrastructure/ipsec-vpn) para conectividade segura com seu data center.
* Alta disponibilidade com VRAs em cluster e uplinks duais.
* Criação de log e auditoria de eventos de segurança.

## Objetivos 
{: #objectives}

* Implementar um Virtual Router Appliance (VRA)
* Definir VLANs e sub-redes IP para implementar máquinas virtuais e servidores bare metal
* Proteger o VRA e o gabinete com regras de firewall

## Serviços usados
{: #products}

Este tutorial usa os serviços do {{site.data.keyword.Bluemix_notm}} a seguir:
* [Virtual Router Appliance](https://{DomainName}/catalog/infrastructure/virtual-router-appliance)

Este tutorial pode incorrer em custos. O VRA está disponível apenas em um plano de precificação mensal.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

  ![Arquitetura](images/solution33-secure-network-enclosure/Secure-priv-enc.png)
</p>

1. Configurar VPN
2. Implementar VRA 
3. Criar Virtual Server
4. Acesso de rota por meio do VRA
5. Configurar firewall de gabinete
6. Definir zona APP
7. Definir zona INSIDE

## Antes de Começar
{: #prereqs}

### Configurar o acesso VPN

Neste tutorial, o gabinete de rede criado não fica visível na Internet pública. O VRA e quaisquer servidores serão acessíveis somente por meio da rede privada e você usará sua VPN para conectividade. 

1. [Assegure-se de que seu acesso VPN seja permitido](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Você deve ser um **Usuário principal** para ativar o acesso VPN ou contatar o usuário principal para obter acesso.
     {:tip}
2. Obtenha suas credenciais de Acesso VPN selecionando seu usuário na [Lista de usuários](https://{DomainName}/iam#/users).
3. Efetue login na VPN por meio da [interface da web](https://www.softlayer.com/VPN-Access) ou use um cliente VPN para [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) ou [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Para o cliente VPN, use o FQDN de um único ponto de acesso VPN do data center na [página de acesso da web VPN](https://www.softlayer.com/VPN-Access), no formato *vpn.xxxnn.softlayer.com* como o Endereço do gateway.
   {:tip}

### Verificar permissões da conta

Entre em contato com o usuário principal da infraestrutura para obter as permissões a seguir:
- **Permissões rápidas** - Usuário básico
- **Rede** para que seja possível criar e configurar o gabinete, Todas as permissões de rede são necessárias. 
- **Serviços** para gerenciar Chaves SSH

### Fazer upload de chaves SSH

Por meio do portal, [faça upload da chave pública SSH](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-getting-started-tutorial#getting-started-tutorial) que será usada para acessar e administrar a rede VRA e privada.  

### Destinar o data center

Escolha um data center do {{site.data.keyword.Bluemix_notm}} para implementar a rede privada segura. 

### Solicitar VLANs

Para criar o gabinete privado no data center de destino, as VLANs privadas necessárias para os servidores devem primeiro ser designadas. Não há encargos para as primeiras VLANs privadas e públicas. As VLANs adicionais para suportar uma topologia do aplicativo multicamada são debitáveis. 

Para assegurar que VLANs suficientes estejam disponíveis no mesmo roteador do data center e possam ser associadas ao VRA, é aconselhável que elas sejam pedidas. Consulte [Pedindo VLANs](https://{DomainName}/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#order-vlans).

## Provisionar Virtual Router Appliance
{: #VRA}

A primeira etapa é implementar um VRA que fornecerá o roteamento de IP e o firewall para o gabinete de rede privada. A Internet é acessível por meio do gabinete por uma VLAN de trânsito público fornecida pelo {{site.data.keyword.Bluemix_notm}}, um gateway e, opcionalmente, um firewall de hardware que cria a conectividade da VLAN pública para as VLANs de gabinete privado seguro. Neste tutorial de solução, um Virtual Router Appliance (VRA) fornece esse perímetro de gateway e de firewall. 

1. No catálogo, selecione um [Dispositivo de gateway](https://{DomainName}/gen1/infrastructure/provision/gateway)
3. Na seção **Fornecedor de gateway**, selecione AT&T. É possível escolher entre "até 20 Gbps" ou "até 2 Gbps" de Velocidade de uplink.
4. Na seção **Nome do host**, insira um Nome do host e um Domínio para seu novo VRA.
5. Se marcar a caixa de seleção **Alta disponibilidade**, você obterá dois dispositivos VRA trabalhando em uma configuração ativa/de backup usando o VRRP.
6. Na seção **Local**, selecione o Local e o **Pod** no qual você precisa de seu VRA.
7. Selecione Processador único ou Processador dual. Você obterá uma lista de Servidores. Escolha um Servidor clicando em seu botão de opções. 
8. Selecione a quantia de **RAM**. Para um ambiente de produção, é recomendável usar um mínimo de 64 GB de RAM. 8 GB no mínimo para o ambiente de teste.
9. Selecione uma **Chave SSH** (opcional). Essa chave ssh será instalada no VRA, portanto, o usuário vyatta poderá ser usado para acessar o VRA com essa chave.
10. Disco rígido. Mantenha o padrão.
11. Na seção **Velocidades da porta de uplink**, selecione a combinação de velocidade, redundância e interfaces privada e/ou pública que atenda às suas necessidades.
12. Na seção **Complementos**, mantenha o padrão. Se você desejar usar IPv6 na interface pública, selecione o endereço IPv6.

No lado direito, é possível ver o **Resumo do pedido**. Marque a caixa de seleção _Eu li e concordo com os Contratos de Prestação de Serviços de Terceiros listados abaixo:_ e clique no botão **Criar**. Seu gateway será implementado.

A [Lista de dispositivos](https://{DomainName}/classic/devices) mostrará o VRA quase imediatamente com um símbolo de **Clock** nele, indicando que as transações estão em andamento nesse dispositivo. Até que a criação do VRA esteja concluída, o símbolo de **Clock** permanece e, além de visualizar detalhes, não é possível executar nenhuma ação de configuração com relação ao dispositivo.
{:tip}

### Revisar o VRA implementado

1. Inspecione o novo VRA. No [Painel de infraestrutura](https://{DomainName}/classic), selecione **Rede** na área de janela à esquerda, seguido por **Dispositivos de gateway** para acessar a página [Dispositivos de gateway](https://{DomainName}/classic/network/gatewayappliances). Selecione o nome do VRA recém-criado na coluna **Gateway** para continuar com a página Detalhes do gateway. ![](images/solution33-secure-network-enclosure/Gateway-detail.png)

2. Anote os endereços IP `Private` e `Public` do VRA para uso futuro.

## Configuração inicial do VRA
{: #initial_VRA_setup}

1. Em sua estação de trabalho, por meio da VPN SSL, efetue login no VRA usando a conta padrão **vyatta**, aceitando os prompts de segurança SSH. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   Se o SSH solicitar uma senha, a chave SSH não foi incluída na construção. Acesse o VRA por meio do [navegador da web](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#accessing-the-device-using-the-web-gui) usando o `VRA Private IP Address`. A senha é por meio da página [Senhas de software](https://{DomainName}/classic/devices/passwords). Na guia **Configuração**, selecione a ramificação Sistema/login/vyatta e inclua a chave SSH desejada.
   {:tip}

   A configuração do VRA requer que o VRA seja colocado no modo \[edit\] usando o comando `configure`. Quando no modo `edit`, o prompt muda de `$` para `#`. Após uma mudança bem-sucedida na configuração do VRA, é possível visualizar suas mudanças com o comando `compare` e verificar suas mudanças com o comando `validate`. Confirmando uma mudança com o comando `commit`, ela será aplicada à configuração em execução e será salva automaticamente na configuração de inicialização.


   {:tip}
2. Aprimore a segurança permitindo somente login de SSH. Agora que o login de SSH é bem-sucedido por meio da rede privada, desative o acesso por meio de autenticação de ID do usuário/senha. 
   ```
   configure
   set service ssh disable-password-authentication
   commit
   exit
   ```
   {: codeblock}
   Deste ponto em diante neste tutorial, supõe-se que todos os comandos do VRA são inseridos no prompt `edit`, subsequente à inserção de `configure`.
3. Revise a configuração inicial
   ```
   mostrar
   ```
   {: codeblock}

   O VRA é pré-configurado para o ambiente do {{site.data.keyword.Bluemix_notm}} IaaS. Isso inclui o seguinte:
   - Servidor NTP
   - Servidores de nomes
   - SSH
   - Servidor da web HTTPS 
   - Fuso horário padrão EUA/Chicago
4. Configure o fuso horário local conforme necessário. A conclusão automática com a tecla tab listará os potenciais valores de fuso horário
   ```
   set system time-zone <timezone>
   ```
   {: codeblock}
5. Configure o comportamento de ping. O ping não é desativado para auxiliar na resolução de problemas de roteamento e de firewall. 
   ```
   set security firewall all-ping enable
   set security firewall broadcast-ping disable
   ```
   {: codeblock}
6. Ative a operação de firewall stateful. Por padrão, o firewall do VRA é stateless. 
   ```
   set security firewall global-state-policy icmp
   set security firewall global-state-policy udp
   set security firewall global-state-policy tcp
   ```
   {: codeblock}
7. Confirme e salve automaticamente suas mudanças na configuração de inicialização. 
   ```
   commit
   ```
   {: codeblock}

## Pedir o primeiro servidor virtual
{: #order_virtualserver}

Um servidor virtual é criado neste ponto para auxiliar no diagnóstico de erros de configuração do VRA. O acesso bem-sucedido à VSI é validado na rede privada do {{site.data.keyword.Bluemix_notm}} antes de o acesso a ela ser roteado por meio de VRA em uma etapa posterior. 

1. Peça um [servidor virtual](https://{DomainName}/catalog/infrastructure/virtual-server-group)  
2. Selecione **Servidor virtual público** e continue.
3. Na página do pedido:
   - Configure **Faturamento** como **Por hora**.
   - Configure o *Nome do host da VSI* e o *Nome de domínio*. Esse nome de domínio não é usado para roteamento e DNS, mas deve ser alinhado com seus padrões de nomenclatura de rede. 
   - Configure **Local** para o mesmo que o VRA.
   - Configure **Perfil** como **C1.1x1**
   - Inclua a **Chave SSH** que você especificou anteriormente.
   - Configure **Sistema operacional** como **CentOS 7.x - Mínimo**
   - Em **Velocidades da porta de uplink**, a interface de rede deve ser mudada do padrão de *pública e privada* para especificar somente uma **Uplink de rede privada**. Isso assegura que o novo servidor não tenha acesso direto à Internet e o acesso seja controlado pelas regras de roteamento e firewall no VRA.
   - Configure **VLAN privada** para o ID de VLAN da VLAN privada pedida anteriormente.
4. Clique na caixa de seleção para aceitar os contratos de prestação de serviços de 'Terceiros' e, em seguida, **Criar**.
5. Monitore a conclusão na página [Dispositivos](https://{DomainName}/classic/devices) ou por meio de e-mail. 
6. Anote o *Endereço IP privado* da VSI para uma etapa posterior e se, na seção **Rede** na página **Detalhes do dispositivo**, a VSI está designada à VLAN correta. Se não, exclua essa VSI e crie uma nova VSI na VLAN correta. 
7. Verifique o acesso bem-sucedido à VSI por meio de rede privada do {{site.data.keyword.Bluemix_notm}} usando ping e SSH em sua estação de trabalho local na VPN.
   ```bash
   ping <VSI Private IP Address>
   SSH root@<VSI Private IP Address>
   ```
   {: codeblock}

## Rotear acesso à VLAN por meio do VRA
{: #routing_vlan_via_vra}

As VLANs privadas para o servidor virtual terão sido associadas pelo sistema de gerenciamento do {{site.data.keyword.Bluemix_notm}} a esse VRA. Neste estágio, a VSI ainda é acessível por meio de roteamento de IP na rede privada do {{site.data.keyword.Bluemix_notm}}. Agora, você roteará a sub-rede por meio de VRA para criar a rede privada segura e validar confirmando que agora a VSI não está acessível. 

1. Continue com os Detalhes do gateway para o VRA por meio da página [Dispositivos de gateway](https://{DomainName}/classic/network/gatewayappliances) e localize a seção **VLANs associadas** na metade inferior da página. A VLAN associada será listada aqui. 
2. Se for desejado incluir VLANs adicionais neste momento, navegue para a seção **Associar uma VLAN**. A caixa suspensa, *Selecionar VLAN*, deve ser ativada e outras VLANs provisionadas podem ser selecionadas. ![](images/solution33-secure-network-enclosure/Gateway-Associate-VLAN.png)

   Se nenhuma VLAN elegível for mostrada, nenhuma VLAN estará disponível no mesmo roteador que o VRA. Isso requererá que um [chamado de suporte](https://{DomainName}/unifiedsupport/cases/add) seja levantado para solicitar uma VLAN privada no mesmo roteador que o VRA.
   {:tip}
5. Selecione a VLAN que você deseja associar ao VRA e clique em Salvar. A associação inicial de VLAN pode levar alguns minutos para ser concluída. Depois de concluída, a VLAN deve ser mostrada sob o título **VLANs associadas**. 

Neste estágio, a VLAN e a sub-rede associada não são protegidas ou roteadas por meio do VRA e a VSI é acessível por meio da rede privada do {{site.data.keyword.Bluemix_notm}}. O status de VLAN será mostrado como *Bypass efetuado*.

4. Selecione **Ações** na coluna direita, em seguida, **Rotear VLAN** para rotear a VLAN/Sub-rede por meio do VRA. Isso levará alguns minutos. Uma atualização de tela mostrará que ela é *Roteada*. 
5. Selecione o [Nome da VLAN](https://{DomainName}/classic/network/vlans/) para visualizar os detalhes da VLAN. A VSI provisionada pode ser vista, assim como a Sub-rede de IP primário designada. Anote o ID de VLAN privada \<nnnn\> (1199 neste exemplo), pois isso será usado em uma etapa posterior. 
6. Selecione a [sub-rede](https://{DomainName}/classic/network/subnets) para ver os detalhes da sub-rede de IP. Anote a Rede de sub-rede, os Endereços de gateway e o CIDR (/26), pois eles são necessários para a configuração adicional do VRA. 64 endereços IP primários são provisionados na rede privada e, para localizar o endereço do Gateway, isso pode requerer a seleção da página 2 ou 3. 
7. Valide que a sub-rede/VLAN é roteada para o VRA e a VSI **NÃO** é acessível por meio da rede de gerenciamento de sua estação de trabalho usando `ping`. 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}

Isso conclui a configuração do VRA por meio do console do {{site.data.keyword.Bluemix_notm}}. O trabalho adicional para configurar o gabinete e o roteamento de IP é agora executado diretamente no VRA por meio de SSH. 

## Configurar o roteamento de IP e o gabinete seguro
{: #vra_setup}

Quando a configuração do VRA é confirmada, a configuração em execução é mudada e as mudanças são salvas automaticamente na configuração de inicialização.

Se for desejado retornar a uma configuração de trabalho anterior, por padrão, os últimos 20 pontos de confirmação poderão ser visualizados, comparados ou restaurados. Consulte o [Guia de configuração básica do sistema Vyatta Network OS](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation) para obter mais detalhes sobre como confirmar e salvar a configuração.
   ```bash
   show system commit 
   rollback n
   compare
   ```
   {: codeblock}

### Configurar o roteamento de IP do VRA

Configure a interface de rede virtual do VRA para rotear para a nova sub-rede por meio da rede privada do {{site.data.keyword.Bluemix_notm}}.  

1. Efetue login no VRA por SSH. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}
2. Crie uma nova interface virtual com o ID de VLAN privada, o endereço IP do gateway de sub-rede e o CIDR registrados nas etapas anteriores. O CIDR normalmente será /26. 
   ```
   set interfaces bonding dp0bond0 vif <VLAN ID> address <Subnet Gateway IP>/<CIDR>
   commit
   ```
   {: codeblock}
    
   É crítico que o endereço **`<Subnet Gateway IP>`** seja usado. Geralmente, esse é mais do que o endereço de início do endereço de sub-rede. Inserir um endereço do gateway inválido resultará no erro `Configuration path: interfaces bonding dp0bond0 vif xxxx address [x.x.x.x] is not valid`. Corrija o comando e insira novamente. É possível procurá-lo em Rede > Gerenciamento de IP > Sub-redes. Clique na sub-rede que você precisa saber o endereço do Gateway. A segunda entrada na Lista com a Descrição **Gateway** é o endereço IP a ser inserido como <Subnet Gateway IP>/<CIDR>.
   {: tip}

3. Listar a nova interface virtual (vif): 
   ```
   show interfaces
   ```
   {: codeblock}

   Esta é uma configuração de interface de exemplo mostrando o vif 1199 e o endereço do gateway de sub-rede.
    ![](images/solution33-secure-network-enclosure/show_interfaces.png)
4. Validar se a VSI está mais uma vez acessível por meio da rede de gerenciamento de sua estação de trabalho. 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
   
   Se a VSI não estiver acessível, verifique se a tabela de roteamento de IP do VRA está configurada conforme o esperado. Exclua e recrie a rota, se necessário. Para executar um comando show conmmand no modo de configuração, é possível usar o comando run    
   ```bash
    run show ip route <Subnet Gateway IP>
   ```
   {: codeblock}

Isso conclui a configuração de roteamento de IP.

### Configurar gabinete seguro

O gabinete seguro de rede privada é criado por meio da configuração de zonas e regras de firewall. Revise a documentação do VRA na [configuração de firewall](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-manage-your-ibm-firewalls#manage-firewalls) antes de continuar. 

Duas zonas são definidas:
   - INSIDE:  as redes privadas e de gerenciamento da IBM
   - APP: a VLAN do usuário e a sub-rede dentro do gabinete de rede privada		

1. Defina firewalls e padrões.
   ```
   configure
   set security firewall name APP-TO-INSIDE default-action drop
   set security firewall name APP-TO-INSIDE default-log

   set security firewall name INSIDE-TO-APP default-action drop
   set security firewall name INSIDE-TO-APP default-log
   commit
   ```
   {: codeblock}
    
   Se um comando set for executado acidentalmente duas vezes, você receberá uma mensagem *'O caminho de configuração xxxxxxxx não é válido. O nó existe'*. Isso pode ser ignorado. Para mudar um parâmetro incorreto, é necessário primeiro excluir o nó com 'delete security xxxxx xxxx xxxxx'.
   {:tip}
2. Crie o grupo de recursos de rede privada do {{site.data.keyword.Bluemix_notm}}. Este grupo de endereços define as redes privadas do {{site.data.keyword.Bluemix_notm}} que podem acessar o gabinete e as redes que podem ser acessadas por meio do gabinete. Dois conjuntos de endereços IP precisam de acesso para e do gabinete seguro, eles são os data centers de VPN SSL e a Rede de serviço do {{site.data.keyword.Bluemix_notm}} (rede de back-end/privada). [Intervalos de IP do {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges) fornece a lista integral de intervalos de IP que precisam ser permitidos. 
   - Defina o endereço de VPN SSL dos data centers que você está usando para acesso à VPN. Na seção VPN SSL de Intervalos de IP do {{site.data.keyword.Bluemix_notm}}, selecione os pontos de acesso à VPN para seu data center ou cluster de DC. O exemplo aqui mostra as variações de endereços de VPN para os data centers de Londres do {{site.data.keyword.Bluemix_notm}}.
     ```
     set resources group address-group ibmprivate address 10.2.220.0/24
     set resources group address-group ibmprivate address 10.200.196.0/24
     set resources group address-group ibmprivate address 10.3.200.0/24
     ```
     {: codeblock}
   - Defina as variações de endereços para a ‘Rede de serviço (na rede de back-end/privada)’ do {{site.data.keyword.Bluemix_notm}} para WDC04, DAL01 e seu data center de destino. O exemplo aqui é WDC04 (dois endereços), DAL01 e LON06.
     ```
     set resources group address-group ibmprivate address 10.3.160.0/20
     set resources group address-group ibmprivate address 10.201.0.0/20
     set resources group address-group ibmprivate address 10.0.64.0/19
     set resources group address-group ibmprivate address 10.201.64.0/20
     commit
     ```
     {: codeblock}
3. Crie a zona APP para a VLAN e sub-rede do usuário e a zona INSIDE para a rede privada do {{site.data.keyword.Bluemix_notm}}. Designar os firewalls criados anteriormente. A definição de zona usa os nomes da interface de rede do VRA para identificar a zona associada a cada VLAN. O comando para criar a zona APP requer que o ID de VLAN da VLAN associada ao VRA anterior seja especificado. Isso é destacado abaixo como `<VLAN ID>`.
   ```
   set security zone-policy zone INSIDE description "IBM Internal network"
   set security zone-policy zone INSIDE default-action drop
   set security zone-policy zone INSIDE interface dp0bond0
   set security zone-policy zone INSIDE to APP firewall INSIDE-TO-APP 

   set security zone-policy zone APP description "Application network"
   set security zone-policy zone APP default-action drop

   set security zone-policy zone APP interface dp0bond0.<VLAN ID> 
   set security zone-policy zone APP to INSIDE firewall APP-TO-INSIDE 
   ```
   {: codeblock}
4. Confirme a configuração e, em sua estação de trabalho, verifique usando o ping se o firewall está agora negando o tráfego por meio do VRA para a VSI: 
   ```
   commit
   ```
   {: codeblock}

   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
5. Defina regras de acesso ao firewall para udp, tcp e icmp.
   ```
   set security firewall name INSIDE-TO-APP rule 200 protocol icmp
   set security firewall name INSIDE-TO-APP rule 200 icmp type 8
   set security firewall name INSIDE-TO-APP rule 200 action accept 
   set security firewall name INSIDE-TO-APP rule 200 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 100 action accept 
   set security firewall name INSIDE-TO-APP rule 100 protocol tcp
   set security firewall name INSIDE-TO-APP rule 100 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 110 action accept 
   set security firewall name INSIDE-TO-APP rule 110 protocol udp
   set security firewall name INSIDE-TO-APP rule 110 source address ibmprivate
   commit

   set security firewall name APP-TO-INSIDE rule 200 protocol icmp
   set security firewall name APP-TO-INSIDE rule 200 icmp type 8
   set security firewall name APP-TO-INSIDE rule 200 action accept 
   set security firewall name APP-TO-INSIDE rule 200 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 100 action accept 
   set security firewall name APP-TO-INSIDE rule 100 protocol tcp
   set security firewall name APP-TO-INSIDE rule 100 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 110 action accept 
   set security firewall name APP-TO-INSIDE rule 110 protocol udp
   set security firewall name APP-TO-INSIDE rule 110 destination address ibmprivate
   commit
   ```
   {: codeblock}
6. Valide o acesso ao firewall. 
   - Confirme se o firewall INSIDE-TO-APP está agora permitindo o tráfego ICMP e udp/tcp em sua máquina local
     ```bash
     ping <VSI Private IP Address>
     SSH root@<VSI Private IP Address>
     ```
     {: codeblock}
   - Confirme se o firewall APP-TO-INSIDE está permitindo o tráfego ICMP e udp/tcp. Efetue login na VSI usando SSH e execute ping de um dos servidores de nomes do {{site.data.keyword.Bluemix_notm}} em 10.0.80.11 e 10.0.80.12.
     ```bash
     SSH root@<VSI Private IP Address>
     [root@vsi  ~]# ping 10.0.80.11 
     ```
     {: codeblock}
7. Valide o acesso continuado à interface de gerenciamento do VRA por meio de SSH em sua estação de trabalho. Se o acesso for mantido, revise e salve a configuração. Caso contrário, uma reinicialização do VRA retornará para uma configuração de trabalho. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   ```
   show security  
   ```
   {: codeblock}

### Depurando regras de firewall

Os logs de firewall podem ser visualizados por meio do prompt de comandos operacional do VRA. Nessa configuração, somente o tráfego descartado para cada Zona é registrado para auxiliar no diagnóstico de configuração incorreta de firewall.  

1. Revise os logs de firewall para o tráfego negado. A revisão periódica dos logs identificará se os servidores na zona APP estão tentando, validamente ou erroneamente, entrar em contato com os serviços na rede IBM. 
   ```
   show log firewall name INSIDE-TO-APP
   show log firewall name APP-TO-INSIDE
   ```
   {: codeblock}
2. Se os serviços ou servidores não puderem ser contatados e nada for visto nos logs de firewall. Verifique se o tráfego de IP de ping/ssh esperado está presente na interface de rede do VRA por meio da rede privada do {{site.data.keyword.Bluemix_notm}} ou na interface do VRA para a VLAN usando o `<VLAN ID>` de antes.
   ```bash
   monitor interface bonding dp0bond0 traffic
   monitor interface bonding dp0bond0.<VLAN ID> traffic
   ```

## Proteger o VRA
{: #securing_the_vra}

1. Aplique a política de segurança do VRA. Por padrão, o zoneamento de firewall baseado em política não protege o acesso ao próprio VRA. Isso é configurado por meio do Control Plane Policing (CPP). O VRA fornece um conjunto de regras básicas de CPP como um modelo. Mescle-o em sua configuração:
   ```bash
   merge /opt/vyatta/etc/cpp.conf 
   ```
   {: codeblock}

Isso cria um novo conjunto de regras de firewall denominado `CPP`, visualize as regras adicionais e confirme no modo \[edit\]. 
   ```
   show security firewall name CPP
   commit
   ```
   {: codeblock}
2. Protegendo o acesso SSH público. Devido a um problema pendente neste momento com o firmware do Vyatta, não é recomendado usar `set service SSH listen-address x.x.x.x` para limitar o acesso administrativo do SSH na rede pública. Como alternativa, o acesso externo pode ser bloqueado por meio do firewall de CPP para o intervalo de endereços IP públicos usados pela interface pública do VRA. O `<VRA Public IP Subnet>` usado aqui é o mesmo que o `<VRA Public IP Address>` com o último octeto sendo zero (x.x.x.0). 
   ```
   set security firewall name CPP rule 900 action drop
   set security firewall name CPP rule 900 destination address <VRA Public IP Subnet>/24
   set security firewall name CPP rule 900 protocol tcp
   set security firewall name CPP rule 900 destination port 22
   commit 
   ```
   {: codeblock}
3. Valide o acesso administrativo SSH do VRA sobre a rede interna da IBM. Se o acesso for perdido para o VRA por meio de SSH após executar confirmações, será possível acessar o VRA por meio do Console do KVM disponível na página Detalhes do dispositivo do VRA por meio do Menu Suspenso de Ação.

Isso conclui a configuração do gabinete seguro de rede privada que protege uma única zona de firewall contendo uma VLAN e uma sub-rede. Zonas de firewall, regras, servidores virtuais e bare metal, VLANs e sub-redes adicionais podem ser incluídos seguindo as mesmas instruções. 

## Remover recursos
{: #removeresources}

Nesta etapa, você limpará os recursos para remover o que criou acima.

O VRA está em um plano pago mensal. O cancelamento não resulta em um reembolso. Sugere-se cancelar apenas se esse VRA não for necessário novamente no próximo mês. Se um cluster de Alta Disponibilidade dual VRA for necessário, esse VRA único poderá ser atualizado na página [Detalhes do gateway](https://{DomainName}/classic/network/gatewayappliances/).
{:tip}  

- Cancelar quaisquer servidores virtuais ou servidores bare-metal
- Cancelar o VRA
- Cancele quaisquer VLANs adicionais por chamado de suporte. 

## Conteúdo relacionado
{: #related}

- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Sub-redes de IP estático e móvel](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-and-ips#about-subnets-and-ips)
- [IBM QRadar Security Intelligence Platform](http://www-01.ibm.com/support/knowledgecenter/SS42VS)
- [Documentação do Vyatta](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
