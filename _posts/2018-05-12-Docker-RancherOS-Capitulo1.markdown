---
title:  "Rancher OS - Capitulo 1"
date:   2018-05-12 16:31:23
categories: [Docker, RancherOS]
tags: [Docker, DevOps, Contenedores, Images, RancherOS]
---
Hoy les voy a hablar sobre los "orquestadores" de contenedores que estan en clusters, ya sea en Docker Swarm, Kubernetes, etc.
Toda esta investigación surge el día que vi en un cliente, una infraestructura montada con Docker Swarm y administrada con un orquestador llamado Portainer. Hablando con el cliente me comentaba que todavia no sabia como hacer para tener un solo orquestador de Portainer principal y orquestar todos los nodos de Docker Swarm.
En este capitulo les voy a hablar, entre otras cosas, de Rancher OS. El orquestador mas pequeño que se haya conocido. No solo es capaz de administrar la infraestructura de contenedores, sino capaz de crear la misma... y tan solo pesa 20 MB.

![RancherOS_Logo]({{ site.baseurl }}/images/rancheros_logo1.png)

## Que es Rancher OS? ##

Rancher es una plataforma (PaaS, Platform as a Service) open source que corre en Docker. Permite desplegar aplicaciones en container en cualquier infraestructura en producción, apoyándose en herramientas open source.
En la parte de orquestación de containers, implementa varias soluciones pudiendo elegir entre Docker Swarm, Kubernetes o Mesos.

### Los puntos fuertes de Rancher OS ###

**1. Infraestructura**

La plataforma cuenta con un apartado Hosts para gestionar de forma visual las máquinas o instancias de diferentes clouds ya sea AWS (Amazon), Azure (Microsoft), Digitalocean… Se puede decir que es una consola visual para la administración de instancias.

La abstracción que da de todas las peculiaridades de cada cloud supone una gran ventaja ya que desde un punto de vista del desarrollador apenas necesita conocimientos de sistemas para poder levantar instancias en Amazon o Azure, por ejemplo.

![RancherOS_Consola_Host]({{ site.baseurl }}/images/rancheros_01.PNG)

**2. Entornos**

Hoy en día cualquier compañía, suele hacer uso de varios entornos para llevar sus features a producción. Normalmente hablamos de tres entornos como mínimo: DEV, PRE y PRO (en algunos casos añaden un cuarto entorno que es integrado INT ).

Para poder tener todos estos entornos funcionando se ha de invertir varias horas y personas de sistemas en su creación y configuración. Aquí es donde entran en juego las ventajas más prometedoras Rancher, que permite:

***-Crear tantos entornos como se necesite solo con unos cuantos clicks.***

***-Elegir el orquestador de containers de entre Cattle, Mesos, Kubernetes y Docker Swarm.***

***-Administrar usuarios y roles para los distintos entornos.***

![RancherOS_Consola_Env]({{ site.baseurl }}/images/rancheros_02.PNG)

**3. Aplicaciones**

Rancher organiza las aplicaciones o servicios en Stacks; siempre que se quiere desplegar un container o aplicación previamente hay que crear un Stack.

Otro de los aspectos más importantes de Rancher es su catálogo de aplicaciones. Este catálogo es público, la comunidad open source puede contribuir con sus aplicaciones para que todos los usuarios de Rancher Community. También ofrece la posibilidad de tener catálogo de aplicaciones privado.

![RancherOS_Consola_Catalogo]({{ site.baseurl }}/images/rancheros_03.PNG)

**4. El espacio**

RancherOS una distribución de Linux minimalista de 27MB para correr nuestras imágenes de Docker.

### Instalacion de Rancher OS ###

La instalacion es muy sencilla, Rancher OS se puede instalar en un host sin HA o con HA. En este caso voy a instalar un host con Rancher OS sin HA.
Previamente tenemos que tener instalado Docker en nuestro host.

```bash
sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server
```

Luego de ejecutar el comando, tendremos un contenedor corriendo con Rancher OS. Basicamente nos queda ingresar al servidor desde un navegador web http://SERVER_IP:8080.

Como mencionaba anteriormente, Rancher OS se puede instalar con HA, o tambien conectandolo a una base de datos externa como ser MYSQL.

### Agregar un host ###

Rancher OS maneja agentes para poder establecer la comunicacion entre él y sus host, es por eso que debemos instalar dicho agente. Es un proceso muy sencillo. 
Basta con agregar los host´s desde la consola de Rancher OS, siguiendo estos pasos:

**1. Dentro del menu, elegimos Infrastructure > Hosts**

**2. Segimos los pasos que nos marca el asistente para instalar el agente en el host**

**3. Corremos el comando en el host que queremos que Rancher OS administre**


![RancherOS_Consola_Host]({{ site.baseurl }}/images/rancheros_01.PNG)

### Environments ###

Rancher OS nos permite manejar varios entornos. Asimismo nos permite crear diferentes clusters, por ejemplo; Docker Swarm.
Para crear un cluster de Docker Swarm, tenemos que seguir estos pasos:

**1. Dentro del menu, vamos a la opción Manage Environments.**

**2. Seleccionamos la opción, Add Environment.**

**3. Seleccionamos que cluster queremos administrar, Docker Swarm, Kubernetes, Mesos, etc.**

**4. Por ultimo, seleccionamos la opción Create.**


![RancherOS_Consola_Env]({{ site.baseurl }}/images/rancheros_02.PNG)

Siguiendo estos pasos tendremos un cluster creado y administrado mediante Rancher OS. En los proximos capítulos les voy a mostrar mas sobre esta poderosa herramienta.

### Links de interés: ###

[Rancher OS - Capítulo 2][RancherOS2]

[Docker Official Page][Docker]

[Rancher OS Official Page][RancherOS]

[RancherOS2]: http://blog.mauriciovillagran.uy/2018/Docker-RancherOS-Capitulo2/

[Docker]: https://www.docker.com/

[RancherOS]: https://rancher.com/rancher-os/
