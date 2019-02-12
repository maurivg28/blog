---
title:  "Ansible Container"
date:   2018-12-23 23:00:00
categories: [Ansible]
tags: [Azure, Ansible, Containers]
---
Hace un tiempo que vengo investigando todo lo relacionado al mundo de los contenedores. Todos sabemos del potencial que Docker nos da para construir imagenes a traves de una Docker file. Buscando en la red, me preguntaba como seria automatizar encima de los contenedores y como hace bastante que trabajo con Ansible,
se me ocurrio investigar si podia cumplir este objetivo. A continuacion les voy a hablar de Ansible Container.

![AnsibleContainer_Logo]({{ site.baseurl }}/images/Ansible-Container-Logo.png)


## Introducción ##

Ansible Container es un proyecto de código abierto que tiene como objetivo permitir la automatización de todo el proceso de creación, despliegue y administración de contenedores a través de Ansible playbooks.

## ¿Qué es Ansible Container? ##

Ansible Container es una tecnología creada para proporcionar un flujo de trabajo centrado en Ansible para construir, ejecutar, probar y desplegar contenedores.
Ansible Container permite la creación completa de contenedores Linux con formato Docker y poder orquestar a partir de Ansible Playbooks. Su principal función es crear una aplicación a partir de un fichero YAML, en lugar de usar un fichero Dockerfile, pudiendo enumerar las funciones que queremos que tenga el contenedor, además de mejorar la forma de crear y configurar contenedores a partir de plantillas, datos encriptados, etc.
Ansible permite a los desarrolladores y operadores implementar entornos y aplicaciones de forma más rápida y sencilla, pudiendo automatizar las actividades rutinarias como la configuración de red, implementaciones en la nube y la creación de entornos de desarrollo.
Con las capacidades de Ansible Container, los usuarios serán capaces de automatizar su infraestructura y sus redes con Ansible Playbooks, usando Playbooks adicionales para incluir sus aplicaciones y desplegarlas en una plataforma de contenedor de aplicaciones.
Junto a Ansible Container, el proyecto Ansible, ha lanzado nuevos módulos Kubernetes que permiten la producción de plantillas Kubernetes directamente desde el Ansible Playbook. Esta función permite construir contenedores Linux e implementar a una plataforma de aplicación de contenedores basada en Kubernetes como Red Hat Open Shift, de manera mucho más ágil y eficiente.

## Docker File vs Ansible ##

A continuacion veremos un ejemplo de como esta armado un docker file:

```sh
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*
```
Ahora veremos que podemos usar los roles de Ansible y, por lo tanto, utilizar completamente los beneficios de las variables, las plantillas y los módulos para obtener algo con una sintaxis intuitiva, como esto:

```yaml
- name: Install Packages
  package:
      name: "{{ packages }}"
      state: present
```

## Objetivo ##

El objetivo es crear y administrar contenedores a través de una receta Ansible sin tener que utilizar ficheros Dockerfile y desplegar una aplicación de prueba para mostrar el funcionamiento.

## Manos a la obra ##

**Instalacion**

Para realizar la instalación de Ansible Container se necesitan instalar previamente las siguientes dependencias:

- docker
- python
- python-pip (pip)
- ansible

**Creando el ambiente**

**Nota:** Este ambiente esta generado sobre una maquina virtual (Ubuntu 18.04) con todos los requerimientos mencionados anteriormente. Asimismo toda la instalacion y configuracion se hizo con el usuario root.

Si no quieren copiar y pegar el codigo que les voy a ir mostrando, pueden clonar directamente el proyecto.
https://github.com/maurivg28/AnsibleContainerDemo

Vamos a crear un directorio para nuestro proyecto:

```sh
mkdir /ansible-container-demo
cd /ansible-container-demo
```

**Instalando Ansible Container**

Ansible Container depende del motor del contenedor para construir, ejecutar y desplegar su proyecto. Cuando se instala Ansible Container, se debe especificar qué motores se desea que admita su instalación. Los motores soportados actualmente son:

- k8s Kubernetes
- docker Docker engine
- openshift RedHat Openshift

