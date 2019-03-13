# OpenWRT_OpenvSwitch

Este tutorial tem como objetivo o uso do OpenWRT em um roteador comum, fazendo o uso do Open vSwitch
proporcionando o uso do OpenFlow SDN, que proporciona o desacoplamento entre os dados e o Plano de coontrole,
permitindo assim o uso de diversos controladores e apps de gerência para realizar o controle.

### Hardware necessário

Para realização deste tutorial será necessário o uso de um roteador doméstico modelo: TP-Link TL-WR1043ND versão 2.1 que será regravado com uma firmware [OpenWRT](https://g.co/kgs/EHYNN2) rodando o Open VSwitch que utiliza o OpenFlow.

Existem outros roteadores que podem ser configurados de maneira semelhante, porém com alguma diferença de configuração, para isto confira a lista de dispositivos que tem suporte à OpenWRT. [Confira a lista](https://openwrt.org/toh/views/toh_fwdownload)

_(Dispositivos com menos de 4MB de flash e/ou menos de 32MB de RAM sofrem com limitações de usabilidade, extensibilidade e instabilidade)_

##**Cuidado** !
:fire: :skull:Este procedimento de "reflashing" em um roteador consequentemente anulará a garantia e poderá torná-lo "bricked", sendo assim irrecuperável. **Continue por sua conta e risco...**:no_entry::no_entry::no_entry:

 
