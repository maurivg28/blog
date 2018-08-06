---
title:  "Ansible + Jenkins"
date:   2018-08-05 21:46:00
categories: [Ansible]
tags: [Ansible, Azure, Cloud, Microsoft, DevOps]
---
Siguiendo un poco todo el tema de Ansible y el despliegue de configuraciones que les contaba en el capítulo anterior. Se me ocurrió automatizar el proceso de dicha configuración. Es por eso que les voy a contar acerca de una herramienta la cual esta pensada para este tipo de automatizaciones. Vamos a hablar de Jenkins, cómo es su instalación, etc. También les voy a contar como configurar el plugin de Ansible en Jenkins para poder lanzar los procesos de CI/CD desde esta herramienta.

![Jenkins_Logo]({{ site.baseurl }}/images/jenkins_new_0.jpg)

## Instalación ##

Su instalación es bastante sencilla, yo lo instale en Centos 7. Lo vamos a poder instalar siguiendo estos sencillos pasos:

- Agregar los repositorios correspondientes:
```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins
```

- Instalar JAVA:
```bash
sudo yum install java
```

- Iniciar el servicio de Jenkins y configurarlo para que levante al iniciar el sistema:
```bash
sudo service jenkins start
sudo chkconfig jenkins on
```

Una vez que tenemos instalado los paquetes necesarios y hemos iniciado el servicio, podremos acceder a la siguiente url para terminar con con su configuración:

http://127.0.0.1:8080

## Jenkins y Ansible ##

Una vez que hemos terminado de configurar Jenkins, debemos instalar ansible y algunos otros paquetes para que todo funcione de manera correcta.

- Instalar Ansible:
```bash
sudo yum install ansible
```

- Instalar python-pip, necesario para instalar modulos que necesita ansible:
```bash
sudo yum install epel-release
sudo yum install python-pip
```

- Instalar pipwinrm, necesario para conectarse a un servidor Windows:
```bash
pip install "pywinrm>=0.2.2"
```

## Jenkins y Git ##

Tambien debemos instalar Git, para poder configurar dicho plugin con Jenkins y que el mismo a la hora de ejecutar el job o tarea, pueda clonar el repositotio donde tenemos nuestros playbooks o recetas de ansible.

- Instalar Git:
```bash
sudo yum install Git
```

## Ansible plugin y Git ##

Debemos seguir una serie de pasos para instalar y configurar dichos plugins. Primero vamos a instalar el plugin de ansible ya que por defecto no viene instalado:

- En el menú de configuración de Jenkins, ir a **Administrar Jenkins**
- Ir a **Administrar Plugins**
- Seleccionamos la solapa **Todos los plugins**
- Buscamos Ansible y lo instalamos.

Una vez que tenemos el plugin de ansible instalado, vamos a pasar a configurarlo:

- En el menú de configuración de Jenkins, ir a **Administrar Jenkins**
- Ir hacia **Global Tool Configuration**
- Buscar el plugin llamado Ansible, y pinchar sobre el boton **Añadir Ansible**
- Completamos con el nombre **ansible 2.4.2.0** y el path **/usr/bin/** y luego guardamos los cambios.

Para configurar el plugin de Git deben seguir los mismos pasos que usamos para configurar el plugin de ansible, solo que para Git debemos completar lo siguiente:

- En el campo name, completamos con **Git 1.8.3.1** y en el campo path completamos con, **/usr/bin/git**

Una vez que tenemos estos dos plugins configurados, vamos a pasar a configurar el job o tarea. Les voy a dejar un video para que vean lo sencillo que es. Obviamente que jenkins nos permite generar ciclos mas complejos, pero la idea de este tutorial es mostrarles como lo hice funcionar con ansible.

## Menos a la obra ##

{%include youtubePlayer.html id="jcgaaj4aoZM"%}

## Links de interes ##

[Docs Ansible Official Page][Ansible]

[Ansible]: https://docs.ansible.com/

[Jenkins Official Page][Ansible]

[Jenkins]: https://jenkins.io/