Para especificar que motores instalar, se indica el nombre separado por comas, entre corchetes como parte del comando de pip install

En mi caso voy a instalar Ansible Container para el motor de Docker:

```sh
pip install "ansible-container[docker]"
```

**Inicializacion**

Los proyectos realizado con Ansible Container se identifican por su estructura de sistema de archivos. La ruta del proyecto contiene toda la fuente y archivos necesarios para construir y orquestar contenedores:

**Conductor Container**

El despliegue de Ansible Container a través del proceso de construcción "build", ocurre dentro de un contenedor especial llamado "Conductor Container".
Contiene todo lo necesario para construir imágenes de contenedores. Durante la construcción de contenedores se instala todas las dependencias necesarias a partir del "Conductor Container".
Debido a esto se recomienda que la distribución del "Conductor Container" sea la misma distribución de los contenedores creados. Por ejemplo, si está creando imágenes de contenedores basadas en Centos 7, es buena idea usar la distribución Centos 7 en el "Conductor Container". Dicha imagen de distribución se especifica en el fichero de configuración "container.yml".

**Creando los archivos necesarios**

Ansible Container proporciona una forma de iniciar un proyecto simplemente ejecutando `ansible- container init` desde el directorio de su proyecto, el cual crearía los siguientes ficheros:

```sh
ansible.cfg
ansible-requirements.txt
container.yml
meta.yml
requirements.yml
```

Para iniciar el proceso de compilación se ejecuta `ansible-container build`. Construye y lanza el "Conductor Container". Él ejecuta instancias a partir de las imágenes del contenedor base que se especifica en el fichero `container.yml`.
Para iniciar el contenedor se ejecuta `ansible-container run`. Inicia los contenedores de las imágenes compiladas descritas en el fichero "container.yml".
Para cargar las imágenes creadas en un registro de contenedores se ejecuta `ansible-container deploy`.


**¿Qué se especifica en los ficheros que hacen que esto funcione?**

**Container.yml**

Es un fichero de sintaxis YAML donde se describe los servicios, cómo construirlos y ejecutarlos, los repositorios, etc. Es similar a otros formatos de contenedores como Docker Compose o OpenCompose.
Ansible Container utiliza este archivo para determinar qué imágenes crear, qué contenedores ejecutar y conectar, y qué imágenes enviar a su repositorio. Se ejecuta automáticamente.

Un ejemplo del fichero “container.yml”.

```sh
# container.yml Syntax Version (don't change this)
version: "2"
settings:
  # Settings for the "conductor" container.
  # The Conductor container is a container used to
  # orchestrate the build of other containers in the project
  conductor:
    # Base image for the conductor container
    base: "ubuntu:xenial"
  # Title of the Project
  project_name: "ansible-container-demo"

services:
  
  # DB Backend
  redis:
    from: "redis"
    ports:
      - "6379"

  # Application
  flask:
    from: "ubuntu:xenial"
    roles:
      # Applying a role to a container
      - role: flask
    ports:
      - "5000"
    command: ["gunicorn", "--bind", "0.0.0.0:5000", "app:app", "--chdir", "/app"]
  
  # Proxy Frontend
  nginx:
    from: "ubuntu:xenial"
    roles:
      # Syntax for expressing extra role variables
      - role: "nginx"
        # These varibles are passed to the nginx config template
        nginx_proxy_host: "flask"
        nginx_proxy_port: 5000
        nginx_listen_port: 8080
    ports:
      - "8080:8080"
    command: ["nginx", "-c", "/etc/nginx/nginx_ansible.conf", "-g", "daemon off;"]

```

Estos parámetros indican lo siguiente

1) Establece la versión 2.
2) Cada uno de los contenedores que desea establecer debe estar bajo la palabra clave "services".
3) La imagen que se especifica, es la imagen base del contenedor.
4) Cada rol debe estar en el directorio "roles/" del proyecto creado.
5) Los puertos que abre el contenedor por donde escucha la aplicación.
6) Se necesita levantar todos los servicios necesarios para desplegar la aplicación,debido a que en los contenedores por defecto hay que activar los demonios.
7) Opcionalmente se puede especificar la opción "dev_overrides". Durante la compilación esta opción anula la configuración de su entorno en producción.

