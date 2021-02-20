# Projeto_finalINTRODUÇÃO:

Em diversas máquinas e processos industriais são utilizados sensores com controle de temperatura e umidade. Pensou-se em um projeto final com uma leitura de um sensor de temperatura e umidade, amostragem em um display LCD e o controle desses pontos, com aquecedores, refrigeração, umidificador e desumidificador. O controlador pode ser utilizado em diversas aplicações onde necessita-se um controle de temperatura e umidade.
No projeto contempla um liga / desliga de uma saída via wifi, para diversas aplicações.

MATERIAL E RECURSOS UTILIZADOS:

Utilizou-se para o projeto os itens:
•	Placa ESP32;
•	Display LCD 16 x 2;
•	Fonte 5Vcc;
•	Diodo 4007;
•	Resistor 1K OHM;
•	Protoboard;
•	Trimpot 10K OHM;
•	Cabos para ligações;

O software utilizado foi o Visual Code com o PlataformIO.

ESQUEMA ELÉTRICO:

Esquema segue no fim do trabalho.

CÓDIGO FONTE:

Abaixo segue o código fonte da programação.
/*  Autor: Fernando Henrique Dorigan
    RA: 200012997 */

/*Include de bibliotecas utilizadas para a aplicação*/
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "freertos/semphr.h"
#include <WiFi.h>
#include <Arduino.h>
#include <LiquidCrystal.h>
#include <DHT_U.h>
#include <DHT.h>

/*Define de pinos utilizados para aplicação*/
#define aquec   34  /*LED ligado ao GPIO34*/
#define refri   14  /*LED ligado ao GPIO14*/
#define bico    36  /*Bico agua ligado ao GPIO 36*/
#define desu    39  /*Desumidificador ligado ao GPIO39*/
#define led     02  /*Led placa ao GPIO2*/
#define bt      10  /*Botão ligado ao pino GPIO12*/
#define DHTTYPE 21  /*DHT 22  (AM2302), AM2321*/

uint8_t DHTPin = 4; /* DHT Sensor*/              
DHT dht(DHTPin, DHTTYPE); /*Inicializa o sensor DTH*/
 
float t; /*Temperatura*/
float h; /* Umidade*/

/*Acesso a rede wifi*/
char* ssid     = "FERNANDO&CARINA";
char* password = "dorigan107";

/*Acesso a porta*/
WiFiServer server(80);

/* Variáveis para Armazenar o handle da Task */
TaskHandle_t xTask2Handle;
TaskHandle_t xTask3Handle;
TaskHandle_t xTask4Handle;

/* Protótipo das Tasks*/
void vTask1(void *pvParameters ); 
void vTask2(void *pvParameters ); 
void vTask3(void *pvParameters );
void vTask4(void *pvParameters );

/*pinos para display*/
const int rs = 22, en = 23, d4 = 18, d5 = 19, d6 = 5, d7 = 15;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

/* Funções auxiliares */
void vInitHW(void);

void setup() {
  vInitHW();  /* Configuração do Hardware */

  /*Configurações das task's*/
  xTaskCreatePinnedToCore(vTask1, "vTask1", configMINIMAL_STACK_SIZE + 2048,  NULL,  1, NULL, 0); /*core 0*/
  xTaskCreate(vTask2, "vTask2", configMINIMAL_STACK_SIZE + 2048,  NULL,  3,  &xTask2Handle);      /*core 1*/
  xTaskCreate(vTask3, "vTask3", configMINIMAL_STACK_SIZE + 2048,  NULL,  2,  &xTask3Handle);      /*core 1*/
  xTaskCreate(vTask4, "vTask4", configMINIMAL_STACK_SIZE + 3048,  NULL,  1,  &xTask4Handle);      /*core 1*/
 
  /*Inicializa o display*/
  lcd.begin(16, 2);

}

