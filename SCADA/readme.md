# Modulo de SCADA y Controladores industriales


Para la sección de SCADA y controladores industriales, el proceso se desglosó en dos partes: una correspondiente al **lavado y pintura de los marcos**, y otra al **ensamble de las bicicletas**, referenciadas como **parte 1** y **parte 2**, respectivamente.


La lógica de la parte 1 fue programada en un **PLC S7-1200 de Siemens**, mientras que la lógica de la parte 2 se implementó en un **CompactLogix 5330 ERM de la marca Allen-Bradley**. Para permitir la comunicación entre ambos controladores, se utilizaron **dos módulos SIMATIC IoT2040**, en los cuales se ejecutaba **Node-RED**. La comunicación se estableció mediante el protocolo **MQTT**, permitiendo el intercambio de tags entre los dos sistemas.

Para el servidor MQTT (broker), se utilizó **HiveMQ**, desplegado en la nube a través de un contenedor en **AWS**. Los módulos IoT se suscribían a los tópicos correspondientes y enviaban la información a sus respectivos PLCs.

## Arquitectura de Comunicaciones

En cuanto a la arquitectura de comunicaciones, se plantea conectar los diferentes **sensores ambientales** a los módulos IoT mediante el protocolo **Modbus**, mientras que los **sensores de presencia o conteo** estarán asociados directamente a su respectivo PLC. Toda la información recolectada será enviada a un **servidor MQTT**, centralizando los datos para su monitoreo y análisis.
<div style="text-align: center;">
<img width="800" height="652" alt="Arquitectura de comunicaciones SCADA" src="https://github.com/user-attachments/assets/8ba918ba-6af4-4ea7-a33f-9fb53b7dd269" />
</div>



## Programación PLC SIEMENS S1200

Para la programación del controlador, lo primero que se realiza es un **análisis por etapas (GRAFCET)** de cómo debería funcionar este subproceso.

<img width="2000" height="2760" alt="Parte_1_Grafcet" src="https://github.com/user-attachments/assets/dd14d9a7-318d-40f6-83d6-bccd9d50f3e2" />

Como se puede observar, lo primero que se debe hacer al iniciar el proceso es **encender el horno** y esperar a que alcance un **humbral de precalentamiento adecuado**, de manera que, cuando las bicicletas lleguen, ya esté a la temperatura ideal para el curado de la pintura. Una vez se cumple esta condición (**Transición 2**), se procede a **encender la banda 1**, iniciando así el flujo de bicicletas.

Posteriormente, se presenta una **bifurcación de tipo AND**, donde se ejecutan en paralelo cuatro subprocesos: uno por cada subestación de lavado y el de pintura. En estos subprocesos se busca **optimizar el uso de recursos**, de modo que si, por alguna razón, hay un slot o gancho vacío entre bicicletas, los **actuadores correspondientes se apaguen automáticamente** y permanezcan inactivos hasta que se detecte nuevamente una bicicleta.

Esta lógica se **alinea con los principios de Lean Manufacturing**, ya que busca minimizar los **desperdicios operativos**, evitando el uso innecesario de energía o componentes en ausencia de producto. Este enfoque contribuye a una operación más eficiente y sostenible, promoviendo el uso inteligente de los recursos disponibles.

Posteriormente, esta lógica se implementa en lenguaje **LADDER**, considerando que las etapas paralelas tienen asociadas sensores y actuadores individuales. Por ello, se opta por estructurar el programa en **subprocesos separados**.

El programa se compone de tres bloques:


- `MAIN`
- `Automatic`
- `Block_1` (Proceso de variables analógicas)

En el bloque `MAIN`, se llama al bloque que procesa las **entradas analógicas** (`IW64` e `IW66`) y a la función `Automatic`, cuya ejecución se controla mediante un **tag** modificado desde el dashboard de **Node-RED**.
<br>

<img width="450" height="536" alt="Bloque MAIN_TIA_PORTAL" src="https://github.com/user-attachments/assets/7aeb0258-40a7-43d6-8d00-340f5c800148" />



En `Block_1`, las variables analógicas se **normalizan** (conversión de entero a real), teniendo en cuenta los valores máximos indicados en el **datasheet** del banco. Luego, se escalan en un rango aproximado de 0 a 100, y sus valores se almacenan en una **base de datos**, para ser accedidos desde Node-RED.
<br>
<img width="450" height="493" alt="Bloque Block_1_TIA" src="https://github.com/user-attachments/assets/1672d54c-e46e-4cba-a105-3722b10d06b7" />



