---
layout: post
title: Posts Antigos 2 - Gerador de funções com Intel Galileo, um resistor e um capacitor
published: false
---
Esse post vai mostrar uma maneira de se fazer um gerador de funções usando a placa Galileo Gen2, a IDE do Arduino para programação e um filtro passa-baixas. Com certeza não é o projeto de gerador de funções mais preciso e eficiente de todos, mas me ajudou a entender um pouco mais sobre a placa da Intel e seus limites, além de aprender como fazer um programa com mais de um arquivo na IDE do Arduino.
        Uma vantagem é a simplicidade do hardware, qualquer um que tenha a disposição uma Intel Galileo (neste exemplo foi usada a geração 2) pode montar esse gerador. A ideia é usar a grande capacidade de processamento e memória da Galileo para gerar ondas de diversas formas, com uma larga faixa de frequência e bastante precisão. 
         O método escolhido para gerar as ondas foi a síntese digital direta (em inglês, DDS).

O método DDS
        Aprendi bastante coisa sobre essa maneira de gerar ondas aqui e aqui; e me inspirei pra fazer o programa nesse projeto aqui .

        Resumindo, na síntese digital direta o sinal periódico que se deseja gerar é armazenado em amostras digitais que são “mandadas” para um conversor digital-analógico em um tempo certo. Este tempo irá determinar a frequência do sinal. Uma maneira de utilizar um microcontrolador para implementar este método é usando um vetor para guardar cada amostra da onda e uma saída PWM seguida de um filtro passa-baixas para a conversão digital-analógica.       
        Usando esse método, a frequência do sinal gerado depende da frequência do PWM e do número da amostras do sinal. Logo, para alcançar altas frequências precisa-se de um PWM com altas frequências também.

A saída PWM da Galileo
        Existem várias maneiras de gerar uma saída PWM na Galileo, a mais comum é utilizar um chip que está na placa justamente pra isso. Na IDE do arduino essas saídas são obtidas via comando AnalogWrite(pino, dutyCicle); e tem frequências de funcionamento padrões para trabalhar com servo motores (48Hz).
        Pode-se mudar essa frequência via comandos do Linux como visto aqui. A frequência máxima obtida dessa maneira é 13,97 KHz. Frequência suficiente se pensarmos em aplicações como controlar um servo motor, mas baixa para geração de sinais. A solução foi então criar uma nova classe que implemente um PWM para a Galileo e alcance frequências MUITO MAIORES.
        Uma maneira organizada de criar uma classe é colocando ela em um arquivo separado. Para criar uma classe em um arquivo separado na IDE do Arduino devemos fazer o seguinte: abrir o sketch, clicar na "setinha" do canto superior direito e criar duas novas abas (uma com extensão .h e outra com extensão .cpp). Elas devem ter o mesmo nome, eu chamei de PWMGalileoGen2.h e PWMGalileoGen2.cpp.


Abrindo novos arquivos

        O cabeçalho (arquivo .h) tem o seguinte código:

#ifndef PWMGalileoGen2_h
#define PWMGalileoGen2_h

class PWMGalileoGen2{
  public:
    PWMGalileoGen2();
    void periodo(int resolucao, int dutyCicle);
  
  private:
    unsigned int latchValue;
    unsigned int bmask;
    void tOn(int n);
    void tOff(int n);
};

#endif

Vou explicar um pouco como a classe funciona. Na verdade essa classe não programa uma saída PWM comum, na qual você pode ativá-la com um certo duty cicle e continuar o fluxo do programa normalmente. Ela tem um método void periodo(int resolução, int dutyCicle); que gera um único período da onda quadrada no pino 2 com o tempo determinado pela resolução. Se quisermos uma saída PWM contínua, devemos chamar esse método continuamente. Como teremos que variar o duty cicle do PWM o tempo todo conforme a amostra do sinal muda, esta solução de PWM resolve o problema.
O código (arquivo .cpp) fica assim:

