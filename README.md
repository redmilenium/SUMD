# SUMD
GENERACION DE PROTOCOLO SUMD Y COMPROBACION EN FC (CONTROLADORA DE VUELO DE DRONE)
Las FC (controladora de vuelo) se utilizan para gestionar el funcionamiento de drones, aviones, barcos, etc. Disponen de acelerometro, giroscopo, barometro, etc, etc.
En el caso de los drones y aviones, permiten la estabilización del modelo por medio de la IMU (acelerometro y giroscopo).
Los datos para que nuestro equipo haga lo que queremos, llegan a la FC desde el receptor. 

Habitualmente la comunicacion entre el receptor u la FC es mediante UART a 115.200 bps 8N1 y existen multiples protocolos. Uno de ellos es el SUMD.
Dentro de la comunicación deberemos enviar datos como el valor del acelerador, si queremos avanzar o retroceder, ir a la izquierda o a la derecha, girar sobre nosotros mismos, y multiples posibilidades mas.
![image](https://github.com/redmilenium/SUMD/assets/48222471/4e02af22-0013-4e3b-9e83-98b5ba68169a)

Ejemplo de una FC
![image](https://github.com/redmilenium/SUMD/assets/48222471/d6d00baa-defd-4236-8f48-c92592b5d460)


