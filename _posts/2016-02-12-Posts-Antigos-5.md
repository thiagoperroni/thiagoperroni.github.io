---
layout: post
title: Posts Antigos 5 - Maneiras de programar o ESP8266 
published: false
---
Existem várias maneiras de programar o ESP8266, neste post vou ensinar a configurar e usar duas IDEs que facilitam a programação do ESP: o ESPlorer e a IDE Arduino. Escolhi essas duas porque acho que são as maneiras mais fáceis e as que tem mais suporte da comunidade na internet.
ESPlorer
O ESPlorer é uma IDE que suporta vários firmwares: o dos comandos AT (que já vem quando vc compra o ESP), NodeMcu (programação através da linguagem lua), MicroPython e parece que vai sair um modo Frankenstein (seja lá como isso funcione). Destes acho que o melhor firmware é o NodeMcu, mas para usá-lo é preciso fazer o flash.

Janela principal do ESPlorer. A esquerda, o editor de texto para escrever os programas. A direita, um terminal para debug.

Flasheando o firmware
Para fazer o flasinhg do firmware usei o esptool (é um nome melhor que o outro), que é um programa em python muito bom para flashear firmwares no ESP. Baixe os arquivos e instale usando o comando: 
python setup.py install
Agora é preciso ter o firmware. Acho que a forma mais fácil de ter o firmware do NodeMcu mais atualizado é usando esse site aqui, você recebe o firmware por email em no máximo 8,5 minutos. Com ele ainda é possível customizar o firmware, escolhendo que módulos deseja ter.
Uma vez com o esptool instalado corretamente e com o firmware em mãos, é preciso fazer um ritual:
1. conecte o ESP em uma porta USB e descubra qual porta é (no windows será a COMX, sendo X algum número). Mostrei uma maneira de fazer isso nesse post.
2. coloque o ESP no modo de flash. Para fazer isso basta resetar o ESP com o pino GPIO0 conectado ao terra.
3. na linha de comando, vá até a pasta que possui o arquivo .bin e digite o comando (lembrando de trocar o "COMX" pela porta certa e o "arquivo_que_tem
_o_firmware.bin" pelo nome certo):
python esptool.py -port COMX write_flash 0x00000 arquivo_que_tem_o_firmware.bin 
4. se não aparecer nenhuma mensagem de erro, basta tirar o ESP do modo flash resetando ele com o GPIO0 conectado ao Vcc e pronto!
Aqui um exemplo na linha de comando. Usei os comandos "python esptool.py -h" para mostrar todas as possibilidades do programa, e o comando do item 3 para mostrar a resposta esperada. 


obs.: a maioria dos erros ao fazer o flash do firmware ocorre porque o ESP não encontra-se em modo de flash, certifique-se de que executou corretamente o passo 2.


Usando o ESPlorer

Agora, abra o ESPlorer (que pode ser baixado daqui) e, com o ESP conectado, clique no botão de atualizar para ele encontrar a porta USB em que o ESP está conectado. Em seguida escolha tal porta e abra o terminal clicando em "open". Como mostra a imagem abaixo, a mensagem "NodeMCU firmware detected" indica sucesso.



Caso apareça a mensagem "Can't autodetect firmware, because proper answer not received." não chore ainda! Há duas coisas que podem ter acontecido:

1. Firmware foi instalado corretamente e já existe um programa rodando nele que atrapalhou a resposta. Clique em "format" no canto superior direito do ESPlorer e tente abrir novamente o terminal.
2. O firmware realmente não foi instalado corretamente. Repita o processo de flashear o firmware.

Pronto. Já é possível escrever seus programas em lua e testá-los no ESP de maneira rápida. Basta escrever um código na janela da esquerda (nomeie o arquivo de init.lua para tê-lo inicializado toda vez que o ESP for inicializado) e clicar em "Save to ESP", no canto inferior esquerdo. Veja aqui tudo o que é possível fazer com o firmware NodeMcu.



Exemplo de código com um server TCP sendo enviado para o ESP
IDE Arduino
Também é possível programar o ESP usando a IDE Arduino, como se ela fosse uma placa qualquer da Arduino. A partir da versão 1.6 da IDE já é possível adicionar a placa através do boards manager:

1. Vá em preferências e adicione o link: http://arduino.esp8266.com/stable/package_esp8266com_index.json na janela "Additional Boards Manager URL's".





2. Vá em Ferramentas>Placa>Boards Manager e instale o pacote de nome ESP8266.




3. Escolha a ESP que você possui (no caso das ESP-01 use a "Generic ESP8266 Module") e a porta. Pronto, já pode programar o ESP como se fosse uma Arduino.




Lembrando que a IDE compila um novo firmware, então toda vez que for carregar algum sketch você tem que deixar o ESP em modo de flash. Para fazer isto deve-se resetar o ESP com o pino GPIO0 conectado ao terra.

Como disse, tem muito material na internet sobre essas IDEs e o ESP8266. Vou deixar algumas dicas:

Fórum do ESP8266: http://www.esp8266.com/index.php?sid=4ce9e4866b9e23ed4e1435fed5611178
FAQ interessante sobre o NodeMcu e a linguagem lua: http://www.esp8266.com/wiki/doku.php?id=nodemcu-unofficial-faq
Bíblia do ESP8266: http://neilkolban.com/tech/esp8266/