Finalmente, en el bloque `Automatic`, que contiene la rutina automática del PLC, se programan las condiciones bajo las cuales los **actuadores se energizan**, según las transiciones definidas en el diagrama GRAFCET.

<br>
<img width="300" height="582" alt="Bloque Auto_TIA_1" src="https://github.com/user-attachments/assets/eca3f979-30bc-4d2d-ba8d-a87a678e1851" />
<img width="300" height="572" alt="Bloque Auto_TIA_2" src="https://github.com/user-attachments/assets/d5af9681-2226-4975-9cf7-19dcc4778952" />



Esta misma lógica se replica para el resto de **actuadores de aire y pintura**, manteniendo la estructura modular del programa.



## Programación PLC CompactLogix 5330


La segunda parte del proceso cuenta con una **línea principal de ensamble**, conectada a diversas **estaciones de subensamble**. La lógica definida establece que la banda transportadora solo puede avanzar si **todas las estaciones de subensamble se encuentran en estado OK**, es decir, que cada una haya completa
do su proceso satisfactoriamente.

Para esto, se dispone de una **botonera en cada estación**, mediante la cual los operarios indican que la estación está lista. La **detención de la banda** ocurre cuando la bicicleta llega a una **posición de referencia**, es decir, cuando ha alcanzado la siguiente estación de trabajo.

La lógica fue representada mediante un **diagrama GRAFCET**, y a partir de este se desarrolló el control en lenguaje **LADDER**, facilitando su implementación en el PLC CompactLogix.


Se desarrolló también un **esquemático funcional** que describe el comportamiento esperado de los actuadores:

<img width="500" height="1377" alt="Parte_2_Grafcet" src="https://github.com/user-attachments/assets/6f494111-a925-4c04-80e2-6346be5d1fd2" />

Cabe resaltar que en cada subestación se contempla la presencia de un **monitor industrial**, el cual indica a los operarios qué producto deben ensamblar en ese momento y cuántas unidades restan por producir.

Para más detalles, se recomienda revisar el archivo `.ACD`, en el cual se evidencia de forma más directa la **traducción del GRAFCET a LADDER**, con las siguientes convenciones:
- Variables `T` para transiciones
- Variables `E` para estaciones
- Comunicación con el variador de frecuencia mediante **tags provenientes de Node-RED**

---

## Node-RED y SCADA

Para la comunicación con Node-RED y el sistema SCADA, se desarrollaron **tres flujos principales**:

1. **Supervisión de sensores y máquinas** en la planta de producción.
2. **Supervisión de variables ambientales**, como temperatura, humedad, y partículas en el aire.
3. **Control manual de equipos y actuadores**, desde una interfaz accesible para el operario.
Además, se incluye una **sección de alarmas**, que se activa cuando alguna variable excede los límites preestablecidos.
<br>
<img width="800" height="1013" alt="SCADA_1" src="https://github.com/user-attachments/assets/0d34b00b-bb75-4ecf-8a57-15dcb317a987" />
<br>
<img width="800" height="1078" alt="SCADA_2" src="https://github.com/user-attachments/assets/ccf4cb8c-7cbd-4c42-b7ab-e31dbe8d3956" />
<br>
<img width="800" height="900" alt="SCADA_3" src="https://github.com/user-attachments/assets/9eae82c7-3342-48eb-8492-9db55b95511a" />

Se muestra igualmente los flujos en node red:



Flujo de supervisión
<br>
<img width="800" height="717" alt="Node-Red_FLUJO_SUPERVISION" src="https://github.com/user-attachments/assets/8fc7d3d9-c21b-4680-950c-6adb068efadd" />


Flujo de MODBUS o sensor de ambiente
<br>
<img width="800" height="786" alt="Node-Red_ FLUJO SENSOR AMBIENTE" src="https://github.com/user-attachments/assets/20ee7361-dc78-47d5-9209-fea034904f38" />

Flujo indicadores de produccion
<br>
<img width="597" height="335" alt="Node-Red_FLUJO ORDENES DEL DIA" src="https://github.com/user-attachments/assets/401f61d0-a36f-4502-a98b-67bae0e51d94" />
