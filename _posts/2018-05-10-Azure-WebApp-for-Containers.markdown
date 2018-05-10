---
title:  "Azure - Webapp for containers"
date:   2018-05-10 2:01:00
categories: [Webapp for Containers]
tags: [WebApp, DevOps, Hashicorp, Containers, Azure]
---
Hace unos meses que estoy junto con algunos compañeros de la oficina trabajando en levantar una APP previamente desarrollada, en Microsoft Azure. Es una infraestructura muy desafiante, la cual cuenta con varios componentes muy interesantes.
Particularmente les voy a contar algo que nos paso utilizando el servicio de WebApp para alojar el front end de dicha aplicacion y como lo pudimos resolver.

## Azure WebApp for Containers ##

Azure webapp for containers, es uno de los nuevos servicios de azure el cual a traves de una webapp, podemos desplegar imagenes de Docker.

Particularmente toda la infraetsructura esta desarrollada con Terraform, y uno de los desafios que nos surgieron fue que el servicio que les mencionaba anteriormente no esta liberado por Hashicorp.

## Entonces... Que hicimos? ##

Luego de dedicar algunas horas de investigacion, vimos que podemos integrar Terraform con templates de Azure ARM, los cuales estan escritos en lenguaje JSON.
Entonces nos pusimos manos a la obra...

## La receta ##

A continuacion les paso a explicar como desarrollamos la receta para desplegar este nuevo componente e integrarlo con toda la infraestructura.


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

```JSON
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



[Packer Demo][Packer_Demo]

[Packer_Demo]: https://www.youtube.com/watch?v=8FM2bG3SZsA&t=128s

### Links de interés: ###

[Packer Official Page][Packer]

[Packer Docs][Packer_Docs]

[Packer]:      https://www.packer.io/

[Packer_Docs]: https://www.packer.io/docs/index.html
