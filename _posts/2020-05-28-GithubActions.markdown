---
title:  "Github Actions"
date:   2020-05-28 19:30:00
categories: [DevOps]
tags: [Terraform, Azure, CI/CD, Microsoft, DevOps]
---
Hoy en dia, en el mundo de DevOps, existen una infinidad de herramientas para la integracion continua y el desplegue o entrega continua, que nos aseguren entregas que aporten valor.

## Introducción ##

Como comentaba anteriormente, existen varias herramientas, que aseguren, compilaciones, pruebas e implementaciones consistentes y repetibles. Contamos con herramientas como ser AzureDevops, Jenkins, por nombrar algunas.
Hace poco descubri que Github ha liberado Github actions, y a continuacion vamos a ver de manera practica como realizar un despliegue sencillo de infraestructura, utilizando terraform para la infraestructura como codigo, y Azure como proveedor de Nube.

## Manos a la obra ##

**Previamente vamos a necesitar lo siguiente:**

1- Una suscripcion de Azure.
2- Un repositirio de Github.
3- Visual Studio Code para la escritura del codigo.
4- Terraform.

**Creacion de secretos en Github**

Github cuenta con una opcion para guardar toda la informacion sencible. Para nuestro ejemplo vamos a necesitar guardar tres secretos, ```clientId```, ```clientSecret```, y ```tenantId```. Estos secretos los usara Terraform para autenticar contra Azure.

Para crear estos secretos, debemos ir al repositorio de Github, click en **Settings**, y luego en **Secrets**.

![Github_Secrets]({{ site.baseurl }}/images/github_secrets.png)

Luego debemos hacer click en **New Secret** y cargar los valores mencionados anteriormente.

![Github_Secrets_Values]({{ site.baseurl }}/images/github_secrets_values.png)

**Configuracion de Terraform**

Luego que hemos configurado los requisitos previos, vamos a configurar Terraform para el despliegue de la infraestructura.
En este caso vamos a desplegar algo muy sencillo como ser un **Resource Group**

Para contar con un codigo que sea reusable, vamos a configurar un archivo de variables llamado **variables.tf** 

Este archivo tendra las siguientes variables, estas variables tienen la funcion de conectarnos con Azure:

- ```sub```

- ```client_secret```

- ```client_id```

- ```tenant_id```

El archivo **variables.tf** se vera de la siguiente manera:

```bash
variable "sub" { type = string }
variable "client_secret" { type = string }
variable "client_id" { type = string }
variable "tenant_id" { type = string }
```

**Creando el archivo de configuracion terraform.tfvars**

Una ves que hemos creado el archivo de variables, vamos a configurar otro archivo llamado `terraform.tfvars`, el cual tiene como funcion pasar ciertos valores al momento de la ejecucion del workflow.

El archivo **.tfvars** contendra los valores correspondientes a las variables que hemos definido en el archivo `variables.tf`. 
Estos valores seran los siguientes:

```bash
sub = id_de_la_suscripcion
client_id = id_del_service_principal
tenant_id = tennant_id_de_la_cuenta
```

Cabe aclarar que estos valores no seran cargados en el archivo `terraform.tfvars` ya que contienen datos sencibles, y estos datos seran tomados de los **secrets** que hemos configurado anteriormente en la configuracion de Github.

Para concluir con la configuracion de la infraestructura como codigo, se creara un archivo llamado `main.tf` el cual contendra todo lo necesario para desplegar lo que es requerido a nivel de Azure.

Este archivo se vera de la siguiente manera:

```bash
provider "azurerm" {
  subscription_id = var.sub
  client_id       = "${var.client_id}"
  client_secret   = "${var.client_secret}"
  tenant_id       = "${var.tenant_id}"
}

resource "azurerm_resource_group" "prod_app" {
  name     = "prod-app-rg"
  location = "eastus2"

  tags = {
    environment = "Prod"
  }
}
```
Una vez que hemos configurado todo, en el repositorio de Github deberiamos tener los siguientes archivos:

- main.tf

- variables.tf

- terraform.tfvars

**Configuracion de Github actions Workflow**

En este punto, se ha creado la configuración de Terraform para implementar el grupo de recursos de Azure. ¡Finalmente es hora de entrar en las acciones de GitHub! Todos los flujos de trabajo se definen usando la sintaxis llamada YAML.

