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

Ansible Container, es un orquestador el cual nos permite crear imagenes para nuestros contenedores. En vez de crear un contenedor a traves de un docker file, podemos hacerlo a travez de un archivo YAML, aplicando todas las ventajas que nos ofrece Ansible, como ser invocar roles para aplicarlos a la imagen.
Pero Ansible Container no se detiene ahi, tambien nos permite enviar las imagenes a registros publicos y privados.

## Porque usar Ansible Container? ##

En los últimos años, ha habido una explosión de interés en Microservicios basados ​​en contenedores y herramientas de orquestación automatizada como Ansible. Sin embargo, lograr que estos sistemas funcionen juntos siempre ha sido un dolor de cabeza. Con el fin de crear imágenes de contenedores, generalmente comenzaría creando archivos Docker (Docker Files) desordenados, encadenando servicios junto con Docker-Compose, y luego, finalmente, tratando de organizar la implementación a través de una plataforma como Kubernetes (tal vez utilizando Ansible para automatizar parte del proceso).

Ansible Container elimina el dolor de la ecuación y le permite crear directamente imágenes de Docker utilizando los conceptos existentes de Ansible. En resumen, podremos crear contenedores usando Ansible.

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

La razón por la que los Dockerfiles son tan difíciles de entender se debe a la forma en que utilizan las capas. Esencialmente, cada nueva línea en un Dockerfile crea una nueva "capa", que se agrega al tamaño del archivo. Como las imágenes del contenedor están diseñadas para ser compartidas, a menudo es ventajoso asegurarse de que los tamaños de las imágenes se mantengan lo más pequeños posible, lo que da como resultado formas creativas para unir los comandos y unificar tanto como sea posible en cada línea.

Aunque la estrategia de capas en Ansible Container no está clara en este momento, es evidente que el uso de tareas y roles de Ansible es una gran mejora en un flujo de trabajo basado en Dockerfile.

Si bien la sintaxis de un docker file no es complicada, veremos que la sintaxis de ansible es mucho mas sencilla. Esto es uno de los puntos a favor por los cuales prefiero usar herramientas como estas.

## Manos a la obra ##
 Paquetes necesarios:

- docker
- python
- python-pip (pip)
- ansible

**Creando el ambiente**

**Nota:** Este ambiente esta generado sobre una maquina virtual (Ubuntu 18.04) con todos los requerimientos mencionados anteriormente. Asimismo toda la instalacion y configuracion se hizo con el usuario root.

Vamos a crear un directorio para nuestro proyecto:

```sh
mkdir /ansible-container-demo
cd /ansible-container-demo
```

**Instalando Ansible Container**

Ansible container se distribuye como un paquete basico, el cual podemos instalarlo para los siguientes motores de contenedores:

- k8s Kubernetes
- docker Docker engine
- openshift RedHat Openshift

En mi caso voy a instalar Ansible Container para el motor de Docker:

```sh
pip install "ansible-container[docker]"
```

Luego ejecutamos el siguiente comando:

```sh
ansible-container init
```

El comando anterior deberia generar la siguiente estructura dentro de nuestra carpeta:

```sh
ansible.cfg
ansible-requirements.txt
container.yml
meta.yml
requirements.yml
```

Podemos visitar la guia rapida que la pondre al final del articulo para ver mas en detalle la descripcion de cada archivo.

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
    name: "{{ item }}"
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

[Ansible Container Installation][Ansible Container Installation]

[Ansible Container Installation]: https://docs.ansible.com/ansible-container/installation.html

[Ansible Container Started Guide][Ansible Container Started Guide]

[Ansible Container Started Guide]: https://docs.ansible.com/ansible-container/getting_started.html

