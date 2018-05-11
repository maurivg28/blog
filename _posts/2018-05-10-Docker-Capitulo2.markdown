---
title:  "Docker - Capitulo 2"
date:   2018-05-09 23:15:23
categories: [Docker]
tags: [Docker, DevOps, Contenedores, Images]
---
Docker se ha convertido en una herramienta indispensable hoy en día para todos los que somos Desarrolladores de aplicaciones o SysAdmins, ya que como hablamos en el primer capítulo nos brinda un entorno muy simple de aplicar en nuestras maquinas, no nos consumirá tantos recursos y podremos desarrollar / probar nuestros scripts en distintos sistemas operativos.


![Docker_Logo]({{ site.baseurl }}/images/Docker_Logo.png)

## Manos a la obra ##

Ahora vamos a ver la parte practica y algunos comandos que nos van a ser muy utiles para manejar Docker.

### Instalar Docker en Centos 7 ###

Como siempre procedemos a actualizar nuestro sistema con el siguiente comando
```bash
yum -y update
```
Ahora instalamos los paquetes necesarios para Docker
```bash
yum -y install docker docker-registry
```
Colocamos el servicio de Docker para que inicie siempre cuando se inicie la maquina
```bash
systemctl enable docker.service
```
Ahora iniciamos el Servicio de Docker
```bash
systemctl start docker.service
```
Con este comando podemos tener el estatus actual de Docker
```bash
systemctl status docker.service
```

### Comandos útiles de Docker ###

***docker images:*** listado de imágenes ya descargadas
```bash
[infra@localhost ~]$ sudo docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
ubuntu latest c5f1cf30c96b 6 days ago 120.7 MB
centos 6.6 881cca81e67f 6 months ago 202.6 MB
```
***docker pull <ID imagen>:*** descargar imágen para tenerla disponible para los contenedores
```bash
[infra@localhost ~]$ sudo docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
6d28225f8d96: Pull complete 
166102ec41af: Pull complete 
d09bfba2bd6a: Pull complete 
c80dad39a6c0: Pull complete 
a3ed95caeb02: Pull complete 
Digest: sha256:5718d664299eb1db14d87db7bfa6945b28879a67b74f36da3e34f5914866b71c
Status: Downloaded newer image for ubuntu:latest
```
***docker rmi  <ID imagen>:*** borrado de una imagen
```bash
[infra@localhost ~]$ sudo docker rmi ubuntu
Untagged: ubuntu:latest
Deleted: sha256:c5f1cf30c96b5b55c0e6385f2ecb791790eacfdc874500ec3dd865789e358dd1
Deleted: sha256:6966dfd905fe3357994340c67347285cfce8a1791fc22806c9c0f427fbdeec40
Deleted: sha256:65faf101139189314da25357e4704f3412877d4b881d86944e9616630f2a4faa
Deleted: sha256:713a70d252b71a53cf3d090acef5c1bae668cb489f5d8b1205f2cb9e0a6cd68a
Deleted: sha256:3417308f5ad0aa1dd0af30817cd1bdedeb11971023a8f54668b17a29078ced1c
Deleted: sha256:7aae4540b42d10456f8fdc316317b7e0cf3194ba743d69f82e1e8b10198be63c
```
***docker history <ID imagen>:*** historial de una imagen
```bash
[infra@localhost ~]$ sudo docker history ubuntu
IMAGE CREATED CREATED BY SIZE COMMENT
c5f1cf30c96b 6 days ago /bin/sh -c #(nop) CMD ["/bin/bash"] 0 B 
<missing> 6 days ago /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/ 1.895 kB 
<missing> 6 days ago /bin/sh -c rm -rf /var/lib/apt/lists/* 0 B 
<missing> 6 days ago /bin/sh -c set -xe && echo '#!/bin/sh' > /u 701 B 
<missing> 6 days ago /bin/sh -c #(nop) ADD file:ffc85cfdb5e66a5b4f 120.7 MB
```
***Crear Contenedor de Ubuntu con acceso a shell***
```bash
[infra@localhost ~]$ sudo docker run -d -P -p 22:22 --name ubuntuinfra -it ubuntu:latest
4426ff3bf6603b19181844381c7f9707a71c600659e58e5917952d06a344e12d
```
***Listado de Contenedores activos***
```bash
[infra@localhost ~]$ sudo docker ps -a
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
4426ff3bf660 ubuntu:latest "/bin/bash" 4 minutes ago Up 4 minutes 0.0.0.0:22->22/tcp ubuntuinfra
```
***Iniciar Contenedor***
```bash
[infra@localhost ~]$ sudo docker start ubuntuinfra
ubuntuinfra
```
***Conectarnos al Contenedor (Tip: Luego de ejecutar el attach presiona ENTER nuevamente para activar el contenedor)***
```bash
[infra@localhost ~]$ sudo docker attach ubuntuinfra
root@4426ff3bf660:/#
root@4426ff3bf660:/# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04 LTS"
```
**Nota: Con Crtl +D podemos salir del contenedor y volver a nuestro entorno normal**

***Borrar Contenedor***
```bash
[infra@localhost ~]$ sudo docker rm ubuntuinfra
ubuntuinfra
[infra@localhost ~]$ sudo docker ps -a
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
```
***Cómo obtener Ip Privada del contenedor***
```bash
sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' $CID
```
Con estos comandos vamos a poder de alguna manera administrar una intraestructura propia de contenedores.

### Links de interés: ###

[Docker Official Page][Docker]

[Docker]: https://www.docker.com/
