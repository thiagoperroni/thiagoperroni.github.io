---
layout: post
title: Posts Antigos 4 - Testando um PPTC
published: false
---
O PPTC, ou fusistor, é um componente usado para a proteção de circuitos eletrônicos contra altas correntes. A diferença dele para um fusível normal é que, após retirar a sobrecarga de corrente ele volta ao funcionamento normal, sem precisar ser substituído. Por conta dessa característica também é chamado de fusível resetável.


Cuidado! Ele não é um capacitor excitado.

O fusistor funciona como um PTC (resistor que varia com a temperatura). A temperatura ambiente ele funciona como um curto, mas com o aumento da corrente o componente esquenta, ao esquentar sua resistência aumenta, diminuindo a corrente fornecida para o circuito e protegendo o mesmo. Após esfriar sua resistência abaixa e ele volta a funcionar como um curto, sem interferir no funcionamento do circuito.

Nesse post vou documentar um teste simples para determinar duas características importantes de um fusistor: a corrente de disparo (a partir de qual corrente a resistência do fusistor aumenta) e o tempo levado para ele disparar (após quanto tempo ela aumenta). Lógico que esses dados e muito outros podem ser encontrados no datasheet, mas os testes podem ser úteis para identificar as características de fusistores xing-lings sem identificação ou que não sejam muito confiáveis.

Para fazer o teste usei, além do próprio fusistor, uma fonte de corrente variável, um multímetro para medir tensão, um sensor de corrente e uma Arduino UNO.

Sensor de corrente
Para medir a corrente passada pelo fusistor foi usado o sensor ACS758-100B, que consegue medir até 100 Amperes em qualquer sentido, gerando uma saída de 0 a 5V com uma precisão de 20mV por ampere. Este sinal é recebido pela Arduino e medido pela entrada analógica.




Montagem
A montagem ficou da seguinte maneira:




Como pode-se ver, o circuito é um curto. Isso é proposital para conseguir controlar a corrente fornecida pela fonte sem precisar de um resistor muito potente, já que para os testes precisa-se de uns 6 amperes no mínimo.

Corrente de disparo
Para determinar a corrente de disparo foi usado o seguinte código na Arduino:

void setup() {
  Serial.begin(9600);
}

void loop() {
  int sensor = analogRead(A0);
  float tensao = sensor * (5.0 / 1023.0);
  float corrente = (tensao - 2.5) / 0.02;
  Serial.println(corrente);
  delay(1);
}

Este código fornece a corrente que passa pelo sensor. E como tudo está em série, é a corrente que passa por todo o circuito. Fez-se então a medida da tensão sobre o fusistor em diferentes valores de corrente para poder ter a resistência dele em função da corrente.




Pode-se ver que a resistência, após um certo tempo, aumentou muito quando passou uma corrente de 4 Ampere pelo fusistor. E a medida da corrente após o disparo foi de 200 mA.

Tempo para disparar
Para determinar o tempo de disparo (time-to-time) foi usado o seguinte código na Arduino:

unsigned long inicio;
int zero;
int erro;

void setup() {
  Serial.begin(9600);
  Serial.println("contagem do tempo de disparo");

  zero = 512; //leitura do sensor sem nenhuma corrente passando
  erro = 3;
}

void loop() {
  inicio = millis();
  int sensor = analogRead(A0);

  if (sensor < zero - erro || sensor > zero + erro){
    unsigned long tempo = time_to_trip(inicio);
    if (tempo > 100){
      Serial.print("time to trip (ms): ");
      Serial.println(tempo);
    }
  }

  delay(1);
}

unsigned long time_to_trip (unsigned long comeco){
  int sensor;
  //Serial.println(comeco);
  do{
    sensor = analogRead(A0);
    delay(1);
  } while (sensor < zero - erro || sensor > zero + erro);

  return millis() - comeco;
}

Este código "conta" o tempo em que o fusistor foi submetido a uma corrente maior que 4A. Com ele foi possível determinar o tempo de disparo do fusistor, que é o tempo que ele demora para aumentar sua resistência após ser submetido a uma alta corrente.




Como vimos, o tempo para o fusistor esquentar e aumentar sua resistência não é instantâneo, por isso é importante ter certeza que o equipamento que está sendo protegido pelo fusistor suporte alguns segundos com sobrecarga.
