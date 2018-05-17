---
title:  "Rancher OS - Capitulo 2"
date:   2018-05-13 18:23:23
categories: [Docker, RancherOS, Swarm]
tags: [Docker, DevOps, Contenedores, Images, RancherOS, Swarm]
---
Continuando con el mundo de los contenedores y esta fabulosa herramienta llamada Rancher OS. Hoy les voy a mostrar de forma práctica como funciona la herramienta. Vamos a crear una insfraestructura montada en la nube de Microsoft Azure, la cual tendra los siguienes componentes:
- Balanceador externo de carga.
- Grupo de disponibilidad para las maquinas virtuales.
- Cuatro maquinas con Ubuntu 16.04 LTS.
- Docker, Swarm, y rancher OS.
- Como contenedor de pruebas desplegaremos un Apache con Nginx.
Asimismo, les estaré mostrando como se comporta el cluster ante la caida de uno de los nodos. Verán que el contenedor con apache que vamos a desplegar nunca perderá conexión y podra ser accedido sin afectar a los usuarios.

![RancherOS_Logo]({{ site.baseurl }}/images/rancheros_logo1.png)

## Introducción. ##

Continuando con el mundo de los contenedores y esta fabulosa herramienta llamada Rancher OS. Hoy les voy a mostrar de forma práctica como funciona la herramienta. Vamos a crear una insfraestructura montada en la nube de Microsoft Azure, la cual tendra los siguienes componentes:
- Balanceador externo de carga.
- Agrupo de disponibilidad para las maquinas virtuales.
- Cuatro maquinas con Ubuntu 16.04 LTS.
- Docker, Swarm, y rancher OS.
- Como contenedor de pruebas desplegaremos un Apache con Nginx.
Asimismo, les estaré mostrando como se comporta el cluster ante la caida de uno de los nodos. Verán que el contenedor con apache que vamos a desplegar nunca perderá conexión y podra ser accedido sin afectar a los usuarios.

### Infraestructura ###

**1. Diagrama**

![RancherOS_Diagrama]({{ site.baseurl }}/images/Swarm_Cluster.PNG)

Aquí veremos graficamente lo que les mencionaba anteriormente, el tipo de infraestructura que voy a desplegar.

### Manos a la obra ###

En el siguiente video veremos en acción estas fabulosas herramientas.

{%include youtubePlayer.html id="gEpNyVgQkU8"%}

En una próxima entrega les estaré hablando de la nueva version de Rancher OS, la cual nos permite crear clusters en la nube y esta pensada para administrar cluster con Kubernetes.

### Links de interés: ###

[Docker Official Page][Docker]

[Rancher OS Official Page][RancherOS]

[Rancher OS - Capítulo 3][RancherOSCap3]


[Docker]: https://www.docker.com/

[RancherOS]: https://rancher.com/rancher-os/

[RancherOSCap3]:http://blog.mauriciovillagran.uy/2018/Docker-RancherOS-Capitulo3/
