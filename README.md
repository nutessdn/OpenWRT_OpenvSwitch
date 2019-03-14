# OpenWRT_OpenvSwitch

Este tutorial tem como objetivo o uso do OpenWRT em um roteador comum, fazendo o uso do Open vSwitch
proporcionando o uso do OpenFlow SDN, que proporciona o desacoplamento entre os dados e o Plano de coontrole,
permitindo assim o uso de diversos controladores e apps de gerência para realizar o controle.

### Hardware necessário

Para realização deste tutorial será necessário o uso de um roteador doméstico modelo: TP-Link TL-WR1043ND versão 2.1 que será regravado com uma firmware [OpenWRT](https://g.co/kgs/EHYNN2) rodando o Open VSwitch que utiliza o OpenFlow.

Existem outros roteadores que podem ser configurados de maneira semelhante, porém com alguma diferença de configuração, para isto confira a lista de dispositivos que tem suporte à OpenWRT. [Confira a lista](https://openwrt.org/toh/views/toh_fwdownload)

_(Dispositivos com menos de 4MB de flash e/ou menos de 32MB de RAM sofrem com limitações de usabilidade, extensibilidade e instabilidade)_

### Cuidado


:fire: :skull:Este procedimento de "reflashing" em um roteador consequentemente anulará a garantia e poderá torná-lo "bricked", sendo assim irrecuperável. **Continue por sua conta e risco...**:fire: :skull:


### Transformando o roteador para OpenWRT

Faça as configurações básicas para iniciar o procedimento

Conecte a uma porta LAN um dispositivo que possa se comunicar usando SSH. Defina um ip estático 192.168.1.2 com mascara 255.255.255.0 (ou use DHCP), agora conecte-se com SSH em 192.168.1.1. Defina uma senha segunra para o root.

### Compilar o OpenWRT com uma imagem do Open vSwitch

_Para Compilar a firmware foi utilizado o Ubuntu 17.04._

#### Clonando o repositório do OpenWRT
No host de compilação, clone o OpenWRT: _(obs: no GitHub, não diretamente do site do OpenWRT)_

**$ git clone https://github.com/openwrt/openwrt.git**

#### Instalando as dependências

      sudo apt-get update
      sudo apt-get install git-core build-essential libssl-dev      libncurses5-dev unzip gawk zlib1g-dev
      sudo apt-get install subversion mercurial
      sudo apt-get install gcc-multilib flex gettext

#### Atualizando Feeds  

      cd openwrt
      ./scripts/feeds update -a
      ./scripts/feeds install -a

#### Make MenuConfig

      make MenuConfig

Alrere a opção selecionanda para o hardware utilizado( selecione a opção TP-Link TL-WR1043N / ND  para TP-Link TL-WR1043ND Hardware Version 2.1)

![Passo 01](https://raw.githubusercontent.com/nutessdn/OpenWRT_OpenvSwitch/master/imagens/passo1.png)



# Em Construção ...
