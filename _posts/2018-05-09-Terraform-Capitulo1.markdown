---
title:  "Terraform"
date:   2018-05-09 22:06:23
layout: [post]
categories: [Terraform]
tags: [Terraform]
---
Introducción a Terraform:
Hoy en día nos encontramos con grandes retos como afrontar el despliegue de infraestructura como código de una manera que sea rápida, fácil, segura, que nos permita desplegar dicha infraestructura en múltiples proveedores de nube y on-premise.
Para ello y otro tipo de problemas, nos encontramos con una herramienta como Terraform, capaz que facilitarnos la codificación de la infraestructura que vamos a desplegar.


![Terraform_Logo]({{ site.baseurl }}/images/Terraform_Logo.png)

## Introducción ##

Hoy en día nos encontramos con grandes retos como afrontar el despliegue de infraestructura como código de una manera que sea rápida, fácil, segura, que nos permita desplegar dicha infraestructura en múltiples proveedores de nube y on-premise.
Para ello y otro tipo de problemas, nos encontramos con una herramienta como Terraform, capaz que facilitarnos la codificación de la infraestructura que vamos a desplegar.

## Que soluciones nos ofrece terraform? ##

Cuanto mas grande se vuelve un servicio, mas difícil se hace su administración y la operación de manera manual. También pueda que exista un servicio en particular para nuestra aplicación que deba correr en un proveedor de nube especifico y que tengamos ligado otro servicio corriendo en otro proveedor de nube, por tanto esto nos enfrenta a utilizar dos consolas de administración distintas o CLI diferentes para el aprovisionamiento de la aplicación.

Si ademas pretendemos crear y destruir entornos bajo demanda en distintos coluds para una misma aplicación, puede convertirse en una tarea tediosa y corremos el peligro de no hacerlo de una forma predictiva.

La solución a estos problemas pasaría por una herramienta que nos permita modelar nuestra infraestructura como código. Encontramos herramientas propias de cada proveedor de nube, como ser Cloudformation para AWS, AzureCli o VisualStudio para Azure. Pero estas herramientas están ligadas al proveedor de nube en donde se desea desplegar la infraestructura.

Sin embargo, Terraform nos permite codificar nuestra infraestructura, atendiendo las necesidades de nuestro servicio y ofreciendo un amplio abanico de proveedores de nube.

## Como funciona? ##

Terraform se basa en tres sencillos pasos para el despliegue de la infraestructura.

![Terraform_img1]({{ site.baseurl }}/images/Terraform_img1.PNG)

### Write ###

Mediante ficheros de configuración, generamos la sintaxis propuesta por terraform (un lenguaje muy fácil y declarativo).
Normalmente usamos uno o mas ficheros para definir la arquitectura, una serie de variables para poder parametrizar nuestro despliegue y algunas variables de salida donde podremos obtener datos necesarios, como ser IP, FQDN del conjunto de componentes que forman parte de la infraestructura.

A continuación voy a mostrar un ejemplo sencillo, usando un proveedor de Docker, generado en un archivo llamado main.tf:

```json
#Config Docker provider

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

resource "docker_container" "jenkins" {
    image = "${docker_image.jenkins.latest}"
    name = "terraform-demo"
    ports = {
       internal = 8080
       external = 8080
       internal = 5000
       external = 5000
       }
 }

resource "docker_image" "jenkins" {
    name = "jenkins:latest"
}

```

### Plan ###

Una vez codificada nuestra infraestructura podemos hacer uso de dos comandos:

El primero es terraform plan, el cual nos mostrara todos los recursos definidos en nuestro código.
A continuación vamos a utilizar otro comando terraform apply, y veremos como terraform se encarga de aplicar el plan definido anteriormente y desplegar nuestra infraestructura.

```yaml
terraform apply
docker_image.jenkins: Refreshing state... (ID: sha256:d5c0410b1b443d3ed805078d498526590ae76fc42a1369bc814eb197f5ee102bjenkins:latest)
docker_container.jenkins: Refreshing state... (ID: 0904115a38630b0f8c7871da9eba9971c3a4ad67acb20d694ef20adc53225d5b)
docker_container.jenkins: Creating...
  bridge:                   "" => "<computed>"
  gateway:                  "" => "<computed>"
  image:                    "" => "sha256:d5c0410b1b443d3ed805078d498526590ae76fc42a1369bc814eb197f5ee102b"
  ip_address:               "" => "<computed>"
  ip_prefix_length:         "" => "<computed>"
  log_driver:               "" => "json-file"
  must_run:                 "" => "true"
  name:                     "" => "terraform-demo"
  ports.#:                  "" => "1"
  ports.376939551.external: "" => "5000"
  ports.376939551.internal: "" => "5000"
  ports.376939551.ip:       "" => ""
  ports.376939551.protocol: "" => "tcp"
  restart:                  "" => "no"
docker_container.jenkins: Creation complete

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

The state of your infrastructure has been saved to the path
below. This state is required to modify and destroy your
infrastructure, so keep it safe. To inspect the complete state
use the `terraform show` command.

State path: terraform.tfstate
```
Tras la ejecución del comando hemos obtenido como resultado un contenedor con una imagen de jenkins y un fichero llamado terraform.tfstate. Dicho fichero almacena el estado actual de la infraestructura y es consultado por Terraform para obtener la información antes de aplicar nuevos cambios sobre la misma. Si borramos el fichero y volvemos a aplicar el plan, obtendremos un mensaje de error indicando que no puede aplicar los cambios:

```yaml
Error applying plan:

1 error(s) occurred:

* docker_container.jenkins: Unable to create container: container already exists

Terraform does not automatically rollback in the face of errors.
Instead, your Terraform state file has been partially updated with
any resources that successfully completed. Please address the error
above and apply again to incrementally change your infrastructure.
```
Para solucionar este problema debemos guardar dicho archivo de estado en lo que llamamos backend (storage de Azure, S3, etc) o también podremos guardar dicho archivo en un sistema de control de versiones (GIT).

### Create ###

Por ultimo ya llegamos a este proceso en donde tenemos desplegados y creados todos los componentes de nuestra aplicación ya sea en un solo proveedor de nube como en varios al mismo tiempo.

### Otras ventajas ###

Terraform no solo soporta el despliegue en múltiples proveedores de nube, sino que tambienes compatible con múltiples herramientas de configuracion que pueden ser invocadas desde nuestro codigo para configurar componentes específicos de nuestra aplicación.



[Terraform Official Page][Terraform]
[Terraform Docs][Terraform_Docs]

[Terraform]:      http://terraform.io
[Terraform_Docs]: https://www.terraform.io/docs/index.html
