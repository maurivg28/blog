---
title:  "Docker - Capitulo 1"
date:   2018-05-09 23:15:23
categories: [Docker]
tags: [Docker, DevOps, Contenedores, Images]
---
No hace mucho que me toco dar soporte a una infraestructura montada en un ambiente de contenedores, es por eso que les voy a contar un poco acerca de Docker.
Docker es una de las herramientas de la que se está hablando mucho, esto es así ya que tiene varios aspectos interesantes que cambian la forma de desarrollar aplicaciones. Docker es una forma de ejecutar procesos de forma aislada pero también se compone de herramientas para construir imágenes y un repositorio para compartirlas.

![Docker_Logo]({{ site.baseurl }}/images/Docker_Logo.png)

## Introduccion ##

Los contenedores son la vía que nos coloca Docker para tener el mismo uso que con las maquinas virtuales creadas de la forma tradicional. Docker utiliza estos contenedores para aislar uno o más procesos. Estos procesos en el Host necesitan Memoria, CPU, Acceso a la Red y espacio en disco.

Es un ambiente perfecto para que las aplicaciones puedes funcionar de forma correcta. Esto incluye algunos ejecutables y librerías específicas, además de la librería estándar de C (libc). Obviamente el Kernel está detrás de muchos de estos componentes y todo el acceso al Kernel es abstracta mediante la librería libc.

## Que es una imagen de Docker ##

Una parte básica de la introducción a Docker es entender qué es una imágen. Ya comentamos que Docker y toda su funcionalidad depende o esta principalmente en el manejo de Contenedores ahora bien si bien los contenedores son muy importantes debemos conocer de donde nacen y para esto conoceremos el concepto “Imagen” que viene siendo en si nuestro “sistema operativo” osea podemos decir que tenemos una imagen de Centos, Ubuntu o Debian.

Ahora bien lo bueno de las imágenes es que pueden ser más que solo un sistema operativo y podemos decir que tenemos una imagen de Nginx o una imagen de MySQL o de Memcached ,que estos últimos en si son servicios pero en Docker podemos crear imágenes a partir de unos servicios ya instalados y hasta configurados.

## Que es un contenedor? ##

Los contenedores de Docker nacen a partir de una imagen y en estos contenedores podemos solo ejecutar e instalar servicios, viene siendo como crear una maquina virtual a partir de una imagen (snapshot) pero muchísimo más ligera. Viene siendo como el siguiente ejemplo: tenemos una Imagen de Ubuntu 14.04 con Django instalado y vamos a crear 3 Contenedores a partir de esta imagen de Ubuntu seria algo como esto.

![Docker_Ejemplo1]({{ site.baseurl }}/images/contenedores-docker-01.png)

Ahora los contenedores al igual que las maquinas virtuales tradicionales están aisladas del host, luego cada contenedor debe tener su ID único y un nombre que sea legible por cualquier humano esto es netamente para identificar cada contenedor y luego es necesario que cada contenedor exponga los servicios que necesite y Docker permite exponer puertos del contenedor para que el Host identifique cuando tratemos de entrar a cada puerto lo que es conocido como el port forwarding  podemos ver un ejemplo en la siguiente imagen con los contenedores que hemos creado en el ejemplo anterior

![Docker_Ejemplo2]({{ site.baseurl }}/images/contenedores-docker-02.png)

Los contenedores están diseñados para ejecutar aplicaciones, es decir, no están originalmente pensados para ejecutar todo lo que lleva una máquina aunque si se puede utilizar los contenedores como máquinas virtuales, pero como veremos perderemos gran flexibilidad ya que la principal funcionalidad que queremos es poder separar la parte de ejecución con los datos.

Esto permite actualizar los servicios de forma rápida y ligera sin afectar los datos de tu aplicación por ejemplo tienes tus datos de tu aplicación en tu máquina Host, luego crear un contenedor y colocar compartida esta carpeta dentro del contenedor, para finalmente después en este contenedor administrar cómo se ejecutarán por ejemplo servicios como Nginx o Apache, y que estos últimos puedan despachar los archivos que están en la carpeta compartida.

## Caracteristicas de una imagen de Docker ##

 - Portátil: pueden ser versionadas en los repositorios de Docker Hub, o guardarse como un archivo tar.
 - Estática: el contenido no se puede cambiar, a menos que hagas una nueva imagen.

## Características de un contenedor ##

**Tiempo de ejecución:** cada contenedor se ejecuta en un solo proceso.

**Permisos de escritura:** sólo tendrá permiso a sus propios archivos y a los volúmenes asociados .

**Capas:** es en una imagen en base a un sistema operativo.

Estos términos aparecen en diversos contextos, y es importante ver cómo se relacionan entre sí. Ahora que tenemos estos fundamentos básicos, podemos pasar a conocer más sobre otro importante concepto de Docker: los volúmenes.

## Que son los volúmenes de Docker? ##

Los volúmenes son para mantener los datos más allá de la vida útil de su contenedor. Son espacios dentro del contenedor que almacenan datos fuera de ella, lo que le permite destruir / reconstruir / cambiar  las veces que queramos nuestro contenedor y se mantendrán intactos sus datos.

Docker permite definir qué partes son la aplicación y qué partes son sus datos. Uno de los mayores cambios en la mentalidad de que hay que hacer cuando se trabaja con Docker es que los contenedores deben ser efímeros y desechables.

Los volúmenes son específicos de cada contenedor, puedes crear varios contenedores de una sola imagen y definir el volumen para cada uno. Los volúmenes se almacenan en el sistema de archivos del servidor que ejecuta el Docker. Todo lo que no es un volumen se almacena en otro tipo de sistema de archivos, pero lo veremos más adelante, aquí podemos ver un ejemplo en la siguiente imagen.

![Docker_Ejemplo3]({{ site.baseurl }}/images/contenedores-docker-03.png)

## Conlusión ##

Ahora hemos visto que es el Docker, cuales son sus principales características, asi pudimos conocer desde donde nacen los Contenedores y a que llamamos Imágenes para luego integrarlo con nuestros Volúmenes de datos con estos conceptos bastante claros podemos pasar a la parte practica de como administrar nuestro Docker para esto vamos a crear un articulo completo en donde podremos aprender cada comando para comenzar a trabajar con Docker.

### Links de interés: ###

[Docker Capitulo 2][DockerCap2]

[DockerCap2]:http://blog.mauriciovillagran.uy/2018/Docker-Capitulo2/

[Docker Official Page][Docker]

[Docker]: https://www.docker.com/
