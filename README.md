# SUMD 
Generación de protocolo SUMD y comprobación en FC (Controladora de vuelo de drones)

Las FC (controladora de vuelo) se utilizan para gestionar el funcionamiento de drones, aviones, barcos, etc. Disponen de acelerometro, giroscopo, barometro, etc, etc.
En el caso de los drones y aviones, permiten la estabilización del modelo por medio de la IMU (acelerometro y giroscopo).

Los datos para que nuestro equipo haga lo que queremos, llegan a la FC desde el receptor. 

Habitualmente la comunicacion entre el receptor u la FC es mediante UART a 115.200 bps 8N1 y existen multiples protocolos. Uno de ellos es el SUMD.

Asi se ve una trama del protocolo SUMD
![image](https://github.com/redmilenium/SUMD/assets/48222471/6684be81-f879-4d13-aa7c-cbf37fb2c543)

El primer byte indica el fabricante

El segundo solo tiene 2 valores validos 0x01 (Rx ok) y 0x81 (indica a la FC que se active Fail Safe), cualquier otro valor invalida la trama.

El tercero indica el numero de canales que estamos enviando. El máximo son 32. 

Los siguientes y por parejas indican el valor de cada canal.
![image](https://github.com/redmilenium/SUMD/assets/48222471/d3f150a5-a479-450f-b6e7-30b12788469b)

El valor a enviar para cada canal se calcula multiplicando por 8 el valor en microsegundos que queremos enviar.

Y los 2 últimos son el calculo del CRC de la trama. Si el CRC es incorrecto, la trama será desechada.

Dentro de la comunicación deberemos enviar datos como el valor del acelerador, si queremos avanzar o retroceder, ir a la izquierda o a la derecha, girar sobre nosotros mismos, y multiples posibilidades mas.

![image](https://github.com/redmilenium/SUMD/assets/48222471/4e02af22-0013-4e3b-9e83-98b5ba68169a)

Ejemplo de una FC
![image](https://github.com/redmilenium/SUMD/assets/48222471/d6d00baa-defd-4236-8f48-c92592b5d460)

Habitualmente se conecta un receptor a la entrada de la FC para el control en remoto del aeromodelo (drone, avion, etc.)

Vista del montaje de pruebas:

![image](https://github.com/redmilenium/SUMD/assets/48222471/565410bc-8053-4724-aeb1-603c0275064e)

He simulado, mediante un ESP32 el protocolo con el siguiente soft:

```
/*
    gestion de la FC del drone
    - envio de datos con protocolo SUMD 16 CANALES
*/

#include "CRC16.h"
#include "CRC.h"
unsigned long seven_milis;

char str[40] = {0xa8,0x01,0x10,   //16x2 + 3 =35     27
0x22,0x60,  //1100
0x2e,0xe0,  //1500
0x3b,0x60,  //1900
0x41,0xa0,  //2100
0x1f,0x40,  //1000
0x2e,0xe0,  //1500
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2E,0xE0};

CRC16 crc;


void calcula_crc()
{
  crc.setPolynome(0x1021);
  crc.add((uint8_t*)str, 35);
  str[35] = crc.calc() >> 8;
  str[36] = crc.calc() & 0xFF;  
  crc.reset();
  for (int alfa = 0; alfa <= 36; alfa++) // envio el array completo: datos + crc
    {
     Serial1.print(str[alfa]);
    }
}

 
void setup() 
{
  // Initialize Serial Monitor
  Serial.begin(115200);

  Serial1.begin(115200, SERIAL_8N1,26,25,true);   // SUMD FC
  Serial1.write("\n");
}
 
void loop() 
{
   // rutina para el envio de protocolo SUMD a la FC del drone 
   if(millis()-seven_milis>10)//10
   {
    seven_milis=millis();
    calcula_crc(); // obtiene el CRC,lo añade al final del array y lo envia
    }
   
 }

```
En la FC he cargado y configurado el soft Betafligh para comprobar el correcto funcionamiento del simulador de protocolo SUMD

![image](https://github.com/redmilenium/SUMD/assets/48222471/1c6954cf-9bd0-4bd7-ba0e-fd88ef8e259f)

El soft del ESP32 esta simplificado al máximo para hacer la comprobación de su correcto funcionamiento.

El envio de tramas SUMD deber ser constante y en intervalos no superiores a 10 milisegundos, de otra forma, la FC entiende que se ha cortado la comunicación con el receptor y activa el modo SAFE, procediendo a aterrizar o volviendo al punto de despeque en el caso de llevar GPS y estar asi configurado.

Ejemplo de tren de tramas no correcto:

![image](https://github.com/redmilenium/SUMD/assets/48222471/734500ad-caa1-4fa1-8deb-2232953e7c17)

Y su efecto en la FC del drone:

![image](https://github.com/redmilenium/SUMD/assets/48222471/c579d7f5-8283-40ba-b4bd-16e0418e1615)




Por tanto, el tren de tramas debe presentar un aspecto mas o menos asi (la separacion entre ellas no llega a los 10 ms):

![image](https://github.com/redmilenium/SUMD/assets/48222471/6de3dc03-e972-4821-8b9a-5cae92928627)

Puede ocurrir que al cargar con más codigo al ESP32, "descuide" el envio de tramas SUMD al estar ejecutando otras rutinas y el envio de tramas SUMD se vea afectado, aumentando el tiempo entre ellas a mas de 10 ms.

Dado que ESP32 dispone de 2 nucleos, esto se soluciona cargando el trabajo del envio de las tramas SUMD al  nucleo ocioso.

Mientras que el nucleo titular se encarga del resto:
```
/*
    gestion de la FC del drone
    - envio de datos con protocolo SUMD 16 CANALES
*/

#include "CRC16.h"
#include "CRC.h"
unsigned long seven_milis;

char str[40] = {0xa8,0x01,0x10,   //16x2 + 3 =35     27
0x22,0x60,  //1100
0x2e,0xe0,  //1500
0x3b,0x60,  //1900
0x41,0xa0,  //2100
0x1f,0x40,  //1000
0x2e,0xe0,  //1500
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2e,0xe0,
0x2E,0xE0};

CRC16 crc;


void calcula_crc()
{
  crc.setPolynome(0x1021);
  crc.add((uint8_t*)str, 35);
  str[35] = crc.calc() >> 8;
  str[36] = crc.calc() & 0xFF;  
  crc.reset();
  for (int alfa = 0; alfa <= 36; alfa++) // envio el array completo: datos + crc
    {
     Serial1.print(str[alfa]);
    }
}

//**************************
TaskHandle_t Task1;
//**************************
//************************************************************************************************
void Task1code( void * pvParameter) 
{
 while(1)
 {
    // rutina para el envio de protocolo SUMD a la FC del drone 
   if(millis()-seven_milis>10)//10
   {
     calcula_crc(); // obtiene el CRC,lo añade al final del array y lo envia
     seven_milis=millis();
   }
 }
}

 
void setup() 
{
//*************************************************************************************************
xTaskCreatePinnedToCore(
     Task1code, /* Function to implement the task */
     "protocolo SUMD", /* Name of the task */
     10000,  /* Stack size in words */
      NULL,  /* Task input parameter */
      0,  /* Priority of the task */
      &Task1,  /* Task handle. */
      0); /* Core where the task should run */
//*************************************************************************************************


  // Initialize Serial Monitor
  Serial.begin(115200);

  Serial1.begin(115200, SERIAL_8N1,26,25,true);   // SUMD FC invertido
  Serial1.write("\n");
}
 
void loop() 
{

 // ya tienes recursos de procesador para que hagas otras cosas
   
 }

```

SEÑAL INVERTIDA

A veces puede ocurrir que una controladora necesita que la señal SUMD le llegue invertida.

Esto sería una señal en modo normal:

![image](https://github.com/redmilenium/SUMD/assets/48222471/9053d44e-5a34-4ff4-aea2-87db71d75b2c)

Y esta sería una señal invertida:

![image](https://github.com/redmilenium/SUMD/assets/48222471/4c776cfe-1adb-4038-8443-73a18c5cbda0)

La diferencia es el punto de partida de la señal. En el primer caso la señal en reposo esta a 1 lógico y en el segundo caso esta a 0 lógico.

Esto se configura asi en el ESP32:

Serial1.begin(115200, SERIAL_8N1,26,25,true);   // SUMD FC señal invertida

Serial1.begin(115200, SERIAL_8N1,26,25);   // SUMD FC señal normal

De esta manera evitamos añadir transistores al circuito para invertir la señal.













