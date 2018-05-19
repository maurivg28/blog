---
title:  "Azure Migrate"
date:   2018-05-19 12:00:00
categories: [AzureMigrate]
tags: [AzureMigrate, Azure, Cloud, Microsoft]
---
Hace unos días que estamos planificando para un cliente un sitio de contingencia en la nube, en donde pueda tener toda su infraestructura lista en caso de que ocura un desastre. Siempre nos preguntamos el "como", y ahí surgen las interrogantes de costos, tamaños de las maquinas virtuales, etc. El día de hoy voy a contarles sobre un servicio de Azure el cual hace todo el trabajo por nosotros y de alguna manera nos responde todas estas interrogantes. Vamos a hablar sobre Azure Migrate.

![AzureMigrate_Logo]({{ site.baseurl }}/images/azuremigrate_logo.png)

## Introducción ##

Azure migrate es un servicio gratuito de Microsoft Azure, el cual mediante un colector, analiza toda la infraestructura y arma un informe en base a lo que ha relevado. Azure Migrate elebora un informe muy completo, el cual destaco los siguientes puntos importantes que debemos tener en cuenta a la hora de crear o replicar las maquinas virtuales en la nube:

- Evalua si las maquinas virtuales onpremise se pueden migrar.
- Recomienda que tamaño de maquina virtual es el adecuado para cerar la maquina virtual en la nube.
- Realiza una estimación de los costos, para saber cuanto gasto mensual tendremos con la nueva infraestructura en la nube.
- Realiza un analisis de dependencia entre maquinas virtuales.

## Limitaciones ##

- Actualmente, solo se pueden evaluar maquinas virtuales que esten en ambientes de virtualización, montados sobre VMWare y administrados por Vcenter Server (5.5, 6.0, 6.5).
- Soporta la migración de maquinas virtuales montadas sobre Hyper-V. Igualmente para este caso se recomienda utilizar la herramienta de planificación de Azure Site Recovery.
- Soporta un descubrimiento de hasta 1500 maquinas virtuales de una sola vez, en un solo proyecto.
- El servicio de Azure Migrate, solo está disponible en las regiones West Central US y East US. 
- Azure migrate solo soporta discos administrados para realizar la evaluación de las mquinas que estarán en la nube.

## Cómo trabaja Azure Migrate? ##

1- En nuestra suscripción de Microsoft Azure, creamos el servicio de Azure Migrate.
2- Azure Migrate utiliza un colector, la cual es una appliance que se encarga de descubrir y obtener la información necesaria sobre la infraestructura onpremise. Dicha appliance se puede descargar y desplegar mediante Vcenter. La misma esta en un formato Open Virtialization Appliance (OVA).
3- El colector, se conecta a VCenter ingresando las credenciales necesarias, para luego iniciar el descubrimiento.
4- El colector recopila los datos utilizando las herramientas de powershell (Power-CLI) para vmware. Este colector no instala nada en los host de VMWare o en las maquinas virtuales. Los datos recopilados incluyen informacion de las VM como ser (núcleos, memoria, tamaño de discos y adaptadores de red). Tambien se recopilan datos de rendimiento de las maquinas virtuales, incluidos, uso de CPU y memoria, IOPS de disco, rendimiento de disco (Mbps) y tráfico saliente de red (Mbps). 
5- Los datos son pusheados al servicio de azure migrate y podemos verlos en el portal.
6- A efectos de evaluación, el colector organiza maquinas virtuales descubiertas en grupos. Por ejemplo, agrupa maquinas virtuales que ejecutan una misma aplicación. 
7- Una vez que forma el grupo, se crea una evaluación para dicho grupo.
8- Una vez que termina la evaluación, se puede visualizar desde el portal o descargarla a un archivo de Excel.

![AzureMigrate_Works]({{ site.baseurl }}/images/azuremigrate_01.png)

## Que puertos necesito? ##

Azure migrate utiliza el puerto seguro 443 para establecer la comunicación.

## Que sucede luego del informe de evaluación? ##

Una vez que tenemos el informe de evaluación, podemos usar otras herramientas como ser Azure Site Recovery o Veeam Backup, para realizar la migración y replicación de la infraestructura onpremise a la nube de azure.

### Links de interés: ###

[Azure Migrate Official Page][AzureMigrate]

[AzureMigrate]: https://azure.microsoft.com/es-es/services/azure-migrate/
