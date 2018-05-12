---
title:  "Docker Swarm - Capitulo 1"
date:   2018-05-12 16:31:23
categories: [Docker, Swarm]
tags: [Docker, DevOps, Contenedores, Images, Swarm]
---
Siguendo en el mundo de los contenedores y de Docker. Hoy les voy a hablar de una herramienta propia de Docker la cual me quedaba por investigar. Docker Swarm.


![DockerSwarm_Logo]({{ site.baseurl }}/images/Docker_SwarmLogo.png)

A partir de la versión 1.12 de Docker es muy sencillo hacer un clúster formado por varios Docker Hosts mediante lo que se conoce como Swarm Mode.

### Que es Docker Swarm? ###

Su definición es tan sencilla como decir que: "Docker Swarm es una herramienta nativa que permite construir un clúster de máquinas docker llamadas Nodos".

### Que es un nodo? ###

En un clúster de Swarm existen dos tipos de nodo, Manager y Worker.

Los nodos Manager son los encargados de gestionar el clúster. Entre todos los Manager se elige automáticamente un líder y éste es el encargado de mantener el estado del clúster.

Los Manager son también los encargados de distribuir las tareas o tasks (unidades básicas de trabajo) entre todos los nodos Worker, los cuales reciben estas tareas y las ejecutan.

Los nodos Manager por defecto también actúan como nodos Worker aunque se puede cambiar su configuración para que sólo asuman tareas de Manager.

![Docker_Swarm_Func]({{ site.baseurl }}/images/Docker_Swarm_Func01.png)

### Servicios y tareas ###

Un servicio define las tareas que serán ejecutadas dentro del clúster.

Cuando creamos un servicio le indicamos a Swarm qué imagen y qué parametrización se utilizará para crear los contenedores que se ejecutarán después como tareas dentro del clúster.

Existen dos tipos de servicios, replicados y globales:

	En un servicio replicado, Swarm creará una tarea por cada réplica que le indiquemos para después distribuirlas en el clúster. Por ejemplo, si creamos un servicio con 4 réplicas, Swarm creará 4 tareas.

	En un servicio global, Swarm ejecutará una tarea en cada uno de los nodos del clúster.

Como hemos dicho antes, las tareas son la unidad de trabajo dentro de Swarm. Realmente son la suma de un contenedor más el comando que ejecutaremos dentro de ese contenedor.

Los Manager asignan tareas a los nodos Worker de acuerdo al número de réplicas definidas por el servicio. Una vez que la tarea es asignada a un nodo ya no se puede mover a otro, tan sólo puede ejecutarse o morir.

Ante la caída de una tarea, Swarm es capaz de crear otra similar en ese u otro nodo para cumplir con el número de réplicas definido.

### Balanceo ###

Swarm tiene un sistema de balanceo interno para exponer servicios hacia el exterior del clúster.

Un Manger es capaz de publicar automáticamente un puerto generado al azar en el rango 30000-32767 para cada servicio, o bien, nosotros podemos publicar uno específico.

Cualquier sistema externo al clúster podrá acceder al servicio en este puerto publicado a través de cualquier nodo del clúster, independientemente de que ese nodo esté ejecutando una tarea del servicio o no.

Todos los nodos del clúster enrutarán a una tarea que esté ejecutando el servicio solicitado.

Además, Swarm cuenta con un DNS interno que asigna automáticamente una entrada a cada uno de los servicios desplegados en el clúster.

### Creacion del cluster ###

Para crear un clúster con Swarm tenemos que partir de un nodo destinado a ser Manager. Este nodo debe tener Docker 1.12 o superior ya instalado.

Suponiendo que la IP del nodo es 10.0.0.40, ejecutamos el siguiente comando:

```bash
$ docker swarm init --advertise-addr 10.0.0.40
Swarm initialized: current node (cf2mywzpa3pd77z23u5jciqwi) is now a manager.

To add a worker to this swarm, run the following command:  
    docker swarm join \
    --token SWMTKN-1-50edlk935u9qgvrs8alhpzf1awgdil2dmfs4zgpd8ue2ltkmww-3y9tb0pjnieqao9ahkutpvpxe \
    10.0.0.40:2377

To add a manager to this swarm, run the following command:  
    docker swarm join \
    --token SWMTKN-1-50edlk935u9qgvrs8alhpzf1awgdil2dmfs4zgpd8ue2ltkmww-06m3vta11n4ihjyr1ytycwqvf \
    10.0.0.400:2377
```

Al ejecutar el comando hemos inicializado este nodo como Manager.

Con el parámetro obligatorio --advertise-addr le indicamos la IP del Manager que se utilizará internamente para las conexiones de Swarm. Si omitimos el puerto tomará el 2377 por defecto.

