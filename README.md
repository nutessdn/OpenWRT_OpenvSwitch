# OpenWRT_OpenvSwitch

Este tutorial tem como objetivo o uso do OpenWRT em um roteador comum, fazendo o uso do Open vSwitch
permitindo o uso do OpenFlow SDN, que proporciona o desacoplamento entre os dados e o Plano de controle,
permitindo assim o uso de diversos controladores e apps de gerência para realizar o controle.

### Hardware necessário

Para realização deste tutorial será necessário o uso de um roteador doméstico modelo: TP-Link TL-WR1043ND versão 2.1 que será regravado com uma firmware [OpenWRT](https://g.co/kgs/EHYNN2) rodando o Open VSwitch que utiliza o OpenFlow.

Existem outros roteadores que podem ser configurados de maneira semelhante, porém com alguma diferença de configuração, para isto confira a lista de dispositivos que tem suporte à OpenWRT. [Confira a lista](https://openwrt.org/toh/views/toh_fwdownload)

_(Dispositivos com menos de 4MB de flash e/ou menos de 32MB de RAM sofrem com limitações de usabilidade, extensibilidade e instabilidade)_

## Cuidado!


:fire: :skull:Este procedimento de "reflashing" em um roteador consequentemente anulará a garantia e poderá torná-lo "bricked", sendo assim irrecuperável. **Continue por sua conta e risco...**:fire: :skull:


### Transformando o roteador para OpenWRT

Faça as configurações básicas para iniciar o processo:

Conecte a uma porta LAN um dispositivo que possa se comunicar usando SSH. Defina um ip estático 192.168.1.2 com mascara 255.255.255.0 (ou use DHCP), agora conecte-se com SSH em 192.168.1.1 (gateway padrão). Defina uma senha segura para o root.

### Compilar o OpenWRT com uma imagem do Open vSwitch

    _Para Compilar a firmware foi utilizado o Ubuntu 17.04._

### Clonando o repositório do OpenWRT
No host de compilação, clone o OpenWRT: _(obs: no GitHub, não diretamente do site do OpenWRT)_

**$ git clone https://github.com/openwrt/openwrt.git**

### Instalando as dependências

      sudo apt-get update
      sudo apt-get install git-core build-essential libssl-dev      libncurses5-dev unzip gawk zlib1g-dev
      sudo apt-get install subversion mercurial
      sudo apt-get install gcc-multilib flex gettext

### Atualizando Feeds  

      cd openwrt
      ./scripts/feeds update -a
      ./scripts/feeds install -a

### Make MenuConfig

      make MenuConfig

Alrere a opção selecionanda para o hardware utilizado( selecione a opção TP-Link TL-WR1043N / ND v2  para TP-Link TL-WR1043ND Hardware Version 2.1)

![Passo 01](https://raw.githubusercontent.com/nutessdn/OpenWRT_OpenvSwitch/master/imagens/passo1.png)

Agora selecione **Kernel Modules -> kmod-tun:**

![Passo2](https://raw.githubusercontent.com/nutessdn/OpenWRT_OpenvSwitch/master/imagens/passo2.png)

**Exit** volte para tela inicial, vá para **Network -> Open vSwitch** e inclua a opção selecionada utilizando _"Y"_:
![Passo3](https://raw.githubusercontent.com/nutessdn/OpenWRT_OpenvSwitch/master/imagens/passo3.png)

**Save**  depois **Exit**:

![Passo3](https://raw.githubusercontent.com/nutessdn/OpenWRT_OpenvSwitch/master/imagens/salvar.png)

Isto deve demorar um pouco...

        make kernel_menuconfig

Quando concluir o processo aparecerá outro menu. Vá para **Networking support -> Networking options** e inclua a opção **Hierarchical Token Bucket (HTB):**

![Passo4](https://raw.githubusercontent.com/nutessdn/OpenWRT_OpenvSwitch/master/imagens/passo4.png)

### Run Make

Este processo pode demorar algumas horas...

        make

### Deseja autenticação Wi-Fi?

**No nosso exemplo não se faz necessária a utilização de Wi-Fi**

Na contrução padrão do OpenWRT com Open vSwitch não permite a autenticação Wi-Fi, se deseja utiliza-la veja:
https://forum.openwrt.org/viewtopic.php?id=59129


### Copy Image

Vá até o diretório onde os arquivos são:

      cd bin/ar71xx

Neste contém diversos arquivos incluindo:

      openwrt-ar71xx-generic-tl-wr1043nd-v2-squashfs-sysupgrade.bin


Use o __SCP__ para copiar o arquivo para o roteador:

      scp ./openwrt-ar71xx-generic-tl-wr1043nd-v2-squashfs-sysupgrade.bin USERNAME@192.168.1.1:tmp


### Upgrade

_Nota: É aconselhado fazer o backup das configurações **etc** primeiro._

Certo que a imagem realmente está no diretório _/tmp_ do TP-Link, podemos usar o **sysupgrade**:

        sysupgrade -v /tmp/openwrt-ar71xx-generic-tl-wr1043nd-v2-squashfs-sysupgrade.bin


### Configure o OpenWRT

O OpenWRT necessita ser configurado para trabalhar junto com o Open vSwitch.

### Dropbear (servidor SSH)

Configure o Dropbear para escutar a interface WAN e também a interface LAN. Com isso temos um método adicional para acessar e administrar o dispositivo, reguardado de um eventual bloqueio.  
_
Nota: Caso deseje reconfigura-lo para um roteador de internet, esta não é uma configuração segura, então lembre-se de remover as configurações de WAN._

Faça o backup das configurações do Dropbear:

      cp /etc/config/dropbear /etc/config/dropbear.original

Adicione estas linhas em _/etc/config/dropbear_ para WAN, o arquivo completo é:

        config dropbear
          option PasswordAuth 'on'
          option Port '22'
          option Interface 'lan'

        config dropbear
          option PasswordAuth 'on'
          option Port '22'
          option Interface 'wan'

### Firewall

O firewall (_/etc/config/firewall_) deve permitir o encaminhamento:

        config defaults
        option syn_flood        1
        option input            ACCEPT
        option output           ACCEPT
        option forward          ACCEPT

### Network

Faça o backup das configurações de rede:

        cp /etc/config/network /etc/config/network.original

Devemos substituir a configuração original por uma que dê suporte ao OpenVSwitch.
Agora separamos cada uma das portas físicas em uma vlan própria e setamos a porta WAN para controle fora da banda do openflow.

_Com isso a comunicação switch/controlador se dará através de uma rede separada da rede para que esta possa ser controlada._


Este será o novo arquivo de rede, copie e cole em  _/etc/config/network_:

          config interface 'loopback'
                  option ifname 'lo'
                  option proto 'static'
                  option ipaddr '127.0.0.1'
                  option netmask '255.0.0.0'

          config globals 'globals'
          	option ula_prefix 'fd7d:e6fd:cd37::/48'

          #Porta 5 com IP estatico para acessar os ssh e para o controlador_

          config interface 'lan'
                  option ifname 'eth0'
                  option force_link '1'
                  option type 'bridge'
                  option proto 'static'
                  option ipaddr '192.168.2.1'
                  option netmask '255.255.255.0'
                  option defaultroute '1'
                  option gateway '192.168.2.2'
                  option dns '8.8.8.8'

          config switch
                  option name 'switch0'
                  option reset '1'
                  option enable_vlan '1'

          config switch_vlan
                  option device 'switch0'
                  option vlan '5'
                  option ports '5 0'

          config switch_vlan
                  option device 'switch0'
                  option vlan '1'
                  option ports '0t 1'

          config switch_vlan
                  option device 'switch0'
                  option vlan '2'
                  option ports '0t 2'

          config switch_vlan
                  option device 'switch0'
                  option vlan '3'
                  option ports '0t 3'

          config switch_vlan
                  option device 'switch0'
                  option vlan '4'
                  option ports '0t 4'

          config interface
                  option ifname 'eth0.1'
                  option proto 'static'
                  option ipv6 '0'

          config interface
                  option ifname 'eth0.2'
                  option proto 'static'
                  option ipv6 '0'

          config interface
                  option ifname 'eth0.3'
                  option proto 'static'
                  option ipv6 '0'

          config interface
                  option ifname 'eth0.4'
                  option proto 'static'
                  option ipv6 '0'

          config interface
                  option ifname 'br-lan'
                  option proto 'static'


Feito isto, edite o arquivo  _/etc/rc.local_  inserindo os comandos à seguir. Assim, será realizado inicialização automática do ovsdb-server(este faz o armazenamento das tabelas de fluxo) ao ligar o switch, juntamente com o estabelecimento a comunicação local e remota:

          ovs-ctl start;
          ovs-ctl stop;

          ovsdb-server --remote=punix:/var/run/openvswitch/db.sock \
                               --remote=ptcp:6640:192.168.2.1 \
                               --private-key=db:Open_vSwitch,SSL,private_key \
                               --certificate=db:Open_vSwitch,SSL,certificate \
                               --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
                               --pidfile --detach

          ovs-vsctl --no-wait init
          ovs-vswitchd --pidfile --detach

Agora iremos configurar o OpenWrt para trabalhar com o OpenVSwitch. Mas antes disso, salve e reinicie o roteador através do comando

        reboot


### Configure o Open vSwitch

Agora é a hora de configurar o Open vSwitch como uma bridge _br0_, adicionar portas vlans pre-definidas e configurar o OpenFlow para comunicar com o controlador.

Digite os comandos no terminal, sendo _br0_ o nome da bridge e o IP:porta referente a maquina onde o controlador irá rodar. Leve em consideração que a porta utilizada pode variar de um controlador para outro, portanto procure na documentação qual a porta utilizada pelo controlador desejado.

        ovs-vsctl add-br br0
        ovs-vsctl add-port br0 eth0.1 -- set interface eth0.1 ofport_request=1
        ovs-vsctl add-port br0 eth0.2 -- set interface eth0.2 ofport_request=2
        ovs-vsctl add-port br0 eth0.3 -- set interface eth0.3 ofport_request=3
        ovs-vsctl add-port br0 eth0.4 -- set interface eth0.4 ofport_request=4
        ovs-vsctl set-controller br0 tcp:192.168.2.2:6653

Por fim, podemos ainda setar alguns atalhos para facilitar no uso de alguns comandos
Edite o arquivo _/etc/profile_ a adicione essas linhas:

**OpenWRT Network Commands:**

        alias nwr='/etc/init.d/network restart'

**Open vSwitch Commands:**

        alias ovshow='ovs-vsctl show'
        alias ovmacs='ovs-appctl fdb/show br-lan'
        alias ovrestart='/etc/init.d/openvswitch restart'

**Open vSwitch OpenFlow Commands:**

        alias ofshow='ovs-ofctl show br-lan'
        alias offlows='ovs-ofctl dump-flows br-lan'
        alias ofports='ovs-ofctl dump-ports br-lan'



# Em Construção ...
