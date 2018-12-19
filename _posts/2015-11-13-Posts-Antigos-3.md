---
layout: post
title: [Posts Antigos #3] Primeiros Passos com o ESP8266
published: false
---
 O ESP8266 é um microcontrolador fabricado pela Espressif. O chip ficou bem famoso na comunidade Maker por ter conexão WiFi e custar algo em torno de 3 dólares. Ele é muito útil para aplicações de internet das coisas e automação residencial. Pude trabalhar com a breakout mais comum do chip, a ESP8266-01, e nesse post mostrarei quais são os primeiros passos a serem feitos após ter uma plaquinha dessas em mãos.

ESP8266-01
Descrição dos pinos
        Apesar do ESP8266 ter uns 20 pinos de GPIO, essa breakout deixa acessível apenas os pinos estritamente necessários para a placa funcionar. A vantagem é que o módulo fica pequeno, a desvantagem é ter apenas 2 pinos GPIO (para resolver isso vc pode fazer como esse russo aqui). As funções dos pinos são as seguintes:
TX e RX: fazem a comunicação serial UART e também é o meio usado para instalar firmwares na placa.
GPIO0 e GPIO2: pinos digitais de entrada e saída. Eles também são responsáveis por determinar o modo de inicialização da placa (sabe quando vc liga o windows apertando o F11 para entrar em modo de segurança? Então, a ESP funciona parecido: ligar a placa com GPIO0 e GPIO1 ligados no Vcc, por exemplo, faz ela entrar no modo de operação normal. Farei um post sobre isso em breve).
CH_PD: pino que habilita o chip, deve estar conectado no Vcc para funcionar.
RST: pino de reset, ativo quando é conectado ao GND.
Vcc e GND: alimentação, a placa funciona com 3,3 Volts.
Hardware para testes
        Para poder atualizar o firmware da placa, carregar programas ou então checar se ela ainda esta funcionando mesmo depois de ter alimentado no 220V, é preciso um hardware que faça a conexão da placa com o computador pela porta USB. Eu sugiro esse esquemático aqui (tirado da bíblia do ESP8266, do INRI Neil Kolban):

Obs.1) o "FTDI Basic Programer" pode ser qualquer conversor TTL ou cabo FTDI. Estou usando um cabo FTDI modificado (o mesmo desse post aqui), que transmite os dados em 3,3V, assim não preciso do divisor de tensão R4 e R5 entre o cabo e o ESP e posso alimentá-lo pela própria USB. Mas se vc tiver um cabo convencional de 5V, é de EXTREMA IMPORTÂNCIA o divisor para não queimar a placa!
Obs.2) vc pode montar esse circuito na protoboard ou em uma pcb. A pcb da mais trabalho, mas pode evitar ruídos indesejáveis durante o teste.

        Para saber se está tudo certo com a montagem e o ESP, abra um terminal serial no seu computador (pode ser o Putty, o terminal da IDE do Arduino ou algum outro de sua preferência), colocando a porta que o ESP está conectado e o baud rate de 115200 bps (ou 9600, dependendo da versão). Digite então "AT" e vc deverá receber um "OK" como resposta. Se isso não ocorrer, verifique novamente a montagem. Caso ainda esteja dando problema, talvez seja preciso atualizar o firmware (também farei um post sobre isso em breve).



Configurações importantes caso for usar o putty
        

Exemplo de teste na IDE Arduino
        Com esse hardware básico e a placa funcionando, vc ja está preparado para se aventurar no mundo da interent das coisas com o ESP8226!