**Meta.yml**

Es un fichero donde se puede compartir el proyecto en "Ansible Galaxy" para que otros puedan utilizarlo como plantilla para crear sus propios proyectos.

**Ansible-requirements.txt**

Es un fichero donde se especifica las dependencias de la biblioteca Python que podría necesitar nuestra aplicación. Este archivo sigue el formato pip para las dependencias de Python. Al ejecutar Ansible cuando se crea el "Conductor Container", estas dependencias se instalan.

**Requirements.yml**

Es un fichero donde se especifica los roles del fichero “container.yml”, si están en "Ansible Galaxy" o en un repositorio remoto.

**Ansible.cfg**

Es un fichero donde se establece la configuración de Ansible dentro del contenedor durante la compilación.

**.Dockerignore**

Es un fichero donde se establece los archivos que se desea ignorar durante la construcción del contenedor.


**Ansible roles**

Uno de los mayores beneficios de usar Ansible-Container sobre Dockerfiles y Docker Compose es la capacidad de construir contenedores fácilmente utilizando la sintaxis de Ansible.

A continuacion vamos a crear los siguientes roles:

- flask
- nginx

Desde el directorio del proyecto, ejecutaremos el siguiente comando para crear los subdirectorios:

```sh
mkdir -p roles/nginx/tasks roles/nginx/templates roles/flask/tasks roles/flask/files
```
La estructura del proyecto deberia quedar asi:

```sh
├── ansible.cfg
├── container.yml
├── ansible-requirements.txt
├── meta.yml
├── requirements.yml
└── roles
    ├── flask
    │   ├── files
    │   └── tasks
    └── nginx
        ├── tasks
        └── templates
```

**Flask**

Vamos a crear una aplicacion en flask muy sencilla, para ello cree un nuevo archivo app.py en el directorio roles / flask / files con el siguiente contenido:

```python
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

En nuestro playbook de ansible, se copiara el archivo **app.py** dentro de la imagen de docker y se instalaran todas las dependencias necesarias. Todo esto mientras se esta construyendo la imagen de docker.

Para crear el playbook de Ansible, cree un nuevo archivo **main.yml** en el directorio **roles/flask/tasks** con el siguiente contenido:

```python
---
- name: Update APT Repositories
  apt:
    update_cache: yes
    cache_valid_time: 600

- name: Install APT Requirements
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - "python-pip"

- name: Install Pip Requirements
  pip:
    #name: "{{ item }}"
    state: present
  with_items:
    - "flask"
    - "gunicorn"
    - "redis"

- name: Create App Folder
  file: 
   path: "/app"
   state: directory

- name: Copy Flask App
  copy:
    src: app.py
    dest: /app/app.py
```

**Nginx**

Para configurar nuestra instancia de proxy Nginx, crearemos dinámicamente un archivo de configuración con nuestros parámetros deseados. Para crear la plantilla Nginx, cree un nuevo archivo **virtualhost.j2** en el directorio **roles/nginx/templates** con el siguiente contenido:

```python
# THIS FILE IS MANAGED BY ANSIBLE
worker_processes 1;