#include "Arduino.h"
#include "PWMGalileoGen2.h"

PWMGalileoGen2::PWMGalileoGen2(){
  pinMode(2, OUTPUT_FAST);
  latchValue = fastGpioDigitalRegSnapshot(GPIO_FAST_IO2); //le o valor atual
  bmask = 0x000000ff & GPIO_FAST_IO2; //retira a mascara que indentifica o pino 2
  if (latchValue & bmask) latchValue = GPIO_FAST_IO2 & !bmask; //comeca sempre em LOW
}

void PWMGalileoGen2::periodo(int resolucao, int dutyCicle){
  tOn(dutyCicle);
  tOff(resolucao - dutyCicle);
}

void PWMGalileoGen2::tOn(int n){
  latchValue = GPIO_FAST_IO2 | bmask;
  for(int i = 0; i < n; i++)
    fastGpioDigitalRegWriteUnsafe (GPIO_FAST_IO2, latchValue);
}

void PWMGalileoGen2::tOff(int n){
  latchValue = GPIO_FAST_IO2 & !bmask;
  for(int i = 0; i < n; i++)
    fastGpioDigitalRegWriteUnsafe (GPIO_FAST_IO2, latchValue);
}

O importante a comentar sobre esta parte do código são os palavrões fastGpioDigitalRegWriteUnsafe(GPIO_FAST_IO2,latchValue); e fastGpioDigitalRegSnapshot(GPIO_FAST_IO2);. Estes comandos são equivalentes aos famosos DigitalWrite(); e DigitalRead(); do arduino, porém se usados corretamente na Galileo fazem com que o tempo de resposta seja incrivelmente rápido, podendo chegar a 2,93MHz!(veja mais neste post). Assim é possível alcançar frequências maiores do que o PWM comum da Galileo.
Para testar esta classe, podemos escrever este programa de teste no scketch:

#include "Arduino.h"
#include "PWMGalileoGen2.h"

PWMGalileoGen2* pwm;
int resolucao = 100;
int duty_cicle = 50;//50 por cento
int duty = int (resolucao*duty_cicle/100);

void setup(){
  pwm = new PWMGalileoGen2();
}

void loop(){
    pwm->periodo(resolucao, duty);
}

Podemos ver o resultado no osciloscópio: 


Exemplo de PWM a 100KHz e 10% de duty cicle



Exemplo de PWM a 1,47MHz

Pode-se notar que só é possível alcançarmos os 1,47MHz se colocarmos a menor resolução possível e 50% de duty cicle, ou seja, quanto maior a frequência, menor a resolução.

Gerador de funções
       Lembrando que queremos usar o PWM como forma de converter sinais digitais para analógicos. Agora que já temos como fazer isso, é preciso criar as amostras digitais dos sinais que queremos gerar e colocá-las em um vetor.
        Seguindo a mesma estratégia de separar a principais partes do programa em objetos, vamos também criar uma classe para gerar os sinais assim como foi feito com o PWM, só que desta vez mais rápido.

GeradorSinal.h:

#ifndef GeradorSinal_h
#define GeradorSinal_h

#include "Arduino.h"

class GeradorSinal{
  public:
    GeradorSinal(int numeroDePassos);
    void geraOndaSenoidal(int *sinal, int resolucao);
    void geraOndaTriangular(int *sinal, int resolucao);
    void geraOndaQuadrada(int *sinal, int resolucao);   

  private:
    int _numeroDePassos;
};

#endif

GeradorSinal.cpp:

#include "Arduino.h"
#include "GeradorSinal.h"

GeradorSinal::GeradorSinal(int numeroDePassos){
  _numeroDePassos = numeroDePassos;
}

void GeradorSinal::geraOndaSenoidal(int *sinal, int resolucao){
  double valor;
  float rad = 2*PI/_numeroDePassos;  //tamanho do passo em rad
  for (int i = 0; i < _numeroDePassos; i++){
    valor = (sin(rad*i) + 1)/2;     //senoide entre 0 e 1
    sinal[i] = (int) (valor*resolucao);
  }
}

