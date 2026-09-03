
 # Dispositivos lowtech e interfaces interactivas 

## Dispositivo-lowtech-para-ninos
## Nombre del dispositivo: *Piping call*


## Equipo

| Integrantes |
|---|
| Mártin Palacios Araya |
| Imay Pizarro Luna |


**Problematica:** Dispositivo low-tech de orientación y comunicación bidireccional para el resguardo de menores.


## Descripción

La propuesta consiste en un sistema interactivo compuesto por dos dispositivos vinculados entre sí, diseñado bajo un enfoque de bajo costo. El sistema busca facilitar el reencuentro mediante información de ubicación, distancia y orientación, complementada con señales sonoras y un sistema de comunicación basado en mensajes predeterminados. Además, contempla mecanismos de alerta que permitan solicitar apoyo ante situaciones que requieran asistencia externa.


## Desafío o problematica  inicial

Las soluciones comerciales de localización y monitoreo infantil pueden representar una barrera económica para algunas familias, limitando el acceso a herramientas que permitan apoyar la supervisión de niños en situaciones de extravío.

Cuando un niño de entre 4 y 11 años se separa de su cuidador en un espacio concurrido, puede experimentar desorientación y dificultades para comunicar su situación o solicitar ayuda de manera autónoma. Al mismo tiempo, el cuidador debe iniciar una búsqueda sin contar necesariamente con información inmediata que facilite la localización o la comunicación con el menor.

Esta situación evidencia la necesidad de explorar una alternativa accesible que permita mantener un vínculo funcional entre ambos usuarios y apoyar las acciones necesarias durante un eventual extravío.


## Objetivo 

Diseñar un sistema interactivo de bajo costo compuesto por dos dispositivos vinculados, orientado a apoyar el reencuentro entre un cuidador y un niño en situaciones de extravío.

El sistema deberá permitir conocer la ubicación y distancia relativa entre ambos dispositivos, facilitar la orientación hacia el usuario vinculado y proporcionar información que apoye la estimación del desplazamiento necesario para el reencuentro. Asimismo, incorporará mecanismos de alerta sonora y comunicación mediante mensajes predeterminados, además de una función destinada a solicitar ayuda ante situaciones de emergencia.


## Usuarios y contexto

El sistema está dirigido a dos usuarios principales: cuidadores, como padres, madres o tutores, y niños y niñas de entre 4 y 11 años.

Su uso se plantea principalmente en espacios urbanos de alta concurrencia, como parques, plazas y centros comerciales, donde existe la posibilidad de que un niño se aleje o pierda contacto visual con su cuidador.

Debido a las características del usuario infantil, la interacción debe considerar una comunicación simple, comprensible y de rápida ejecución. Por su parte, el cuidador requiere información que le permita orientar sus acciones y facilitar el proceso de búsqueda.


## Variables

Las principales variables consideradas por el sistema corresponden a la relación espacial entre ambos dispositivos vinculados. Estas incluyen:

-Distancia entre los dispositivos, expresada en metros.

-Dirección o orientación aproximada hacia la ubicación del otro dispositivo.

-Coordenadas de ubicación para su representación cartográfica.

-Tiempo estimado de desplazamiento o reencuentro, expresado en minutos.

-Estado de comunicación entre ambos dispositivos.

-Activación de alertas sonoras o mensajes predeterminados.


## Hipótesis

Si se diseña un sistema compuesto por dos dispositivos vinculados que combine funciones de localización, orientación, alerta y comunicación simplificada, entonces será posible entregar a cuidadores y niños información inmediata que facilite la identificación de su posición relativa y la coordinación de acciones ante una situación de extravío.


## Referentes
## *Referente 1: Reloj GPS con Arduino*
![Reloj GPS con Arduino](imagenes-referentes/referente1.png)

Sacamos beneficios de esta idea por el formato del dispositivo, como sus componentes y codificación.

Usando un SIM28M GPS receiver module como receptor y un Módulo GPS NEO-6M con una antena.

(https://www.electronicsforu.com/electronics-projects/gps-clock-arduino)


## *Referente 2: Módulo GPS con Arduino*
![Módulo GPS con Arduino](imagenes-referentes/referente2.png)

Ejemplo de un código para arduino que muestra la localización del dispositivo mediante la respuesta del registro de salida.

Usando el Módulo GPS NEO-6M


(https://naylampmechatronics.com/blog/18_tutorial-modulo-gps-con-arduino.html)


## *Referente 3: Gps Rastreador*
![Gps Rastreador](imagenes-referentes/referente3.png)

Ejemplo de una forma práctica de ocultar el gps y volverlo algo cotidiano, se podría presionar de manera convencional sin que tuviera algún indicio de alertar a otros, un ejemplo de hipótesis para nuestro producto final.



(https://articulo.mercadolibre.cl/MLC-613843320-gps-rastreador-anti-perdida-mascotas-bolsos-ninos-_JM?matt_tool=16931662&utm_source=google_shopping&utm_medium=organic
)


## *Referente 4: AirTag*
![AirTag](imagenes-referentes/referente4.png)

A partir del referente de AirTag, adaptamos la idea de proyectar en el teléfono móvil los datos clave del dispositivo low-tech vinculado, permitiendo así integrar múltiples funcionalidades de monitoreo.




(https://www.apple.com/cl/airtag/)








## Índice de la bitácora

- [S01 - Entrega 01](bitacora/S01.md)
- [S02 - Entrega 02](bitacora/S02.md)
- [S03 - Entrega 03](bitacora/S03.md)