events {
  worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    proxy_buffer_size   128k;
    proxy_buffers   4 256k;
    proxy_busy_buffers_size   256k;

    server {
        listen {{ nginx_listen_port }};

        location / {
            proxy_pass http://{{ nginx_proxy_host }}:{{ nginx_proxy_port }}/;
            proxy_hide_header X-Frame-Options;
            proxy_set_header X-Forwarded-Host $host:$server_port;
            proxy_set_header X-Forwarded-Server $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

Luego, implementaremos el proceso para instalar y configurar el archivo de configuración de plantilla dentro de nuestro playbook de Ansible. Para crear el playbook, cree un nuevo archivo **main.yml** en el directorio **roles/nginx/tasks** con el siguiente contenido:

```python
---
- name: Update APT
  apt:
    update_cache: yes
    cache_valid_time: 600

- name: Install APT Requirements
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - "nginx"

- name: Copy config template
  template:
    src: virtualhost.j2
    dest: /etc/nginx/nginx_ansible.conf
```

**Como funciona de proyecto?**

Ahora que todos los archivos están en su lugar, podemos comenzar a unir el proyecto.

Para organizar proyectos de múltiples contenedores, Ansible Container utiliza un archivo con formato YAML, **container.yml**. Este archivo describe qué imágenes usar, qué contenedores ejecutar, atributos de contenedor y qué hacer con las imágenes creadas. Esto es muy similar a un archivo Docker Compose.

Estudie el archivo container.yml a continuación, luego lo utilice como base para reemplazar el contenido del archivo container.yml en la raíz del proyecto.

```python
# container.yml Syntax Version (don't change this)
version: "2"
settings:
  # Settings for the "conductor" container.
  # The Conductor container is a container used to
  # orchestrate the build of other containers in the project
  conductor:
    # Base image for the conductor container
    base: "ubuntu:xenial"
  # Title of the Project
  project_name: "ansible-container-demo"

services:
  
  # DB Backend
  redis:
    from: "redis"
    ports:
      - "6379"

  # Application
  flask:
    from: "ubuntu:xenial"
    roles:
      # Applying a role to a container
      - role: flask
    ports:
      - "5000"
    command: ["gunicorn", "--bind", "0.0.0.0:5000", "app:app", "--chdir", "/app"]
  
  # Proxy Frontend
  nginx:
    from: "ubuntu:xenial"
    roles:
      # Syntax for expressing extra role variables
      - role: "nginx"
        # These varibles are passed to the nginx config template
        nginx_proxy_host: "flask"
        nginx_proxy_port: 5000
        nginx_listen_port: 8080
    ports:
      - "8080:8080"
    command: ["nginx", "-c", "/etc/nginx/nginx_ansible.conf", "-g", "daemon off;"]
```

**Ejecutando el proyecto**

Finalmente, ahora podemos construir y ejecutar nuestros contenedores Docker. Hay que asegurarse de estar en el directorio raíz del proyecto y ejecutamos el siguiente comando:

```sh
ansible-container build
```

El comando anterior procesará los playbooks y construirá las imágenes de Docker. Una vez que se haya completado con éxito, podemos correr el contenedor con el siguiente comando:

```sh
ansible-container run
```

**Final**

¡Por Fin! Hemos utilizado satisfactoriamente Ansible para crear una aplicación de múltiples contenedores en Docker. La aplicación se divide en tres niveles: **Presentación (nginx), Lógica (flask) y Almacenamiento (redis)**, y es ideal para usar como base para iniciar sus propios proyectos.


### Comandos utiles ###

**ansible-container init**

Creates files in the current directory to get you started. Read the comments, and edit to suit your needs.

**ansible-container install**

Downloads Ansible-Container-ready roles from Ansible Galaxy, and installs them in your project.

**ansible-container build**

Creates images from your Ansible playbooks.

**ansible-container run**

Launches the containers specified in the orchestration document, container.yml, for testing the built images. The format of container.yml is nearly identical to Docker Compose.

**ansible-container deploy**

Pushes the project's container images to a registry of your choice, and generates a playbook capable of deploying the project on a supported cloud provider.

### Demo ###

{%include youtubePlayer.html id="1hPisA4ORIU"%}

### Links de interés: ###

A continuacion dejo los links que utilice como referencia para armar este articulo:

[Ansible Container Installation][Ansible Container Installation]

[Ansible Container Installation]: https://docs.ansible.com/ansible-container/installation.html

[Ansible Container Started Guide][Ansible Container Started Guide]

[Ansible Container Started Guide]: https://docs.ansible.com/ansible-container/getting_started.html

[Ansible Container Doc][Ansible Container Doc]

[Ansible Container Doc]: https://docs.ansible.com/ansible-container/

[Ansible Container Integration][Ansible Container Integration]

[Ansible Container Integration]: https://www.ansible.com/integrations/containers/ansible-container

[Revistacloudcomputing][Revistacloudcomputing]

[Revistacloudcomputing]: https://www.revistacloudcomputing.com/2016/06/red-hat-presenta-ansible-container/