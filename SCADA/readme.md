# Modulo de SCADA y Controladores industriales

Para la seccion de SCADA y controladores industriales, se desgloso el proceso en 2 partes, una de lavado y pintura de los marcos, y otra de ensamble de las 
bicicletas, referenciadas como parte 1 y parte 2 respectivamente. La lógica de la parte 1 fue programada en un PLC S1200 de SIEMENS, y la lógica de la segunda parte fue montada en un COMPACTLOGIX 5330 ERM de la marca Allen Bradley. y para poder comunicarlos, se utilizaron 2 modulos SIMATIC IoT 2040, sobre los cuales corría node-red, y por medio de protocolos MQTT, que pudieran ver los tags entre sí. 
Para la comunicación MQTT, se utilizó HiveMQ, montado en nube sobre un contenedor de AWS, este era nuestro servidor o Broker, y los modulos IoT se suscribian a los topicos y mandaban la información a los PLC's

## Programación PLC SIEMENS S1200
para la programacion del controlador, lo primero que se hace es un analisis por etapas (Grafcet) de como debería funcionar este subproceso. 

