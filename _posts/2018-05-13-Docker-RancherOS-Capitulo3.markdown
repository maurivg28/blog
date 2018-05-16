---
title:  "Rancher OS - Capitulo 3"
date:   2018-05-14 20:52:00
categories: [Docker, RancherOS, Swarm]
tags: [Docker, DevOps, Contenedores, Images, RancherOS, Swarm]
---
Continuando con el capítulo 3 de Rancher, les quiero comentar que ya esta disponible la versión 2.0 de esta herramienta la cual trae nuevas funciones dejando de lado otras con las que contaba en su versión 1.x.
Hoy vamos a manejar varios conceptos nuevos. En capítulos anteriores hablamos de Docker Swarm, y hoy en varias ocaciones les voy a mencionar a Kubernetes.

![RancherOS_Logo]({{ site.baseurl }}/images/rancheros_logo1.png)

## La evolución de Rancher OS 2.0 ##

Cómo ya sabemos, Rancher OS es una plataforma de administración de contenedores diseñada para organizaciones que implementan contenedores en producción. Rancher hace que sea fácil ejecutar Kubernetes en todas partes, cumplir con los requisitos de TI y capacitar a los equipos de DevOps.

### Dónde esta Docker Swarm? ###

Kubernetes se ha convertido en el estándar de orquestación de contenedores. La mayoría de los proveedores de nube y virtualización ahora lo ofrecen como infraestructura estándar. Los usuarios de Rancher tienen la opción de crear clústeres de Kubernetes con Rancher Kubernetes Engine (RKE) o servicios en la nube de Kubernetes, como GKE (Google Cloud Kubernetes Engine), AKS (Azure Kubernetes Service)  y EKS (Amazon Elastic Kubernetes Services). Los usuarios de Rancher también pueden importar y administrar sus clústeres existentes de Kubernetes creados con cualquier distribución o instalador de Kubernetes. Mientras que segun expertos, Docker Swarm esta pensado para ambientes pequeños. 
En próximos capitulos haremos una comparacion de las ventajas de usar uno y otro.

### Nuevas características ###

Rancher admite la autenticación centralizada para todos los clústeres de Kubernetes bajo su control. Por ejemplo, se puede:

- Usar credenciales de Active Directory para acceder a los clústeres de Kubernetes alojados por proveedores de la nube, como GKE.

- Configurar políticas de seguridad y control de acceso en todos los usuarios, grupos, proyectos, clústeres y nubes.

- Ver la salud y la capacidad de los clústeres de Kubernetes desde un solo panel.

### Más poder para el equipo de DevOps ###

Rancher proporciona una interfaz de usuario intuitiva para que los equipos de DevOps administren la carga de trabajo de sus aplicaciones. El usuario no necesita tener un conocimiento profundo de los conceptos de Kubernetes para comenzar a usar Rancher. El catálogo de Rancher contiene un conjunto de útiles herramientas DevOps. Rancher está certificado con una amplia selección de productos de ecosistemas nativos de la nube, que incluyen, por ejemplo, herramientas de seguridad, sistemas de monitoreo, registros de contenedores y controladores de almacenamiento y redes.

La siguiente figura ilustra el rol que juega Rancher en las organizaciones de TI y DevOps. Cada equipo implementa sus aplicaciones en las nubes públicas o privadas que elijan. Los administradores de TI obtienen visibilidad y aplican políticas en todos los usuarios, clústeres y nubes.

![RancherOS_Diagrama]({{ site.baseurl }}/images/rancheros_04.PNG)

### Manos a la Obra ###

A continuación veremos una guía rapida sobre como funciona esta nueva versión de Rancher OS. Desplegaré un host con Docker y Rancher OS como orquestador de un cluster de Kubernetes que montaremos en la nube de Microsoft Azure.

***Host de administración ***

**Sistema Operativo:**

- Ubuntu 16.04 Lts
- Rancher OS 2.0

**Hardware:**

- 4 GB RAM

**Sofrware:**

- Docker version 17.03 o superior.

***Instalación de Rancher***

Una vez conectados al host de administracion vía ssh, ejecutaremos el siguiente comando:

```bash
$ sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
```

***Login***

1 -Abrir un navegador e ingresar a la dirección IP del servidor de Rancher OS https://Server_IP

2- Crear la contraseña del usuario admin. (Por defecto: admin).

***Crear un cluster***

1- Desde la pagina de **Cluster**, click en **Add Cluster**

2- Elegir ***Custom***

3- Ingresar el nombre del cluster.

4- Saltear la opción **Member Roles** y **Cluster Options**. Las configuraremos mas adelante.

5- Click **Next**.

6- Desde **Node Role**, seleccionar todos los roles: **etcd, Control,**y **Worker**.

7- Rancher detectará automaticamente las direcciones IP que usará para la comunicacion entre el cluster. Si queremos cambiar la configuracion lo tendremos que hacer en la sección **Node Address**.

8- Saltear la opcion **Labels**.

9- Copiar el comando que veremos en pantalla en el portapapeles.

10- Ingresar al host Linux, pegar y ejecutar el comando.

11- Cuando la ejecucuón de comando termine, haremos click en **Done**.

***Crear un WorkLoad***

Ya estamos listos para crear el primer Workload. Un Workload es un objeto que incluye PODS junto con otros archivos e información necesaria para implementar su aplicación.

En este Workload, implementaremos una aplicacion con Nginx.

1- Desde la pagina **Clusters**, abrir el cluster previamente creado.

2- Desde el menú principal **Dashboard**, seleccionar **Projects**.

3- Abrir **Default** projects.

4- Click en **+**, **Deploy**.

5- Ingresamos el nombre del **Workload**.

6- Desde la opción **Docker image**, ingresamos `nginx`

8- Desde la opción **As a**, nos aseguramos que la opción **Node port (on every node)** este seleccionada.

9- Desde la opción **On listening port**, dejamos la opción **Random** seleccionada.

10- Desde la opción **Publish the container port**, configuramos el puerto 80.

11- Dejaremos las opciones restantes por defecto, ya que las configuraremos mas adelante.

12- Click **Launch**.

Una vez que el proceso termine podremos acceder a nuestra aplicación. A continuación les voy a dejar un video en el cual muestro todo el proceso mencionado anteriormente.


###Video de demostración###

En el video, verán como se configura un cluster de Kubernetes montado en Azure desde Rancher OS. Asimismo verán que la infraestructura cuenta con los siguientes componentes: 

- Grupo de disponibilidad para los host.
- Balanceador de carga.
- Host Linux con Ubuntu, dentro de este host está instalado Rancher y Docker.
- Tres nodos con Ubuntu.
- Aplicación con Nginx.

En próximas ediciones les voy a mostrar como desplegar un cluster de kubernetes como servicio sobre Azure.

***Parte 1 - Creacion del cluster.***

{%include youtubePlayer.html id="BbYKKENSRd8"%}

***Parte 2 - Creacion de los contenedores.***

{%include youtubePlayer.html id="3YqvpxuIIhM"%}


### Links de interés: ###

[Docker Official Page][Docker]

[Rancher OS Official Page][RancherOS]

[Video - Rancher OS y Swarm en acción][Video]

[Docker]: https://www.docker.com/

[RancherOS]: https://rancher.com/rancher-os/

[Video]: https://youtu.be/gEpNyVgQkU8