Para crear el workflow, debemos ir al repositorio de Github y hacer click en **Actions**. Vamos a encontrarnos con una infinidad de workflow ya pre configurados, en nuestro caso haremos click sobre la opcion **set up a workflow yourself**.

![Github_Actions_Yourself]({{ site.baseurl }}/images/github_actions_yourself.png)

Ahora vamos a construir nuestro flujo en formato de archivo YAML, por defecto el mismo se llamara `main.yaml`.

Primero vamos a definir el nombre del flujo de la siguiente manera:

```name: Desliegue con Terraform```

Luego vamos a definir lo que comunmente se llama **trigger**, el cual va a ser el disparador del flujo. En nuestro caso lo vamos a disparar cada vez que hagamos un commit al repositorio.

```on: [push]```

Dentro de `steps`, se van a especificar las etapas que correran en el workflow.

El primer paso verifica el código en la rama **master**. Verificar el código significa llevar el código a donde sea que esté ejecutando el código.

La palabra clave `uses` significa qué se llama a la **API** que se utilizará para interactuar con el recurso específico. En nuestro caso, vamos a interactuar con la API de `actions/checkout` utilizando la rama **master**.

```bash
- name: "Checkout"
  uses: actions/checkout@master
```

**Inicializacion**

El primer paso es donde inicializaremos nuestro codigo para que sea interpretado por Terraform. 
En el siguiente fragmento de archivo YAML, el wokflow está ejecutando código creado por Hashicorp para ejecutar Terraform en flujos de trabajo de GitHub.

La palabra clave with describe qué componentes usará en la llamada API dentro de la palabra clave `uses`.

Como estamos llamando a la API `terraform-GitHub-actions`, los componentes con son nuestras opciones que tenemos.

```bash
- name: "Terraform Init"
  uses: hashicorp/terraform-github-actions@master
  with:
    tf_actions_version: 0.12.13
    tf_actions_subcommand: "init"
```

**Planning**

El siguiente paso en el archivo YAML es la etapa de `terraform plan`. Este step del `plan` pasa por cada recurso y confirma si lo que se está creando es correcto. Por ejemplo, se podría obtener un error que indique que le falta un parámetro requerido en el recurso que se está utilizando.

La palabra clave `args` se utiliza para pasar los argumentos al momento que corre el workflow. El parametro `var` permite pasar las variables a Terraform de manera externa.

```bash
- name: "Terraform Plan"
  uses: hashicorp/terraform-github-actions@master
  with:
    tf_actions_version: 0.12.13
    tf_actions_subcommand: "plan"
    args: '-var="client_secret=${{ secrets.clientSecret }}"'
```

**Deploy**

Por ultimo, llegamos al paso de despliegue. Cuando el workflow pasa el paso de inizializacion y plan de Terraform, se ejecuta el comando `apply` el cual se encargara de desplegar los recursos.

```bash
- name: "Terraform Apply"
  uses: hashicorp/terraform-github-actions@master
  with:
	  tf_actions_version: 0.12.13
    tf_actions_subcommand: "apply"
		args: '-var="client_secret=${{ secrets.clientSecret }}"'
```

**Archivo yaml completo**

```bash
name: Terraform deploy to Azure

on: [push]

jobs:
	build:
		runs-on: ubuntu-latest
		
		steps:
		- name: "Checkout"
		  uses: actions/checkout@master

		- name: "Terraform Init"
		  uses: hashicorp/terraform-github-actions@master
		  with:
		    tf_actions_version: 0.12.13
		    tf_actions_subcommand: "init"

		- name: "Terraform Plan"
		  uses: hashicorp/terraform-github-actions@master
		  with:
		    tf_actions_version: 0.12.13
		    tf_actions_subcommand: "plan"
		    args: '-var="client_secret=${{ secrets.clientSecret }}"'
		
		- name: "Terraform Apply"
		  uses: hashicorp/terraform-github-actions@master
		  with:
			  tf_actions_version: 0.12.13
		    tf_actions_subcommand: "apply"
				args: '-var="client_secret=${{ secrets.clientSecret }}"'
```

**Commit**

Luego que tenemos el archivo YAML creado, lo vamos a guardar en el repositorio de Github.
Para ello haremos click en el boton commit.

![Github_Commit]({{ site.baseurl }}/images/github_commit_button.png)

Por ultimo veremos como corre nuestro workflow.

![Github_Commit]({{ site.baseurl }}/images/github_monitoring.png)

![Github_Commit]({{ site.baseurl }}/images/github_monitoring.png)