La salida del comando nos muestra dos tokens. Cada uno de ellos sirve para unir nodos Manager y Worker adicionales.

Añadiremos ahora un nodo Worker al clúster. Para ello y desde la consola del Worker ejecutamos:

```bash
$ docker swarm join --token  SWMTKN-1-50edlk935u9qgvrs8alhpzf1awgdil2dmfs4zgpd8ue2ltkmww-3y9tb0pjnieqao9ahkutpvpxe 10.0.0.40:2377
This node joined a swarm as a worker.  
```

Como se puede ver hemos utilizado el token para nodos Worker. Le indicamos también la IP y puerto de uno de los nodos Manager del clúster existente.

Si queremos añadir otro Manager el comando sería el mismo, pero usaríamos el otro token.

### Gestión de servicios ###

Para dar de alta un nuevo servicio nos vamos a la consola de un Manager y ejecutamos lo siguiente:

```bash
$ docker service create --replicas 5 --name helloworld alpine ping ejemplo.com.uy
```

Donde --name es el nombre del servicio y --replicas es el número de tareas de este servicio que queremos crear.

Ahora podemos ver un listado de todos los servicios en el clúster:

```bash
$ docker service ls

ID            NAME        SCALE  IMAGE   COMMAND  
9uk4639qpg7n  helloworld  1/1    alpine  ping ejemplo.com.uy
```

Podemos ver información sobre el servicio:

```bash
$ docker service inspect --pretty helloworld

ID:     9uk4639qpg7npwf3fn2aasksr  
Name:       helloworld  
Mode:       REPLICATED  
 Replicas:      5
Placement:  
UpdateConfig:  
 Parallelism:   5
ContainerSpec:  
 Image:     alpine
 Args:  ping docker.com
 ```

 Podemos ver todas las tareas que se están ejecutando para este servicio:

```bash
 $ docker service ps helloworld

ID                         NAME          SERVICE     IMAGE   LAST STATE         DESIRED STATE  NODE  
8p1vev3fq5zm0mi8g0as41w35  helloworld.1  helloworld  alpine  Running 3 minutes  Running        worker2  
```

Si queremos aumentar o disminuir el número de réplicas ejecutamos:

```bash
$ docker service scale helloworld=10

helloworld scaled to 10
```

Y después comprobamos en qué nodos están corriendo las nuevas réplicas:

```bash
$ docker service ps helloworld

ID                         NAME          SERVICE     IMAGE   LAST STATE          DESIRED STATE  NODE  
8p1vev3fq5zm0mi8g0as41w35  helloworld.1  helloworld  alpine  Running 7 minutes   Running        worker2  
c7a7tcdq5s0uk3qr88mf8xco6  helloworld.2  helloworld  alpine  Running 24 seconds  Running        worker1  
6crl09vdcalvtfehfh69ogfb1  helloworld.3  helloworld  alpine  Running 24 seconds  Running        worker1  
auky6trawmdlcne8ad8phb0f1  helloworld.4  helloworld  alpine  Running 24 seconds  Accepted       manager1  
ba19kca06l18zujfwxyc5lkyn  helloworld.5  helloworld  alpine  Running 24 seconds  Running        worker2  
```

### Rolling Updates ###

Otra funcionalidad de Swarm Mode son los Rolling Updates, es decir, la capacidad de gestionar actualizaciones en los servicios.

Si la imagen de un servicio se ha actualizado o cambia, podemos decirle a Swarm que actualice el servicio y, a su vez, todas las tareas asociadas al servicio.

La actualización de las tareas se puede hacer secuencialmente o en paralelo.

Por defecto se hace secuencialmente. Si queremos paralelizar las actualizaciones debemos indicarlo en el momento de la creación del servicio:

```bash
$ docker service create --replicas 4 --name apache --update-delay 10s --update-parallelism 2 apache:3.0.6
```

Donde --update-delay es el tiempo que esperará hasta empezar con la actualización de la siguiente tarea, y donde --update-parallelism será el número de tareas que actualizará en paralelo.

Para realizar una actualización le indicamos la nueva imagen:

```bash
$ docker service update --image apache:3.0.7 apache
```

### Conclusión ##

Es muy sencillo montar un ambiente de contenedores, con alta disponibilidad utilizando esta herramienta nativa de Docker. Actualmente ya se esta utilizando no solo para ambientes de desarrollo y testing, sino para ambientes de producción.


### Links de interés: ###

[Docker Official Page][Docker]

[Docker Swarm Docs][Swarm Docs]

[Docker]: https://www.docker.com/
[Swarm Docs]: https://docs.docker.com/engine/swarm/
