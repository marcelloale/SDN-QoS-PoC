# Prova de Conceito de QoS do OpenFlow

Este código fornece uma rede emulada para testar o QoS de Rede Programável através de Filas do OpenFlow 1.0.

## Instruções

As seguintes instruções são projetadas para funcionar em um Ubuntu 16.04. Deve funcionar em outros sistemas operacionais Linux com os pacotes corretos instalados.

### Requisitos

- Emulador de rede Mininet modificado. Fornecemos um Mininet modificado que permite o uso de TCLinks (links emulados pelo mininet com largura de banda e/ou atraso fixos) juntamente com switches virtuais com filas (openvswitch).
- openvswitch-switch.
- Controlador SDN (openvswitch-testcontroller).

### Instalação

Para instalar o emulador de rede mininet modificado (se tiver alguma duvida sobre o [install.sh do Mininet](https://github.com/marcelloale/SDN-QoS-PoC/blob/master/mininet/INSTALL)):

```bash
cd mininet
./util/install.sh -n
``` 

Para instalar o openvswitch-switch:

```bash
sudo apt-get update && sudo apt-get install openvswitch-switch
```

Para instalar o Controlador SDN:

```bash
sudo apt-get install openvswitch-testcontroller
```

Pare o controlador SDN e desative-o na inicialização. O script `upgrade_QoS.py` fornecido gerenciará o ciclo de vida do controlador:

```bash
sudo systemctl stop openvswitch-testcontroller
sudo systemctl disable openvswitch-testcontroller
```

### Executando
O Mininet está configurado para executar a seguinte topologia de rede.
![Topologia da Rede](network_topology.png)

Lançamos a topologia mininet com um script python:

```bash
sudo python upgrade_QoS.py 
```

Este comando abre três terminais e dois clientes vlc.


Usamos o [video-small.3gp](http://mirrors.standaloneinstaller.com/video-sample/P6090053.3gp) nesta prova de conceito.

Iniciamos a emissão do fluxo de vídeo executando no terminal “RTP Stream Emiter” o seguinte comando:

```bash
./send_video.sh CAMINHO_PARA_O_ARQUIVO_DE_VIDEO
```

Ambos os clientes vlc (usuário regular e premium) devem começar a reproduzir o vídeo.

Geramos um fluxo de dados UDP de baixa prioridade que competirá pela largura de banda com o usuário regular (durante 300 segundos). Fazemos isso digitando no terminal “UDP Noise Sender” o comando:
 
```bash
iperf -u -c 10.0.0.5 -b 5M -t 300
```

O usuário *regular* começará a experimentar problemas de qualidade: seu fluxo de vídeo compartilha a fila de saída do switch ‘core2’ com o fluxo de dados com ruído UDP.

Podemos dinamicamente reatribuir o usuário regular para à fila de alta prioridade no switch core2 para melhorar sua qualidade de experiência. Enviamos uma regra OpenFlow para fazer essa reatribuição. 
Abrimos um novo terminal e executamos o script `set_priority`:


```bash
./set_priority.sh 10.0.0.1 10.0.0.4 hi
```

Também podemos diminuir o QoE do usuário *premium* com o mesmo script:


```
./set_priority.sh 10.0.0.1 10.0.0.3 lo
```

O último comando fará com que o usuário *premium* comece a experimentar problemas de vídeo, porque reatribuímos dinamicamente seu fluxo de vídeo para a fila regular.

