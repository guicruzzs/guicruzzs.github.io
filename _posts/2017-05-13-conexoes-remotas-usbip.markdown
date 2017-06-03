---
layout: post
title:  "Conexões remotas de USB via usbip"
date:   2017-05-13
---

<p class="intro"><span class="dropcap">I</span>magine utilizar a rede para compartilhar uma conexão de um dispositivo USB, seja ele uma câmera, um HD externo, mouse, impressora, ou qualquer outro dispositivo. Há uma ferramenta no repositório Linux que resolve isso.</p>

Foi com essa necessidade que eu conheci o [usbip](http://usbip.sourceforge.net/). Ele foi criado para transmitir pela rede portas USB (*USB over IP*). O usbip abstrai uma conexão USB em um device genérico (*server*), por meio do VHCI, e a partir daí transmitir os dados pra outro computador via rede. Isso permite que este receptor (*client*) dos dados USB possa montar seu próprio device e injetar a comunicação que ele recebe via rede.

Acabei de descrever o processo como se ele fosse numa mão única, um *read-only*, mas não é verdade. O *client* também pode enviar os dados pro *server* pela mesma comunicação anteriormente estabelecida.

Quem gerencia os devices que subirão, bem como a comunicação entre *clients* e *server* é o ilustríssimo usbip. E você não leu errado, *clients* no plural mesmo, é possível que um mesmo dispositivo seja compartilhado com diversas outras máquinas.

O usbip foi criado por **Takahiro Hirofuchi** com uma [publicação inicial](https://www.usenix.org/legacy/events/usenix05/tech/freenix/hirofuchi/hirofuchi_html/index.html) que explica os conceitos e a arquitetura da aplicação. Caso você queira entender melhor como as coisas funcionam debaixo do capô, eu sugiro que olhe também o [site original da aplicação](http://usbip.sourceforge.net/) e o [repositório](https://github.com/torvalds/linux/tree/master/tools/usb/usbip) que atualmente está no *core tools* do Linux (obrigado L. Torvalds).

O usbip foi concebido para integrar devices entre diferentes sistemas operacionais. Fiz alguns testes com Linux e Windows e verifiquei que funciona perfeitamente **apenas entre duas máquinas com Linux**. Segui as instruções, instalando e compilando os drivers pro Windows, mas aparentemente surgiu alguma incompatibilidade que não foi tratada. Gostaria de ter testado também no macOS antes dessa publicação, mas não foi possível, então sinta-se à vontade :).

### Instalação

Atualmente a versão estável do usbip é a 2.0. Para instalar, basta que você baixe o [código fonte](https://github.com/torvalds/linux/tree/master/tools/usb/usbip) e o compile, como utilizo a distribuição Ubuntu 16.04, instalei o pacote do *tools-generic* da seguinte forma:

{% highlight shell %}
$ sudo apt-get install linux-tools-generic
{% endhighlight %}

#### Atenção
Utilizaremos duas aplicações: **`usbip`** e **`usbipd`**.

No caso do Ubuntu, você terá a aplicação disponível na pasta `/usr/lib/linux-tools/versao-do-kernel-generic/`
onde `versao-do-kernel-generic` pode e vai variar de acordo com a versão da *tools-generic*.

Caso você tenha compilado, essas duas aplicações estarão disponíveis por padrão dentro da pasta `src`.

### Server

Antes de rodar a aplicação que irá distribuir um dispositivo USB, é preciso habilitar 2 módulos:
{% highlight shell %}
$ sudo modprobe usbip_core
$ sudo modprobe usbip_host
{% endhighlight %}

Basta então que invoque a aplicação *server*, em sua respectiva pasta:
{% highlight shell %}
$ sudo ./usbipd
usbipd: info: starting usbipd (usbip-utils 2.0)
usbipd: info: listening on 0.0.0.0:3240
usbipd: info: listening on :::3240
{% endhighlight %}

A aplicação responde avisando onde estará disponível a comunicação com outro device, no caso por padrão, na porta `3240`.

Se você não quiser ver os logs e/ou não interagir diretamente com a aplicação, basta utilizar a opção `-D` que roda como *daemon*.

É possível visualizar os dispositivos locais disponíveis por meio do comando:
{% highlight shell %}
$ ./usbip list -l
 - busid 1-1.1.1 (413c:8161)
   Dell Computer Corp. : Integrated Keyboard (413c:8161)

 - busid 1-1.1.2 (413c:8162)
   Dell Computer Corp. : Integrated Touchpad [Synaptics] (413c:8162)

 - busid 2-1.8 (0c45:6481)
   Microdia : unknown product (0c45:6481)
{% endhighlight %}

A partir dessa listagem é possível disponibilizar algum dispositivo para conexão, basta executar o comando `bind`:
{% highlight shell %}
$ sudo ./usbip bind -b 1-1.1.2 #1-1.1.2 é o ID do dispositivo listado anteriormente
usbip: info: bind device on busid 1-1.1.2: complete
{% endhighlight %}

### Client

Agora em outra máquina, é possível conectar-se ao *server* e conversar com os dispositivos que estão disponíveis. Para isso, é preciso saber o endereço IP da máquina que está rodando o *server*, nos exemplos abaixo assumirei que esse IP é `192.168.0.17`.

Além de ter também a aplicação usbip instalada nessa máquina, os seguintes módulos precisam ser ativados:
{% highlight shell %}
$ sudo modprobe usbip_core
$ sudo modprobe vhci-hcd
{% endhighlight %}

Para listar os dispositivos disponíveis do *server* (anteriormente habilitados pelo `bind`), execute:
{% highlight shell %}
$ ./usbip list -r 192.168.0.17
Exportable USB devices
======================
 - 192.168.0.17
    1-1.1.2: Dell Computer Corp. : Integrated Touchpad [Synaptics] (413c:8162)
           : /sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.1/1-1.1.2
           : (Defined at Interface level) (00/00/00)

{% endhighlight %}

Basta então, se conectar ao dispositivo que desejar através do seu ID:
{% highlight shell %}
$ sudo ./usbip attach -r 192.168.0.17 -b 1-1.1.2
{% endhighlight %}

E voilà! Você consegue então utilizar por meio da rede, um dispositivo USB que está em outra máquina!

### Considerações finais

As possibilidades de utilização são ampliadas se você pensar que o seu USB *server* pode ser RaspberryPi, por exemplo. Utilizo essa aplicação para alcançar alguns cenários de IoT, mas começar pelo mouse pode ser uma boa (no meu caso foi bem divertido).

A taxa de transferência de dados pode ser um problema dependendo do tipo de dispositivo USB que você deseja conectar via rede, como estou aplicando em cenários menores, isso me atende, mas quando se trata de disco externo ou mesmo pen-drive nota-se um atraso bem grande.

Utilize o `lsusb` pra te ajudar quando tiver dúvida sobre os dispositivos conectados em sua máquina. Isso serve tanto pro *client* quanto pro *server*, pode ser uma boa baliza para seus experimentos e testes.

A maior parte dos comandos precisa ser executado com permissão de usuário `root`, isso acontece porque é preciso manipular dispositivos internamente, o que pede algum grau de permissão além do usuário simples. Mas fique tranquilo, não há código malicioso nessa aplicação, ela é canônica e foi avaliada por uma equipe qualificada.

Para desconectar os dispositivos, utilize os comandos `detach` e `unbind` no *client* e *server* respectivamente. O comando `help` pode te ajudar bastante nisso ;).

Não há autenticação para acessar os dispositivos, como você pode ter observado. Portanto, seja cuidadoso ao compartilhar os *devices*, garantir que somente as pessoas devidas tem acesso a essa rede é um bom caminho.

Não há necessidade alguma do *server* ter os drivers de utilização do *device*, apenas o *client*. O que de certa forma é incrível, pois no final a rede acaba sendo um mero condutor do sinal!

Aproveite, há muitas possibilidades para serem exploradas :)

Bom dia e boa sorte!