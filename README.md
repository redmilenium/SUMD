# SUMD
GENERACION DE PROTOCOLO SUMD Y COMPROBACION EN FC (CONTROLADORA DE VUELO DE DRONE)

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
En la FC he cargado y configurado el soft Betafligh y se puede ver el correcto funcionamiento del simulador de protocolo SUMD

![image](https://github.com/redmilenium/SUMD/assets/48222471/1c6954cf-9bd0-4bd7-ba0e-fd88ef8e259f)

El soft del ESP32 esta simplificado al máximo para hacer la comprobación de su correcto funcionamiento.











