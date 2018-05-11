---
title:  "Azure - Webapp for containers"
date:   2018-05-10 2:01:00
categories: [Webapp for Containers]
tags: [WebApp, DevOps, Hashicorp, Containers, Azure, GuayoyoLabs, Howlermonkey]
---
Hace unos meses que estoy junto con algunos compañeros de la oficina trabajando en el despliegue de una infraestructura como codigo para una aplicacion. Es un proyecto muy desafiante, el cual cuenta con varios servicios de Microsoft Azure (WebApp, Cluster de Kubernetes - AKS, MongoDB, RedisCache, etc).
Particularmente les voy a contar algo que nos paso utilizando el servicio de WebApp para alojar una parte dicha aplicacion.
En el siguiente articulo les voy a hablar del servicio WebApp For Containers.


## El Problema ##

Uno de los problemas que se presentaron en este proyecto fue, que la aplicacion (Front-End) era incompatible con el servicio de WebApp, ya sea usando planes Windows o Linux.
Fue asi que decidimos usar un nuevo servicio de WebApp que salio publicado en Azure...

## Webapps con contenedores? ##

Azure webapp for containers. Es uno de los nuevos servicios de azure, el cual nos permite integrar imagenes de Docker, ya sea desde repositorios publicos (DockerHub), o repositorios privados (Azure Container Registry).

### Manos a la Obra ###

Particularmente toda la infraestructura esta desarrollada con Terraform, y uno de los desafios fue que este servicio, que anteriormente les mencionaba, no esta liberado por Hashicorp.

### Entonces... Como lo resolvimos? ###

Luego de dedicar algunas horas de investigacion, vimos que podemos integrar Terraform con templates de Azure ARM, los cuales estan escritos en lenguaje JSON.
Entonces comenzamos a desarrollar la nueva receta para desplegar este nuevo servicio y asi poder probar la aplicacion.

### La receta ###

A continuacion les paso a explicar como desarrollamos la receta para desplegar este nuevo componente e integrarlo con toda la infraestructura.

Partimos de un archivo main.tf el cual tenemos declarados los siguientes componentes:

***- Resource Group (Grupo de recursos).***
***- App Service Plan.***
***- Azure RM Template Deplyment.***

Aqui les dejo el codigo que utilizamos... Lo separe en dos partes para que se note el codigo escrito en el formato que usa Terraform y en otra parte el template de ARM en formato JSON.

Parte 1
```yaml
#Resource Group
resource "azurerm_resource_group" "rg" {
  name     = "${var.resource_group_name}"
  location = "${var.location}"

  tags {
    dept        = "Guayoyo Labs"
    environment = "${terraform.workspace}"
  }
}

#APP Service Plan
resource "azurerm_app_service_plan" "serviceplan" {
  name                = "hm-service-plan-${terraform.workspace}"
  location            = "${var.location}"
  resource_group_name = "${var.resource_group_name}"
  kind                = "linux"

  sku {
    tier     = "${var.webapp_sku}"
    size     = "${var.webapp_size}"
    capacity = "${var.webapp_capacity}"
  }

  # Necesario para que Azure fuerce un asp Liux
  # Ver https://github.com/terraform-providers/terraform-provider-azurerm/issues/602
  properties {
    reserved = true
  }

  tags {
    dept        = "Guayoyo Labs"
    environment = "${terraform.workspace}"
  }
}
```
Parte 2
```yaml
#Template ARM para WebApp For Containers
resource "azurerm_template_deployment" "tmpl" {
  name                = "${var.template_name}"
  resource_group_name = "${azurerm_resource_group.rg.name}"

  template_body = <<DEPLOY
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "app_service_plan_id": {
      "type": "string",
      "metadata": {
        "description": "App Service Plan ID"
      }
    },
    "name": {
      "type": "string",
      "metadata": {
        "description": "App Name"
      }
    },
    "image": {
      "type": "string",
      "metadata": {
        "description": "Docker image"
      }
    }
  },
  "resources": [
    {
      "apiVersion": "2016-08-01",
      "kind": "app,linux,container",
      "name": "[parameters('name')]",
      "type": "Microsoft.Web/sites",
      "properties": {
        "name": "[parameters('name')]",
        "siteConfig": {
          "alwaysOn": true,
          "appSettings": [
            {
              "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
              "value": "false"
            }
          ],
          "linuxFxVersion": "[concat('DOCKER|hmregistry.azurecr.io/howlermonkey-', parameters('image'), 'app:test-be')]"
        },
        "serverFarmId": "[parameters('app_service_plan_id')]"
      },
      "location": "[resourceGroup().location]"
    }
  ]
}
DEPLOY

  parameters {
    "name"                = "HowlerMonkey-webapp"
    "image"               = "webapp"
    "app_service_plan_id" = "${azurerm_app_service_plan.serviceplan.id}"
  }

  deployment_mode = "Incremental"
}
}
```

### Links de interés: ###

[Azure WebApp for Containers][AzureWebAppforContainers]

[AzureWebAppforContainers]: https://azure.microsoft.com/es-es/services/app-service/containers/

[Howlermonkey][Howlermonkey]

[Guayoyo Labs][GuayoyoLabs]

[Howlermonkey]: https://howlermonkey.io/

[GuayoyoLabs]: https://guayoyolabs.com/