void GeradorSinal::geraOndaTriangular(int *sinal, int resolucao){
  float valor = 0;
  float acrescimo = 2*resolucao/_numeroDePassos;     //tamanho do acrescimo em cada passo
  for (int i = 0; i < _numeroDePassos; i++){
    sinal[i] = (int) (valor);
    if (valor >= resolucao) acrescimo = -acrescimo; //pico da onda
    valor += acrescimo;
  }
}

void GeradorSinal::geraOndaQuadrada(int *sinal, int resolucao){
  for(int i = 0; i < _numeroDePassos; i++){
    if (i < _numeroDePassos/2) sinal[i] = resolucao;
    else sinal[i] = 0;
  }
}

Foram implementados 3 métodos para gerar ondas senoidais, quadradas e triangulares. A variável inteira “numeroDePassos” indica o numero de amostras por período.

O filtro passa-baixas
        Como disse acima, o hardware é bem simples: um resistor em série com um capacitor e seus valores determinados pela frequência de corte (pode-se entender mais sobre passa-baixas aqui). Mas surgiu um problema: ao mudar a frequência da onda, também muda-se a frequência do PWM, logo a frequência de corte também muda. Resultado: os valores do resistor e do capacitor devem ser diferentes dependendo da frequência desejada.
        A solução foi usar um potenciômetro para variar a frequência de corte do filtro. O hardware final ficou assim:


Potenciômetro de 10K e capacitor de 100nF
Testes finais
         Temos até agora 5 arquivos, o sketch principal e as duas classes, e o hardware pronto. Vamos reescrever então o sketch para fazer os testes finais.

#include "Arduino.h"
#include "GeradorSinal.h"
#include "PWMGalileoGen2.h"

const int numeroDePassos = 100;
int frequencia = 1000;
int resolucao = (int) (2*2940000/(frequencia*numeroDePassos));
GeradorSinal* gerador;
PWMGalileoGen2* pwm;
int sinalGerado[numeroDePassos];

void setup(){
  gerador = new GeradorSinal(numeroDePassos);
  gerador->geraOndaSenoidal(sinalGerado, resolucao);
  pwm = new PWMGalileoGen2();
}

void loop(){
  for(int i = 0; i < numeroDePassos; i++)
    pwm->periodo(resolucao, sinalGerado[i]);
}

        Nesse sketch, devemos colocar na variável inteira “numeroDePassos” o número de amostras por período e na variável inteira "frequência" a frequência desejada em Hz. Quanto maior for o o valor de numeroDepassos, maior será a precisão da onda, mas a frequência máxima será menor. A forma de onda é gerada dentro do setup(). No exemplo acima, o gerador irá gerar uma onda senoidal de 1000 Hz e com 100 amostras por período da onda.


Imagem do osciloscópio - onda senoidal de 1KHz

Neste outro exemplo temos uma onda senoidal a 100 Hz. A onda em amarelo é a onda depois do filtro, e a verde antes do filtro. Pode-se notar a variação do duty cicle do PWM.



        Outros exemplos:


Onda quadrada a 500Hz


Onda triangular a 1KHz

Onda senoidal a 20KHz

Comentários finais
         Nos testes alcancei ondas senoidais e triangulares de até 30KHz e ondas quadradas de até 500KHz, o que achei bem grande se comparado com outros projetos de geradores microcontrolados. No entanto a precisão não ficou la essas coisas, esse é o ponto a ser melhorado, talvez mudando o filtro por um melhor ou a forma como a síntese digital direta é implementada.
        Outra melhoria é implementar alguma interface que possibilite a geração de ondas genéricas, isso pode ser feito usando o Processing ou outro programa do gênero.
          Por fim, gostaria de dizer que achei este post um pouco confuso rsrs, então se tiverem alguma dúvida ou sugestão, coloquem nos comentários que eu tentarei responder.
