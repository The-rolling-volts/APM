# Modulo de SCADA y Controladores industriales

Para la seccion de SCADA y controladores industriales, se desgloso el proceso en 2 partes, una de lavado y pintura de los marcos, y otra de ensamble de las 
bicicletas, referenciadas como parte 1 y parte 2 respectivamente. La lógica de la parte 1 fue programada en un PLC S1200 de SIEMENS, y la lógica de la segunda parte fue montada en un COMPACTLOGIX 5330 ERM de la marca Allen Bradley. y para poder comunicarlos, se utilizaron 2 modulos SIMATIC IoT 2040, sobre los cuales corría node-red, y por medio de protocolos MQTT, que pudieran ver los tags entre sí. 
Para la comunicación MQTT, se utilizó HiveMQ, montado en nube sobre un contenedor de AWS, este era nuestro servidor o Broker, y los modulos IoT se suscribian a los topicos y mandaban la información a los PLC's

## Programación PLC SIEMENS S1200
para la programacion del controlador, lo primero que se hace es un analisis por etapas (Grafcet) de como debería funcionar este subproceso. 
![Logo](images/Parte_1_Grafcet.png)
Como se puede ver, lo primero que se debe hacer cuando se inicia el proceso es encender el horno, y esperar que llegue a un humbral de precalentamiento adecuado, de manera que cuando las ciclas lleguen, ya este en la temperatura ideal para el curado de la pintura, una vez se cumpla esta condicion (Transicion 2) se procede a encender la banda 1, y con ella el flujo de las bicilcetas, posteriormente, vemos una bifurcacion de tipo AND, donde tenemos 4 subprocesos ejecutandose en paralelo, uno por cada subestacion de lavado y el de pintura. en estos subprocesos se busca optimizar recursos, y es que si por alguna razón, hay un slot o gancho vacio entre ciclas, se deben apagar los actuadores correspondientes, y no encenderse hasta que se vuelva a detectar una cicla, esta logica se alinea con la filosofia de lean manufacturing, donde se busca reducir al máximo los desperdicios