void vInitHW(void) {
  Serial.begin(112500); /*Inicializa comunicação serial com baudrate de 9600 bps */
  dht.begin();  /*Inicializa o sensor*/
  pinMode(aquec, OUTPUT); /* configura pino como saída*/
  pinMode(refri, OUTPUT); /* configura pino como saída*/
  pinMode(bico,  OUTPUT); /* configura pino como saida*/
  pinMode(desu,  OUTPUT); /* configura pino como saida*/
  pinMode(led,   OUTPUT); /* configura pino como saida*/
  pinMode(bt, INPUT_PULLUP);  /* configura pino como entrada*/

  /*Apresenta na serial a conexão wifi*/
  Serial.println();
  Serial.println();
  Serial.print("Conectando ");
  Serial.println(ssid);

  /*liga wifi*/
  WiFi.begin(ssid, password);

  /*status wifi*/
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  /*mostra na serial wifi conectato*/
  Serial.println("");
  Serial.println("WiFi conectado.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  
  server.begin();

  /*Inicialização do display*/
  lcd.setCursor(0,0); /*Posiciona o cursor na coluna 0, linha 0*/
  lcd.print ("Fernando Dorigan"); /*Envia o texto entre aspas para o LCD*/
  delay(5000);  /*Delay para leitura do texto*/
}	

/*Loop faz a leitura e carrega nas variaveis*/
void loop() {
  h = dht.readHumidity(); /*variavel de leitura umidade relativa*/
  t = dht.readTemperature();  /*variavel de leitura temperatura*/
  vTaskDelay(pdMS_TO_TICKS(100));
}

/*Task 1 mostra no LCD textos e valores*/
void vTask1 (void *pvParameters ){
  (void) pvParameters;

  while(1) {
    lcd.setCursor(0, 0);  /*Posiciona o cursor na coluna 0, linha 0*/
    lcd.print("TEMP: ");  /*Envia o texto entre aspas para o LCD*/
    lcd.setCursor(6, 0);  /*Posiciona o cursor na coluna 6, linha 0*/
    lcd.print(t); /*Envia valor de temperatura para o LCD*/
    lcd.setCursor(0, 1);  /*Posiciona o cursor na coluna 0, linha 1*/
    lcd.print("UMID: ");  /*Envia o texto entre aspas para o LCD*/
    lcd.setCursor(6, 1);  /*Posiciona o cursor na coluna 6, linha 1*/
    lcd.print(h); /*Envia valor de umidade para o LCD*/
    vTaskDelay(pdMS_TO_TICKS(500));
  }
}

/*Task2 logica do software*/
void vTask2(void *pvParameters ){
  (void) pvParameters;
  

  while(1) {

    if (t > 26.5){  /*ponto de controle de temperatura*/
      digitalWrite(aquec,LOW);  /*desliga o aquecedor*/
      digitalWrite(refri,HIGH); /*liga a refrigeração*/
      lcd.setCursor(12, 0); /*Posiciona no LCD coluna 12, linha 0*/
      lcd.print("REFR"); /*Mostra no LCD que refrigeração está acionada*/
    }
    else {  /*Abaixo do ponto de controle*/
      digitalWrite(refri,LOW);  /*desliga refrigeração*/
      digitalWrite(aquec,HIGH); /*liga aquecimento*/
      lcd.setCursor(12, 0); /*Posiciona no LCD coluna 12, linha 0*/
      lcd.print("AQUE");/*Mostra no LCD que o aquecimento está acionado*/
    }

    if (h > 70){  /*ponto de controle de umidade*/
      digitalWrite(bico,LOW); /*desliga o bico de umidade*/
      digitalWrite(desu,HIGH);  /*liga o desumidificador*/
      lcd.setCursor(12, 1); /*Posiciona no LCD coluna 12, linha 1*/
      lcd.print("DESU");  /*Mostra no LCD que o desumidificador está acionado*/
    }

    else {  /*Abaixo do ponto de controle*/
      digitalWrite(desu,LOW); /*desliga o desumidificador*/
      digitalWrite(bico,HIGH);  /*liga o bico de umidade*/
      lcd.setCursor(12, 1); /*Posiciona no LCD coluna 12, linha 1*/
      lcd.print("BICO");  /*Mostra no LCD que o bico está acionado*/
    }
    vTaskDelay(pdMS_TO_TICKS(500));
  }
}

/*Task3 mostra na serial a temperatura e umidade*/
void vTask3(void *pvParameters ){
  (void) pvParameters;

  while(1){
    /*Mostra a temperatura no serial monitor e no display*/
    Serial.print("Temperatura: "); 
    Serial.print(t);
    Serial.print(" *C  ");

    /*Mostra a umidade no serial monitor e no display*/
    Serial.print("Umidade : "); 
    Serial.print(h);
    Serial.println(" %"); 
    vTaskDelay(pdMS_TO_TICKS(3000));
  }
}

  /*Task4 wifi acende e apaga led*/
  void vTask4(void *pvParameters ){
    (void) pvParameters;

    /*controla as conexões de wifi para acender led*/
    while(1){

      WiFiClient client = server.available();   /*Verifica as conexões*/

      if (client) {                    /*se conexão ok*/
        Serial.println("New Client."); /*print mensagem de novo cliente*/
        String currentLine = "";       /*string para conter os dados*/

        while (client.connected()) {            

          if (client.available()) {             
            char c = client.read();             
            Serial.write(c);                    

            if (c == '\n') {                    

              if (currentLine.length() == 0) {
                client.println("HTTP/1.1 200 OK");
                client.println("Content-type:text/html");
                client.println();

                /* HTTP na pagina web*/
                client.print("Click <a href=\"/H\">here</a> to turn the LED on pin 5 on.<br>");
                client.print("Click <a href=\"/L\">here</a> to turn the LED on pin 5 off.<br>");

                /* Print na tela http*/
                client.println();
                break;
              } 
                else {    
                currentLine = "";
              }
            } 
              else if (c != '\r') {  
              currentLine += c;      
          }

            /* Check se é "GET /H" or "GET /L":*/
            if (currentLine.endsWith("GET /H")) {    /*Quando H*/
              digitalWrite(led, HIGH);               /*Liga led*/
            }
            if (currentLine.endsWith("GET /L")) {    /*Quando l*/
              digitalWrite(led, LOW);                /*Desliga led*/
            }
        }
      }
        /*Desliga conexão*/
        client.stop();
        Serial.println("Client Disconnected.");
      }
    }
  } 


EXPLICAÇÃO CODIGO FONTE:

O código fonte está comentado, porém seguem algumas considerações:
Na primeira parte temos as bibliotecas para o funcionamento do código.
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "freertos/semphr.h"
#include <WiFi.h>
#include <Arduino.h>
#include <LiquidCrystal.h>
#include <DHT_U.h>
#include <DHT.h>

Nessa parte temos as definições de pinos e variáveis.
#define aquec   34  /*LED ligado ao GPIO34*/
#define refri   14  /*LED ligado ao GPIO14*/
#define bico    36  /*Bico agua ligado ao GPIO 36*/
#define desu    39  /*Desumidificador ligado ao GPIO39*/
#define led     02  /*Led placa ao GPIO2*/
#define bt      10  /*Botão ligado ao pino GPIO12*/
#define DHTTYPE 21  /*DHT 22  (AM2302), AM2321*/

uint8_t DHTPin = 4; /* DHT Sensor*/              
DHT dht(DHTPin, DHTTYPE); /*Inicializa o sensor DTH*/
 
float t; /*Temperatura*/
float h; /* Umidade*/


Nessa parte temos o acesso a rede wifi e configuração da porta do webserver.
/*Acesso a rede wifi*/
char* ssid     = "FERNANDO&CARINA";
char* password = "dorigan107";

/*Acesso a porta*/
WiFiServer server(80);


Nessa parte temos as configurações das task’s.

/* Variáveis para Armazenar o handle da Task */
TaskHandle_t xTask2Handle;
TaskHandle_t xTask3Handle;
TaskHandle_t xTask4Handle;

/* Protótipo das Tasks*/
void vTask1(void *pvParameters ); 
void vTask2(void *pvParameters ); 
void vTask3(void *pvParameters );
void vTask4(void *pvParameters );

Nessa parte seguem os pinos utilizados para ligação do display. Utilizou-se a ligação do display para 4 bits.
/*pinos para display*/
const int rs = 22, en = 23, d4 = 18, d5 = 19, d6 = 5, d7 = 15;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

No setup, tem-se as configurações das task’s e acionamento do display.
/* Funções auxiliares */
void vInitHW(void);

void setup() {
  vInitHW();  /* Configuração do Hardware */

  /*Configurações das task's*/
  xTaskCreatePinnedToCore(vTask1, "vTask1", configMINIMAL_STACK_SIZE + 2048,  NULL,  1, NULL, 0); /*core 0*/
  xTaskCreate(vTask2, "vTask2", configMINIMAL_STACK_SIZE + 2048,  NULL,  3,  &xTask2Handle);      /*core 1*/
  xTaskCreate(vTask3, "vTask3", configMINIMAL_STACK_SIZE + 2048,  NULL,  2,  &xTask3Handle);      /*core 1*/
  xTaskCreate(vTask4, "vTask4", configMINIMAL_STACK_SIZE + 3048,  NULL,  1,  &xTask4Handle);      /*core 1*/
 
  /*Inicializa o display*/
  lcd.begin(16, 2);
}

Nessa parte temos a inicialização da serial, configurações dos pinos como entrada e saídas. O sensor e o web server são inicializados.
Na serial, tem-se os print’s de acesso no wifi, para verificação se está conectado e mostra o IP para acesso.
void vInitHW(void) {
  Serial.begin(112500); /*Inicializa comunicação serial com baudrate de 9600 bps */
  dht.begin();  /*Inicializa o sensor*/
  pinMode(aquec, OUTPUT); /* configura pino como saída*/
  pinMode(refri, OUTPUT); /* configura pino como saída*/
  pinMode(bico,  OUTPUT); /* configura pino como saida*/
  pinMode(desu,  OUTPUT); /* configura pino como saida*/
  pinMode(led,   OUTPUT); /* configura pino como saida*/
  pinMode(bt, INPUT_PULLUP);  /* configura pino como entrada*/

  /*Apresenta na serial a conexão wifi*/
  Serial.println();
  Serial.println();
  Serial.print("Conectando ");
  Serial.println(ssid);

  /*liga wifi*/
  WiFi.begin(ssid, password);

  /*status wifi*/
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  /*mostra na serial wifi conectato*/
  Serial.println("");
  Serial.println("WiFi conectado.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  
  server.begin();

Nessa parte temos a inicialização do display e uma mensagem inicial no display que aguarda 5 segundos. 
/*Inicialização do display*/
  lcd.setCursor(0,0); /*Posiciona o cursor na coluna 0, linha 0*/
  lcd.print ("Fernando Dorigan"); /*Envia o texto entre aspas para o LCD*/
  delay(5000);  /*Delay para leitura do texto*/
}	

No loop temos a leitura do sensor de temperatura e umidade e transferir para as variáveis.
/*Loop faz a leitura e carrega nas variaveis*/
void loop() {
  h = dht.readHumidity(); /*variavel de leitura umidade relativa*/
  t = dht.readTemperature();  /*variavel de leitura temperatura*/
  vTaskDelay(pdMS_TO_TICKS(100));
}




NA Task1 temos as configurações do display para escrita, com as posições e leituras dos sensores.
/*Task 1 mostra no LCD textos e valores*/
void vTask1 (void *pvParameters ){
  (void) pvParameters;

  while(1) {
    lcd.setCursor(0, 0);  /*Posiciona o cursor na coluna 0, linha 0*/
    lcd.print("TEMP: ");  /*Envia o texto entre aspas para o LCD*/
    lcd.setCursor(6, 0);  /*Posiciona o cursor na coluna 6, linha 0*/
    lcd.print(t); /*Envia valor de temperatura para o LCD*/
    lcd.setCursor(0, 1);  /*Posiciona o cursor na coluna 0, linha 1*/
    lcd.print("UMID: ");  /*Envia o texto entre aspas para o LCD*/
    lcd.setCursor(6, 1);  /*Posiciona o cursor na coluna 6, linha 1*/
    lcd.print(h); /*Envia valor de umidade para o LCD*/
    vTaskDelay(pdMS_TO_TICKS(500));
  }
}

Na Task2 temos a lógica do programa, com os pontos de controle e de acionamento para temperatura baixa, alta, umidade baixa e alta, assim ligando as saídas para aquecedor, refrigeração, bico umidade e desumidificador.
/*Task2 logica do software*/
void vTask2(void *pvParameters ){
  (void) pvParameters;
  

  while(1) {
    //lcd.clear();

    if (t > 26.5){  /*ponto de controle de temperatura*/
      digitalWrite(aquec,LOW);  /*desliga o aquecedor*/
      digitalWrite(refri,HIGH); /*liga a refrigeração*/
      lcd.setCursor(12, 0); /*Posiciona no LCD coluna 12, linha 0*/
      lcd.print("REFR");  /*Mostra no LCD que refrigeração está acionada*/
    }
    else {  /*Abaixo do ponto de controle*/
      digitalWrite(refri,LOW);  /*desliga refrigeração*/
      digitalWrite(aquec,HIGH); /*liga aquecimento*/
      lcd.setCursor(12, 0); /*Posiciona no LCD coluna 12, linha 0*/
      lcd.print("AQUE");  /*Mostra no LCD que o aquecimento está acionado*/
    }

    if (h > 70){  /*ponto de controle de umidade*/
      digitalWrite(bico,LOW); /*desliga o bico de umidade*/
      digitalWrite(desu,HIGH);  /*liga o desumidificador*/
      lcd.setCursor(12, 1); /*Posiciona no LCD coluna 12, linha 1*/
      lcd.print("DESU");  /*Mostra no LCD que o desumidificador está acionado*/
    }

    else {  /*Abaixo do ponto de controle*/
      digitalWrite(desu,LOW); /*desliga o desumidificador*/
      digitalWrite(bico,HIGH);  /*liga o bico de umidade*/
      lcd.setCursor(12, 1); /*Posiciona no LCD coluna 12, linha 1*/
      lcd.print("BICO");  /*Mostra no LCD que o bico está acionado*/
    }
    vTaskDelay(pdMS_TO_TICKS(500));
  }
}


Na Task3 temos um acesso a serial, mostrando a temperatura e umidade, pensou-se como uma opção caso o display apresente algum defeito.
/*Task3 mostra na serial a temperatura e umidade*/
void vTask3(void *pvParameters ){
  (void) pvParameters;

  while(1){
    /*Mostra a temperatura no serial monitor e no display*/
    Serial.print("Temperatura: "); 
    Serial.print(t);
    Serial.print(" *C  ");

    /*Mostra a umidade no serial monitor e no display*/
    Serial.print("Umidade : "); 
    Serial.print(h);
    Serial.println(" %"); 
    vTaskDelay(pdMS_TO_TICKS(3000));
  }
}

Na Task4 temos o acesso wifi, com parâmetros para ativar uma saída a distância (Webserver). Pensou-se em uma saída para acesso remoto, como alarme, acionamento de segurança, liga / desliga entre outras aplicações diversas.
  /*Task4 wifi acende e apaga led*/
  void vTask4(void *pvParameters ){
    (void) pvParameters;

    /*controla as conexões de wifi para acender led*/
    while(1){

      WiFiClient client = server.available();   /*Verifica as conexões*/

      if (client) {                             /*se conexão ok*/
        Serial.println("New Client.");          /*print mensagem de novo cliente*/
        String currentLine = "";                /*string para conter os dados*/

        while (client.connected()) {            

          if (client.available()) {             
            char c = client.read();             
            Serial.write(c);                    

            if (c == '\n') {                    

              if (currentLine.length() == 0) {
                client.println("HTTP/1.1 200 OK");
                client.println("Content-type:text/html");
                client.println();

                /* HTTP na pagina web*/
                client.print("Click <a href=\"/H\">here</a> to turn the LED on pin 5 on.<br>");
                client.print("Click <a href=\"/L\">here</a> to turn the LED on pin 5 off.<br>");

                /* Print na tela http*/
                client.println();
                break;
              } 
                else {    
                currentLine = "";
              }
            } 
              else if (c != '\r') {  
              currentLine += c;      
          }

            /* Check se é "GET /H" or "GET /L":*/
            if (currentLine.endsWith("GET /H")) {    /*Quando H*/
              digitalWrite(led, HIGH);               /*Liga led*/
            }
            if (currentLine.endsWith("GET /L")) {    /*Quando l*/
              digitalWrite(led, LOW);                /*Desliga led*/
            }
        }
      }
        /*Desliga conexão*/
        client.stop();


CONCLUSÃO:

Com o desenvolvimento do projeto, pode-se concluir que o aprendizado foi concluído com a prática e aplicação dos itens apresentados em aula.
O uso do FreeRtos faz o código ser mais eficiente, organizado e sem travar em tarefas, além de ter possibilidade da realização de diferentes tarefas ao mesmo tempo.
Para realização de testes por partes do programa é mais eficiente, pois há facilidades em separar as tarefas e desativa-las caso necessário algum teste.
Com a divisão, a manutenção é fácil, pois cada tarefa é criada separadamente e aplicações são executadas isoladamente.


REFERÊNCIAS:

Para o desenvolvimento do projeto, utilizou-se o material apresentado em aula, bem como os sites abaixo:

•	https://www.embarcados.com.br/freertos-8-2-3/
•	https://www.usinainfo.com.br/blog/esp32-tutorial-com-primeiros-passos/
•	https://www.filipeflop.com/blog/aprenda-a-configurar-a-rede-wifi-do-esp32-pelo-smartphone/#:~:text=Funcionamento%20do%20circuito&text=Caso%20o%20bot%C3%A3o%20tenha%20sido,acender%C3%A1%20indicando%20sucesso%20na%20conex%C3%A3o.
•	https://www.arduinoportugal.pt/controlo-leds-atraves-pagina-html-nodemcu-marco-2017/
•	https://medium.com/@valdiney/ligando-e-desligando-led-por-via-do-wifi-usando-esp8266-node-mcu-4b0ef85ce6a3
•	https://www.filipeflop.com/blog/aprenda-a-configurar-a-rede-wifi-do-esp32-pelo-smartphone/
•	https://techtutorialsx.com/2018/05/05/esp32-arduino-temperature-humidity-and-co2-concentration-web-server/
•	https://lastminuteengineers.com/multiple-ds18b20-esp32-web-server-tutorial/
•	https://randomnerdtutorials.com/esp32-web-server-sent-events-sse/
•	https://create.arduino.cc/projecthub/harshmangukiya/create-esp8266-web-server-9c32ac
•	https://blogmasterwalkershop.com.br/embarcados/esp8266/webserver-com-o-shield-wifi-esp8266-para-arduino/
•	https://www.arduino.cc/reference/en/libraries/wifiwebserver/
•	https://www.embarcados.com.br/esp32-macaddress/
•	https://www.arduino.cc/en/Reference/LiquidCrystal
•	http://labdegaragem.com/forum/topics/transi-o-de-telas-no-display-16x2
•	https://forum.fazedores.com/t/transicao-de-telas-no-display-16x2/4232/2
•	https://savjee.be/2020/01/multitasking-esp32-arduino-freertos/
•	https://blog.eletrogate.com/iot-com-modulo-wifi-esp8266-basico/
•	https://www.curtocircuito.com.br/blog/analise-climatica-esp32-blynk
•	https://www.fernandok.com/2018/03/esp32-lora-dispositivos-de-seguranca.html
•	https://blogmasterwalkershop.com.br/embarcados/nodemcu/nodemcu-como-criar-um-web-server-e-conectar-a-uma-rede-wifi/
•	https://www.fernandok.com/2017/10/blog-post.html
•	https://arduinogetstarted.com/tutorials/arduino-temperature-sensor-lcd
•	https://www.filipeflop.com/blog/controlando-um-lcd-16x2-com-arduino/
LICENÇA DO PROJETO:

https://github.com/fdorigan/Projeto_final.git


