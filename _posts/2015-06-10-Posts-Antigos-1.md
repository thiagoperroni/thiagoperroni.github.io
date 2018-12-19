---
layout: post
title: Posts Antigos 1 - Acessando o console do linux embarcado na placa Intel Galileo Gen 2
published: false
---
Como todos sabem, a Intel Galileo possui um sistema operacional Linux embarcado dentro dela. Logo, é bem útil que se possa acessar seu console para executar alguns comandos como acionar portas digitais e mudar as configurações das saídas PWM. Este post irá mostrar um tutorial de como acessar o console do Linux de uma Intel Galileo Gen 2 usando um terminal de comunicação serial e um cabo FTDI. Antes de começar, você irá precisar de:(além de um computador com entrada USB e a Intel Galileo Gen 2, claro)

- Um cabo FTDI 3,3V como este aqui: https://www.sparkfun.com/products/9717
- Um software de comunicação serial (neste tutorial usei o Putty: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)

Observações importantes
1) Se você possui a Intel Galileo (a versão anterior), o cabo de comunicação é outro! Este tutorial irá avisar quase sempre se alguma coisa não for compatível com a primeira geração da placa, mas recomenda-se que quem tiver a Galileo busque na internet outros tutoriais de apoio.
2) A Galileo Gen 2 não possui um conversor de nível TTL como na placa "antiga". Logo, é imprescindível que o próprio cabo transmita os dados em 3,3V. O pino de Vcc pode ser 5V, pois ele não vai a lugar algum na placa.
3)Neste tutorial foi utilizado o sistema operacional Windows 7. Provavelmente irá funcionar em outras versões do Windows. Para quem usa Mac ou Linux, recomenda-se que busquem na internet outros tutoriais de apoio (no fim do post darei algumas dicas para encontrar)

Passo a passo

1)     Conecte o conector de 6 pinos do cabo na placa (veja na figura o lado correto) e o USB no computador



2)     Se é a primeira vez que você conecta um cabo FTDI no seu computador, provavelmente irá precisar instalar o driver manualmente (que pode ser encontrado em: http://www.ftdichip.com/FTDrivers.htm). Caso tenha dificuldades basta seguir este ótimo tutorial da Sparkfun: https://learn.sparkfun.com/tutorials/how-to-install-ftdi-drivers/all

3)     Abra o terminal com algumas configurações como: número da porta COMX (no exemplo é a COM10, para ver qual seu número entre na janela “Gerenciador de Dispositivos” do Windows); o tipo de conexão é serial; e a velocidade é de 115200 bps.




4)     Ao clicar em "Open", uma janela igual a esta se abrirá:




digite “root” e aperte ENTER. Pronto, você já pode aproveitar todas possibilidades que o Linux da Galileo pode te oferecer.

Dicas de comandos legais
                Para ver o nome da sua plataforma, tente digitar:
root@clanton:~# cd /sys/devices/platform/GalileoGen2/
root@clanton:/sys/devices/platform/GalileoGen2# cat modalias
                E aparecerá:
platform:GalileoGen2
                
                Para a Galileo seria algo mais ou menos assim:
root@clanton:~# cd /sys/devices/platform/Galileo
root@clanton:/sys/devices/platform/Galileo# cat modalias
platform:Galileo

                Se você digitar o comando abaixo, irá receber todos os comandos que podem ser executados na Galileo:

root@clanton:/sys/devices/platform/GalileoGen2# busybox


Onde encontrar mais informações
                Este tutorial foi feito seguindo as instruções do livro "Intel Galileo and Intel Galileo Gen 2: API Features and Arduino Projects for Linux Programmers" do Manoel Carlos Ramon. O livro pode ser baixado gratuitamente em http://www.apress.com/9781430268390?gtmf=c e tem várias outras aplicações legais da placa, além de instruções pra quem usa Linux e Mac.
